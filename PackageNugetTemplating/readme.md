## Implémentation et Flux de Travail NuGet

### 1. Flux de Travail Actuels

*   **workflow.release.create.PackageNugetTemplating** :
    *   Objectif : Création d'une release préliminaire (brouillon) du package NuGet.
    *   Référence : [workflow.release.create.PackageNugetTemplating.md](https://github.com/Microservice-Architecture-NET-A-Rename/Package-Nuget-Templating/blob/main/PackageNugetTemplating/docs/workflow.release.create.PackageNugetTemplating.md)
*   **workflow.release.publish.PackageNugetTemplating** :
    *   Objectif : Publication du package NuGet.
    *   Référence : [workflow.release.publish.PackageNugetTemplating.md](https://github.com/Microservice-Architecture-NET-A-Rename/Package-Nuget-Templating/blob/main/PackageNugetTemplating/docs/workflow.release.publish.PackageNugetTemplating.md)

### 2. Processus de Développement des Packages

#### Priorisation des Sources NuGet

*   **Objectif** : Privilégier les packages en cours de développement par rapport aux versions publiées.
*   **Méthode** : Utilisation d'un flux local (`LocalFeed`) pour les packages en développement. Ce flux est priorisé dans la configuration NuGet.
*   **Configuration (`nuget.config`)** :

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources>
        <clear />
        <add key="LocalFeed" value="C:\LocalNuGet" />
        <add key="Github" value="https://nuget.pkg.github.com/Microservice-Architecture-NET-A-Rename/index.json" />
        <add key="Nuget" value="https://api.nuget.org/v3/index.json" />
    </packageSources>

    <packageSourceMapping>
        <packageSource key="LocalFeed">
            <package pattern="*" /> <!-- Priorité absolue -->
        </packageSource>
        <packageSource key="Github">
            <package pattern="*" />
        </packageSource>
        <packageSource key="Nuget">
            <package pattern="*" />
        </packageSource>
    </packageSourceMapping>
</configuration>
```

*   **Explication de la configuration** :
    *   `packageSources` : Définit les sources de packages NuGet.
    *   `LocalFeed` : Pointe vers un dossier local où sont stockés les packages en développement.
    *   `packageSourceMapping` : Définit la priorité des sources. Ici, `LocalFeed` a la priorité absolue grâce à `<package pattern="*" />.`.

#### Tests Locaux

*   **Procédure** :
    1.  Se placer dans le répertoire du projet NuGet.
    2.  Créer le package avec la version souhaitée et l'envoyer vers le flux local :

    ```bash
    dotnet pack -p:PackageVersion=4.0.0 --configuration Release --output C:\LocalNuGet
    ```

    3.  Ajouter le package au projet en utilisant la commande `dotnet add package` :

    ```bash
    dotnet add package PackageNugetTemplating -v 4.0.0
    ```

*   **Important** : NuGet utilisera la configuration de mapping des sources pour résoudre les dépendances, priorisant `LocalFeed`.

#### Commandes Utiles

*   Lister les sources NuGet configurées :

```bash
dotnet nuget list source --format Detailed
```

*   Lister les packages disponibles dans le flux local :

```bash
dotnet list package --source LocalFeed
```

*   Nettoyer le cache NuGet en cas de problèmes :

```bash
dotnet nuget locals all --clear
```

## Évolutions Futures

### 1. Templates de Configuration

*   **Objectif** : Simplifier la configuration des sources NuGet pour les nouveaux projets.
*   **Description** : Créer un template qui configure automatiquement les sources NuGet nécessaires.

### 2. Template de Projet Pré-Initialisé

*   **Objectif** : Faciliter la création de nouveaux projets NuGet avec un workflow de publication intégré.
*   **Description** :
    *   Créer un template de projet avec un fichier `.csproj` pré-rempli (description, auteur, société, etc.).
    *   Intégrer directement les workflows de création de release et de publication NuGet (éventuellement sous forme de fichiers GitHub Actions).
    *   Expliquer l'importance de respecter une nomenclature stricte pour le nom du projet.

## Ressources Additionnelles

*   Publication de packages Node.js avec GitHub Packages : [https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-nodejs-packages](https://docs.github.com/en/actions/use-cases-and-examples/publishing-packages/publishing-nodejs-packages)
*   Utilisation du registre NuGet avec GitHub Packages : [https://docs.github.com/fr/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry](https://docs.github.com/fr/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)
*   Gestion des versions de packages NuGet : [https://learn.microsoft.com/en-us/nuget/concepts/package-versioning?tabs=semver20sort](https://learn.microsoft.com/en-us/nuget/concepts/package-versioning?tabs=semver20sort)
*   Packages de symboles (`.snupkg`) pour le débogage : [https://learn.microsoft.com/en-us/nuget/create-packages/symbol-packages-snupkg](https://learn.microsoft.com/en-us/nuget/create-packages/symbol-packages-snupkg)
*   Référence de la CLI NuGet (sources) : [https://learn.microsoft.com/fr-fr/nuget/reference/cli-reference/cli-ref-sources](https://learn.microsoft.com/fr-fr/nuget/reference/cli-reference/cli-ref-sources)
*   Publication de packages NuGet sur Azure Artifacts : [https://learn.microsoft.com/fr-fr/azure/devops/artifacts/nuget/publish?view=azure-devops](https://learn.microsoft.com/fr-fr/azure/devops/artifacts/nuget/publish?view=azure-devops)
*   Création de templates de projets avec `dotnet new` : [https://learn.microsoft.com/fr-fr/dotnet/core/tools/dotnet-new](https://learn.microsoft.com/fr-fr/dotnet/core/tools/dotnet-new)
*   Flux NuGet locaux : [https://learn.microsoft.com/en-us/nuget/hosting-packages/local-feeds](https://learn.microsoft.com/en-us/nuget/hosting-packages/local-feeds)


