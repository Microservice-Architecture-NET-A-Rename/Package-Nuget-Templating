## Workflow GitHub Actions pour la Publication de Packages NuGet

### Vue d'Ensemble
Ce workflow automatise la cr�ation et la publication d'un package NuGet dans le **GitHub Package Registry** (registre priv�) � chaque publication d'une release. Il utilise `dotnet.exe` pour compiler/empaqueter le projet et le NuGet CLI pour publier le package.

### D�clencheur
Le workflow se d�clenche lorsqu'une nouvelle release est publi�e :
```yaml
on:
  release:
    types: [published]
```

### Variables Cl�s
- `NUGET_AUTH_TOKEN`: Jeton d'authentification g�n�r� automatiquement via `secrets.GITHUB_TOKEN`
- `NUGET_VERSION`: Version du package extraite du fichier `.csproj`

### Job : publish_nuget_package
**Configuration** :
```yaml
runs-on: ubuntu-latest
permissions:
  contents: read
  packages: write
```

#### �tapes D�taill�es

##### 1. Checkout du Code
```yaml
- uses: actions/checkout@v4
```

##### 2. Configuration de .NET
```yaml
- name: Setup .NET
  uses: actions/setup-dotnet@v3
  with:
    dotnet-version: 7.0.x
```

##### 3. Empaquetage avec dotnet.exe
```yaml
- name: Pack
  run: dotnet pack --configuration Release /p:Version=$NUGET_VERSION
  env:
    NUGET_VERSION: ${{ github.ref_name }}
```
**Pourquoi utiliser dotnet.exe ?**
- Int�gration native avec les projets .NET
- R�solution automatique des d�pendances
- G�n�ration de fichier `.nupkg` pr�t � publier
- Support multi-plateforme

##### 4. Publication sur GitHub Package Registry
```yaml
- name: Push
  run: |
    dotnet nuget push **/*.nupkg \
    --api-key ${{ secrets.GITHUB_TOKEN }} \
    --source https://nuget.pkg.github.com/${{ github.repository_owner }}/index.json
```
**Sp�cificit�s du flux priv�** :
- Stockage s�curis� dans l'�cosyst�me GitHub
- Contr�le d'acc�s via permissions GitHub
- Int�gration transparente avec les workflows CI/CD
- Historique de versions tra�able

### Avantages Cl�s
1. **Automatisation compl�te** : De la release � la publication
2. **S�curit� renforc�e** :
   - Authentification via `GITHUB_TOKEN`
   - Pas de credentials externes
3. **Gestion de version** :
   - Version extraite automatiquement du tag de release
   - Coh�rence entre code et package

### Utilisation du Flux Priv�
Pour consommer le package dans d'autres projets :
1. Ajouter la source NuGet dans `nuget.config` :
```xml
  <packageSources>
     <add key="github" value="https://nuget.pkg.github.com/VOTRE_ORGA/index.json" />
  </packageSources>
```

2. Authentifier avec un PAT ayant acc�s `read:packages`

### Ressources Utiles
- [Documentation GitHub sur les packages NuGet](https://docs.github.com/fr/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
- [Syntaxe workflow GitHub Actions](https://docs.github.com/fr/actions/using-workflows/workflow-syntax-for-github-actions)


