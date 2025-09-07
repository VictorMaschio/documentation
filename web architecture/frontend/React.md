 # <center> React </center> 

Cette implémentation de React utilise une image **node:alpine**, qui est la plus légère implémentation de node facilement utilisable. Elle utilise aussi un serveur de développement **Vite** et sa librairie associé **Vitest**. Elle utilise **Eslint** comme linter et **Prettier** comme formatter et **Typedoc** comme générateur de documentation.

**Rappel** : A chaque fois que vous chercher à télécharger une image docker ou une dépendance, utiliser le VPN pour ne pas être bloquer (erreur de certificat TLS signé en boucle).

## Adapter la configuration docker et nginx 

1. Ajouter à votre fichier docker-compose.yml un service pour le frontend :

```yml
  frontend:
    image: node:24.4-alpine
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
  node:24.4-alpine \
  sh -c "yes | npm create vite@latest . -- --template react-ts && npm install react-router-dom && npm install --save-dev vitest jsdom @vitest/ui @testing-library/react @testing-library/jest-dom prettier eslint-plugin-prettier typedoc typedoc-plugin-markdown"  
```

2. Modifier les scripts dans le `package.json`. cela vous permettra de lancer n'importe lequel en modifiant la variable `COMMAND_FRONT` dans votre `.env` et en relancant tous les services avec `docker compose up`, ou un seul service avec `docker compose up frontend` (Par exemple pour executer les test frontend).

```json
  "scripts": {
    "dev": "vite",
    "build": "tsc -b --verbose && vite build",
    "preview": "vite preview",
    "lint:check": "eslint \"src/**/*.{ts,tsx}\"",
    "lint:fix": "eslint \"src/**/*.{ts,tsx}\" --fix",
    "format:check": "prettier --check \"src/**/*.{js,ts,jsx,tsx,json,css,md}\"",
    "format:fix": "prettier --write \"src/**/*.{js,ts,jsx,tsx,json,css,md}\"",
    "test": "vitest",
    "test:coverage": "vitest run --coverage",
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

1. Modifier la configuration vite `vite.config.ts` comme ceci :

```ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

