# Emlékeztetõ #

## Kérdések ##
1. Milyen elnevezési konveciót használunk csomagokra?
2. Mi a kapcsolat a csomag és a fájlrendszeren elhelyezkedõ mappaszerkezet között?
2. Milyen láthatósági módosítokkal találkoztál?
3. Mit nevezünk a függvény szignatúrájának?
4. Túlterhelhetõ-e a függvény a visszatérési értékére nézve?
5. Milyen paraméterátadási mód(ok) vannak?
6. Hogy használunk másik csomagban lévõ osztályokat?
7. Mit jelent az `import pkg.*;`?
8. Rekurzív-e az `import`?
9. Csak statikus metódusokat és mezõket szeretnénk importálni, milyen lehetõségünk van?
10. Mi történik, ha egy adott néven két osztály is szerepel különbözõ csomagokban, és mind a kettõt importáljuk? Hogy oldjuk meg a problémát?

## Környezet beállítása ##
Emlékeztetõ:

``` dos
	Microsoft Windows XP [verziószám: 5.1.2600]
	(C) Copyright 1985-2001 Microsoft Corp.
	
	c:\tmp>set PATH=%PATH%;c:\Program Files\Java\jdk1.6.0_12\bin\
	
	c:\tmp>javac -version
	javac 1.6.0_12
	
	c:\tmp>javac HelloWorldApp.java
	
	c:\tmp>java HelloWorldApp
	Hello World!
	
	c:\tmp>
```

# Csomagok #
Modularizáció, névütközések feloldása, hozzáférés szabályozás, etc. (mint a
C++ namespace). Osztályok, interfészek gyûjteménye. Használható a `*` wildcard.
Alapértelmezetten látszik a `java.lang.*` csomag minden eleme, minden mást
importálni kell (anélkül ún. _fully qualified classname_ segítségével
hivatkozhatunk, pl. `java.util.Vector`): 

``` java
import java.util.Vector;	// 1 tipushoz  
import java.math.*;			// Minden package-beli tipus lathatova valik  
	  
import java.awt.*;			// GUI  
import java.awt.event.*;	// GUI - esemenykezeles  
import javax.swing.*;		// Advancedebb GUI  
import java.util.*;			// Adatstrukturak  
import java.io.*;			// IO  
import java.util.regex.*;	// Regexp  
	  
// static import: minden static konstans lathato az adott osztalybol  
// fenntartasokkal hasznalni  
import static java.lang.Math.*;  
```

A fordítás nehézkes, nincs rekurzív `javac -R *.java`. Leképezés a
fájlrendszerre: minden `.` karakterrel szeparált rész egy könyvtárat jelent,
fordítás a gyökérkönyvtárból történik. Static importot csak offtosan
(strukturáltság, enkapszuláció, egységbezárás ellen hat - használjatok helyette
dedikált osztályt vagy interfészt). Csomag definíciója a Java fájl legelején:

``` java
package pkg;
	
// Import utasitasok
	
public class HelloWorldApp {
    public static void main(String args[]) {
        System.out.println("Hello World!");
    }
};
```

> **Fontos** Ha csomagokat használunk, akkor mindig a *package root* alól adjuk ki a fordításhoz/futtatáshoz szükséges parancsokat (azaz abból a könyvtárból, ami a legfelsõ szintû csomagokat tartalmazza)! Erre azért van szükség, mert ha nem állítod be kézzel a `CLASSPATH` változó értékét (ez azon útvonal, ahol a Java alapértelmezés szerint keresi a szükséges osztályokat), akkor az a `.` (azaz az aktuális) könyvtár. Így ha pl. a `foo.A` osztály hivatkozik egy `foo.bar.B` osztályra, és a `foo` könyvtárból fordítod az `A` osztályt, akkor a fordító/futtatókörnyezet a `foo` konyvtárban keres egy másik `foo`, majd abban egy `bar` könyvtárat!
> 
> Tehát a lényeg: **mindig a package root könyvtárból fordítsunk, futtassunk!**

**Fordítás** a Java fájl **teljes útvonalának** megadásával:

	C:\tmp>javac pkg/HelloWorldApp.java

**Futtatáshoz** azonban a **teljes hivatkozási név** szükséges (*fully qualified classname*):
	
	C:\tmp>java pkg.HelloWorldApp
	Hello World!

