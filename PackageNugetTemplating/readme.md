# Package-Nuget-Templating

## Objectif du projet

Ce projet vise � d�velopper un template pour faciliter la cr�ation rapide de packages .NET, incluant tous les workflows n�cessaires � leur publication et � la g�n�ration automatique de notes de version.

## �tat actuel

Le projet est actuellement en phase de conception et de documentation. Aucune impl�mentation n'a encore �t� r�alis�e.

## Fonctionnalit�s envisag�es

### Cr�ation et gestion de packages

- Utilisation de `dotnet.exe` pour la gestion des packages, offrant une meilleure int�gration avec l'�cosyst�me .NET.
- Mise en place d'un processus de test avant publication.
- Int�gration avec GitHub Package Registry pour les flux priv�s.

### Versioning

- Utilisation de Nerdbank.GitVersioning pour l'incr�mentation automatique des num�ros de version.

### Tests locaux

- D�veloppement d'une m�thode pour tester localement les packages NuGet sans publication.

### Workflow de publication

- Cr�ation d'un workflow inspir� du projet MicroServiceTemplating.
- Automatisation du processus de versioning et de publication.

## Prochaines �tapes

1. Finaliser la documentation pour chaque �tape du processus.
2. Impl�menter le workflow de cr�ation, test et publication.
3. Int�grer le versioning automatique avec Nerdbank.GitVersioning.
4. D�velopper des instructions pour les tests locaux.
5. Configurer l'int�gration avec GitHub Package Registry.

## Ressources � explorer

- [GitVersion Documentation](https://gitversion.net/docs/)
- [GitHub Packages Documentation](https://docs.github.com/fr/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
- [NuGet Package Versioning](https://learn.microsoft.com/en-us/nuget/concepts/package-versioning?tabs=semver20sort)
- [NuGet Symbol Packages](https://learn.microsoft.com/en-us/nuget/create-packages/symbol-packages-snupkg)
- [Signing NuGet Packages](https://learn.microsoft.com/fr-fr/nuget/create-packages/sign-a-package)
- [NuGet CLI Reference](https://learn.microsoft.com/fr-fr/nuget/reference/cli-reference/cli-ref-sources)
- [Azure DevOps Artifacts](https://learn.microsoft.com/fr-fr/azure/devops/artifacts/nuget/publish?view=azure-devops)
- [Nerdbank.GitVersioning](https://github.com/dotnet/Nerdbank.GitVersioning)

Note : Ce projet est en cours de d�veloppement. Les fonctionnalit�s et les processus d�crits sont sujets � modification au fur et � mesure de l'avancement du projet.
