## Tuto installation

#### PHP

 * Pour installer le nécessaire pour php, se référer à ce site :
 https://openclassrooms.com/courses/concevez-votre-site-web-avec-php-et-mysql/preparer-son-ordinateur-2
et ensuite installer composer https://getcomposer.org/download/

 __ou__
 * Pour Linux http://laravel.com/docs/5.0/homestead
 * Pour windows
http://laragon.org/

#### MongoDB et elasticsearch

* Aller sur <https://www.mongodb.org/downloads?_ga=1.118639489.902302554.1453231495#production> pour récupérer MongoDB.

* Aller sur https://www.elastic.co/downloads/past-releases/elasticsearch-1-4-4 pour avoir elasticsearch. ( Java doit être installé sur l'ordinateur ).

* Le plugin qui fera le lien entre MongoDB et elasticsearch a besoin du plugin _elasticsearch-mapper-attachment_. Entrer le code suivant dans le dossier bin de elasticsearch :

```
plugin -install elasticsearch/elasticsearch-mapper-attachments/2.4.1
```
S'il y a une erreur : "JAVA_HOME environment variable must be set!" se référer à ceci : http://www.wikihow.com/Set-Java-Home

 * Le plugin principal à présent :
 ```
 plugin -install com.github.richardwilly98.elasticsearch/elasticsearch-river-mongodb/2.0.9
 ```

 * Démarrer à présent MongoDB en mode replicaSet (dossier bin de MongoDB):
````
mongod --port 27017 --dbpath <path> --replSet rs0
````
< path > est le chemin du dossier où MongoDB stockera les données.

* Exécuter mongo et taper :
````
cfg = {
   "_id" : "rs0",
   "version" : 1,
   "members" :
      [ { "_id" : 0,
          "host" : "localhost:27017"
      } ]
}
rs.initiate(cfg)
````
Et voilà !

Pour créer un nouvel index, il suffit de taper :
````
curl -XPUT "localhost:9200/_river/test/_meta" -d '
{
  "type": "mongodb",
  "mongodb": {
    "db": "test",
    "collection": "restaurants"
  },
 "index": {
   "name": "resto",
   "type": "restau"
 }
}'
````
* < name > le nom de l'index
* < databaseName > nom de la base de donnée
* < CollectionName > nom de la collection
* < type > type de document ( pour qualifier les documents json ; pas très important )

Pour avoir une interface pour intéragir avec elasticsearch :

  * ``elasticsearch/bin/plugin -install mobz/elasticsearch-head ``
  *  Ouvrir http://localhost:9200/_plugin/head/


Tuto dont je me suis inspiré : http://blog.viseo-bt.com/indexation-recherche-documents-mongodb-elasticsearch/
