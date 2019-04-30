# Hozzájárulás a projekthez

`scroll down for English version`

A projekthez bárki szabadon hozzájárulhat tudástár (wiki) vagy fejlesztési ötlet, javaslat hozzáadásával akár szöveges megjegyzés (issue) akár kód (pull request) formájában. A hozzájáruláskor a hozzájáruló automatikusan elfogadja a projekt licensz (MIT) által meghatározott feltételeket.

## 1) Hogyan tudom 2 eltérő API verzió változásait tételesen megnézni? (diff)

A fájlok közötti különbség megtekintésére a GitHub több nézetet is felkínál. A legegyszerűbb 2 commit összehasonlítása, de lehetőség van URL param alapján is különbségeket vizsgálni. 

Például, az 1.1 és a 2.0 közötti különbséget az alábbi URL-en lehet megnézni: https://github.com/nav-gov-hu/Online-Invoice/commit/0da32edd71c68da2e529d704f06825317767245e?diff=split

A változásokat javasoljuk összeolvasni minden esetben az aktuális [CHANGELOG.md](https://github.com/nav-gov-hu/Online-Invoice/blob/master/src/schemas/nav/gov/hu/OSA/CHANGELOG_2.0.md) fájllal, ami segít az áttekintésben.

## 2) Fejlesztési ötletem vagy kérdésem van (issue)

Az API **koncepcionális** működésére vonatozó kérdés vagy ötlet esetén issue-t lehet feladni a fejlesztőknek: https://github.com/nav-gov-hu/Online-Invoice/issues

Minden XSD módosításhoz készül külön CHANGELOG az adott branchen, ezt minden esetben kérjük az issue feladása előtt előzetesen tanulmányozni! Minden feladott issue-t megválaszolunk, de kérjük, hogy lehetőség szerint kerüljük a duplikációkat. Minden issue nyilvános, a kérdést és a választ is mindenki láthatja. Amennyiben az issue javaslat és annak tartalmával egyetértünk, úgy a javaslatot befogadjuk és a projekt kanban táblájára is felkerül mint feladat. A módosítást a későbbiekben új commit alatt hozzá fogjuk adni a projekthez.

Kérjük a tárgynak megfelelő sablon használatát az issue-k alatt kiválasztani, a következők alapján:

- kérdés esetén: Kérdés-válasz / Q&A issue
- javaslat esetén: Fejlesztési kérés / Feature request

Kérjük a sablon tárgy mezőjében a [] címke utáni részt annak megfelelően kitölteni, amire az issue vonatkozik!

Felhívjuk a figyelmet, hogy a GitHub nem helpdesk, tehát konkrét éles működést vagy problémát érintő kérdésre jelen platformon nincs lehetőségünk válaszolni. Ezek megoldására az interfészdokumentáció 7.2 pontjában írt lehetőségek állnak nyitva továbbra is. Ezen túlmenően kérjük figyelembe venni, hogy tisztán üzleti, számlázási kérdésre választ a GitHub keretei között csak legfeljebb a sikeres és helyes adatszolgáltatáshoz szükséges mélységig áll lehetőségünkben adni. A fenti feltételeknek nem megfelelő issue-kat törölni fogjuk.

## 3) Tudástárat szeretnék írni (wiki)

A NAV támogatja a közösségi tudásmegosztást, hiszen a számlázóprogramok fejlesztőinek elkötelezett és magas színvolalú munkája elengedhetetlenül szükséges eleme az Online Számla Rendszer sikerességének. Ennek fényében a GitHubon keretet és lehetőséget kínálunk egy olyan tudásbázis kialakítására, amely nyitva áll minden hozzájárulni kívánó fejlesztő előtt. A hozzájárulás vonatkozhat (a teljesség igénye nélkül) üzleti logikára, design patternre, vagy adott programnyelv konkrét rész vagy funkció implementációjára is, lényegében bármire, ami a projekthez csatlakozóknak hasznos lehet.

A wiki csak GitHub userrel, bejelentkezést követően érhető el: https://github.com/nav-gov-hu/Online-Invoice/wiki

A wiki tartalmát és struktúráját a közösség önszerveződő módon, szabadon meghatározhatja. A közösség ennek során felelősséggel tartozik a README.md-ben megfogalmazott a moderációs elvek betartásáért és betartatásáért. A wiki tartalmának szakmai helyességéért a NAV nem vállal felelősséget, annak biztosítása a hozzájáruló és a közösség feladata.

A projekt alatt található wiki szerkesztéséhez jelenleg nincs szükség csoport tagságra, ez azonban az idő előre haladásával még változhat.

## 4) Saját kódot szeretnék adni (pull request)

Figyelemmel arra, hogy a tárhely csak az XSD-t és annak tervezeteit tartalmazza, pull requestet jelenleg csak a sémaleíróra, valamint a hozzá tartozó sample XML fájlokra lehet feladni. Az ettől eltérő fájlokat (is) tartalmazó PR-eket el fogjuk utasítani. Ahogy a tárhely bővül, úgy lesz egyre több lehetőség más jellegű PR-ek feladására is.

Ha a pull request tartalmával egyetértünk, úgy a változtatást mergeljük és ha a váloztatáshoz szerver oldali módosításra is szükség van, úgy a feladat a projekt kanban táblájára is felkerül. Ha a pull request hibás vagy hiányos, úgy kérni fogjuk annak javítását vagy kiegészítését.

Pull request csak GitHub userrel, bejelentkezést követően érhető el: https://github.com/nav-gov-hu/Online-Invoice/pulls

### 4.1) Pull request feladás folyamat
1. Saját fejlesztő környezet felállítása (IDE, GIT, stb) kliens oldalon.
2. Online-Invoice repository forkolása.
3. A forkolt repository klózonása saját gépre.
4. Új branch létrehozása a névkonvenciónak megfelelően.
5. A kódok módosítása, ellenőrzés, tesztelés. Figyelemmel arra, hogy pull request még csak nem funkcionális kódra adható fel, csak statikus kódelemzést kérünk (tehát a séma legyen jól formázott és valid), unit és fejlesztői tesztek írása még nem elfogadási követelmény.
6. Commit(ok) a saját lokális branchre. 
7. A commitált változtatások pusholása a saját repository fork alá.
8. Pull request feladása a GitHub felületén. A pull requestet az XSD azon főverziója szerinti branchre kell feladni, amelyre a módosítási igény vonatkozik. (véleményezés időszak alatt ez mindig a 'master' branch) A projektben támogatott a merge commit és a squash is, de több commit esetén preferáljuk a squasholást. (https://github.blog/2016-04-01-squash-your-commits/)

### 4.2) Pull requestek kezelése
- Minden pull request kötelező eleme a leírás, a leírás nélküli PR-eket elutasítjuk.
- A pull request leírása legyen annyira tömör és egyértelmű amennyire lehetséges, és derüljön ki belőle, hogy mi volt az igény, amit az adott PR tartalmaz.
- Pull request csak akkor mergelhető, ha a reviewerek megfelelőnek találták.
- Ha a pull request hiányos vagy hibás, akkor a review során kérni fogjuk a javítását vagy a kiegészítését.

### 4.3) Pull request névkonvenciók
A pull requesteket kérjük minden esetben az alábbi névkonvenció szerint elnevezni: `[típus]/[változtatások tömören]`. 

A `[típus]/` prefix az alábbi értékeket veheti fel:

- `feature/` = új funkcionalitás hozzáadása
- `try/` = javaslat kísérleti jelleggel
- `fix/` = javítás, pontosítás

A `[változtatások tömören]` postfix tartalmazza azon üzleti igényt, amire a módosítás irányul. Például:

- `[feature]/[modifyWithoutMaster keresőparaméter hozzáadása a /queryInvoiceDigest operációhoz]`
- `[fix]/[annotácíó elírás javítása a ProductCodeCategoryType típusban]`

Nem helyesek a többértelmű, túl általános megfogalmazások (pl: /queryInvoiceDigest operáció módosítása), ezeket lehetőség szerint kerüljük.

---------------------------------------------------------------------------------------------------------------------------------------------

Translation in progress, we are going to publish the new version of the file ASAP.