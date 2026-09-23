# TP Docker 2

## Exercice : Optimisation d’une application et de son dockerfile


**0. Etat initial**

Problèmes :

| Problème | Pourquoi |
|----------|----------|
| `FROM node:latest` | Image très lourde |
| `COPY node_modules` | Inutile, `npm install` le refait |
| Pas de `.dockerignore` | Tout le dossier est copié dans l'image, donc pas optimiser |
| `apt-get install ...` | Paquets pas utilisés par l'application et donc inutile |
| `EXPOSE 3000 4000 5000` | L'app utilise seulement le port 3000 |
| `NODE_ENV=development` | On veut une image de production pas sur la branche de developpement |
| `USER root` | Pas de sécurité, compte admin |

```bash
docker build -t node-app:step0 .
docker images
docker run -p 3000:3000 node-app:step0
```
```
REPOSITORY          TAG       IMAGE ID       CREATED             SIZE
node-app            step0     6bff675b924f   1 second ago   1.88GB
```

**1. Nettoyage du depot**

```bash
git rm --cached .DS_Store
```
```
Docker version 28.5.1, build e180ab8
```
