Do rozwiązania zadania posłużyłem się bazą danych [GetGlue](http://getglue-data.s3.amazonaws.com/getglue_sample.tar.gz). Są to dane z serwisu [IMDB](http://imdb.com) z lat 2007 - 2012 dotyczące filmów i seriali. 

### Import

Importu dokonałem następującym poleceniem:

```sh
mongoimport --db imdb -c imdb --type json --file /media/pc/14599F0F06EA98FB/nosql/getglue_sample.json
```

Czas dla tej operacji wyniósł 19m10s. Czas jest zawyżony (- byłem zmuszony do wrzucenia jsonów na inną partycję, gdyż skończyło mi się miejsce na linuxowej. Jest to inna partycja (ntfs) niż ta, na której jest linux (ext4), co odbija się dość znacząco na czasie importu. Wcześniej (operując na tej samej partycji) mongo importowało około ~21 500 rekordów na sekundę (a czas wyniósł około 14m30s), natomiast w tym wypadku zaczęło od ~23 000 na sekundę, po czym zaczęło sukcesywnie spadać (nawet do ~16 000!). 

Można przypuszczać, że problemy z wydajnością Mongo pod Windowsem mogą być uzależnione od systemu plików (w końcu Windowsy od XP w górę raczej używają ntfsa), aczkolwiek nie ukrywam, że byłoby to bardzo dziwne.

### Agregacje - przygotowanie

Do agregacji wykrzystałem bibliotekę [pymongo](http://api.mongodb.org/python/current/). 

Bibliotekę instalujemy za pomocą pip wpisując:
 
pip install pymongo

### Agregacja nr 1

Szukamy 10 filmów z największą ilością pozytywnych komentarzy:

#####MongoDB

```sh
db.imdb.aggregate( 
	{ $match: { "action": "Liked" }},
	{ $match: { "comment": {$ne: ""} } }, 
	{ $group: { _id: "$title", count: {$sum: 1} } }, 
	{ $sort: { count: -1 } }, { $limit: 10 } );
```

#####pymongo

```sh
from pymongo import MongoClient
db = MongoClient().imdb
db.imdb.aggregate( 
	{ "$match": { "action": "Liked" }},
	{ "$match": { "comment": {"$ne": ""} } }, 
	{ "$group": { "_id": "$title", "count": {"$sum": 1} } }, 
	{ "$sort": { "count": -1 } }, 
	{ "$limit": 10 } );
```
	
#####Rezultat:

```sh
{
	"result" : [
		{
			"_id" : "Lord of the Rings: The Return of the King",
			"count" : 815
		},
		{
			"_id" : "Fight Club",
			"count" : 725
		},
		{
			"_id" : "Iron Man",
			"count" : 722
		},
		{
			"_id" : "Pulp Fiction",
			"count" : 721
		},
		{
			"_id" : "The Hangover",
			"count" : 691
		},
		{
			"_id" : "X-Men",
			"count" : 667
		},
		{
			"_id" : "Monsters, Inc.",
			"count" : 644
		},
		{
			"_id" : "Braveheart",
			"count" : 638
		},
		{
			"_id" : "Kill Bill: Vol. 1",
			"count" : 631
		},
		{
			"_id" : "WALL-E",
			"count" : 624
		}
	],
	"ok" : 1
}
```

![Alt text](/master/img/aggr1.png)

### Agregacja nr 2

Szukamy 10 najpopularniejszych filmów:

#####MongoDB

```sh

db.imdb.aggregate(
    { $match: {"modelName": "movies"  } },
    { $group: {_id: "$title", count: {$sum: 1}} },
    { $sort: {count: -1} },
    { $limit : 10}
    );
	
```

```sh

#####pymongo

from pymongo import MongoClient
db = MongoClient().imdb
db.imdb.aggregate(
    { "$match": {"modelName": "movies"  } },
    { "$group": {"_id": "$title", "count": {"$sum": 1}} },
    { "$sort": {"count": -1} },
    { "$limit" : 10}
    );
```

#####Rezultat:

```sh
{
	"result" : [
		{
			"_id" : "The Twilight Saga: Breaking Dawn Part 1",
			"count" : 87521
		},
		{
			"_id" : "The Hunger Games",
			"count" : 79340
		},
		{
			"_id" : "Marvel's The Avengers",
			"count" : 64356
		},
		{
			"_id" : "Harry Potter and the Deathly Hallows: Part II",
			"count" : 33680
		},
		{
			"_id" : "The Muppets",
			"count" : 29002
		},
		{
			"_id" : "Captain America: The First Avenger",
			"count" : 28406
		},
		{
			"_id" : "Avatar",
			"count" : 23238
		},
		{
			"_id" : "Thor",
			"count" : 23207
		},
		{
			"_id" : "The Hangover",
			"count" : 22709
		},
		{
			"_id" : "Titanic",
			"count" : 20791
		}
	],
	"ok" : 1
}
```

![Alt text](/master/img/aggr2.png)

### Agregacja nr 3

Szukamy 10 najpopularniejszych seriali:

#####MongoDB

```sh
db.imdb.aggregate(
    { $match: {"modelName": "tv_shows"  } },
    { $group: {_id: "$title", count: {$sum: 1}} },
    { $sort: {count: -1} },
    { $limit : 10}
    );
```
	
#####pymongo

```sh

from pymongo import MongoClient
db = MongoClient().imdb
db.imdb.aggregate(
    { "$match": {"modelName": "tv_shows"  } },
    { "$group": {"_id": "$title", "count": {"$sum": 1}} },
    { "$sort": {"count": -1} },
    { "$limit" : 10}
    );

	```
	
#####Rezultat:

```sh

{
	"result" : [
		{
			"_id" : "The Big Bang Theory",
			"count" : 260686
		},
		{
			"_id" : "Fringe",
			"count" : 187910
		},
		{
			"_id" : "Nikita",
			"count" : 150683
		},
		{
			"_id" : "Glee",
			"count" : 146799
		},
		{
			"_id" : "Supernatural",
			"count" : 130454
		},
		{
			"_id" : "True Blood",
			"count" : 122913
		},
		{
			"_id" : "The Walking Dead",
			"count" : 119369
		},
		{
			"_id" : "The Vampire Diaries",
			"count" : 118000
		},
		{
			"_id" : "Game of Thrones",
			"count" : 108548
		},
		{
			"_id" : "Once Upon a Time",
			"count" : 99515
		}
	],
	"ok" : 1
}

```

![Alt text](/master/img/aggr3.png)

### Agregacja nr 4

Szukamy 12 reżyserów, którzy nakręcili najwięcej filmów (wcześniej było 10, ale dwa pierwsze wyniki zwracają "not available" i "various directors", a chodzi o nazwiska).

#####MongoDB

```sh

db.imdb.aggregate(
    { $match: {"modelName": "movies" || "tv_shows"  } },
    { $group: {_id: {"dir": "$director", id: "$title"}, count: {$sum: 1}} },
    { $group: {_id: "$_id.dir" , count: {$sum: 1}} },
    { $sort: {count: -1} },
    { $limit : 12}
    );
	
```

```sh

#####pymongo

from pymongo import MongoClient
db = MongoClient().imdb
db.imdb.aggregate(
    { "$match": {"modelName": "movies" || "tv_shows"  } },
    { "$group": {"_id": {"dir": "$director", "id": "$title"}, "count": {"$sum": 1}} },
    { "$group": {"_id": "$_id.dir" , "count": {"$sum": 1}} },
    { "$sort": {"count": -1} },
    { "$limit" : 12}
    );
	
```

#####Rezultat:

```sh

{
	"result" : [
		{
			"_id" : "not available",
			"count" : 1474
		},
		{
			"_id" : "various directors",
			"count" : 54
		},
		{
			"_id" : "alfred hitchcock",
			"count" : 50
		},
		{
			"_id" : "michael curtiz",
			"count" : 48
		},
		{
			"_id" : "woody allen",
			"count" : 47
		},
		{
			"_id" : "jesus franco",
			"count" : 43
		},
		{
			"_id" : "takashi miike",
			"count" : 43
		},
		{
			"_id" : "ingmar bergman",
			"count" : 42
		},
		{
			"_id" : "john ford",
			"count" : 42
		},
		{
			"_id" : "robert mckimson",
			"count" : 41
		},
		{
			"_id" : "steven spielberg",
			"count" : 41
		},
		{
			"_id" : "robert altman",
			"count" : 40
		}
	],
	"ok" : 1
}

```

![Alt text](/master/img/aggr4.png)