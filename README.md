# Frontend Dockerfile

## Création du projet

Le projet a été créé avec Vite :

```bash
npm create vite@latest frontend-dockerfile
```

Puis les dépendances ont été installées :

```bash
cd frontend-dockerfile
npm install
```

Pour vérifier que l'application fonctionne avant sa conteneurisation :

```bash
npm run dev
```

Par défaut, Vite utilise le port :

```text
5173
```

---

# Dockerisation de l'application

## Dockerfile

À la racine du projet, un fichier nommé `Dockerfile` a été créé.

```dockerfile
FROM node:24

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

---

## Explication du Dockerfile

### Image de base

```dockerfile
FROM node:24
```

L'image officielle **Node.js 24** est utilisée comme base de l'image Docker.

Elle fournit notamment Node.js et npm, nécessaires pour installer les dépendances et lancer Vite.

### Répertoire de travail

```dockerfile
WORKDIR /app
```

Cette instruction définit `/app` comme répertoire de travail à l'intérieur du conteneur.

Les instructions suivantes seront donc exécutées depuis ce dossier.

### Copie des fichiers npm

```dockerfile
COPY package*.json ./
```

Les fichiers `package.json` et `package-lock.json` sont copiés dans le conteneur.

Ils permettent à npm de connaître les dépendances nécessaires au projet.

### Installation des dépendances

```dockerfile
RUN npm install
```

Cette instruction installe les dépendances du projet.

Les dépendances de développement sont également installées car **Vite est nécessaire pour lancer le serveur de développement**.

### Copie du projet

```dockerfile
COPY . .
```

Le reste des fichiers du projet est ensuite copié dans le répertoire `/app` du conteneur.

### Documentation du port

```dockerfile
EXPOSE 5173
```

Cette instruction indique que l'application utilise le port `5173` à l'intérieur du conteneur.

`EXPOSE` ne publie pas le port sur la machine hôte. Le mapping du port est réalisé au lancement du conteneur avec l'option `-p`.

### Démarrage de Vite

```dockerfile
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

Cette commande démarre le serveur de développement Vite.

L'option :

```text
--host 0.0.0.0
```

est importante dans Docker.

Sans elle, Vite peut écouter uniquement sur l'interface locale du conteneur et l'application ne serait alors pas accessible depuis la machine hôte.

---

# Fichier .dockerignore

À la racine du projet, un fichier `.dockerignore` a également été créé :

```dockerignore
node_modules
dist
.git
```

Ce fichier permet d'exclure certains fichiers et répertoires du contexte envoyé à Docker pendant la construction de l'image.

En particulier :

- `node_modules` : les dépendances sont installées directement dans l'image avec `npm install` ;
- `dist` : les fichiers de compilation ne sont pas nécessaires ici ;
- `.git` : l'historique Git n'est pas utile dans l'image ;

Cela permet notamment de réduire le contexte de construction envoyé à Docker.

---

# Construction de l'image

Depuis la racine du projet :

```bash
docker build -t frontend-dockerfile .
```

Cette commande construit une image Docker nommée :

```text
frontend-dockerfile
```

---

# Lancement du conteneur

Une fois l'image construite :

```bash
docker run --name frontend-test -p 4242:5173 frontend-dockerfile
```

Le mapping :

```text
4242:5173
```

signifie que :

- `4242` est le port utilisé sur la machine hôte ;
- `5173` est le port utilisé par Vite dans le conteneur.

L'application est alors accessible sur :

```text
http://localhost:4242
```

---

# Consultation des logs

Pour afficher les logs du conteneur :

```bash
docker logs frontend-test
```

Il est également possible de suivre les logs en temps réel avec :

```bash
docker logs -f frontend-test
```

---

# Arrêt et suppression du conteneur

Pour arrêter le conteneur :

```bash
docker stop frontend-test
```

Pour le supprimer :

```bash
docker rm frontend-test
```

Il est également possible de forcer directement sa suppression :

```bash
docker rm --force frontend-test
```

---

# Modification du projet

Les sources de l'application sont copiées dans l'image au moment du :

```dockerfile
COPY . .
```

Une modification des fichiers sur la machine hôte ne modifie donc pas automatiquement les fichiers déjà présents dans l'image Docker.

Après une modification du projet ou du `Dockerfile`, il faut reconstruire l'image :

```bash
docker build -t frontend-dockerfile .
```

Puis supprimer l'ancien conteneur :

```bash
docker rm --force frontend-test
```

Et créer un nouveau conteneur :

```bash
docker run --name frontend-test -p 4242:5173 frontend-dockerfile
```

Un simple redémarrage de l'ancien conteneur ne suffit pas, car celui-ci utilise toujours l'image avec laquelle il a été créé.

---

# Structure principale

```text
frontend-dockerfile/
├── src/
├── public/
├── .dockerignore
├── Dockerfile
├── package.json
├── package-lock.json
├── index.html
└── README.md
```

---

# Commandes principales

### Installer les dépendances en local

```bash
npm install
```

### Lancer l'application sans Docker

```bash
npm run dev
```

### Construire l'image Docker

```bash
docker build -t frontend-dockerfile .
```

### Créer et lancer le conteneur

```bash
docker run --name frontend-test -p 4242:5173 frontend-dockerfile
```

### Afficher les logs

```bash
docker logs frontend-test
```

### Supprimer le conteneur

```bash
docker rm --force frontend-test
```

### Reconstruire puis relancer après une modification

```bash
docker build -t frontend-dockerfile .
docker rm --force frontend-test
docker run --name frontend-test -p 4242:5173 frontend-dockerfile
```

---

## Résultat

L'image Docker contient l'application React ainsi que ses dépendances.

Lors du démarrage du conteneur, Vite est lancé sur :

```text
0.0.0.0:5173
```

Le port `5173` du conteneur est publié sur le port `4242` de la machine hôte grâce à :

```bash
-p 4242:5173
```

L'application peut ainsi être consultée depuis :

```text
http://localhost:4242
```
