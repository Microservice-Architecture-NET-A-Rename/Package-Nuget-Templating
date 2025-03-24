# Package-Nuget-Templating

## Objectif du projet

Ce projet vise à développer un template pour faciliter la création rapide de packages .NET, incluant tous les workflows nécessaires à leur publication et à la génération automatique de notes de version.

## État actuel

Le projet est actuellement en phase de conception et de documentation. Aucune implémentation n'a encore été réalisée.

## Fonctionnalités envisagées

### Création et gestion de packages

- Utilisation de `dotnet.exe` pour la gestion des packages, offrant une meilleure intégration avec l'écosystème .NET.
- Mise en place d'un processus de test avant publication.
- Intégration avec GitHub Package Registry pour les flux privés.

### Versioning

- Utilisation de Nerdbank.GitVersioning pour l'incrémentation automatique des numéros de version.
- Stratégie de versionnage uniformisée définie dans un fichier `version.json` à la racine du projet.
- Utilisation de tags conventionnés pour mettre à jour des packages spécifiques : `{project-name}-v1.0`.

### Templates

Deux templates seront créés :
1. Configuration du versionnage
2. Ajout d'un nouveau projet avec un .csproj pré-initialisé dans la solution générée par le premier template

### Tests locaux

- Développement d'une méthode pour tester localement les packages NuGet sans publication.

### Workflow de publication

- Création d'un workflow déclenché à la création d'un tag.
- Création d'une release en mode brouillon du package à publier.
- Publication du package sur GitHub Package Registry lors de la publication de la release.

## Configuration du projet

### Installation de Nerdbank.GitVersioning

```bash
dotnet tool install --tool-path .config/dotnet-tools nbgv
.config/dotnet-tools/nbgv install
```

### Empaquetage des packages NuGet

```bash
dotnet pack
```

### Structure du projet

- Chaque projet .NET dans la solution représentera un package NuGet.
- Les informations telles que la description, l'auteur et la société du package seront centralisées dans le .csproj.
- Important : Pour le PackageId, tous les mots doivent être collés.

### Notes de Release

Les notes de release seront conventionnées comme suit :
```
feat(project-name) description 
fix(project-name) description
```

## Prochaines étapes

1. Finaliser la documentation pour chaque étape du processus.
2. Implémenter le workflow de création, test et publication.
3. Intégrer le versioning automatique avec Nerdbank.GitVersioning.
4. Développer des instructions pour les tests locaux.
5. Configurer l'intégration avec GitHub Package Registry.

## Ressources à explorer

- [GitVersion Documentation](https://gitversion.net/docs/)
- [GitHub Packages Documentation](https://docs.github.com/fr/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
- [NuGet Package Versioning](https://learn.microsoft.com/en-us/nuget/concepts/package-versioning?tabs=semver20sort)
- [NuGet Symbol Packages](https://learn.microsoft.com/en-us/nuget/create-packages/symbol-packages-snupkg)
- [Signing NuGet Packages](https://learn.microsoft.com/fr-fr/nuget/create-packages/sign-a-package)
- [NuGet CLI Reference](https://learn.microsoft.com/fr-fr/nuget/reference/cli-reference/cli-ref-sources)
- [Azure DevOps Artifacts](https://learn.microsoft.com/fr-fr/azure/devops/artifacts/nuget/publish?view=azure-devops)
- [Nerdbank.GitVersioning](https://github.com/dotnet/Nerdbank.GitVersioning)

Note : Ce projet est en cours de développement. Les fonctionnalités et les processus décrits sont sujets à modification au fur et à mesure de l'avancement du projet.
