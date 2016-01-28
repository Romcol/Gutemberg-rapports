# Tuto

Vous pourrez obtenir un exemple de jeu de données [ici](https://docs.mongodb.org/getting-started/cpp/import-data/). J'utiliserai ce jeu de données pour donner des exemples de codes.

### Elasticsearch

Elasticsearch ne sera utilisé uniquement pour faire des requêtes de recherche. Il existe des fonctions pour insérer des données, mettre à jour, etc. Mais, elles ne font que modifier l'index et pas la base de données.

Pour commencer, il faut insérer dans le code :

```php
require 'vendor/autoload.php';

$client = Elasticsearch\ClientBuilder::create()->build();
```

Cela permet de communiquer avec l'instance d'Elasticsearch.

Pour récupérer un document à partir de son id :
```php
$params = [
    'index' => 'resto',
    'type' => 'restau',
    'id' => '569eaac8a2e885c3f5013135',
 ];

$results = $client->get($params);
print_r($results);
```

Au format JSON, cela donne le document ( ce ne sera pas forcément le même pour votre base de données) :
```json
{
  "address": {
     "building": "1007",
     "coord": [ -73.856077, 40.848447 ],
     "street": "Morris Park Ave",
     "zipcode": "10462"
  },
  "borough": "Bronx",
  "cuisine": "Bakery",
  "grades": [
     { "date": { "$date": 1393804800000 }, "grade": "A", "score": 2 },
     { "date": { "$date": 1378857600000 }, "grade": "A", "score": 6 },
     { "date": { "$date": 1358985600000 }, "grade": "A", "score": 10 },
     { "date": { "$date": 1322006400000 }, "grade": "A", "score": 9 },
     { "date": { "$date": 1299715200000 }, "grade": "B", "score": 14 }
  ],
  "name": "Morris Park Bake Shop",
  "restaurant_id": "30075445"
}
```

Cependant, dans php, le fichier obtenu est (pour avoir cet affichage, il faut mettre ```<pre>``` au début du fichier et ```</pre>``` à la fin)

```php
Array
(
    [_index] => resto
    [_type] => restau
    [_id] => 569eaac8a2e885c3f5013135
    [_version] => 1
    [found] => 1
    [_source] => Array
        (
            [address] => Array
                (
                    [zipcode] => 10462
                    [coord] => Array
                        (
                            [0] => -73.856077
                            [1] => 40.848447
                        )

                    [street] => Morris Park Ave
                    [building] => 1007
                )

            [restaurant_id] => 30075445
            [name] => Morris Park Bake Shop
            [cuisine] => Bakery
            [_id] => 569eaac8a2e885c3f5013135
            [borough] => Bronx
            [grades] => Array
                (
                    [0] => Array
                        (
                            [date] => 2014-03-03T00:00:00.000Z
                            [grade] => A
                            [score] => 2
                        )

                    [1] => Array
                        (
                            [date] => 2013-09-11T00:00:00.000Z
                            [grade] => A
                            [score] => 6
                        )

                    [2] => Array
                        (
                            [date] => 2013-01-24T00:00:00.000Z
                            [grade] => A
                            [score] => 10
                        )

                    [3] => Array
                        (
                            [date] => 2011-11-23T00:00:00.000Z
                            [grade] => A
                            [score] => 9
                        )

                    [4] => Array
                        (
                            [date] => 2011-03-10T00:00:00.000Z
                            [grade] => B
                            [score] => 14
                        )

                )

        )

)
```

Pour une recherche dans le contenu des documents, voilà un exemple de recherche du terme "Bronx" parmi les noms de quartiers des restaurants :
```php
$json = '{

    "query": {
        "bool": {
            "must": [
                {
                    "query_string": {
                        "default_field": "restau.borough",
                        "query": "Bronx"
                    }
                }
            ],
            "must_not": [ ],
            "should": [ ]
        }
    },
    "from": 0,
    "size": 10,
    "sort": [ ],
    "aggs": { }

}';

$params = [
    'index' => 'resto',
    'type' => 'restau',
    'body' => $json
];

$results = $client->search($params);
print_r($results);
```

* La partie ```must``` contient les termes que doit avoir les documents recherchés.
* La partie ```must_not``` les termes que le document ne doit pas contenir
* La partie ```should``` les documents qui auront ces termes seront mis en haut de la liste
* ```from``` correspond à l'indice du document par lequel vous voulez que la liste des résultats commence
* ```size``` est le nombre de résultats que l'on veut récupérer
* ```sort``` permet de trier les résultats ( ce sera détaillé plus tard )
* ```aggs``` permet de faire des aggrégations ( ce sera détaillé plus tard )

Il n'est pas obligatoire de mettre à chaque fois tous les éléments, par exemple pour chercher le terme "french" dans tous les éléments :

```php
$json = '{

    "query": {
        "query_string": {
            "default_field": "_all",
            "query": "french"
        }
    }
}';

$params = [
    'index' => 'resto',
    'type' => 'restau',
    'body' => $json
];

$results = $client->search($params);
print_r($results);
```

Infos supplémentaires : https://www.elastic.co/guide/en/elasticsearch/client/php-api/2.0/index.html

### MongoDB

Pour commencer, il faut mettre :

```php
require 'vendor/autoload.php';

$m = new MongoClient();
   
   // select a database
   $db = $m->test;   
   $collection = $db->restaurants;
```
"test" est le nom de la base de données.

Pour insérer un nouveau document, il suffit de faire :
```php
$collection->insert($doc);
```
```$doc``` aura bien sûr été définit avant, au format de php :
```php
 Array
        (
            [restaurant_id] => 30075445
            [name] => Morris Park Bake Shop
            [cuisine] => Bakery
            [_id] => 569eaac8a2e885c3f5013135
// ...
)
```

Pour mettre à jour un document
```php
$collection->update($critere, array('$set' => array("name" => "newName")));
```
* ```$critere``` est le critere qui permet de trouver le(s) document(s) que vous voulez modifier.
Pour utiliser le id comme critére : ``` $critere = array('_id' => new MongoId("56aa28854e8aa90828000029")); ```
* "set" permet de modifier un champ sans écraser les autres
* Je mettrai plus de détails sur la mise à jour au fur et à mesure

Pour supprimer un document :
```php
$collection->remove($critere);
```

Infos supplémentaires : https://secure.php.net/manual/fr/mongo.manual.php