// https://vite.dev/config/
export default defineConfig({
  plugins: [react()],
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

1. Créer dans le dossier `src` les dossiers `components` et `pages`.

2. Créer dans `components` le fichier `navbar.tsx` :

```tsx
import { NavLink } from "react-router-dom";
import "../App.css";

function NavBar() {
  return (
    <nav
      style={{
        display: "flex",
        justifyContent: "space-between",
        padding: "1rem",
        width: "400px",
      }}
    >
      <NavLink to="/vite">Vite</NavLink>
      <NavLink to="/react">React</NavLink>
    </nav>
  );
}

export { NavBar };
```

3. Créer dans `pages` respectivement les fichiers :

- `react.tsx`

```tsx
import { useState } from "react";
import reactLogo from "../assets/react.svg";
import "../App.css";
import { NavBar } from "../components/navbar";

/**
 * ReactComponent displays a simple UI with the React logo and a counter.
 *
 * The component includes a navigation bar, a link to the React website,
 * and a counter button that increments the count on each click.
 *
 * @returns {JSX.Element} A component rendering a React logo link and counter button.
 */
function ReactComponent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <NavBar />
      <div>
        <a href="https://react.dev" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
      </div>
      <h1>React</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
      </div>
      <p className="read-the-docs">Click on the React logo to learn more</p>
    </>
  );
}

export { ReactComponent };
```

- `vite.tsx`

```tsx
import { useState } from "react";
import viteLogo from "/vite.svg";
import "../App.css";
import { NavBar } from "../components/navbar";

/**
 * ViteComponent displays a simple UI with the Vite logo and a counter.
 *
 * The component includes a navigation bar, a link to the Vite website,
 * and a counter button that increments the count on each click.
 *
 * @returns {JSX.Element} A component rendering a Vite logo link and counter button.
 */
function ViteComponent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <NavBar />
      <div>
        <a href="https://vite.dev" target="_blank">
          <img src={viteLogo} className="logo" alt="Vite logo" />
        </a>
      </div>
      <h1>Vite</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
      </div>
      <p className="read-the-docs">Click on the Vite logo to learn more</p>
    </>
  );
}

export { ViteComponent };
```

4. Modifier le App.tsx comme ceci :

```tsx
import { BrowserRouter, Route, Routes } from "react-router-dom";

import { ReactComponent } from "./pages/react";
import { ViteComponent } from "./pages/vite";
import { NavBar } from "./components/navbar";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<NavBar />} />
        <Route path="/vite" element={<ViteComponent />} />
        <Route path="/react" element={<ReactComponent />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

Vous devriez voir la modification s'effectuer grâce au hot-reload et pouvoir naviguer entre les pages.

## Implémenter les test

1. Créez dans le dossier `frontend/src` un dossier le dossier `__tests___` et son sous dossier `units` dans lequel vous viendrez crée vos test unitaire.

2. Créez le fichier `vite.test.tsx` ci-dessous :

```tsx
import "@testing-library/jest-dom";
import { render, screen, fireEvent } from "@testing-library/react";
import { describe, expect, test, vi } from "vitest";
import { MemoryRouter } from "react-router-dom";
import { ViteComponent } from "../../pages/vite";

vi.mock("../../pages/navbar", () => {
  return {
    default: () => <nav data-testid="mock-navbar">Mocked NavBar</nav>,
  };
});

describe("ViteComponent", () => {
  test("renders Vite logo with correct alt text", () => {
    // Given
    render(
      <MemoryRouter>
        <ViteComponent />
      </MemoryRouter>
    );
    const logo = screen.getByAltText(/vite logo/i);

    // Then
    expect(logo).toBeInTheDocument();
    expect(logo).toHaveClass("logo");
  });

  test("increments count on button click", () => {
    // Given
    render(
      <MemoryRouter>
        <ViteComponent />
      </MemoryRouter>
    );
    const button = screen.getByRole("button", { name: /count is 0/i });

    // When
    fireEvent.click(button);

    // Then
    expect(button).toHaveTextContent("count is 1");
  });
});
```

A ce moment ci, vous pouvez changer la variable `COMMAND_FRONT` à `test` et utiliser `docker compose up frontend`

## Implémenter les mecanismes de linting et formattage

### Configurer lint et formattage au sein du projet

1. Modifier le fichier `eslint.config.js` comme ceci :

```js
import js from '@eslint/js'
import globals from 'globals'
import reactHooks from 'eslint-plugin-react-hooks'
import reactRefresh from 'eslint-plugin-react-refresh'
import tseslint from 'typescript-eslint'
import { globalIgnores } from 'eslint/config'
import prettierPlugin from 'eslint-plugin-prettier'

export default tseslint.config([
  globalIgnores(['dist']),
  {
    files: ['**/*.{ts,tsx}'],
    extends: [
      js.configs.recommended,
      tseslint.configs.recommended,
      reactHooks.configs['recommended-latest'],
      reactRefresh.configs.vite,
    ],
    languageOptions: {
      ecmaVersion: 2020,
      globals: globals.browser,
    },
    plugins: {
      prettier: prettierPlugin,
    },
    rules: {
      ...prettierPlugin.configs.recommended.rules,
      'prettier/prettier': 'error',
    },
  },
])
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

## Implémentation de la génération de documentation automatique

Créer un fichier `typedoc.json` dans le dossier `frontend` :

```json
{
  "name": "frontend docs",
  "out": "../docs/frontend",
  "plugin": ["typedoc-plugin-markdown"],
  "exclude": ["node_modules/**", "src/__tests__/**/*"],
  "entryPoints": ["src/**/*.tsx"]
}
```
