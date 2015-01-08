Sprz�t, na kt�rym pracuj�: Pentium B970 @ 2.3 GhZ, 6 GB RAM, Linux Mint Xfce 17

#Zadanie 3

###ANAGRAMY

Plik [word_list.txt](http://wbzyl.inf.ug.edu.pl/nosql/doc/data/word_list.txt) wczyta�em za pomoc� nast�puj�cego polecenia:

``` 
mongoimport -d anagrams -c anagrams --type csv --file /media/pc/863E2B1D3E2B05B1/nosql/word_list.txt -f "words"```

Nast�pnie skorzysta�em z mapreduce:

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

Do wykonania zadania wykorzysta�em [plik z artyku�ami z wikipedii](http://dumps.wikimedia.org/plwiki/latest/plwiki-latest-pages-articles-multistream.xml.bz2). Plik jest w formacie xml, co oznacza, �e nie da si� go zaimportowa� do mongo - trzeba go skonwertowa�. Moje pr�by skonwertowania tak du�ego pliku xml do jsona zako�czy�y si� fiaskiem - wi�kszo�� narz�dzi, kt�re pr�bowa�em (np. [[1]](https://github.com/parmentf/xml2json), [[2]](https://github.com/hay/xml2json), [[3]](https://github.com/yihuang/xml2json) - wszystkie nazywaj� si� tak samo, bo xml2json) wywala�a b��dy w stylu OutOfMemoryException i MemoryError (z powodu wielko�ci pliku), tak wi�c postanowi�em spr�bowa� z konwersj� do csv. W tym celu pos�u�y�em si� narz�dziem [XML2CSVGenericConverter](http://sourceforge.net/projects/xml2csvgenericconverter/files/?source=navbar):

```
time java -jar XML2CSVGenericConverter_V1.0.0.jar -v -i /nosql/wiki.xml -o /nosql/
```

Jak na tak du�y plik konwersja zako�czy�a si� moim zdaniem stosunkowo szybko:


```
real	23m23.156s
user	13m18.764s
sys	1m28.312s
```

Nast�pnie zaimportowa�em baz� do mongo za pomoc� polecenia

```
time mongoimport -d wiki -c wiki --type csv --file wiki.csv --headerline
```

ale przy tym poleceniu wyst�powa� b��d:

```
exception:CSV file ends while inside quoted field
```

Znalaz�em opis tego problemu i rozwi�zanie wygl�da nast�puj�co:

```
time mongoimport -d wiki -c wiki --type tsv --file wiki.csv --headerline --ignoreBlanks
```

Import potrwa� troch� d�u�ej:

```
real	51m50.747s
user	6m13.195s
sys	1m21.482s
```

Nast�pnie skorzysta�em z mapreduce:

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
