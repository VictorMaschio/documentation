 # <center> Vue </center> 

Cette implémentation de NodeJS utilise une image **node:alpine**, qui est la plus légère implémentation de node facilement utilisable. Elle utilise la librairie **Jest** pour les tests, **Eslint** comme linter, **Prettier** comme formatter et **Typedoc** comme générateur de documentation automatique.

**Rappel** : A chaque fois que vous chercher à télécharger une image docker ou une dépendance, utiliser le VPN pour ne pas être bloquer (erreur de certificat TLS signé en boucle).

## Adapter la configuration docker et nginx 

1. Ajouter à votre fichier docker-compose.yml un service pour le frontend :

```yml
  backend:
    image: node:22.17-alpine
    container_name: backend
    volumes: backend
      - ./backend:/home/app
      - ./docs:/home/docs
    working_dir: /home/app
    command: ["sh", "-c", "npm i && npm run ${COMMAND_BACK}"]
    networks:
      - default
```

2. Ajouter un chemin qui redirige vers votre backend dans la configuration nginx :

```conf
events {}

http{
    server{
        listen 0.0.0.0:80;
        client_max_body_size 50M; # Définit la taille maximum des requètes (images, vidéos, ...)

        # Définit l'adresse à laquelle le backend est accessible
        location /api {
            proxy_pass http://backend:8081/;
        }
    }
}
```

## Initialiser l'application

1. Executer la commande suivante au niveau du `docker-compose.yml`, normalement à racine de votre projet.

```bash
docker run --rm -it \
  -v "$(pwd)/backend:/home/app" \
  -w /home/app \
  node:22.17-alpine \
  sh -c "npm init -y && npm install express && npm install --save-dev typescript ts-node @types/node nodemon jest ts-jest @types/jest supertest @types/supertest typedoc typedoc-plugin-markdown"
```

2. Modifier les scripts dans le `package.json`. cela vous permettra de lancer n'importe lequel en modifiant la variable `COMMAND_FRONT` dans votre `.env` et en relancant tous les services avec `docker compose up`, ou un seul service avec `docker compose up frontend` (Par exemple pour executer les test frontend).

```json
    "scripts": {
        "dev": "nodemon src/app.ts",
        "build": "tsc -b --verbose",
        "lint:check": "eslint \"src/**/*.{ts}\" --cache",
        "lint:fix": "eslint \"src/**/*.{ts}\" --cache --fix",
        "format:check": "prettier --list-different \"src/**/*.{js,ts,json}\"",
        "format:fix": "prettier --write \"src/**/*.{js,ts,json}\"",
        "test": "jest --detectOpenHandles",
        "test:coverage": "jest --coverage --detectOpenHandles",
        "doc": "npx typedoc"
    },
```

Aussi pour installer une dépendance, vous pouver utiliser :

- si votre conteneurs est déjà lancé, utiliser dans un autre terminal :

```bash
docker exec frontend npm install <package-name>
```

- si le conteneur n'est pas lancé :

```bash
docker compose run --rm frontend npm install <package-name>
```

## Permettre le hot-reload

Créer dans `backend` la configuration nodemon `nodemon.json` comme ceci :

```json
{
    "ignore": ["src/dist/*", "uploads/__test__/*"],
    "legacyWatch": true
}
```

## Implémenter les mecanismes de linting et formattage

### Configurer lint et formattage au sein du projet

1. Créer le fichier `eslint.config.js` dans le dossier `src` comme ceci :

```js
import pluginVue from 'eslint-plugin-vue';
import pluginPrettier from 'eslint-plugin-prettier';
import pluginTypeScript from '@typescript-eslint/eslint-plugin';
import tsParser from '@typescript-eslint/parser';
import prettierConfig from 'eslint-config-prettier/flat';
import globals from 'globals';

export default [
  ...pluginVue.configs['flat/recommended'],
  prettierConfig,
  {
    files: ['**/*.{ts,vue}'],
    plugins: {
      prettier: pluginPrettier,
    },
    rules: {
      'prettier/prettier': 'error',
    },
    languageOptions: {
      sourceType: 'module',
      globals: {
        ...globals.browser,
      },
    },
  },
  {
    files: ['**/*.ts'],
    languageOptions: {
      parser: tsParser,
      parserOptions: {
        ecmaVersion: 'latest',
        sourceType: 'module',
      },
    },
    plugins: {
      '@typescript-eslint': pluginTypeScript,
    },
  },
];
```

2. Créer un fichier `.prettierrc` dans le dossier `frontend` :

```json
{
    "printWidth": 80,
    "tabWidth": 4,
    "useTabs": false,
    "semi": true,
    "singleQuote": true,
    "trailingComma": "all",
    "bracketSpacing": true
}
```

3. Modifier le fichier `tsconfig.json` :
```json
{
    "compilerOptions": {
        "baseUrl": ".",
        "paths": {
            "@/*": ["src/*"]
        }
    },
    "files": [],
    "references": [
        { "path": "./tsconfig.app.json" },
        { "path": "./tsconfig.node.json" }
    ]
}
```

4. Modifier `tsconfig.app.json` :
```json
{
    "extends": "@vue/tsconfig/tsconfig.json",
    "compilerOptions": {
        "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",

        /* Linting */
        "strict": true,
        "noUnusedLocals": true,
        "noUnusedParameters": true,
        "erasableSyntaxOnly": true,
        "noFallthroughCasesInSwitch": true,
        "noUncheckedSideEffectImports": true,

        "baseUrl": ".",
        "paths": {
            "@/*": ["src/*"]
        }
    },
    "include": ["src/**/*.ts", "src/**/*.tsx", "src/**/*.vue"]
}
```

A ce moment ci, vous pouvez utiliser `docker compose up frontend` et changer la variable `COMMAND_FRONT` à :
- `lint:check`
- `lint:fix`
- `format:check`
- `format:fix`

Ceux-ci qui virendront vérifier ou réparer la syntaxe ou le format du code. 

### Configurer VS Code pour le faire à l'enregistrement

1. Installer les plugins suivant : 
- [Eslint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

Sur VS code, executer `CTRL + SHIFT + p` et rechercher `settings.json` pour le workspace. Modifier le comme ceci :

```json
{
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": "explicit"
    },
    "eslint.alwaysShowStatus": true,
    "eslint.format.enable": true,
    "prettier.requireConfig": true,
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        "typescript",
        "typescriptreact",
        "vue",
        "html",
        "json",
        "json5",
        "yaml"
    ],
    "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```
Maintenant, si vous modifier l'indentation d'un fichier ou une virgule et un point virgule, la correction se fera à l'enregistrement du fichier.

## Créer le fichier de configuration de test

Créer dans le dossier `backend` le fichier `jest.config.ts` :
```ts
import type { JestConfigWithTsJest } from 'ts-jest';

const config: JestConfigWithTsJest = {
    preset: 'ts-jest',
    testEnvironment: 'node',
    testMatch: ['**/**/*.test.ts'],
    verbose: true,
    forceExit: true,
    // clearMocks: true,
    collectCoverage: true,
    coverageReporters: ['lcov', 'json'],
    coverageDirectory: 'coverage',
    testResultsProcessor: 'jest-sonar-reporter',
};

export default config;
```