# Generic #

Egyszerűbb példák (`java.util `csomagból):

``` java
public interface List<E> {
	void add(E x);
	Iterator<E> iterator();
}
	
public interface Iterator<E> {
	E next();
	boolean hasNext();
}
```

`E` - formális típusparaméter, amely aktuális értéket a kiértékelésnél vesz fel
(pl. `Integer`, etc.).

## Altípusosság ##
Nem konvertálhatók, ennek oka:

``` java
List<String> l1 = new ArrayList<String>();
List<Object> l2 = l1; // error
	
// Mert akkor lehetne ilyet csinalni:
l2.add(new Object());
l1.get(0); // reccs, Object -> String castolas
```

Magyarul ha `S <= T =x=> G<S> <= G<T>` - ez pedig ellent mond az ember
megérzésének. Castolni lehet (warning), `instanceof` tilos (fordítási hiba)!

## Wildcardok ##
Probléma: általános megoldást szeretnénk, amely minden collectiont elfogad,
függetlenül az azokban tárolt elemektől (pl. ki szeretnénk őket írni), vagy nem
tudjuk azok konkrét típusát (pl. legacy code). `Collection<Object>` *nem* őse
(ld. előző bekezdés). Ha nem használunk genericeket, megoldható, viszont
warningot generál:

``` java
void print(Collection c) {
	for (Object o : c) System.out.println(o);
}
```

Első próba: használjunk Objectet:

``` java
void print(Collection<Object> c) {
	for (Object o : c) System.out.println(o);
}
```

Fordul? Fordul! Működik? Néha... **Fail!** A gond az, hogy mivel `Collection<X>`
nem leszármazottja a `Collection<Object>` típusnak, kizárólag `Collection<Object>`
paraméterrel hívható meg. Nem túl hasznos...

A megoldás a wildcard használata: `Collection<?>` minden kollekcióra ráillik.
Ilyenkor `Objectként` hivatkozhatunk az elemekre:

``` java
void print(Collection< ? > c) {
	for (Object o : c) System.out.println(o);
}
```

Vigyázat! A `? != Object`! Csak egy ismeretlen típust jelent. Így a következő
kódrészlet is fordítási hibához vezet:

``` java
List< ? > c = ...;
l.add(new Object()); // forditasi hiba
```

Nem tudjuk, hogy mi van benne, lekérdezni viszont lehet (mert tudjuk, hogy
minden objektum az `Object` leszármazottja). Ha lehetne belepakolni, az 
veszélyeztetné a program típusbiztonságát. Az egyetlen kivétel ezalól a
`null` érték (ez mindennek értékül adható).

## Bounded wildcard ##
Amikor tudjuk, hogy adott helyen csak adott osztály leszármazottai
szerepelhetnek, első (rossz) megközelítés:

``` java
abstract class Super {}
class Sub1 extends Super {}
class Sub2 extends Super {}
...
void func(List<Super> l) {...} // Rossz!
```

Probléma: `func()` csak `List<Super>` paraméterrel hívható meg, `List<Sub1>`,
`List<Sub2>` nem lehet paramétere (nem altípus). Megoldás: *bounded wildcard*:

``` java
void func(List< ? extends Super> l) {...}
```

Belepakolni ugyanúgy nem tudunk, mint a `?` esetén, azaz erre fordítási hibát
kapunk:

``` java
void func(List< ? extends Super> l) {
	l.add(new Sub1()); // reccs
}
```

Felfelé is megköthető a wildcard a `<? super T>` jelöléssel.

## Generikus osztályok, függvények ##
Osztálydefinícióban bevezethető típusparaméter az osztályhoz, ez minden
membernél használható. Példa:

