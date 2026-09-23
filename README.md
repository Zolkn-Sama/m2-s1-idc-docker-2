# TP Docker 2

## Exercice : Optimisation d'une application et de son dockerfile

Commandes utilisées à chaque étape pour mesurer la taille de l'image :

```bash
docker build -t node-app:stepN .
docker images
docker run -p 3000:3000 node-app:stepN
```

### 0. Etat initial

Problèmes :

| Problème | Pourquoi |
|----------|----------|
| `FROM node:latest` | Image très lourde |
| `COPY node_modules` | Inutile, `npm install` le refait |
| Pas de `.dockerignore` | Tout le dossier est copié dans l'image |
| `apt-get install ...` | Paquets pas utilisés par l'application |
| `EXPOSE 3000 4000 5000` | L'application utilise seulement le port 3000 |
| `NODE_ENV=development` | On veut une image de production |
| `USER root` | Pas sécurisé, le conteneur tourne en admin |

```
REPOSITORY   TAG     IMAGE ID       CREATED        SIZE
node-app     step0   6bff675b924f   1 second ago   1.88GB
```

### 1. Nettoyage du dépôt et .dockerignore

Suppression du `.DS_Store` du dépôt :

```bash
git rm --cached .DS_Store
```

Ajout d'un `.dockerignore` :

```
node_modules
.git
.DS_Store
sujet
```

Taille : **1.89GB**

> Remarque : les commits des étapes 1 à 3 ne buildent pas tels quels, car le `.dockerignore` exclut `node_modules` alors que le dockerfile fait encore `COPY node_modules` supprimé à l'étape 4. Les tailles de ces étapes ont été mesurées après sans cette ligne par claude qui a verifer mon exercice.

### 2. Suppression du apt-get install

Suppression de la ligne :

```dockerfile
RUN apt-get update && apt-get install -y build-essential ca-certificates locales && ...
```

Taille : **1.81GB**

### 3. Image node alpine

Avant :

```dockerfile
FROM node:latest
```

Après, une image beaucoup plus légère :

```dockerfile
FROM node:22-alpine
```

Taille : **266MB**

### 4. Réorganisation des COPY

Avant :

```dockerfile
COPY node_modules ./node_modules
COPY . /app
RUN npm install
```

Après :

```dockerfile
COPY package*.json ./
RUN npm install --omit=dev
COPY . .
```

- On ne copie plus le `node_modules` de la machine.
- `--omit=dev` n'installe pas `nodemon`, qui ne sert pas en production.

Taille : **262MB**

### 5. Nettoyage du code

Suppression de `mongodb`, qui n'est jamais utilisé dans `server.js` :

```bash
npm uninstall mongodb
```

Suppression de la ligne inutile :

```dockerfile
RUN npm run build
```

Taille : **247MB**

### 6. Configuration de production et sécurité

Avant :

```dockerfile
EXPOSE 3000 4000 5000
ENV NODE_ENV=development
USER root
```

Après :

```dockerfile
ENV NODE_ENV=production
EXPOSE 3000
USER node
```

Taille : **247MB**

### Dockerfile final

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev
COPY . .
ENV NODE_ENV=production
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

```
REPOSITORY   TAG     IMAGE ID       CREATED        SIZE
node-app     final   774556225863   1 second ago   247MB
```

### Statisiques

| Etape | Taille |
|-------|--------|
| 0. Etat initial | 1.88GB |
| 1. .dockerignore | 1.89GB |
| 2. Suppression apt-get | 1.81GB |
| 3. Image alpine | 266MB |
| 4. Réorganisation des COPY | 262MB |
| 5. Nettoyage du code | 247MB |
| 6. Config production | 247MB |

On passe de **1.88GB à 247MB**, se qui est une grande amélioration
