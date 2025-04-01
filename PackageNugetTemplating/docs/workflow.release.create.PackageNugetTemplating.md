# Documentation du Workflow "Create Release PackageNugetTemplating"

## Introduction

Ce workflow GitHub Actions automatise la gestion des versions du package **PackageNugetTemplating** en cr�ant des tags immuables pour suivre son �volution. Il s'inspire des principes de Trunk Based Development, mettant l'accent sur un tronc principal unique (g�n�ralement `main` ou `trunk`) et limitant l'utilisation de branches de longue dur�e.

Dans ce contexte, les releases sont g�r�es principalement via des tags, �vitant ainsi la complexit� des branches de release traditionnelles.

## Objectifs

- Adopter une strat�gie de versionnement conforme � Trunk Based Development.
- Simplifier la gestion des versions en �vitant les branches de release superflues.
- Assurer un suivi structur� des modifications � l'aide de tags immuables.
- G�n�rer automatiquement des notes de release bas�es sur les commits en suivant les conventions de [Conventional Commits](https://www.conventionalcommits.org/fr/v1.0.0/).
- **Garantir l'auto-documentation des modifications** en associant chaque commit et chaque tag � une t�che dans un outil de gestion de projet comme Azure DevOps, Jira ou GitHub Projects.

## Organisation du Workflow

### Philosophie des Tags (Inspir�e de Trunk Based Development)

1. **Tags immuables** : Un tag pointe toujours vers le commit correspondant � la release et n'est jamais modifi� apr�s sa cr�ation.
2. **Absence de branches de release permanentes** : Seuls les tags sont utilis�s pour identifier les versions publi�es.
3. **Branches temporaires pour les correctifs** : En cas de bug sur une version existante, une branche temporaire est cr��e � partir du tag correspondant.
4. **Gestion des correctifs (Hotfixes)** :
   - Un correctif doit **d'abord �tre appliqu� sur la branche principale** (`main` ou `develop`).
   - Ensuite, une **branche temporaire est cr��e depuis le tag concern�** pour appliquer le correctif via un cherry-pick.
   - Une fois valid� et test�, un **nouveau tag** est cr�� pour cette version corrig�e.
   - **Les branches temporaires sont supprim�es apr�s validation** afin de ne pas encombrer l'historique du projet.

### Exemple d'Utilisation des Tags

#### Cr�ation d'un tag pour une release :
```sh
git tag -a PackageNugetTemplating-v4.0.1 -m "PackageNugetTemplating version 4.0.1"
git push origin PackageNugetTemplating-v4.0.1
```

#### Suppression d'un tag (Attention : d�conseill� apr�s publication) :
```sh
git tag -d PackageNugetTemplating-v4.0.1
git push origin --delete PackageNugetTemplating-v4.0.1
```

### Flux de travail

1. **D�veloppement continu sur le tronc principal** (`main` ou `trunk`).
2. **Cr�ation d'un tag** (ex: `PackageNugetTemplating-v4.0.1`) lorsque la version est pr�te � �tre publi�e.
3. **D�clenchement automatique du workflow GitHub Actions**.
4. **G�n�ration des notes de release** � partir des commits.
5. **Publication d'une release GitHub en mode brouillon**.

## D�clenchement

Le workflow est d�clench� lorsqu'un nouveau tag correspondant au format `PackageNugetTemplating-v*.*.*` est pouss� dans le d�p�t.

```yaml
on:
  push:
    tags:
      - "PackageNugetTemplating-v*.*.*"
```

## Permissions

Le workflow n�cessite les permissions suivantes :

```yaml
permissions:
    contents: write
    repository-projects: write
```

## �tapes du Workflow

### 1. Checkout du code

R�cup�re l'historique complet du d�p�t.

```yaml
- name: Checkout code
  uses: actions/checkout@v4
  with:
    fetch-depth: 0
```

### 2. Configuration de Node.js

Installe Node.js en version 18 pour ex�cuter les scripts de g�n�ration des notes de release.

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '18'
```

### 3. Installation des d�pendances

Installe `conventional-changelog-cli` pour la g�n�ration des notes de release.

```yaml
- name: Install dependencies
  run: npm install conventional-changelog-cli
```

### 4. G�n�ration des notes de release

Identifie les tags existants et g�n�re les notes de release entre le dernier tag et le pr�c�dent.

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
        echo "Pas de tag pr�c�dent trouv�. G�n�ration depuis le d�but du projet."
      fi

      RELEASE_NOTES=$(cat release_notes.md)
      RELEASE_NOTES_JSON=$(echo "$RELEASE_NOTES" | jq -R -s '.')
      echo "RELEASE_NOTES=$RELEASE_NOTES_JSON" >> $GITHUB_OUTPUT
```

### 5. R�cup�ration du tag

Extrait le tag actuel � partir de la r�f�rence GitHub.

```yaml
- name: Get tag
  id: tag
  run: |
    TAG=${GITHUB_REF#refs/tags/}
    echo "TAG=$TAG" >> $GITHUB_OUTPUT
```

### 6. Cr�ation de la release GitHub

Publie une release GitHub en mode brouillon avec les notes de release g�n�r�es.

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

1. **Simplicit�** : Pas de gestion complexe des branches de release.
2. **Automatisation** : Cr�ation des notes de release et publication sans intervention manuelle.
3. **Tra�abilit�** : Chaque version est clairement identifi�e par un tag unique.
4. **R�duction des conflits** : Int�gration continue du code.
5. **Gestion des correctifs facilit�e** : Application rapide des hotfixes avec des branches temporaires.

## Conclusion

Ce workflow optimise la gestion des versions en s'appuyant sur Trunk Based Development et l'utilisation de tags immuables. Il �limine la n�cessit� de maintenir des branches de release complexes tout en assurant un suivi pr�cis des modifications et en facilitant l'application de correctifs cibl�s.

