# Installation des outils de bases pour une architecture de développement web dockerisée

Cette installtion comprend les bases pour installler un environnement dockerisé et interragir avec gitlab.

## I. Docker (WSL)

Tout d'abord, vous devez savoir combien d'employés travaillent dans votre entreprise, car l'accord de licence de Docker Desktop n'autorise pas l'installation si votre entreprise compte plus de 500 employés.

### a. Docker Desktop

Si votre entreprise compte moins de 500 employés, vous avez la chance de pouvoir utiliser Docker Desktop.
[Téléchargez Docker Desktop](https://www.docker.com/products/docker-desktop/) et installez-le.

### b. WSL Kernel + Docker

Pour installer Docker sans Docker Desktop, vous aurez besoin d'un noyau Linux. Sous Windows, vous pouvez utiliser WSL (Windows Subsystem for Linux), qui fournit un noyau Linux.

Utilisez la commande suivante pour installer Ubuntu :

```bash
wsl --install -d Ubuntu
```

Si cela fonctionne, vous pouvez passer à la section [Docker sur WSL](#docker-sur-wsl). Sinon veuillez utiliser le [guide de Microsoft](https://learn.microsoft.com/fr-fr/windows/wsl/install-manual). Nous conserverons à la suite de ce paragraphe, une version du processus d'installation qui a fonctionné, au cas où Microsoft l'enlèverait.

Utilisez **PowerShell** en mode **administrateur**.

1. Exécutez la commande suivante :

```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

2. Vérifiez que votre version est au moins la version 1903 ou supérieure, avec la build 18362.1049 ou plus récente, en appuyant sur Win + R et en exécutant `winver`.

3. Exécutez la commande suivante :

```bash
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

4. Redémarrez l'ordinateur.
5. [Téléchargez](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) et installez la mise à jour pour WSL2.
6. Exécutez la commande suivante :

```bash
wsl --set-default-version 2
```

7. [Téléchargez](https://aka.ms/wslubuntu2004) et installez Ubuntu.

### c. Docker sur WSL

1. Connectez-vous à votre instance WSL avec la commande suivante :

```bash
wsl -d Ubuntu
```

2. Mettez à jour la distribution :

```bash
sudo apt update && sudo apt upgrade -y
```

3. Installez les dépendances :

```bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common libssl-dev libffi-dev git wget nano
```

4. Créez un groupe Docker et ajoutez l'utilisateur :

```bash
sudo groupadd docker
sudo usermod -aG docker ${USER}
```

5. Ajoutez la clé GPG du dépôt Docker :

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

6. Installez le dépôt :

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

7. Installez et mettez à jour Docker :

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Vous pouvez maintenant démarrer Docker avec la commande `dockerd`.

## II. Installer Visual Studio Code et son extension WSL

1. [Téléchargez VSCode](https://code.visualstudio.com/download) et installez-le.
2. Installez l'extension [WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl), elle vous permet de coder depuis windows sur votre distribution WSL.
3. Extensions recommandées :
   - [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) : Permet de naviguer facilement entre les branches et les commits.
   - [Conventional Commits](https://marketplace.visualstudio.com/items?itemName=vivaxy.vscode-conventional-commits) : Vous aide à nommer vos commits selon les standards conventionnels.

Avec l'extension WSL, vous pouvez développer au sein de WSL depuis windows et directement créer des shells WSL. Pour cela, cliquer sur l'icone remote explorer sur la barre de gauche, selectionner WSL et ouvrer le dossier `/home`.

## III. Créer une Clé SSH et l’Ajouter à GitLab

Exécutez la commande suivante pour créer une clé SSH, nommez-la comme vous le souhaitez et définissez un mot de passe :

```bash
ssh-keygen -t ed25519
```

Ajoutez ensuite la clé publique à votre compte GitLab. Avant d'interagir avec le dépôt distant, utilisez :

```bash
eval "$(ssh-agent -s)"
ssh-add /chemin/vers/votre/clé/privée
```

## IV. Ajout de certificats dans WSL et connection à Nexus

1. Télécharger les certificats [root](https://consultantssolutec.sharepoint.com/sites/fichierslabsolutec/Lyon/Forms/AllItems.aspx?id=%2Fsites%2Ffichierslabsolutec%2FLyon%2F1%2E%20Documents%20Utiles%2FCertificats%2Froot%2Ecrt&viewid=91efca93%2Df8e4%2D4d8b%2Da2ae%2D71d62a832b05&parent=%2Fsites%2Ffichierslabsolutec%2FLyon%2F1%2E%20Documents%20Utiles%2FCertificats) et [intermediate](https://consultantssolutec.sharepoint.com/sites/fichierslabsolutec/Lyon/Forms/AllItems.aspx?id=%2Fsites%2Ffichierslabsolutec%2FLyon%2F1%2E%20Documents%20Utiles%2FCertificats%2Fintermediate%2Ecrt&viewid=91efca93%2Df8e4%2D4d8b%2Da2ae%2D71d62a832b05&parent=%2Fsites%2Ffichierslabsolutec%2FLyon%2F1%2E%20Documents%20Utiles%2FCertificats).
2. Ajoutez les dans `/usr/local/share/ca-certificates/` via l'explorateur de fichier
3. Mettez à jour les certificats :

```bash
sudo update-ca-certificates
```

4. Connectez vous à Nexus en utilisant vos login SSO:

```bash
docker login --username <username> registry.nexus.preprod.lab.solutec
```

## V. Cloner votre projet et démaré votre shell WSL dans le bon chemin

Éditez votre fichier `~/.bashrc` et ajoutez la ligne suivante à la fin :

```bash
dockerd & cd /mnt/c/Users/<user>/votre/chemin/vers/votre/projet;
```

Cela démarrera Docker et changera votre répertoire de travail à chaque fois que vous vous connecterez à votre distribution. (Si Docker est déjà en cours d'exécution, un avertissement s'affichera indiquant que Docker est déjà lancé).

## VI. Optionnel

Nous vous conseillons de changer votre thème d'icone via l'extension [material-icon](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme) qui permet une meilleure lisibilité.