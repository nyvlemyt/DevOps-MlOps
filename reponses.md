## 1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?

Il vaut mieux sortir les variables d'environnement de l'image et les injecter avec le -e pour des questions :
- de sécurité : on ne veut pas que les venv soient dans un depot git, lisibles par tous, cela evite de figer des secrets dans l'image
- de réutilisabilité : on veut que l'image puisse etre utilisé dans plusieurs environnments avec des venv différents

## 1-2 Why do we need a volume to be attached to our postgres container?

Le plus important c'est pour conserver les données même si le conteneur est supprimé

## 1-3 Document your database container essentials: commands and Dockerfile.

Dans database/Dockerfile j'ai mis : 
```
FROM postgres:17.2-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

Ensuite build l’image : 
```
docker build -t my-database ./database
``` 

Puis lance un conteneur :
```
docker run -d \
  --name postgres-db \
  -p 5432:5432 \
  my-database
``` 

Pour verfier que le conteneur tourne : (une des trois)
```
docker ps
docker logs postgres-db
docker inspect postgres-db
``` 

Créer le réseau :
```
docker network create app-network
``` 

Supprimer ton conteneur postgres si besoin puis relancer en réseau :
```
docker rm -f postgres-db
``` 

Puis :
```
docker run -d \
  --name postgres-db \
  --network app-network \
  -p 5432:5432 \
  my-database
``` 

Lancer Adminer :
```
docker run -d \
  --name adminer \
  --network app-network \
  -p 8090:8080 \
  adminer
``` 

Tester en ouvrant http://localhost:8090 et en se connectant avec les identifiants : 
- System: PostgreSQL
- Server: postgres-db
- Username: usr
- Password: pwd
- Database: db


J'ai ajouté un script d'initialisation dans database/initdb/02-InsertData.sql pour insérer des données dans la base de données à la création du conteneur. 
Le Dockerfile a été modifié pour copier ce script dans le dossier d'initialisation de Postgres :
```
COPY initdb/ /docker-entrypoint-initdb.d/
``` 

Rebuild et relance proprement le conteneur pour que les changements soient pris en compte :
```
docker rm -f postgres-db
docker build -t my-database ./database
docker run -d \
  --name postgres-db \
  --network app-network \
  -p 5432:5432 \
  my-database
``` 

ReLancer Adminer pour verfier que les données ont bien été insérées :
```
docker rm -f adminer
docker run -d \
  --name adminer \
  --network app-network \
  -p 8090:8080 \
  adminer
``` 


Pour faire persister les données même si le conteneur est supprimé, on peut utiliser un volume Docker. 
Par exemple, pour créer un volume nommé "pgdata" et l'attacher au conteneur :
```
docker volume create pgdata
docker rm -f postgres-db
docker run -d \
  --name postgres-db \
  --network app-network \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  my-database
``` 

Pour tester que les données sont bien persistées, on relance Adminer, verifie les données.
```
docker rm -f adminer
docker run -d \
  --name adminer \
  --network app-network \
  -p 8090:8080 \
  adminer
``` 

On supprime le conteneur et on le relance, les données devraient toujours être là :
```
docker rm -f postgres-db
docker run -d \
    --name postgres-db \
    --network app-network \
    -p 5432:5432 \
    -v pgdata:/var/lib/postgresql/data \
    my-database
``` 

## 1-4 Why do we need a multistage build? And explain each step of this dockerfile.
- pour séparer compilation et exécution
- pour ne pas embarquer le JDK et les outils de build dans l’image finale
- pour réduire la taille de l’image
- pour améliorer sécurité et performance
- pour avoir une image finale plus propre et plus proche des pratiques pro


Pour le backend, j'ai créé une classe java puis compilé pour obtenir un fichier Main.class
```
cd backend
cd src
javac Main.java
cd .. 
cd .. 
``` 

Puis j'ai créé un Dockerfile pour le backend et build l'image :
```
docker build -t my-java-hello ./backend 
``` 

Enfin j'ai run le conteneur pour tester que ça fonctionne :
```
docker run --rm --name java-hello my-java-hello
``` 
et j'ai obtenu "Hello World!" dans les logs du conteneur.

##  1-5 Why do we need a reverse proxy?

- Il fournit un point d’entrée unique pour les clients.
- Il masque l’architecture interne de l’application.
- Il empêche l’exposition directe du service backend.
- Il peut gérer le routage, la terminaison SSL, l’équilibrage de charge et la distribution de contenu statique.
- Il améliore la maintenabilité et la sécurité.


## 1-9 Documenter les commandes de publication et les images publiées sur Docker Hub

Pour publier mes images Docker, je me suis d’abord connecté à Docker Hub :
```
docker login
``` 


Ensuite, j’ai tagué chaque image locale avec mon nom d’utilisateur Docker Hub et un tag de version :
```
docker tag my-database melvynlpp/my-database:1.0
docker tag my-backend melvynlpp/my-backend:1.0
docker tag my-httpd melvynlpp/my-httpd:1.0
```

Après cela, j’ai poussé les images vers Docker Hub :
```
docker push melvynlpp/my-database:1.0
docker push melvynlpp/my-backend:1.0
docker push melvynlpp/my-httpd:1.0
```

Images publiées :
```
melvynlpp/my-database:1.0
melvynlpp/my-backend:1.0
melvynlpp/my-httpd:1.0
```

## 1-10 Why do we put our images into an online repo?

#### Pourquoi met-on nos images dans un dépôt en ligne ?

Nous plaçons les images Docker dans un dépôt en ligne afin de pouvoir les partager, les versionner et les réutiliser facilement sur d’autres machines ou par d’autres membres de l’équipe.
Cela facilite également le déploiement, car un serveur ou un autre développeur peut récupérer exactement la même image sans avoir à la reconstruire localement.
Les registres en ligne sont aussi essentiels en environnement professionnel, car ils s’intègrent parfaitement avec les pipelines CI/CD et les workflows de déploiement.


```

``` 

```

``` 
```

``` 
```

``` 