``` java
package generics;
	
public class Pair<T, S> {
	private final T first;
	private final S second;
	    
	public Pair(final T first, final S second) {
		super();
		this.first = first;
		this.second = second;
	}
	    
	public T getFirst() {
		return first;
	}
	public S getSecond() {
		return second;
	}
	    
	// Esetleges null ellenorzeseket tessek elvegezni!
	// Itt az attekinthetoseg kedveert ettol eltekintettem.
	@Override
	public boolean equals(final Object obj) {
		if (obj instanceof Pair) {
			Pair<?, ?> pair = (Pair<?, ?>) obj;
			return first.equals( pair.first ) && second.equals( pair.second );
		}
	        
		return false;
	}

	// Esetleges null ellenorzeseket tessek elvegezni!
	// Itt az attekinthetoseg kedveert ettol eltekintettem.
	@Override
	public int hashCode() {
		return first.hashCode() + second.hashCode();
	}

	@Override
	public String toString() {
		return "(" + first + ", " + second + ")";
	}

}
```

Generikus függvények esetén szintén a definícióban használható. Példa:

``` java
package generics;
	
public class ArrayUtils {
	public static final <T, S extends T> boolean isIn(final T[] arr, final S element) {
		for (final T t : arr) {
			if (t.equals(element)) return true;
		}
	        
		return false;
	}
	    
	public static void main(final String[] args) {
		final String[] sarr = {"a", "b", "c"};
		System.out.println( isIn(sarr, "c") );
	}
}
```

> **Részletesen:** <http://java.sun.com/j2se/1.5/pdf/generics-tutorial.pdf>

## Bemelegítő feladat ##

Valósítsuk meg a generikus Stack adatszerkezetet.
Rendelkezzen két konstruktorral. Egy paraméterek nélküli és egy `int` típusú paramétert váróval.
Abban az esetben, ha a paraméteres konstruktort hívjuk meg, figyeljünk arra, hogy a paraméterben megadott értéknél több elem nem kerülhet a verembe.
Üres konstruktorhívás esetén végtelen (memória nagyságtól függően) mennyiségű adatot tudunk eltárolni.
A stack adatszerkezet az alábbi öt művelettel rendelkezik.
 
 * `push(T)`: `void`  :: berak egy elemet a verembe
 * `pop()`: `T`  :: kivesz egy elemet a veremből
 * `top()`: `T`  :: visszatér a verem tetején lévő elemmel.
 * `size()`: `int`  :: visszatér a veremben lévő elemek számával
 * `isEmpty()`: `boolean`  :: üres verem esetén igaz (true), más esetben hamis (false) a visszatérési értéke

Dobjunk `FullStackException` kivételt, ha tele verem esetén meghívásra kerül a push művelet.
Dobjunk `EmptyStackException` kivételt, ha üres veremre hívjuk meg a top és pop műveleteket.

*Egy megoldás:*

``` java
package stackdemo;

import stackdemo.util.Stack;
import stackdemo.util.FullStackException;
import stackdemo.util.EmptyStackException;

public class StackDemo {

    public static void main(String[] args) throws EmptyStackException {
        Stack<Integer> sInt = new Stack<Integer>();
        Stack<Character> sChar = new Stack<Character>(5);

        try {
            for (int i = 0; i < 10; ++i) {
                // Berakjuk az elemekt: 0..9
                sInt.push(i);
            }
        } catch (FullStackException ex) {
            ex.printStackTrace();
        }
        System.out.println(sInt.top());  // 9
        for (int i = 0; i < 5; ++i) {
            System.out.println(sInt.pop()); //9..5
        }
        System.out.println(sInt.top());  // 4
        System.out.println(sInt.size());  // 5
        System.out.println(sInt.isEmpty());  // false
        for (int i = 0; i < 5; ++i) {
            System.out.println(sInt.pop()); //4..0
        }
        System.out.println(sInt.isEmpty());  // true

        try {
            for (int i = 0; i < 6; ++i) {
                sChar.push(Character.forDigit(i, 10));
            }
        } catch (FullStackException ex) {
            ex.printStackTrace();
        }
    }
}
```

