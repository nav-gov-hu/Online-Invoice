# Changelog 2.0

`scroll down for English version`

## 1) A módosítás igénye általánosságban
A 2.0-ás interfész verzió bevezetésének igénye kettős. Egyrészről, a rendszer 2018 júliusi éles indulása óta eltelt idő keletkeztetett annyi új lehetőséget és igényt, hogy megérje a számlabejelentő interfészből egy új nagy verzióban gondolkodni, másrészt az 1.0 verzió számos inkonzisztenciát és hiányosságot tartalmaz, melyeknek a kivezetésére megérett az idő. A 2.0-ás verzió tehát egyaránt tartalmaz új funkcionalitást és refaktot is. Figyelemmel arra, hogy nagy verzióról van szó és a változások breaking change-t jelentenek mind szerver mind kliens oldalon, a 2.0-ás séma új namespace-t kap. Nagy valószínűséggel számítani kell arra is, hogy a 2.0-ás üzeneteket a rendszer más URL-en fogadja majd mint jelenleg.

### 1.1) Megoldani kívánt főbb problémák:

- A technikai érvénytelenítés osztozik a számlabeküldésre használt /manageInvoice operációval az API struktúrában (ami miatt például lehet tömörítve is beküldeni, ami teljesen értelmetlen) miközben a belső tartalom függ a Data XSD-től is. Az összekapcsolás felesleges tageket (pl: technicalAnnulment boolean megadása) vagy más inkonzisztenciát (pl számla lekérdezésben lehetséges az ANNUL, mint operációs paraméter használata, amire sosincs találat) eredményez, és ellehetetleníti hosszútávon a /manageInvoice operáció moduláris bővítését.
- A számla lekérdezésben használt /queryInvoiceData operációban a bemenet és a kimenet is choice ami antipattern, továbbá ez miatt a válaszban nem lehet azt a méretkorlátot garantálni, ami a beküldések során a POST body méretére vonatkozik. 
- A feldolgozás státusz lekérdezésre használt /queryInvoiceStatus operáció válaszában nem látszik a mentés ténye, azaz nem lehet tudni, mikortól lehet a módosító számláról adatot szolgáltatni. Ez eredményez olyan paradox eseteket, hogy a tranzakció még nincs teljesen feldolgozva, de az egyes számlák már lekérdezhetők mentett számlaként akár a felületen, akár az interfészen.
- A technikai érvénytelenítés jóváhagyási státusza nem kérdezhető le az API-n keresztül, így sem az újraküldés, sem más erre épülő folyamat nem automatizálható kliens oldalon.
- A vevő oldali számlalekérdezés nem lehetséges API-n keresztül.
- Az egyes módosító számlák modificationTimestamp alapján sorrendezhetők, de sem egyediséget, sem sorfolytonosságot nem lehet vizsgálni.
- Nem lehet 1 számlával több számlát módosítani.
- A számla kelte és a módosító okirat kelte 2 külön tag a sémaleíróban, ami a lekérdezésekben felesleges bonyodalmakat okoz. 
- A frontendes és az API-s keresés nem egységes az egyenlőség és a kisebb-nagyobb relációk keresésében, plusz a kisebb-mint és nagyobb-mint keresőmezők XML struktúrája indokolatlanul terjengős.
- Az API XSD-ben rossz a típusosság számos kérés-és válasz elemre, az objektumok nem minden esetben generálhatók le helyesen.
- A CRC32 ellenőrző algoritmust le kell váltani egy kritográfiai hash függvénnyel, ami a teljes üzleti tartalmat védi.
- Nincs a rendszer működéséről metrika lekérdezési lehetőség.

### 1.2) A tervezett megoldások

