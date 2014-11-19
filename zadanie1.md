Sprzęt, na którym pracuję: Pentium B970 @ 2.3 GhZ, 6 GB RAM, Linux Mint Xfce 17

### Zadanie 1a

Najpierw przetworzyłem bazę za pomocą skryptu z repozytorium prowadzącego:

```sh
if [ -z "$1" ] ; then
echo "First replaces all the \\n with spaces then replaces all the \\r with \\n"
echo "Replace header line with: _id,title,body,tags (for mongoDB)"
echo "usage: $0 input.csv output.csv"
exit 1;
fi

tr '\n' ' ' < "$1" | tr '\r' '\n' > "$2"
sed -i '$ d' "$2"

sed -i '1 c "_id","title","body","tags"' "$2" 

```

Później zaimportowałem plik do bazy za pomocą polecenia:

```sh
time mongoimport -c Topics --type csv --file Train2.csv --headerline
```

W wypadku Postgresa wygląda to nieco inaczej - najpierw trzeba stworzyć odpowiednią tabelę:

```sh
CREATE TABLE Topics(
ID INT PRIMARY KEY NOT NULL,
TITLE CHAR(128) NOT NULL,
BODY CHAR(128) NOT NULL,
TAGS CHAR(128) NOT NULL,
);
```

Dopiero potem możemy wypełnić tabelę danymi z pliku:

```sh
COPY Topics FROM '/home/pc/nosql/Train2.csv' DELIMITER ',' CSV;
```

Czasy dla importu wyniosły: MongoDB 7m53s, PostgreSQL 9m01s.

### Zadanie 1b

Do obliczenia ilości zaimportowanych rekordów użyłem polecenia:

```sh
db.Topics.count()
```

które dało wynik 6034195.

Zadanie 1c

```sh
baza = db.Train.find();

var tagsUnique = {};
var tagsNumber = 0;

baza.forEach(function(train){
    var tagsArray = [];
    if(typeof train.tags === "string") {
        tagsArray = train.tags.split(" ");
        db.Train.update({_id: train._id}, {$set: {tags: tagsArray}});
    } else if(typeof train.tags === "number") {
        tagsArray.push(train.tags.toString());
        db.Train.update({_id: train._id}, {$set: {tags: tagsArray}});
    } else {
        tagsArray = train.tags;
    }
    tagsNumber += tagsArray.length;
    tagsArray.forEach(function(tag) {
        if(typeof tagsUnique[tag] === "undefined")
            tagsUnique[tag] = 1;
    });
});

print("Wszystkie: " + tagsNumber);
print("Unikalne: " + Object.keys(tagsUnique).length);

```

### Zadanie 1d

Do rozwiązania zadania użyłem danych ze strony poipoint.pl. Dane dotyczą lokalizacji szkół podstawowych w Polsce. 

Na początek zaimportowałem dane do mongo

```sh
mongoimport --db zadanie -d geo -c schools < /home/pc/nosql/szkoly.json
```

Do bazy wykonałem następujące zapytania:


```sh
db.schools.ensureIndex({"loc" : "2dsphere"})
```
 
by całość miała prawo działać.

Szkoły podstawowe w odległości 50 km od Gdańska


```sh
 db.schools.find( { loc : { $near :
                         { $geometry :
                             { type : "Point" ,
                               coordinates: [ 18.639, 54.360 ] } },
                           $maxDistance : 50000
              } }, { _id: 0 } )
```

[JSON](/data/zapytanie nr 1.json), [GeoJSON](master/data/zapytanie nr 1.geojson)

Szkoły podstawowe w odległości 50 km od Warszawy

```sh
 db.schools.find( { loc : { $near :
                         { $geometry :
                             { type : "Point" ,
                               coordinates: [ 21.020, 52.259 ] } },
                           $maxDistance : 50000
              } }, { _id: 0 } )
```

[JSON](master/data/zapytanie nr 2.json), [GeoJSON](master/data/zapytanie nr 2.geojson)

sdfd
