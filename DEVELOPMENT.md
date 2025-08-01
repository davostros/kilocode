# Guide de développement de Kilo Code

### Glossaire

- **Codebase** : ensemble du code source du projet.

Bienvenue dans le guide de développement de Kilo Code ! Ce document vous aidera à configurer votre environnement et à comprendre comment travailler avec la base de code. Que vous corrigiez des bugs, ajoutiez des fonctionnalités ou souhaitiez simplement explorer le projet, ce guide vous servira de point de départ.

## Prérequis

### Glossaire

- **Git** : système de contrôle de version.
- **Node.js** : environnement d’exécution JavaScript.
- **pnpm** : gestionnaire de paquets rapide.
- **Visual Studio Code** : éditeur de code recommandé pour le développement.

Avant de commencer, assurez‑vous d’avoir installé :

1. **Git** – pour le contrôle de version
2. **Node.js** (version recommandée [v20.19.2](https://github.com/Kilo-Org/kilocode/blob/main/.nvmrc))
3. **pnpm** – gestionnaire de paquets (https://pnpm.io/)
4. **Visual Studio Code** – notre IDE conseillé pour contribuer

## Pour commencer

### Glossaire

- **Fork** : copie personnelle d’un dépôt GitHub.
- **Dépendances** : bibliothèques nécessaires au fonctionnement du projet.

### Installation

1. **Forkez et clonez le dépôt** :

    - **Fork du dépôt** :
        - Rendez‑vous sur le [dépôt GitHub Kilo Code](https://github.com/Kilo-Org/kilocode)
        - Cliquez sur le bouton « Fork » en haut à droite pour créer votre propre copie.
    - **Clonez votre fork** :
        ```bash
        git clone https://github.com/[VOTRE-UTILISATEUR]/kilocode.git
        cd kilocode
        ```
        Remplacez `[VOTRE-UTILISATEUR]` par votre nom d’utilisateur GitHub.

2. **Installez les dépendances** :

    ```bash
    pnpm install
    ```

    Cette commande installe les dépendances pour l’extension principale, l’interface webview et les tests end‑to‑end.

3. **Installez les extensions VS Code** :
    - **Obligatoire** : [ESBuild Problem Matchers](https://marketplace.visualstudio.com/items?itemName=connor4312.esbuild-problem-matchers) – aide à afficher correctement les erreurs de compilation.

Ces extensions ne sont pas indispensables pour lancer l’extension mais sont recommandées pour le développement :

- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) – intègre ESLint dans VS Code.
- [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) – intègre Prettier dans VS Code.

La liste complète des extensions recommandées se trouve [ici](https://github.com/Kilo-Org/kilocode/blob/main/.vscode/extensions.json)

### Structure du projet

### Glossaire

- **Frontend** : partie visible par l’utilisateur.
- **Scripts** : petits programmes utilitaires.

Le projet est organisé en plusieurs répertoires clés :

- **`src/`** – code principal de l’extension
    - **`core/`** – fonctionnalités et outils cœur
    - **`services/`** – implémentations des services
- **`webview-ui/`** – code de l’interface (frontend)
- **`e2e/`** – tests de bout en bout
- **`scripts/`** – scripts utilitaires
- **`assets/`** – ressources statiques (images, icônes…)

## Flux de développement

### Glossaire

- **Hot Reloading** : rechargement automatique des modifications sans redémarrage.

### Lancement de l’extension

Pour exécuter l’extension en mode développement :

1. Appuyez sur `F5` (ou choisissez **Run** → **Start Debugging**) dans VS Code
2. Une nouvelle fenêtre VS Code s’ouvrira avec Kilo Code chargé

### Rechargement à chaud

- **Modifications de la webview** : elles apparaissent immédiatement sans redémarrage
- **Modifications du cœur de l’extension** : elles rechargent automatiquement l’hôte d’extension

En mode développement (NODE_ENV="development"), la modification du cœur déclenche la commande `workbench.action.reloadWindow`, il n’est donc plus nécessaire de redémarrer manuellement le débogueur et les tâches.

> **Important** : dans les versions de production, lorsque vous modifiez le cœur de l’extension, vous devez :
>
> 1. Arrêter le processus de débogage
> 2. Tuer les tâches npm en arrière‑plan (voir capture ci‑dessous)
> 3. Relancer le débogage

<img width="600" alt="Stopping background tasks" src="https://github.com/user-attachments/assets/466fb76e-664d-4066-a3f2-0df4d57dd9a4" />

### Compilation de l’extension

Pour construire un fichier `.vsix` prêt pour la production :

```bash
pnpm build
```

Cette commande va :

1. Construire la webview
2. Compiler le TypeScript
3. Packager l’extension
4. Créer un fichier `.vsix` dans le répertoire `bin/`

### Installation de l’extension compilée

Pour installer votre extension compilée :

```bash
code --install-extension "$(ls -1v bin/kilo-code-*.vsix | tail -n1)"
```

Remplacez `[version]` par le numéro de version actuel.

## Tests

Kilo Code utilise plusieurs types de tests pour garantir la qualité :

### Tests unitaires

Lancez les tests unitaires avec :

```bash
pnpm test
```

Cette commande exécute à la fois les tests de l’extension et ceux de la webview.

### Tests de bout en bout

Pour plus de détails sur les tests E2E, consultez [apps/vscode-e2e](apps/vscode-e2e/).

## Lintage et vérification des types

Assurez‑vous que votre code respecte nos standards de qualité :

```bash
pnpm lint          # Run ESLint
pnpm check-types   # Run TypeScript type checking
```

## Hooks Git

Ce projet utilise [Husky](https://typicode.github.io/husky/) pour gérer les hooks Git, automatisant certaines vérifications avant les commits et les pushes. Les hooks se trouvent dans le répertoire `.husky/`.

### Hook pre-commit

Avant qu’un commit soit finalisé, le hook `.husky/pre-commit` s’exécute :

1.  **Vérification de branche** : empêche de committer directement sur `main`.
2.  **Génération de types** : exécute `pnpm --filter kilo-code generate-types`.
3.  **Vérification du fichier de types** : s’assure que les modifications sur `src/exports/roo-code.d.ts` sont bien ajoutées au commit.
4.  **Lintage** : lance `lint-staged` pour linter et formater les fichiers indexés.

### Hook pre-push

Avant de pousser des changements sur le dépôt distant, le hook `.husky/pre-push` s’exécute :

1.  **Vérification de branche** : empêche de pousser directement sur `main`.
2.  **Compilation** : lance `pnpm run check-types` pour vérifier les types.
3.  **Vérification de changeset** : vérifie qu’un fichier existe dans `.changeset/` et rappelle d’en créer un avec `npm run changeset` si besoin.

Ces hooks aident à maintenir la qualité et la cohérence du code. En cas de problème lors d’un commit ou d’un push, consultez leurs messages pour trouver la cause.

## Dépannage

### Problèmes courants

1. **Extension ne se charge pas** : vérifiez les outils développeur de VS Code (Aide > Basculer les outils développeur) pour voir les erreurs
2. **Webview ne se met pas à jour** : essayez de recharger la fenêtre (Developer: Reload Window)
3. **Erreurs de compilation** : assurez‑vous que toutes les dépendances sont installées avec `pnpm install`

### Conseils de débogage

- Utilisez des instructions `console.log()` dans votre code pour déboguer
- Consultez le panneau Output de VS Code (Affichage > Output) et sélectionnez « Kilo Code » dans la liste déroulante
- Pour les problèmes de webview, ouvrez les outils développeur du navigateur dans la webview (clic droit > « Inspect Element »)

## Contribuer

Nous accueillons volontiers vos contributions à Kilo Code ! Voici comment vous pouvez aider :

1. **Signaler un problème** via [GitHub Issues](https://github.com/Kilo-Org/kilocode/issues)
2. **Trouver un ticket** et proposer un Pull Request avec votre correction
3. **Écrire des tests** pour améliorer la couverture
4. **Améliorer la documentation** sur [kilocode.ai/docs](https://kilocode.ai/docs)
5. **Suggérer une nouvelle fonctionnalité** via [GitHub Discussions](https://github.com/Kilo-Org/kilocode/discussions/categories/ideas) !
6. Vous voulez **implémenter quelque chose de nouveau** ? Super ! Nous serons ravis de vous aider sur [Discord](https://discord.gg/Ja6BkfyTzJ) !

## Communauté

Vos contributions sont les bienvenues ! Pour toute question ou idée, rejoignez notre serveur Discord : https://discord.gg/Ja6BkfyTzJ

Nous avons hâte de recevoir vos contributions et vos retours !