- A technikai érvénytelenítés leválasztásra kerül a /manageInvoice operációról és saját operációt kap, melynek neve /manageAnnulment. Az operációhoz előzetesen ugyan úgy tokent kell kérni, mint a számlabeküldéshez. Az operáción belül ugyan úgy legfeljebb 100 index alatt küldhető be technikai érvénytelenítés. Az egyes tételekhez kapcsolódó operáció külön singleType-ot kap, jelenleg egyetlen felvehető értékkel, az ANNUL enummal. Azon tranzakciókat, amelyek tartalmaznak az ANNUL operáción kívül más műveletet is, szinkron ERROR hibával elutasításra kerülnek, tehát a tranzakciók homogenitását (vagy kizárólag adatszolgáltatás, vagy kizárólag technikai érvénytelenítés) továbbra is be kell tartani. Az érvénytelenítési adatokat ugyan úgy base64 kódoltan kell küldeni. A technikai érvénytelenítés adatai külön sémaleíróba, az invoiceAnnulment.XSD-be kerülnek, így az már független a data sémaleírótól. A szerver a kérésre ugyan úgy tranzakció azonosítót válaszol. Ezzel párhuzamosan a /manageInvoice kérésben a technicalAnnulment boolean törlésre kerül, és operációként csak CREATE, MODIFY, STORNO adható meg, ANNUL már nem.
- A korábbi homogén /queryInvoiceData operációból leválasztásra kerül a paraméteres keresés. A jelenlegi sémában a /queryInvoiceData keresőparaméterként csak számlaszámot fogad el, ezen felül kötelező egy új, invoiceEntity nevű tagban megadni, hogy a számlát kiállítóként vagy vevőként akarjuk-e lekérdezni. Ez rendezi a vevő oldali számla lekérdezés egyik felét is. A válaszban csak a lekérdezett számlaszám teljes adattartalma kerül visszaadásra, plusz az eddigi csomópontok (auditData, invoiceReference, compressedContentIndicator).
> Felhívjuk a figyelmet, hogy vevő oldali számlalekérdezés az interfészen is csak akkor lehetséges, ha a vevő adószáma ki van töltve az adatszolgáltatásban. A vevői adószámot nem tartalmazó számlák nem kereshetők!
- A paraméteres lekérdezés új operációja a /queryInvoiceDigest lesz. Mivel invoiceEntity ebben a kérésben is szerepel, ezzel egyben lehetőség van a vevő oldali számlák paraméteres lekérdezésére is. A keresőparaméterek teljesen új struktúrában adhatók meg, külön csomópontja van a kötelező, az opcionális, a relációs és a tranzakciós paramétereknek is. Az operáció válaszként csak digestet ad vissza, teljes tartalmat már sosem. Ha a listából szükség van valamely számlának a teljes tartalmára, akkor azt a /queryInvoiceData operációval le kell kérdezni.
> Felhívjuk a figyelmet, hogy vevő oldali számlalekérdezés az interfészen is csak akkor lehetséges, ha a vevő adószáma ki van töltve az adatszolgáltatásban. A vevői adószámot nem tartalmazó számlák nem kereshetők!
* Kötelező kereső paraméternek vagy kiállítási dátum tartományt, vagy az alapbizonylat sorszámát kell megadni. 
* Dátum tartomány megadása esetén a működés és a válasz az eddigieknek megfelelő, míg az alapbizonylat sorszám megadásakor az összes olyan számla kivonat visszaadásra kerül, amely az adott számlaszámra hivatkozik.
* Az opcionális kereső paraméterek közé bekerült az ÁFA csoport tagjának adószáma, melyet szintén vevői és kiállítói oldalon is meg lehet adni.
* A relációs kereső paraméterek listaszerűen tartalmazzák a korábban kisebb-mint, nagyobb-mint relációkban kereshető értékeket. Az újdonság, hogy a keresett érték mellett egy ötös listából lehet kiválasztani a relációs operátort. (egyenlő, kisebb, nagyobb, kisebb-mint, nagyobb-mint) A jelenlegi struktúra a jövőbeni bővítéseket is sokkal egyszerűbbé teszi.
* A tranzakciós kereső paraméterekben az átrendezésen kívül csak annyi a változás, hogy összhangban a /manageInvoice operációban kieső ANNUL értékre, keresőparaméternek itt is csak a CREATE, MODIFY, STORNO értékek adhatók már meg.
- A /queryInvoiceStatus válasza opcionálisan visszaad egy annulmentVerificationStatus taget, amely a technikai érvénytelenítés jóváhagyási státuszait tartalmazza, ha a kérésben megadott transactionId egy technikai érvénytelenítést tartalmazó tranzakcióra mutat.
- A /queryInvoiceStatus operációban visszaadott invoiceStatus tag értékkészlete új elemmel, a SAVED értékkel bővült. A SAVED státusz sorrendben a PROCESSING és a DONE között áll, ekkor a tranzakció feldolgozása még nem fejeződött be, de az adott indexen lévő számla már elmentésre került, tehát az adatai lekérdezhetők, a rá módosító vagy stornó adatszolgáltatás már küldhető.
> Felhívjuk a figyelmet, hogy a SAVED státusz visszaadása nem egyenlő azzal, hogy az adatszolgáltatási kötelezettség teljesült, ezt továbbra is kizárólag a DONE státusz jelzi! Ezért a feldolgozás státusz lekérdezést egészen addig folytatni kell, amíg a tranzakció alatt minden tétel DONE vagy ABORTED nem lesz!
- Az API sémaleíróban minden kérés és válasz saját típust kapott.
- A módosító számlák kezelése átalakul a data sémaleíróban. Az invoiceReference csomópontból törlésre kerül a modificationIssueDate, a modificationTimestamp és a lastModificationReference. A törlendők közül a modificationIssueDate fogalmilag összevonásra kerül az invoiceIssueDate taggel, így az a 2.0-tól a számla VAGY a módosító okirat kiállítását jelenti. A többi törölt taget a 2.0-ban nem kell küldeni. Új elemként megjelenik a modificationIndex, amelyben a kliens oldalnak kell a módosítás sorrendiségét jelezni. A tag értéke 1-től kezdődik, logikailag az első módosító vagy stornó számlának kell az 1-es értéket kapnia. A szerver oldalon az egyediség vizsgálatra biztosan ERROR kerül bevezetésre, a sorfolytonosság ellenőrzésének lehetőségét még vizsgáljuk. Amennyiben megvalósításra kerül a sorfolytonosság ellenőrzése is, úgy elképzelhető, hogy a szerver oldali feldolgozásba késleltetés kerül be, ekkor - feltéve, hogy az alapszámla már beérkezett a rendszerbe-, adott időkeret állna rendelkezésre az egyes módosító okirat adatszolgáltatására, és a sorfolytonosság csak a késleltetési idő leteltét követően kerülne vizsgálatra. A modificationIndex a /queryInvoiceDigest válaszában visszaadásra kerül arra az esetre, ha az alapbizonylatra több módosító okiratot állítottak volna ki eltérő számlázó rendszerekből, és a módosításkor nem ismert a helyes következő érték.
* A törölt tagekkel egyidejűleg törlésre kerülnek azok a WARNING üzenetek, melyek ezek összefüggéseit vizsgálták.
- Lehetőség nyílik egy számlával több számlát módosítani.
* Ehhez elsőként szükség van arra, hogy a számlaszám és a számla kiállítának időpontja kikerüljön a jelenlegi helyéről, és az invoiceHead/invoiceData helyett rögtön legfelül, a root element után szerepeljen. A kiemelés azt is jelenti, hogy helytelen kiállítási dátum esetén logikailag nincs lehetőség a módosításra, ezért ennek javítására új technikai érvénytelenítési okot vezettünk be ERRATIC_INVOICE_ISSUE_DATE néven.
* A kiemelést követően lehetőség van a sémában egy choice szerint eldönteni, hogy 'egyes' adatszolgáltatást vagy kötegelt adatszolgáltatást írunk le. Az egyes adatszolgáltatás struktúrája ugyan az, mint az 1.1-ben. A kötegelt adatszolgáltatást batchInvoice néven lehet megképezni. A batchInvoice alatt egy megszámlálhatatlan listaelemben bárhányszor lehet ismételni az 'egyes' adatszolgáltatás szerinti struktúrát, egy batchIndex képzésével.
* A belső tartalomban batchInvoice taget értelemszerűen csak MODIFY vagy STORNO operációkban szabad képezni. További megkötés, hogy az ilyen adatszolgáltatásoknak kívül az API XML-ben csak 1 indexe lehet, tehát a kérés 100 helyett legfeljebb csak 1 adatszolgáltatást tartalmazhat ebben az esetben. Mindkét eset vizsgálatára új aszinkron ERROR kerül majd bevezetésre.
> Egyelőre nem látjuk indokoltnak, hogy bevezessünk 2 új operációt erre az esetre, (pl: BATCH_MODIFY és BATCH_STORNO) de ez a jövőben még változhat.
* A kötegelt módosítás feldolgozására vonatkozó hibaüzenetek könnyebb keresése miatt a /queryInvoiceStatus válaszában visszaadásra kerül az index mellett a batchIndex is, továbbá a pointerekbe mindenhol bekerül a hivatkozott számla száma, az originalInvoiceNumber is.
* A kötegelt módosítás feldolgozása a technikai érvénytelenítés szabályai szerint fog történni, azaz amennyiben bármely batchIndex alatti tétel ERROR miatt elbukik, úgy az egész adatszolgáltatás is el fog bukni az ellenőrzésen.
- Az API-ban új operációként megjelenik egy /querySystemMetrics operáció. A szolgáltatáshoz alap authentikációt kell végezni, mint a token kérésnél. Siker esetén a rendszer visszaadja a rendszer aktuális állapotára vonatkozó metrikákat. 
* A kliens oldali lekérdezések gyakorisága alkalmazás oldalon korlátozva lesz. Ennek kialakításánál a metrika visszaadás költsége is szempont lesz, így ennek hiányában pontos értéket még nem tudunk mondani.
> Az operáció válasza egyelőre base64Binary, amint látjuk pontosan mi lesz benne, adunk ki külön XSD-t a tartalomhoz. Ez nem feltétlenül lesz a 2.0 része időben, de szeretnénk megteremteni már most a lehetőséget rá a sémában.

