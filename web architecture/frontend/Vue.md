 # <center> Vue </center> 

Cette implémentation de Vue utilise une image **node:alpine**, qui est la plus légère implémentation de node facilement utilisable. Elle utilise aussi un serveur de développement **Vite** et sa librairie associé **Vitest**. Elle utilise **Eslint** comme linter et **Prettier** comme formatter.

**Rappel** : A chaque fois que vous chercher à télécharger une image docker ou une dépendance, utiliser le VPN pour ne pas être bloquer (erreur de certificat TLS signé en boucle).

## Adapter la configuration docker et nginx 

1. Ajouter à votre fichier docker-compose.yml un service pour le frontend :

```yml
  frontend:
    image: node:22.17-alpine
    container_name: frontend
    volumes:
      - ./frontend:/home/app
      - ./docs:/home/docs
    environment:
      VITE_DOMAIN: ${DOMAIN}
      VITE_PROTOCOL: ${PROTOCOL}
    working_dir: /home/app
    command: ["sh", "-c", "npm i && npm run ${COMMAND_FRONT}"]
    networks:
      - default
```

2. Ajouter un chemin qui redirige vers votre frontend dans la configuration nginx :

```conf
events {}

http{
    server{
        listen 0.0.0.0:80;
        client_max_body_size 50M; # Définit la taille maximum des requètes (images, vidéos, ...)

        # Définit l'adresse à laquelle le frontend est accessible
        location / {
            proxy_pass http://frontend:8080/;
        }
    }
}
```

## Initialiser l'application

1. Executer la commande suivante au niveau du `docker-compose.yml`, normalement à racine de votre projet.

```bash
docker run --rm -it \
  -v "$(pwd)/frontend:/home/app" \
  -w /home/app \
  node:22.17-alpine \
  sh -c "yes | npm create vite@latest . -- --template vue-ts &&  npm install --save-dev @types/node @types/vue-router vitest @testing-library/vue jsdom @vitest/ui @testing-library/jest-dom prettier eslint-plugin-prettier eslint-plugin-vue eslint-config-prettier @typescript-eslint"  
```

2. Modifier les scripts dans le `package.json`. cela vous permettra de lancer n'importe lequel en modifiant la variable `COMMAND_FRONT` dans votre `.env` et en relancant tous les services avec `docker compose up`, ou un seul service avec `docker compose up frontend` (Par exemple pour executer les test frontend).

```json
  "scripts": {
    "dev": "vite",
    "build": "tsc -b --verbose && vite build",
    "preview": "vite preview",
    "lint:check": "eslint \"src/**/*.{ts,vue}\"",
    "lint:fix": "eslint \"src/**/*.{ts,vue}\" --fix",
    "format:check": "prettier --check \"src/**/*.{js,ts,vue,json,css,md}\"",
    "format:fix": "prettier --write \"src/**/*.{js,ts,vue,json,css,md}\"",
    "test": "vitest",
    "test:coverage": "vitest run --coverage"
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

1. Modifier la configuration vite `vite.config.ts` comme ceci :

```ts
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'
import path from 'path';

// https://vite.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
      alias: {
        '@': path.resolve(__dirname, './src'),
      },
    },
  server: {
    host: "0.0.0.0",
    port: 8080,
    strictPort: true,
    allowedHosts: ['frontend'],
    watch: {
      usePolling: true,
    },
    hmr: {
      host: "localhost",
      clientPort: 80,
    },
  },
  test: {
    globals: true,
    environment: "jsdom",
    coverage: {
      reporter: ["lcov"],
    },
  },
});
```

2. Modifier le reverse proxy comme ceci :

```conf
        location / {
            proxy_pass http://frontend:8080/;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
        }
```

Vous pouvez lancer l'application avec la `commande docker compose up`. Normalement, si votre application est lancée et que vous effectuez une modification, vous devriez voir celle-ci en temps réel sur votre navigateur. Pour tester ceci, nous allons crée la base de la navigation sur l'application.

## Création d'un routeur et de différentes pages

1. Créer dans le dossier `src` les dossiers `components`, `views` et `routers`.

2. Rennomer dans `components` le fichier `HelloWorld.vue` en `NavBar.vue` et remplacer son contenu :

```ts
<template>
    <nav class="navigation">
        <router-link to="/vite">Vite</router-link>
        <router-link to="/vue">Vue</router-link>
    </nav>