Ha esetleg névütközés van (2 azonos nevû osztály), akkor minõsített névvel
érhetjük el az egyiket (pl. `java.util.List`, `java.awt.List`). Importokat
használjatok nyugodtan, nem gáz, nem emészt erõforrást (nem C++, dinamikus
osztálybetöltés van).


## Rekurzív fordítás ##
Alapból nincs rekurzív fordítás, viszont megadható a fordítónak egy fájl,
ami a fordítani kívánt fájlok listáját tartalmazza - ezt a `@` karakterrel
jelezheted.

	# Linux
	$ find -name "*.java" > sources.txt
	$ javac @sources.txt

	:: Windows
	> dir /s /B *.java > sources.txt
	> javac @sources.txt

> **Részletesen** [Itt](http://stackoverflow.com/questions/6623161/javac-option-to-compile-recursively/8769536#8769536)

# Függvények #
Általános prototípus:

	<módosítószavak> <visszatérési érték> <név>( <paraméterek listája> )
	        [ throws <kivétel lista> ] {
	    <utasítás1>;
	    <utasítás2>;
	    ...
	}

Paraméter átadás érték szerint történik (még a referenciák is!).

* Módosítószavak:
	* Láthatóság: `public`, `protected`, `private`. Ha nem definiált, akkor ún.
	  *default* / _package-private_ / *package-level* láthatóság.
	* Lehet `abstract`: ekkor nincs implementáció (mint a C++ _pure virtual_
	  függvényei) leszármazottban kötelezõen felüldefiniálandó
	* Lehet `final`: felüldefiniálhatóság letiltására
	* Lehet `static`: osztály szintû függvény (**Fontos:** static kontextusból
	  csak static módosítóval ellátott hivatkozás szerepelhet)
	* Egyéb, pl. `strictfp`, `native`, `synchronized`, `transient`, `volatile`
	  (utóbbi kettõ **csak** fieldekre). Ezekrõl késõbb.
* Visszatérési érték szerinti csoportosítás:
	* `void`: eljárás
	* Minden egyéb: függvény
* Metódusnév: _lowerCamelCase_ formátumban
* Paraméter átadás: minden paraméter érték szerint adódik át
_még a referenciák is_.

**Szignatúra** a függvény neve és paramétereinek típusa -- más **nem**. Például:

``` java
eredmenyMeghatarozasa( double, int, int )
```

Overloading, overriding.

# +/- Feladatok #

A feladatokat a `gyak2.f1` ill. `gyak2.f2`, ... csomagba rakjátok!

### Faktoriális ###
Írjatok programot, mely kiszámítja a paraméterül kapott szám faktoriálisát.

``` java
    class F1 {
        public static void main(String[] args) {
            System.out.println(fakt(5));
        }
        
        public static int fakt(int n) {
            int res = 1;
            for (int i = 1; i < n; ++i) {
                res *= i;
            }
            return res;
        }
    }
```

### Rekfakt ###
Írj faktoriálist megoldó programot, melyben a függvény rekurzív.

``` java
    class F2 {
        public static void main(String[] args) {
            System.out.println(fakt(5));
        }
        
        public static int fakt(int n) {
            if( n == 0 ) {
                return 1;
            } else {
                return n * fakt(n-1);
            }
        }
    }
```

### Fib ###
Add össze az összes páratlan rész-fibonacci számot egészen addig, amíg az adott rész-fibonacci szám értéke nem éri el 400000.

### Euclid ###
Készítsetek egy függvényt, amely az Euklideszi-algoritmus alapján meghatározza
két szám legnagyobb közös osztóját! Az algoritmus pszeudokódja:

	function gcd(a, b)  
	    if a = 0  
	       return b  
	    while b != 0  
	        if a > b  
	           a := a - b  
	        else  
	           b := b - a  
	    return a  

Készítsd el a függvény rekurzív változatát is!

### Quadratic ###
Készítsetek egy függvényt, amely megadja egy másodfokú egyenlet gyökeit! A
függvény definíciója legyen a következõ:

``` java
private static double[] sqroots(final double a, final double b, final double c) {
	// ...
}
```

A függvény visszatérési értéke legyen:

* üres tömb (nem `null` érték!), ha `a == 0` vagy a diszkrimináns negatív,
* egyelemû tömb, ha egyetlen megoldás van (`D == 0`), illetve
* kételemû tömb, amennyiben két különbözõ megoldás létezik!

A paramétereket a parancssori argumentumok határozzák meg, amiket a
`Double.parseDouble()` segítségével tudsz értelmezni.
