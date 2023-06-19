# Rapport Devops AISSAT Djedjiga

# **TP1**

### 1-1 : Documentez les essentiels du conteneur de la base de données (commands and Dockerfile).

Explication du Dockerfile :
ENV POSTGRES_DB=mydb \ POSTGRES_USER=postgres \ POSTGRES_PASSWORD=postgres : les variables d'environnement définissent les paramètres de configuration pour la base de données. on spécifie le nom de la base de données (mydb), le nom d'utilisateur (postgres) et le mot de passe (postgres).
COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/ : Cette commande copie le fichier SQL 01-CreateScheme.sql dans le répertoire /docker-entrypoint-initdb.d/ du conteneur. 
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/ : Cette commande copie le fichier SQL 02-InsertData.sql dans le répertoire /docker-entrypoint-initdb.d/ du conteneur. Ce fichier contient les instructions pour insérer des données dans la base de données.
EXPOSE 5432 : indique que le container expose le port 5432

#### Étape 1 : Database

*  Construction de l'image de la bd :
``` docker build -t mydatabase . ```
*  Démarrage du container de la bd :
```docker run -d -p 5432:5432 --name database mydatabase```


#### Étape 2 : Init database

* Création du network : ``` docker network create app-network ```
* Reconstruction de l'image de la bd : ``` docker build -t mydatabase . ```
* Démarrage du container de la bd en spécifiant le réseau :
```docker run -d -p 5432:5432 --name database --network app-network mydatabase ```

#### Étape 3 : Persist data

* Construction de l'image de la db avec un volume : ``` docker build -t mydatabase . ```
* Démarrage du conteneur de la db avec un volume monté :
``` docker run -d -p 5432:5432 --name database --network app-network -v mydatabase_data:/var/lib/postgresql/data mydatabase ```


----------------

### 1-2 : Documentez les essentiels du containeur Backend API (commands and Dockerfile).

Les commandes essentielles sont la compilation du code Java (javac), la création de l'image (docker build) et le démarrage du container (docker run).
Le Dockerfile doit spécifier l'image Java appropriée, copier le code Java compilé dans le container et exécuter le code avec la commande java -jar.


### Étape 4 : Backend API

* Construction de l'image du backend : ``` docker build -t mybackend . ```
* Démarrage du container du backend API en spécifiant le network :
``` docker run -d -p 8080:8080 --name backend --network app-network mybackend ```

----------------

### 1-3 : Explication du multistage build et de chaque étape du Dockerfile.

Explication du Dockerfile :
Utilisation d'un build multistage pour simplifier la construction de l'application Java.
Etapes du dockerfile : 
la première étape pour construire le code Java et une pour exécuter le code avec une image JRE.
Le code Java compilé est copié de l'étape précédente à l'aide de COPY --from=0.
Exécution de l'application Java avec la commande java -jar.

Le multistage build permet de séparer la construction du code Java de son exécution dans des images différentes.
#### Étape 5 : Multistage build

* Construction de l'image avec le build multistage : ``` docker build -t myapp . ```
* Démarrage du container de l'application : ``` docker run -d -p 8080:8080 --name app myapp ```

#### Étape 6 : HTTP Server

* Construction de l'image pour le serveur HTTP : ``` docker build -t myhttpserver . ```
* Démarrage du container du serveur HTTP en spécifiant le réseau :
``` docker run -d -p 80:80 --name httpserver --network app-network myhttpserver ```

----------------
### 1-3 Document docker-compose most important commands. 1-4 Document your docker-compose file.

#### Étape 7 : Docker-compose 

* Exécution de la commande ``` docker-compose up ``` pour démarrer tous les containers en arrière-plan.

----------------
### 1-5 Document your publication commands and published images in dockerhub.

#### Étape 8 : Publish

* Connexion au registre Docker : ``` docker login ```
* Publication des images individuellement :
```docker push mydatabase```
```docker push mybackend```
```docker push myhttpserver```
```docker push myapp```

----------------

# **TP2**

### 2-1 What are testcontainers?

Testcontainers est une bibliothèque Java qui fournit des contneurs légers et jetables pour les tests d'intégration. Elle vous permet de créer et de gérer facilement des conteneurs, tels que des bases de données, des courtiers de messages ou d'autres services tiers, dans vos tests. Testcontainers simplifie la configuration et la suppression de ces conteneurs, ce qui facilite les tests de l'interaction de votre application avec des dépendances externes.

En utilisant Testcontainers, vous pouvez exécuter vos tests d'intégration avec de véritables instances des dépendances, telles qu'une base de données PostgreSQL, sans avoir besoin d'une configuration complexe ou de simulations. La bibliothèque gère le cycle de vie des conteneurs, en les démarrant automatiquement avant l'exécution de vos tests et en les arrêtant une fois les tests terminés.

Dans le fichier `pom.xml` fourni, les dépendances pour Testcontainers sont incluses pour permettre son utilisation dans les tests du projet. Les dépendances spécifiques mentionnées (`testcontainers`, `jdbc` et `postgresql`) sont nécessaires pour travailler avec Testcontainers afin de créer et de gérer des conteneurs pour des instances de bases de données PostgreSQL dans les tests d'intégration. Ces dépendances sont déclarées avec une portée `test`, ce qui indique qu'elles ne sont utilisées que pendant la phase de test de l'application.
### 2-2 Document your Github Actions configurations.

### 2-3 Document your quality gate configuration.


----------------

# **TP3**





