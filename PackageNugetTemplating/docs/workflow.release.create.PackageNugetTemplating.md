# Documentation du Workflow "Create Release PackageNugetTemplating"

## Introduction

Ce workflow GitHub Actions automatise la gestion des versions du package **PackageNugetTemplating** en créant des tags immuables pour suivre son évolution. Il s'inspire des principes de Trunk Based Development, mettant l'accent sur un tronc principal unique (généralement `main` ou `trunk`) et limitant l'utilisation de branches de longue durée.

Dans ce contexte, les releases sont gérées principalement via des tags, évitant ainsi la complexité des branches de release traditionnelles.

## Objectifs

- Adopter une stratégie de versionnement conforme à Trunk Based Development.
- Simplifier la gestion des versions en évitant les branches de release superflues.
- Assurer un suivi structuré des modifications à l'aide de tags immuables.
- Générer automatiquement des notes de release basées sur les commits en suivant les conventions de [Conventional Commits](https://www.conventionalcommits.org/fr/v1.0.0/).
- **Garantir l'auto-documentation des modifications** en associant chaque commit et chaque tag à une tâche dans un outil de gestion de projet comme Azure DevOps, Jira ou GitHub Projects.

## Organisation du Workflow

### Philosophie des Tags (Inspirée de Trunk Based Development)

1. **Tags immuables** : Un tag pointe toujours vers le commit correspondant à la release et n'est jamais modifié après sa création.
2. **Absence de branches de release permanentes** : Seuls les tags sont utilisés pour identifier les versions publiées.
3. **Branches temporaires pour les correctifs** : En cas de bug sur une version existante, une branche temporaire est créée à partir du tag correspondant.
4. **Gestion des correctifs (Hotfixes)** :
   - Un correctif doit **d'abord être appliqué sur la branche principale** (`main` ou `develop`).
   - Ensuite, une **branche temporaire est créée depuis le tag concerné** pour appliquer le correctif via un cherry-pick.
   - Une fois validé et testé, un **nouveau tag** est créé pour cette version corrigée.
   - **Les branches temporaires sont supprimées après validation** afin de ne pas encombrer l'historique du projet.

### Exemple d'Utilisation des Tags

#### Création d'un tag pour une release :
```sh
git tag -a PackageNugetTemplating-v4.0.1 -m "PackageNugetTemplating version 4.0.1"
git push origin PackageNugetTemplating-v4.0.1
```

#### Suppression d'un tag (Attention : déconseillé après publication) :
```sh
git tag -d PackageNugetTemplating-v4.0.1
git push origin --delete PackageNugetTemplating-v4.0.1
```

### Flux de travail

1. **Développement continu sur le tronc principal** (`main` ou `trunk`).
2. **Création d'un tag** (ex: `PackageNugetTemplating-v4.0.1`) lorsque la version est prête à être publiée.
3. **Déclenchement automatique du workflow GitHub Actions**.
4. **Génération des notes de release** à partir des commits.
5. **Publication d'une release GitHub en mode brouillon**.

## Gestion des Correctifs (Hotfixes) et Cherry-Picking

### Scénario : Application d'un correctif à une version existante

1. **Reproduire et corriger le bug sur le tronc principal**
    ```sh
    git checkout main
    git pull origin main
    ```
    - Reproduire le bug directement sur `main`
    - Développer le correctif avec tests associés
    - Committer avec un message clair :
    ```sh
    git commit -m "fix: correction du bug X [RELATES #123]"
    ```

2. **Création des branches de hotfix**
    ```sh
    # Pour PackageNugetTemplating-v2.0.0
    git checkout -b hotfix/PackageNugetTemplating-v2.0.1 PackageNugetTemplating-v2.0.0
    # Pour PackageNugetTemplating-v4.0.0 
    git checkout -b hotfix/PackageNugetTemplating-v4.0.1 PackageNugetTemplating-v4.0.0
    ```

3. **Cherry-pick du correctif depuis main**
    ```sh
    # Trouver le hash du commit du correctif
    git log main --oneline

    # Appliquer le correctif sur chaque branche
    git cherry-pick <commit-hash> -x
    ```

4. **Adaptation spécifique à la version**
    - Modifier uniquement ce qui est nécessaire pour assurer la compatibilité
    - Conserver le même comportement fonctionnel que sur `main`

5. **Validation et tagging**
    ```sh
    # Tests spécifiques à la version
    npm run test:compatibility --version=2.0.0

    # Création des tags
    git tag -a PackageNugetTemplating-v2.0.1 -m "Correction du bug X portée depuis main"
    git tag -a PackageNugetTemplating-v4.0.1 -m "Correction du bug X portée depuis main"
    ```

6. **Propagation vers le dépôt**
    ```sh
    git push origin --tags
    git push origin hotfix/PackageNugetTemplating-v2.0.1
    git push origin hotfix/PackageNugetTemplating-v4.0.1
    ```

7. **Suppression des tags obsolètes (si nécessaire)**
    ```sh
    git tag -d PackageNugetTemplating-v4.0.1
    git push origin --delete PackageNugetTemplating-v4.0.1
    ```

### Bonnes Pratiques

1. 🚫 **Interdiction** de développer directement sur les branches de release
2. ✅ **Obligation** de valider le correctif sur `main` avant tout cherry-pick
3. 🔍 **Vérification** systématique des différences avec :
    ```sh
    git diff main..hotfix/PackageNugetTemplating-v2.0.1 -- <fichiers-corrigés>
    ```
4. 🗑️ **Nettoyage** des branches après merge :
    ```sh
    git branch -d hotfix/PackageNugetTemplating-v2.0.1 hotfix/PackageNugetTemplating-v4.0.1
    ```

### Cas Exceptionnel (si le bug n'existe pas sur `main`)
1. Développer le correctif sur la branche concernée.
2. **Backport** vers `main` avec :
    ```sh
    git checkout main
    git cherry-pick <commit-hash> --no-commit
    git commit -m "chore: backport du correctif #123 [FROM PackageNugetTemplating-v2.0.1]"
    ```

---

### Génération et Traçabilité des Notes de Release

Le workflow inclut un mécanisme d'auto-documentation en générant des notes de release détaillées. Ces notes sont créées automatiquement à partir des messages de commit, en respectant la convention [Conventional Commits](https://www.conventionalcommits.org/fr/v1.0.0/). Chaque modification est associée à une tâche spécifique dans un outil de gestion de projet comme Azure DevOps, Jira ou GitHub Projects.

**Avantages :**
-   **Lien direct entre les modifications et les tâches projet** : Chaque commit doit inclure un identifiant de tâche (ex: `feat(core): ajout du support multi-langues [AB#1234]` pour Azure DevOps ou `fix(ui): correction du bug d'affichage (JIRA-456)`).
-   **Meilleure traçabilité** : Les équipes peuvent rapidement identifier les raisons des modifications et retrouver l'historique des décisions.
-   **Automatisation du reporting** : Les releases GitHub contiennent automatiquement des liens vers les tickets et tâches concernées, améliorant la transparence et la communication entre équipes.


## Déclenchement

Le workflow est déclenché lorsqu'un nouveau tag correspondant au format `PackageNugetTemplating-v*.*.*` est poussé dans le dépôt.

```yaml
on:
  push:
    tags:
      - "PackageNugetTemplating-v*.*.*"
```

## Permissions

Le workflow nécessite les permissions suivantes :

```yaml
permissions:
    contents: write
    repository-projects: write
```

## Étapes du Workflow

### 1. Checkout du code

Récupère l'historique complet du dépôt.

```yaml
- name: Checkout code
  uses: actions/checkout@v4
  with:
    fetch-depth: 0
```

### 2. Configuration de Node.js

Installe Node.js en version 18 pour exécuter les scripts de génération des notes de release.

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '18'
```

### 3. Installation des dépendances

Installe `conventional-changelog-cli` pour la génération des notes de release.

```yaml
- name: Install dependencies
  run: npm install conventional-changelog-cli
```

### 4. Génération des notes de release

Identifie les tags existants et génère les notes de release entre le dernier tag et le précédent.

```yaml
- name: Generate Release Notes
  id: release_notes
  run: |
      PACKAGE_NAME="PackageNugetTemplating"

      TAG_COUNT=$(git tag -l "PackageNugetTemplating-v*" | wc -l)

      if [ "$TAG_COUNT" -ge 2 ]; then
        npx conventional-changelog-cli -t $PACKAGE_NAME-v -p angular --commit-path=./$PACKAGE_NAME -r 2 > release_notes.md
      else
        npx conventional-changelog-cli -t $PACKAGE_NAME-v -p angular --commit-path=./$PACKAGE_NAME -r 0 > release_notes.md
        echo "Pas de tag précédent trouvé. Génération depuis le début du projet."
      fi

      RELEASE_NOTES=$(cat release_notes.md)
      RELEASE_NOTES_JSON=$(echo "$RELEASE_NOTES" | jq -R -s '.')
      echo "RELEASE_NOTES=$RELEASE_NOTES_JSON" >> $GITHUB_OUTPUT
```

### 5. Récupération du tag

Extrait le tag actuel à partir de la référence GitHub.

```yaml
- name: Get tag
  id: tag
  run: |
    TAG=${GITHUB_REF#refs/tags/}
    echo "TAG=$TAG" >> $GITHUB_OUTPUT
```

### 6. Création de la release GitHub

Publie une release GitHub en mode brouillon avec les notes de release générées.

```yaml
- name: Create GitHub Release
  uses: softprops/action-gh-release@v2
  if: startsWith(github.ref, 'refs/tags/')
  with:
    body: ${{ fromJson(steps.release_notes.outputs.RELEASE_NOTES) }}
    tag_name: ${{ steps.tag.outputs.TAG }}
    name: Release ${{ steps.tag.outputs.TAG }}
    draft: true
    prerelease: false
    generate_release_notes: false
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Avantages de cette approche

1. **Simplicité** : Pas de gestion complexe des branches de release.
2. **Automatisation** : Création des notes de release et publication sans intervention manuelle.
3. **Traçabilité** : Chaque version est clairement identifiée par un tag unique.
4. **Réduction des conflits** : Intégration continue du code.
5. **Gestion des correctifs facilitée** : Application rapide des hotfixes avec des branches temporaires.

## Conclusion

Ce workflow optimise la gestion des versions en s'appuyant sur Trunk Based Development et l'utilisation de tags immuables. Il élimine la nécessité de maintenir des branches de release complexes tout en assurant un suivi précis des modifications et en facilitant l'application de correctifs ciblés.