``` java
package stackdemo.util;

/*
 * Valósítsuk meg a generikus Stack adatszerkezetet.
 * Rendelkezzen két konstruktorral. Egy paraméterek nélküli és egy int paramétert váróval.
 * Abban az esetben, ha a paraméteres konstruktort hívjuk meg, figyeljünk arra, hogy a paraméterben megadott értéknél több elem nem kerülhet a verembe.
 * Üres konstruktorhívás esetén végtelen (memória nagyságtól függően) mennyiségű adatot tudunk eltárolni.
 * A stack adatszerkezet az alábbi öt művelettel rendelkezik.
 *  * push(T): void  :: berak egy elemet a verembe
 *  * pop(): T  :: kivesz egy elemet a veremből
 *  * top(): T  :: visszatér a verem tetején lévő elemmel.
 *  * size(): int  :: visszatér a veremben lévő elemek számával
 *  * isEmpty(): boolean  :: üres verem esetén igaz (true), más esetben hamis (false) a visszatérési értéke
 * Dobjunk FullStackException kivételt, ha tele verem esetén meghívásra kerül a push művelet.
 * Dobjunk EmptyStackException kivételt, ha üres veremre hívjuk meg a top és pop műveleteket.
 * */

import java.util.List;
import java.util.LinkedList;

public class Stack<T> {
    private List<T> stack = new LinkedList<T>();
    private int maxSize = -1;   // -1 a kezdeti érték. Ha nem kerül átállításra a konstruktorban, akkor sem lesz gond maxSize == size() esetén.

    public Stack() {
    }

    public Stack(int size) {
        this.maxSize = size;
    }

    public void push(T e) throws FullStackException {
        // Azért állítottuk a maxSize kezdeti értékét -1-re, hogy a push akkor is működjön, ha paraméter nélküli konstruktort hívtunk példányosításnál.
        if(this.maxSize == this.size()) {
            throw new FullStackException();
        }
        this.stack.add(e);
    }

    public T pop() throws EmptyStackException {
        if(this.stack.size() == 0) {
            throw new EmptyStackException();
        }
        T e = this.stack.get(this.stack.size() - 1);
        //this.stack.remove(this.stack.size() - 1);
        this.stack.remove(e);
        return e;
    }

    public T top() {
        return this.stack.get(this.stack.size() - 1);
    }

    public int size() {
        return this.stack.size();
    }

    public boolean isEmpty() {
        return this.stack.isEmpty();
    }
}
```

``` java
package stackdemo.util;

public class EmptyStackException extends Exception{
    public EmptyStackException() {
    }
    
    public EmptyStackException(String msg) {
        super(msg);
    }
   
}
```

``` java
package stackdemo.util;

public class FullStackException extends Exception {
    public FullStackException() {
    }
    
    public FullStackException(String msg) {
        super(msg);
    }
}
```

Írjunk raktárnyilvántartó programot.
Olvassuk be az adatokat egy fájlból, melyenk felépítése a következő:
típus : id : nev : ... egyéb adatok
ruha:id:nev:meret:ár
etel:id:nev:lejarat:ár
butor:id:nev:fő részére:ár

Írjunk egy raktár (Raktar) osztályt, mely tartalmazza az összes fáljban lévő terméket.
Gyűjtsd külön a különböző típusú termékeket, ehhez használj Map<String, List<T>> adatszerkezetet.

A Raktár osztály a következő függvényeket tartalmazza: 

    * add(T):void
    * remove(T):void Kitörli az első objectet, ami megegyezik T-vel
    * removeAll(T): void Kitörli az összes elemet.
    * removeAll(String key):void
    * size(): int
    * size(String key):int
    * get(int id): T
    * getAll(String key): List<T>

Inputtext:
ruha:111:pulover:S:12
etel:112:konzerv:2025:25
butor:113:kerti pad:6:250
etel:114:szilva:2012:25
etel:115:szilva:2012:25
butor:116:sofa:2:25
ruha:117:pulover:M:25
etel:118:szilva:2012:25
etel:119:szilva:2012:25
etel:120:szilva:2012:25
etel:121:szilva:2012:25
etel:122:szilva:2012:25