</template>

<style>
.navigation {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    padding: 1rem;
    width: 400px;
}
</style>
```

3. Créer dans `views` respectivement les fichiers :

- `ViteView.vue`

```tsx
<script setup lang="ts">
import { ref } from 'vue';
import Navbar from '@/components/NavBar.vue';
const count = ref(0);
</script>

<template>
    <Navbar />
    <div>
        <a href="https://vite.dev" target="_blank">
            <img src="/vite.svg" class="logo" alt="Vite logo" />
        </a>
    </div>
    <h1>Vite</h1>
    <button type="button" @click="count++">count is {{ count }}</button>
    <p class="read-the-docs">Click on the Vite logo to learn more</p>
</template>

<style scoped>
.logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
}
.logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
    filter: drop-shadow(0 0 2em #42b883aa);
}
.read-the-docs {
    color: #888;
}
</style>
```

- `VueView.vue`

```tsx
<script setup lang="ts">
import { ref } from 'vue';
import Navbar from '@/components/NavBar.vue';
const count = ref(0);
</script>

<template>
    <Navbar />
    <div>
        <a href="https://vuejs.org/" target="_blank">
            <img src="@/assets/vue.svg" class="logo vue" alt="Vue logo" />
        </a>
    </div>
    <h1>Vue</h1>
    <button type="button" @click="count++">count is {{ count }}</button>
    <p class="read-the-docs">Click on the Vue logo to learn more</p>
</template>

<style scoped>
.logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
}
.logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
    filter: drop-shadow(0 0 2em #42b883aa);
}
.read-the-docs {
    color: #888;
}
</style>
```

4. Créer dans `routers` le fichier `router.ts` :

```ts
import { createRouter, createWebHistory } from 'vue-router';
import ViteView from '@/views/ViteView.vue';
import VueView from '@/views/VueView.vue';
import Navbar from '@/components/NavBar.vue';

const routes = [
    { path: '/', name: 'Home', component: Navbar },
    { path: '/vite', name: 'Vite', component: ViteView },
    { path: '/vue', name: 'Vue', component: VueView },
];

const router = createRouter({
    history: createWebHistory(),
    routes,
});

export default router;
```

5. Modifier le `App.vue` comme ceci :

```ts
<script setup lang="ts">
import '@/style.css';
</script>

<template>
    <router-view />
</template>
```

6. Modifier le `main.ts` comme ceci :

```ts
import { createApp } from 'vue';
import './style.css';
import App from './App.vue';
import router from './routers/router';

const app = createApp(App);
app.use(router);
app.mount('#app');
```

Vous devriez voir la modification s'effectuer grâce au hot-reload et pouvoir naviguer entre les pages.

## Implémenter les test

1. Créez dans le dossier `frontend/src` un dossier le chemin `src/__tests___/units/` ans lequel vous viendrez crée vos test unitaire et son sous dossier `views`.

2. Créez le fichier `ViteView.test.ts` ci-dessous :

```ts
import { render, screen, fireEvent } from '@testing-library/vue';
import { describe, it, expect, vi } from 'vitest';
import '@testing-library/jest-dom';
import ViteView from '@/views/ViteView.vue';

// Mock Navbar component globally
vi.mock('@/components/NavBar.vue', () => ({
    default: {
        name: 'Navbar',
        template: '<nav data-testid="mock-navbar">Mocked NavBar</nav>',
    },
}));

describe('YourComponent', () => {
    it('renders Navbar component', () => {
        render(ViteView);
        expect(screen.getByTestId('mock-navbar')).toBeInTheDocument();
    });

    it('renders Vite logo with correct alt text and class', () => {
        render(ViteView);
        const logo = screen.getByAltText(/vite logo/i);
        expect(logo).toBeInTheDocument();
        expect(logo).toHaveClass('logo');
    });

    it('increments count on button click', async () => {
        render(ViteView);
        const button = screen.getByRole('button', { name: /count is 0/i });

        await fireEvent.click(button);

        expect(button).toHaveTextContent('count is 1');
    });
});
```

A ce moment ci, vous pouvez changer la variable `COMMAND_FRONT` à `test` et utiliser `docker compose up frontend`

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
