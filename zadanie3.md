Sprzęt, na którym pracuję: Pentium B970 @ 2.3 GhZ, 6 GB RAM, Linux Mint Xfce 17

#Zadanie 3

###ANAGRAMY

Plik [word_list.txt](http://wbzyl.inf.ug.edu.pl/nosql/doc/data/word_list.txt) wczytałem za pomocą następującego polecenia:

``` 
mongoimport -d anagrams -c anagrams --type csv --file /media/pc/863E2B1D3E2B05B1/nosql/word_list.txt -f "words"
```

Następnie skorzystałem z mapreduce:

```
db.anagrams.mapReduce(
  function()
  {
    emit(Array.sum(this.words.split("").sort()), this.words);
  },
  function(key, values) 
  {
    return values.toString()
  },
  {
    query: {},
    out: "result"
  }
)
```


###WIKIPEDIA

Do wykonania zadania wykorzystałem [plik z artykułami z wikipedii](http://dumps.wikimedia.org/plwiki/latest/plwiki-latest-pages-articles-multistream.xml.bz2). Plik jest w formacie xml, co oznacza, że nie da się go zaimportować do mongo - trzeba go skonwertować. Moje próby skonwertowania tak dużego pliku xml do jsona zakończyły się fiaskiem - większość narzędzi, które próbowałem (np. [[1]](https://github.com/parmentf/xml2json), [[2]](https://github.com/hay/xml2json), [[3]](https://github.com/yihuang/xml2json) - wszystkie nazywają się tak samo, bo xml2json) wywalała błędy w stylu OutOfMemoryException i MemoryError (z powodu wielkości pliku), tak więc postanowiłem spróbować z konwersją do csv. W tym celu posłużyłem się narzędziem [XML2CSVGenericConverter](http://sourceforge.net/projects/xml2csvgenericconverter/files/?source=navbar):

```
time java -jar XML2CSVGenericConverter_V1.0.0.jar -v -i /nosql/wiki.xml -o /nosql/
```

Jak na tak duży plik konwersja zakończyła się moim zdaniem stosunkowo szybko:


```
real	23m23.156s
user	13m18.764s
sys	  1m28.312s
```

Następnie zaimportowałem bazę do mongo za pomocą polecenia

```
time mongoimport -d wiki -c wiki --type csv --file wiki.csv --headerline
```

ale przy tym poleceniu występował błąd:

```
exception:CSV file ends while inside quoted field
```

Znalazłem opis tego problemu i rozwiązanie wygląda następująco:

```
time mongoimport -d wiki -c wiki --type tsv --file wiki.csv --headerline --ignoreBlanks
```

Import potrwał trochę dłużej:

```
real	51m50.747s
user	6m13.195s
sys	1m21.482s
```

Następnie skorzystałem z mapreduce:

```
var map = function() {  
    var id = this.id;
    if (id) { 
        id = id.split(";"); 
        for (var i = id.length - 1; i >= 0; i--) {
            if (id[i])  {    
               emit(id[i], 1
            }
        }
    }
};
var reduce = function( key, values ) {    
    var count = 0;    
    values.forEach(function(v) {            
        count +=v;    
    });
    return count;
}
db.test.mapReduce(map, reduce, {out: "word_count"})
```