### 1.3) A requestSignature számítása a 2.0 verziótól 
A requestSignature értékét a 2.0 verziótól minden operációban SHA2-512 helyett SHA3-512 függvénynel kell számolni. 

A /manageInvoice és a /manageAnnulment operációk alatt a requestSignature számítása továbbra is speciális. A változás az 1.1-hez képest, hogy ezekben az operációkban az indexenkénti részleges értékeket CRC32 helyett már SHA3-512 függvénnyel kell számolni, és az egyes indexek alatti operation és invoice tagek teljes tartalmát össze kell fűzni a számítás előtt.
Például az alábbi index
```xml
<invoiceOperations>
		<compressedContent>false</compressedContent>
		<invoiceOperation>
			<index>1</index>
			<operation>CREATE</operation>
			<invoice>QWJjZDEyMzQ=</invoice>
		</invoiceOperation>
	</invoiceOperations>
</ManageInvoiceRequest>
```
részleges hash értéke a következő
```java
'CREATE' + 'QWJjZDEyMzQ=' -> SHA3-512(CREATEQWJjZDEyMzQ=) -> 4317798460962869bc67f07c48ea7e4a3afa301513ceb87b8eb94ecf92bc220a89c480f87f0860e85e29a3b6c0463d4f29712c5ad48104a6486ce839dc2f24cb
```
A részleges hasheket index szerint monoton növekvő sorrendben össze kell fűzni, és az így képzett requestId+timestamp+signkey+konkatenált részleges hashekre kiszámolni a requestSignate értékét.
> Egy későbbi pull requestben adni fogunk példa XML-eket is.

