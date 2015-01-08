Sprzêt, na którym pracujê: Pentium B970 @ 2.3 GhZ, 6 GB RAM, Linux Mint Xfce 17

#Zadanie 3

###ANAGRAMY

Plik [word_list.txt](http://wbzyl.inf.ug.edu.pl/nosql/doc/data/word_list.txt) wczyta³em za pomoc¹ nastêpuj¹cego polecenia:

``` 
mongoimport -d anagrams -c anagrams --type csv --file /media/pc/863E2B1D3E2B05B1/nosql/word_list.txt -f "words"```

Nastêpnie skorzysta³em z mapreduce:

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

Do wykonania zadania wykorzysta³em [plik z artyku³ami z wikipedii](http://dumps.wikimedia.org/plwiki/latest/plwiki-latest-pages-articles-multistream.xml.bz2). Plik jest w formacie xml, co oznacza, ¿e nie da siê go zaimportowaæ do mongo - trzeba go skonwertowaæ. Moje próby skonwertowania tak du¿ego pliku xml do jsona zakoñczy³y siê fiaskiem - wiêkszoœæ narzêdzi, które próbowa³em (np. [[1]](https://github.com/parmentf/xml2json), [[2]](https://github.com/hay/xml2json), [[3]](https://github.com/yihuang/xml2json) - wszystkie nazywaj¹ siê tak samo, bo xml2json) wywala³a b³êdy w stylu OutOfMemoryException i MemoryError (z powodu wielkoœci pliku), tak wiêc postanowi³em spróbowaæ z konwersj¹ do csv. W tym celu pos³u¿y³em siê narzêdziem [XML2CSVGenericConverter](http://sourceforge.net/projects/xml2csvgenericconverter/files/?source=navbar):

```
time java -jar XML2CSVGenericConverter_V1.0.0.jar -v -i /nosql/wiki.xml -o /nosql/
```

Jak na tak du¿y plik konwersja zakoñczy³a siê moim zdaniem stosunkowo szybko:


```
real	23m23.156s
user	13m18.764s
sys	1m28.312s
```

Nastêpnie zaimportowa³em bazê do mongo za pomoc¹ polecenia

```
time mongoimport -d wiki -c wiki --type csv --file wiki.csv --headerline
```

ale przy tym poleceniu wystêpowa³ b³¹d:

```
exception:CSV file ends while inside quoted field
```

Znalaz³em opis tego problemu i rozwi¹zanie wygl¹da nastêpuj¹co:

```
time mongoimport -d wiki -c wiki --type tsv --file wiki.csv --headerline --ignoreBlanks
```

Import potrwa³ trochê d³u¿ej:

```
real	51m50.747s
user	6m13.195s
sys	1m21.482s
```

Nastêpnie skorzysta³em z mapreduce:

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