``` java
package szallitmanydemo;

import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;
import szallitmanydemo.entities.Etel;
import szallitmanydemo.entities.Ruha;
import szallitmanydemo.entities.Termek;
import szallitmanydemo.util.Raktar;

public class SzallitmanyDemo {

    private final static String RUHA = "ruha";
    private final static String ETEL = "etel";
    private final static String BUTOR = "butor";
    
    public static void main(String[] args) {
        Raktar<Termek> raktar = readFromFile(args[0]);   
        System.out.println(raktar.get(115));
        System.out.println(raktar.size());
        System.out.println(raktar.size(ETEL));
        System.out.println(raktar.remove(raktar.get(115)));
        raktar.removeAll(raktar.get(114));
        for(Termek t : raktar.getAll(ETEL)) {
            System.out.println(t);
        }
    }
    
    public static Raktar<Termek> readFromFile(String fileName) {
        Raktar<Termek> retRaktar = new Raktar<Termek>();
        Scanner sc = null;
        try {
            sc = new Scanner(new File(fileName));
            while(sc.hasNextLine()) {
                String[] datas = sc.nextLine().split(":");
                if(RUHA.equals(datas[0].trim())) {
                    retRaktar.add(new Ruha(
                                    Integer.parseInt(datas[1].trim()),
                                    datas[0].trim(),
                                    datas[2].trim(),
                                    Integer.parseInt(datas[4].trim()),
                                    datas[3].trim()));
                } else if(ETEL.equals(datas[0].trim())) {
                    retRaktar.add(new Etel(
                                    Integer.parseInt(datas[1].trim()),
                                    datas[0].trim(),
                                    datas[2].trim(),
                                    Integer.parseInt(datas[3].trim()),
                                    Integer.parseInt(datas[4].trim())));
                } else if(BUTOR.equals(datas[0].trim())) {
                    retRaktar.add(new Etel(
                                    Integer.parseInt(datas[1].trim()),
                                    datas[0].trim(),
                                    datas[2].trim(),
                                    Integer.parseInt(datas[3].trim()),
                                    Integer.parseInt(datas[4].trim())));
                }
            }
        } catch(FileNotFoundException ex) {
            ex.printStackTrace();
        } finally {
            if( sc != null) {
                sc.close();
            }
        }        
        return retRaktar;
    }    
}

```

``` java
package szallitmanydemo.util;

import java.util.LinkedHashMap;
import java.util.LinkedList;
import java.util.List;
import java.util.Map;

import szallitmanydemo.entities.Termek;

public class Raktar<T extends Termek> {
    private Map<String, List<T>> raktar = new LinkedHashMap<String, List<T>>();
    
    public void add(T e) {   
        if(this.raktar.containsKey(e.getTipus())) {
            List<T> l = this.raktar.get(e.getTipus());
            l.add(e);
        } else {
            List<T> l = new LinkedList<T>();
            l.add(e);
            this.raktar.put(e.getTipus(), l);
        }
    }
    
    public boolean remove(T e) {
        List<T> list = this.raktar.get(e.getTipus());
        return list.remove(e);
    }
    
    public void removeAll(T e) {
        List<T> list = this.raktar.get(e.getTipus());
        for(int i=list.size()-1; i>=0; --i) {
            T t = list.get(i);
            if(e.getNev().equals(t.getNev())) {
                list.remove(t);
            }
        }
    }
    
    public void removeAll(String key) {
        this.raktar.remove(key);
    }
    
    public int size() {
        int count = 0;
        for(String s : this.raktar.keySet()) {
            count += this.raktar.get(s).size();
        }
        return count;
    }
    public int size(String key) {
        return this.raktar.get(key).size();
    }

    public T get(int id) {
        for(String key : this.raktar.keySet()) {
            for(T t : this.raktar.get(key)) {
                if(t.getId() == id) {
                    return t;
                }
            }
        }
        return null;
    }
    
    public List<T> getAll(String key) {
        List<T> retList = this.raktar.get(key);
        return retList;
    }
    
}

```

