# ZH gyakorlás #
Az alábbi feladat a ZH-ra való felkészülést segíti.

## Feladatleírás ##
Készíts egy számítógépes játék értékelő alkalmazást! A rendszer az alábbi
konzolos felülettel rendelkezzen:

	[1] Új értékelés felvétele
	[2] Értékelések rendezett megjelenítése
	[3] Értékelések mentése
	[4] Értékelések betöltése
	---------------------------------------
	Választás: 

### Funkciók ###
Az egyes funkciók részletesebb leírása.

#### Új értékelés felvétele ####
Az értékelésekről alapvetően az alábbi adatokat kell nyilvántartani, ezt
tetszőleges adatszerkezetben megoldhatod:

* Cikkszám, amely egy egyedi azonosító
* A játék neve
* A játékhoz eddig leadott értékelések átlaga százalékban

Példa:

	CIKK-001	Baldur's Gate I		95%
	CIKK-002	Dragon Age			93%
	CIKK-003	Mass Effect 2		91%
	CIKK-004	Trine				84%

Ha a felhasználó egy új értékelést szeretne regisztrálni a rendszerben, akkor a
következőt kell tennünk:

* ha a megadott cikkszám már létezik, akkor csak egy újabb százalékos értékelést
  kell regisztrálni (amely aztán módosítja az eddigi szavazatok átlagát)
* ha a megadott cikkszám még nem létezik, akkor be kell kérni az értékelni
  kívánt játék nevét, majd az első meghatározó értékelést

##### Inputkezelés #####
* Az azonosító formátumát tetszőlegesen választhatod, de erősen ajánlott, hogy
  ne használd benne a `:` karaktert.
* Játék neve nem tartalmazhat `:` karaktert.
* Ha a felhasználó nem megfelelő százalékos értékelést ad (megfelelő input az
  `[1..100]` intervallum bármely eleme), azt egy *saját kivétel* bevezetésével
  kezeld le!

##### Átlagszámítás #####
Erre több megoldás van, legegyszerűbb az eddigi összes értékelést tárolni,
viszont az eddigi értékelések száma, átlaga, valamint az új értékelés alapján
konstans időben meghatározható.

#### Értékelések rendezett megjelenítése ####
A program listázza ki az általa aktuálisan felügyelt értékeléseket, a százalékos
értékelések szerinti csökkenő sorrendben.

Ehhez használd a *Java Collections Framework* által kínált szolgáltatásokat
(`java.util.Collections`)!

#### Perzisztencia ####
Mentsük el a felhasználó által megadott aktuális értékelések listáját az általa
megadott fájlba (az egyes adattagokat `:` karakterrel válasszuk el egymástól).

A program legyen képes ezeket az adatokat visszatölteni, azonban ha hibás az
input, keletkezzen egy saját `Error` (lekezelni nem muszáj)!
