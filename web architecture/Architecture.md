# Créer une architecture de développement web dockerisée stateless avec linting et formatage

## Introduction

- Plateforme agnostique
  Cette architecture fonctionne de manière identique sur Windows, macOS et Linux, grâce à l’utilisation exclusive de Docker et Docker Compose.
  Elle reproduit fidèlement un environnement de production conteneurisé, ce qui permet de réduire drastiquement les bugs liés aux différences d’environnement local/serveur lors du déploiement.

- Léger et stateless
  L’environnement est stateless : les services applicatifs (frontend, backend) ne conservent aucun état entre les exécutions. Les images ne sont pas buildées, seules les volumes sont utilisés pour le partage de code, et la base de données est rendue persistante pour conserver les données pendant le développement. Cela permet un déploiement rapide, néccessitant que docker.

- Hot reload intégré
  Chaque service applicatif (frontend, backend, reverse proxy) est configuré avec un mécanisme de hot reload. Les modifications du code sont automatiquement détectées, et les serveurs redémarrent ou rechargent les modules en temps réel. Cela garantit un feedback immédiat pour les développeurs, sans avoir à relancer manuellement les services.

- Correcteurs de code intégrés
  L’environnement inclut des outils de linting (analyse de syntaxe) et de formatage automatique du code, selon les langages/frameworks utilisés. Ces outils permettent de maintenir un code propre, cohérent et standardisé, tout en réduisant les erreurs humaines et en améliorant la lisibilité.

- Divulgation d'information
  Comprend un reverse proxy qui améliore la cyber sécurité.

Schéma (reverse, front, back, bdd + optionellement une interface pour interragir)

## Création

1. Créer l'architecture de dossier suivante :

```bash
/
│
├── backend/           # Contient tout ce qui concerne le backend
├── CI/                # Contient les configurations utiles pour le pipeline et la production
├── database/          # Contient tout ce qui concerne la base de donnée
├── frontend/          # Contient tout ce qui concerne le frontend
├── proxy/             # Contient tout ce qui concerne le reverse proxy
│
├── docker-compose.yml # Orchestre l'application
├── .env               # Contient les variables d'environnement
└── .gitignore         # Définit les fichier locaux
```

2. Modifier le fichier `.env` comme ceci :

```bash
DOMAIN=localhost
PROTOCOL=http
COMMAND_FRONT=dev
COMMAND_BACK=dev
```

- **DOMAIN** et **PROTOCOL** définissent l'environnement dans lequel l'application tourne (par exemple en production, renseignez l’adresse IP et https).
- **COMMAND_FRONT** et **COMMAND_BACK** permettront de définir le mode de fonctionnement à exécuter pour chaque service lors du lancement de l'application.

3. Modifier le `.gitignore` comme ceci :

```bash
.env
.vscode/
```

4. Crée un ficher de configuration nginx `nginx.conf` dans le dossier `proxy` :

```conf
# Nginx configuration file for the developpement
events {}

http{
    server{
        listen 0.0.0.0:80;
        client_max_body_size 50M; # Définit la taille maximum des requètes (images, vidéos, ...)
    }
}
```

5. Créer un script `reload.sh` dans le dossier `proxy` pour ajouter le hot-reload du proxy :

```bash
#!/bin/bash

while inotifywait -e modify /etc/nginx/nginx.conf; do
    nginx -t && nginx -s reload
done
```

6. Créer un fichier de configuration nginx dans le dossier `CI`, celui-ci servira en production :

```conf
# Nginx configuration file for the production
events {}

http{
    include mime.types;

    server{
        listen 0.0.0.0:80;
        server_name localhost;
        client_max_body_size 50M;
        server_tokens off;

        add_header Content-Security-Policy "default-src 'self'
        style-src 'self' 'unsafe-inline';
        script-src 'self';
        img-src 'self' data:;
        frame-ancestors 'none';
        form-action 'self';"
        always;
        add_header Cross-Origin-Resource-Policy "same-origin" always;
        add_header Cross-Origin-Opener-Policy "same-origin" always;
        add_header X-Content-Type-Options "nosniff";
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), fullscreen=(), payment=(), usb=(), accelerometer=(), autoplay=(), clipboard-read=(), clipboard-write=(), gyroscope=(), magnetometer=(), midi=(), notifications=(), push=(), screen-wake-lock=(), speaker-selection=(), vibrate=(), vr=(), xr-spatial-tracking=()";

        root /<  !!!!!!! Changer le nom du projet lors du passage en production !!!!!!!   >/frontend/dist;
        index index.html;

        location / {
            try_files $uri $uri/ /index.html;
        }

        location /api {
            proxy_pass http://backend:8081;
        }
    }
}
```

7. Créer un fichier docker compose comprenant un service pour le proxy, nous allos par la suite spécifié d'autres services en fonctions de nos besoins.

```yaml
services:
  proxy:
    image: nginx:1.29
    container_name: proxy
    volumes:
      - ./proxy/nginx.conf:/etc/nginx/nginx.conf
      - ./proxy/reload.sh:/reload.sh:ro
    command: /bin/bash -c "apt update && apt install -y inotify-tools && nginx && bash /reload.sh"
    ports:
      - "80:80"
    networks:
      - default

networks:
  default: {}
```

Vous êtes maintenant invité à suivre les configurations plus spécifiques aux technologies que vous avez choisis :

Frontend :

- React
- Angular
- Vue

Backend :

- NodeJS
- Java SpringBoot
- Python