``` java
package szallitmanydemo.entities;

public class Termek {
    private final int id;
    private final String tipus;
    private final String nev;
    private final int ar;
    public Termek(final int id, final String tipus, final String nev, final int ar) {
        this.id = id;
        this.tipus = tipus;
        this.nev = nev;
        this.ar = ar;
    }
    
    public int getId() {
        return this.id;
    }
    public String getTipus() {
        return this.tipus;
    }
    public String getNev() {
        return this.nev;
    }
    public int getAr() {
        return this.ar;
    }
    
    public String toString() {
        return this.id + ":" + this.nev;
    }
    
}
```

``` java
package szallitmanydemo.entities;

public class Ruha extends Termek {
    private final String meret;
    public Ruha(final int id, final String tipus, final String nev, final int ar, final String meret) {
        super(id, tipus, nev, ar);
        this.meret = meret;
    }
    
    public String getMeret() {
        return this.meret;
    }
}
```

``` java
package szallitmanydemo.entities;

public class Etel extends Termek {
    private final int lejarat;
    public Etel(final int id, final String tipus, final String nev, final int ar, final int lejarat) {
        super(id, tipus, nev, ar);
        this.lejarat = lejarat;
    }
    
    public int getLejarat() {
        return this.lejarat;
    }
}
```

``` java
package szallitmanydemo.entities;

public class Butor extends Termek {
    private final int foReszere;
    public Butor(final int id, final String tipus, final String nev, final int ar, final int foReszere) {
        super(id, tipus, nev, ar);
        this.foReszere = foReszere;
    }
    
    public int getFoReszere() {
        return this.foReszere;
    }
}
```


## További feladatok ##

A megoldáshoz készített osztályokat tegyétek a `javagyak.generics` csomagba,
valamint a teszteléshez használt osztályokat (amik a `main()` definícióit is
tartalmazzák) a `javagyak.test` csomagba!

### Triple ###
Készíts el egy generikus `Triple` osztályt, amely 3 (nem feltétlen) különböző
típuból alkotott rendezett hármas! Valósítsd meg vele a szokásos műveleteket
(`equals()`, `toString()`, `hashCode()`)!

### Reverse ###
Készíts egy generikus `reverse()` függvényt, amely egy tetszőleges lista
adatszerkezetet képes visszafelé kiírni a képernyőre!

### Fill ###
Készíts egy generikus `fill()` függvényt, amely a paraméterként kapott,
tetszőleges lista minden elemét lecseréli a megadott elemre.

### addAll() ###
Készíts egy generikus `addAll()` függvényt, amely egy tetszőleges lista
adatszerkezetbe beleteszi a második paraméterként megadott lista összes elemét!

### MiniMax ###
Készíts egy olyan generikus `min()` és `max()` függvényt, amely egy tetszőleges
kollekcióban megkeresi és visszaadja a minimális és maximális elemet! (Cseles!
Kell a `Comparable` interfész is ugye...).

### Tömbös (*) ###
Készíts két generikus függvényt: az egyik segítsen tömböt kollekcióvá
konvertálni, a másik pedig kollekcióból tömböt.

### Stack (*) ###
Készíts egy saját, generikus `Stack` implementációt! Az implementáció
illeszkedjen a Java gyűjtemény keretrendszerébe! Ennek módját tetszőlegesen
megválaszthatod, a reprezentációt is, de a `Collection<E>` mindenképp legyen őse!

### Generikus bináris keresőfa (*) ###
Készítsünk egy egyszerű, általános bináris keresőfa implementációt! A fához
lehessen elemet hozzáadni (`add()`), kiírni, valamint a minimum, maximum elemet
megkeresni (`min()`, `max()`). A típusparaméterének összehasonlíthatónak kell
lennie (`<T extends Comparable<T>>`).

