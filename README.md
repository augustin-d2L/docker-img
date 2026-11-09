  # Laboratoire Docker — Partie 1 : Serveur PostgreSQL
  Image PostgreSQL personnalisée qui initialise automatiquement la base `authentification` avec une table `client`.

# Docker Hub
https://hub.docker.com/r/augustind2l/serveur-postgresql

# Commandes de mise en place
## on construit l'image avec la commande 
```
docker build --load -t postgres-auth .
``` 
le `--load` sert pour enregistrer l'image dans Docker

## créer le volume avec la commande 
```
docker volume create donnees-postgresql
```

## on lance le conteneur avec les paramètres demandés avec la commande 
```
docker run -d --name serveur-postgresql -e POSTGRES_USER=administrateur -e POSTGRES_PASSWORD=MonMotDePasse -e POSTGRES_DB=authentification -p 5433:5432 -v donnees-postgresql:/var/lib/postgresql/data postgres-auth
```

# Vérification de la persistance
## Vérifier qu'on commence bien avec deux clients
![Image 1](https://raw.githubusercontent.com/augustin-d2L/docker-img/refs/heads/main/1.png)
## Ajouter un troisième client
```
docker exec -it serveur-postgresql psql -U administrateur -d authentification -c "INSERT INTO client (username, mot_de_passe) VALUES ('agagnon', 'Password789!');"
```
![Image 2](https://raw.githubusercontent.com/augustin-d2L/docker-img/refs/heads/main/2.png)
## Arrêter et supprimer le conteneur
```
docker stop serveur-postgresql
docker rm serveur-postgresql
```
## Recréer le conteneur avec le même volume
```
docker run -d --name serveur-postgresql -e POSTGRES_USER=administrateur -e POSTGRES_PASSWORD=MonMotDePasse -e POSTGRES_DB=authentification -p 5433:5432 -v donnees-postgresql:/var/lib/postgresql/data postgres-auth
```
## Vérifier que les trois clients sont présents
```
docker exec -it serveur-postgresql psql -U administrateur -d authentification -c "SELECT * FROM client;"
```
![Image 3](https://raw.githubusercontent.com/augustin-d2L/docker-img/refs/heads/main/3.png)

# Publication sur Docker Hub

## Renommer l'image selon la nomenclature demandée
```
docker tag postgres-auth:latest augustind2l/serveur-postgresql:1.0
```
`docker tag` ajoute un deuxième nom à l'image locale `postgres-auth` sans la copier. Le nouveau nom suit la forme `utilisateur/dépôt:version` exigée par Docker Hub.

## Se connecter à Docker Hub
```
docker login
```
Authentifie le poste avec le compte `augustind2l`, ce qui est nécessaire pour publier une image.

## Publier l'image
```
docker push augustind2l/serveur-postgresql:1.0
```
Envoie l'image dans le dépôt public `augustind2l/serveur-postgresql` avec le tag `1.0`.

## Récupérer l'image sur une autre machine
```
docker pull augustind2l/serveur-postgresql:1.0
```
Télécharge l'image publiée. On peut ensuite lancer le conteneur avec la même commande `docker run`, en remplaçant `postgres-auth` par `augustind2l/serveur-postgresql:1.0`.

# Questions
1. Quelle est la différence entre une image et un conteneur Docker ? 
   >L’image est un modèle figé, en lecture seule (comme une classe). Le conteneur est une instance qui tourne à partir de cette image (comme un objet). On peut lancer plusieurs conteneurs à partir d’une même image.
2. Quelle est la différence entre EXPOSE et -p ? 
   >`EXPOSE` (dans le Dockerfile) ne fait que documenter le port sur lequel l’application écoute dans le conteneur, sans rien ouvrir. `-p 5433:5432` (dans `docker run`) crée réellement le lien entre un port de l’hôte et un port du conteneur.
3. Pourquoi utilise-t-on un volume Docker pour conserver les données ? 
   >Un conteneur est éphémère. Quand on le supprime, tout ce qui a été écrit à l’intérieur disparaît. Le volume est un stockage géré par Docker, séparé du conteneur, donc les données survivent à sa suppression ou à son remplacement.
4. Pourquoi le script d'initialisation SQL ne s'exécute-t-il pas à chaque démarrage du conteneur ?
   >L’image postgres n’exécute les fichiers de /docker-entrypoint-initdb.d/ que si le dossier de données est vide, c’est-à-dire au tout premier démarrage.