## 2) Egyéb Módosítások
A felsorolt főbb problémákon felül több adózói igény is érkezett egyes mezők hosszának vagy értékkészletének bővítésére, illetve számos más refakt elem is bekerül a sémaleírókba, az alábbiak szerint. Az összes módosítás tételes vizsgálatához össze lehet DIFF-elni a pull requestben szereplő 2.0-át az 1.1-es állapottal. 
 
### 2.1) API sémaleíró

- software adatok küldése minden requestben kötelező, a belső elemekre új közelezőségek kerültek
- a queryInvoiceData request teljesen új típusra változott (InvoiceQueryType)
- InvoiceResultType átnevezés -> InvoiceDataResultType
- InvoiceQueryParamsType típus bővítése a csoport tag adószámával (groupMemberTaxNumber)
- InvoiceQueryParamsType típusból az adószám paraméter törlése
- InvoiceQueryResultType típus törlése
- TaxpayerAddressType törlésre került, az adószám lekérdező válasza a data:DetailedAddressType típust használja a címadatokhoz

### 2.2) DATA sémaleíró

- a lineExpressionIndicator tag kötelező, a LINE_EXPRESSION_INDICATOR_MISSING ellenőrzés 2.0-nál törölhető
* a lineExpressionIndicator tag default értéke false
- PostalCodeType új patternt kap, mostantól a szóköz és a kötőjel engedélyezett karakter (ha az nem a string elején vagy végén áll), a minimum hossz 3 karakterre csökkent
- lineDescription tag bővítésre kerül 512 hosszúra
- ProductCodeCategoryType új enum értéket kap (TESZOR), az új pattern hossz 5-ről 6-ra nő
- productCodeOwnValue tag bővítésre kerül 255 hosszúra
- invoiceDeliveryDate kötelező, a MANDATORY_CONTENT_MISSING ellenőrzés 2.0-nál törölhető
- InvoiceDataType átnevezése -> InvoiceDetailType
- IndexType kikerült az API-ból és a Data része lett
- árrés adózáshoz új tag került meghatározásra lineConsideration néven, ez a számlasorban szereplő nettő értékkel (lineNetAmount) choice-ban áll

----------------------------------------------------------------------------------------------------------------------------------------------------------

Translation in progress, we are going to publish the new version of the file ASAP.