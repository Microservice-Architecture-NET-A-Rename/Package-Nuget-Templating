## Workflow GitHub Actions pour la Publication de Packages NuGet

### Vue d'Ensemble
Ce workflow automatise la création et la publication d'un package NuGet dans le **GitHub Package Registry** (registre privé) à chaque publication d'une release. Il utilise `dotnet.exe` pour compiler/empaqueter le projet et le NuGet CLI pour publier le package.

### Déclencheur
Le workflow se déclenche lorsqu'une nouvelle release est publiée :
```yaml
on:
  release:
    types: [published]
```

### Variables Clés
- `NUGET_AUTH_TOKEN`: Jeton d'authentification généré automatiquement via `secrets.GITHUB_TOKEN`
- `NUGET_VERSION`: Version du package extraite du fichier `.csproj`

### Job : publish_nuget_package
**Configuration** :
```yaml
runs-on: ubuntu-latest
permissions:
  contents: read
  packages: write
```

#### Étapes Détaillées

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
- Intégration native avec les projets .NET
- Résolution automatique des dépendances
- Génération de fichier `.nupkg` prêt à publier
- Support multi-plateforme

##### 4. Publication sur GitHub Package Registry
```yaml
- name: Push
  run: |
    dotnet nuget push **/*.nupkg \
    --api-key ${{ secrets.GITHUB_TOKEN }} \
    --source https://nuget.pkg.github.com/${{ github.repository_owner }}/index.json
```
**Spécificités du flux privé** :
- Stockage sécurisé dans l'écosystème GitHub
- Contrôle d'accès via permissions GitHub
- Intégration transparente avec les workflows CI/CD
- Historique de versions traçable

### Avantages Clés
1. **Automatisation complète** : De la release à la publication
2. **Sécurité renforcée** :
   - Authentification via `GITHUB_TOKEN`
   - Pas de credentials externes
3. **Gestion de version** :
   - Version extraite automatiquement du tag de release
   - Cohérence entre code et package

### Utilisation du Flux Privé
Pour consommer le package dans d'autres projets :
1. Ajouter la source NuGet dans `nuget.config` :
```xml
  <packageSources>
     <add key="github" value="https://nuget.pkg.github.com/VOTRE_ORGA/index.json" />
  </packageSources>
```

2. Authentifier avec un PAT ayant accès `read:packages`

### Ressources Utiles
- [Documentation GitHub sur les packages NuGet](https://docs.github.com/fr/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
- [Syntaxe workflow GitHub Actions](https://docs.github.com/fr/actions/using-workflows/workflow-syntax-for-github-actions)


