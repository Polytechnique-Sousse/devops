# Utiliser l'image de base Node.js LTS Alpine
FROM node:lts-alpine

# Définir le répertoire de travail dans le conteneur
WORKDIR /usr/src/app

# Copier les fichiers package*.json
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste des fichiers du serveur
COPY . .

# Exposer le port 5000
EXPOSE 5000

ENV MONGO_URI=mongodb://localhost:27017/employees

# Démarrer l'application
CMD ["npm", "start"]
