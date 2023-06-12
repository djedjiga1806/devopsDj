# Rapport Devops AISSAT Djedjiga



#### Question 1-1 : Documentez les essentiels du conteneur de la base de données (commands and Dockerfile).

Explication du Dockerfile :
ENV POSTGRES_DB=mydb \ POSTGRES_USER=postgres \ POSTGRES_PASSWORD=postgres : Ces variables d'environnement définissent les paramètres de configuration pour la base de données. on spécifie le nom de la base de données (mydb), le nom d'utilisateur (postgres) et le mot de passe (postgres).
COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/ : Cette commande copie le fichier SQL 01-CreateScheme.sql dans le répertoire /docker-entrypoint-initdb.d/ du conteneur. Ce répertoire est utilisé par l'image Postgres pour exécuter automatiquement les scripts SQL lors de la création de la base de données.
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/ : Cette commande copie le fichier SQL 02-InsertData.sql dans le répertoire /docker-entrypoint-initdb.d/ du conteneur. Ce fichier contient les instructions pour insérer des données dans la base de données.
EXPOSE 5432 : Cette instruction indique que le container expose le port 5432, le port par défaut pour les connexions à la base de données PostgreSQL.

### Étape 1 : Database

*  Construction de l'image de la base de données :
``` docker build -t mydatabase . ```
*  Démarrage du container de la base de données :
```docker run -d -p 5432:5432 --name database mydatabase```


### Étape 2 : Init database

* Création du réseau Docker : ``` docker network create app-network ```
* Reconstruction de l'image de la base de données : ``` docker build -t mydatabase . ```
* Démarrage du container de la base de données en spécifiant le réseau :
```docker run -d -p 5432:5432 --name database --network app-network mydatabase ```

### Étape 3 : Persist data

* Construction de l'image de la base de données avec un volume : ``` docker build -t mydatabase . ```
* Démarrage du conteneur de la base de données avec un volume monté :
``` docker run -d -p 5432:5432 --name database --network app-network -v mydatabase_data:/var/lib/postgresql/data mydatabase ```


#### Question 1-2 : Documentez les essentiels du containeur Backend API (commands and Dockerfile).

Les commandes essentielles sont la compilation du code Java (javac), la création de l'image (docker build) et le démarrage du conteneur (docker run).
Le Dockerfile doit spécifier l'image Java appropriée, copier le code Java compilé dans le conteneur et exécuter le code avec la commande java -jar.


### Étape 4 : Backend API

* Construction de l'image du backend API : ``` docker build -t mybackend . ```
* Démarrage du container du backend API en spécifiant le réseau :
``` docker run -d -p 8080:8080 --name backend --network app-network mybackend ```

#### Question 1-3 : Explication du multistage build et de chaque étape du Dockerfile.

Explication du Dockerfile :
Utilisation d'un build multistage pour simplifier la construction de l'application Java.
Création d'un Dockerfile avec deux étapes (FROM), une pour construire le code Java et une pour exécuter le code avec une image JRE.
Le code Java compilé est copié de l'étape précédente à l'aide de COPY --from=0.
Exécution de l'application Java avec la commande java -jar.

Le multistage build permet de séparer la construction du code Java de son exécution dans des images différentes.
Chaque étape du Dockerfile est définie par une image de base, des instructions de copie et d'ex from user.

### Étape 5 : Multistage build

* Construction de l'image avec le build multistage : ``` docker build -t myapp . ```
* Démarrage du conteneur de l'application : ``` docker run -d -p 8080:8080 --name app myapp ```

### Étape 6 : HTTP Server

* Construction de l'image pour le serveur HTTP : ``` docker build -t myhttpserver . ```
* Démarrage du conteneur du serveur HTTP en spécifiant le réseau :
``` docker run -d -p 80:80 --name httpserver --network app-network myhttpserver ```

#### 1-3 Document docker-compose most important commands. 1-4 Document your docker-compose file.

### Étape 7 : Docker-compose 

* Exécution de la commande ``` docker-compose up ``` pour démarrer tous les conteneurs en arrière-plan.

#### 1-5 Document your publication commands and published images in dockerhub.

### Étape 8 : Publish

* Publication des images sur Docker hub
* Connexion au registre Docker : ``` docker login ```
* Publication des images individuellement :
```docker push mydatabase```
```docker push mybackend```
```docker push myhttpserver```
```docker push myapp```






