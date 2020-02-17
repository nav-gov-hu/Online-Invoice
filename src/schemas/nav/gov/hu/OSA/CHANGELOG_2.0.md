# Changelog 2.0

`scroll down for English version`

Változtatások 1.1-ről 2.0-ra.

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

- A technikai érvénytelenítés leválasztásra kerül a /manageInvoice operációról és saját operációt kap, melynek neve /manageAnnulment. Az operációhoz előzetesen ugyanúgy tokent kell kérni, mint a számlabeküldéshez. Az operáción belül ugyan úgy legfeljebb 100 indexig küldhető be technikai érvénytelenítés. Az egyes tételekhez kapcsolódó operáció külön singleType-ot kap, jelenleg egyetlen felvehető értékkel, az ANNUL enummal. Azon tranzakciók, amelyek tartalmaznak az ANNUL operáción kívül más műveletet is, szinkron ERROR hibával elutasításra kerülnek, tehát a tranzakciók homogenitását (vagy kizárólag adatszolgáltatás, vagy kizárólag technikai érvénytelenítés) továbbra is be kell tartani. Az érvénytelenítési adatokat ugyan úgy base64 kódoltan kell küldeni. A technikai érvénytelenítés adatai külön sémaleíróba, az invoiceAnnulment.XSD-be kerülnek, így az már független a data sémaleírótól. A szerver a kérésre ugyan úgy tranzakció azonosítót válaszol. Ezzel párhuzamosan a /manageInvoice kérésben a technicalAnnulment boolean törlésre kerül, és operációként csak CREATE, MODIFY, STORNO adható meg, ANNUL már nem.
- A korábbi homogén /queryInvoiceData operációból leválasztásra kerül a paraméteres keresés. A jelenlegi sémában a /queryInvoiceData keresőparaméterként csak számlaszámot fogad el, ezen felül kötelező egy új, invoiceDirection nevű tagban megadni, hogy a számlát kiállítóként vagy vevőként akarjuk-e lekérdezni. Ez rendezi a vevő oldali számla lekérdezés egyik felét is. A válaszban csak a lekérdezett számlaszám teljes adattartalma kerül visszaadásra, plusz az eddigi csomópontok (auditData, invoiceReference, compressedContentIndicator).
> Felhívjuk a figyelmet, hogy vevő oldali számlalekérdezés az interfészen is csak akkor lehetséges, ha a vevő adószáma ki van töltve az adatszolgáltatásban. A vevői adószámot nem tartalmazó számlák nem kereshetők!
- A paraméteres lekérdezés új operációja a /queryInvoiceDigest lesz. Mivel invoiceDirection ebben a kérésben is szerepel, ezzel egyben lehetőség van a vevő oldali számlák paraméteres lekérdezésére is. A keresőparaméterek teljesen új struktúrában adhatók meg, külön csomópontja van a kötelező, az opcionális, a relációs és a tranzakciós paramétereknek is. Az operáció válaszként csak digestet ad vissza, teljes tartalmat már sosem. Ha a listából szükség van valamely számlának a teljes tartalmára, akkor azt a /queryInvoiceData operációval le kell kérdezni.
> Felhívjuk a figyelmet, hogy vevő oldali számlalekérdezés az interfészen is csak akkor lehetséges, ha a vevő adószáma ki van töltve az adatszolgáltatásban. A vevői adószámot nem tartalmazó számlák nem kereshetők!
* Kötelező kereső paraméternek vagy kiállítási dátum tartományt, a beszúrás (feldolgozás) dátum tartományt, vagy az alapbizonylat sorszámát kell megadni. 
* Dátum tartomány megadása esetén a működés és a válasz az eddigieknek megfelelő, míg az alapbizonylat sorszám megadásakor az összes olyan számla kivonat visszaadásra kerül, amely az adott számlaszámra hivatkozik.
* Az opcionális kereső paraméterek közé bekerült az ÁFA csoport tagjának adószáma és az adózó neve, melyet szintén vevői és kiállítói oldalon is meg lehet adni.
* A relációs kereső paraméterek listaszerűen tartalmazzák a korábban kisebb-mint, nagyobb-mint relációkban kereshető értékeket. Az újdonság, hogy a keresett érték mellett egy ötös listából lehet kiválasztani a relációs operátort. (egyenlő, kisebb, nagyobb, kisebb-mint, nagyobb-mint) A jelenlegi struktúra a jövőbeni bővítéseket is sokkal egyszerűbbé teszi.
* A relációs kereső paraméterek közé bekerül a fizetési határidő.
* A tranzakciós kereső paraméterekben az átrendezésen kívül csak annyi a változás, hogy összhangban a /manageInvoice operációban kieső ANNUL értékre, keresőparaméternek itt is csak a CREATE, MODIFY, STORNO értékek adhatók már meg.
* A digest lekérdezés lapozása 10 tételről 100 tételre változik.
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
- Az adózó lekérdezésre használt /queryTaxpayer operáció visszaadja a lekérdezett adózó ÁFA csoport tagságát, valamint séma szinten tartalmazza az adózó rövid nevét.
* A kliens oldali lekérdezések gyakorisága alkalmazás oldalon korlátozva lesz. Ennek kialakításánál a metrika visszaadás költsége is szempont lesz, így ennek hiányában pontos értéket még nem tudunk mondani.
> Az operáció válasza egyelőre base64Binary, amint látjuk pontosan mi lesz benne, adunk ki külön XSD-t a tartalomhoz. Ez nem feltétlenül lesz a 2.0 része időben, de szeretnénk megteremteni már most a lehetőséget rá a sémában.
- Az API-ban új operációként megjelenik egy /queryInvoiceCheck operáció. A szolgáltatáshoz alap authentikációt kell végezni, mint a token kérésnél. Siker esetén a rendszer visszaadja, hogy a lekérdetett számlaszám létezik-e érvényesként a rendszerben, feltéve, hogy a lekérdezést végző adózó szerepel a számlán kiállítóként vagy vevőként.

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

- software adatok küldése minden requestben kötelező, a belső elemekre új kötelezőségek kerültek
- a queryInvoiceData request teljesen új típusra változott (InvoiceQueryType)
- InvoiceResultType átnevezés -> InvoiceDataResultType
- InvoiceQueryParamsType típus bővítése a csoport tag adószámával (groupMemberTaxNumber)
- InvoiceQueryParamsType típusból az adószám paraméter törlése
- InvoiceQueryResultType típus törlése
- TaxpayerAddressType törlésre került, az adószám lekérdező válasza a data:DetailedAddressType típust használja a címadatokhoz
- A beszúrás időpontja data:TimestampType típusra változott és UTC időre kerül konvertálásra a válaszban
- batchIndex opcionálisan megadható az egyes számla lekérdezésekben
- névkonvenció miatt a korábbi /queryInvoiceStatus operáció átnevezésre került /queryTransactionStatus operációra
- a /queryTransactionStatus operáció válaszában szereplő processingResult listaelem számossága unbounded értékre módosult

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
- LineNumberType típusban korlátozásra került a megadható hossz 20 helyiértékre
- QuantityType típus kiterjesztésre került 6-ról 10 tizedes jegyre
- ExchangeRateType típus minimum értéke 0.00-ról 0-ra változott
- TimestampType típusban opcionálissá vált a tört másodperc
- az árrés adózáshoz a sémában fogalmilag összevonásra kerül az ellenérték és a nettó érték

### 3) ÁFA analitika előkészítés

A megfelelő bázis időszaki adatok előállításához a 3.0-ás interfész előtt már szükség van arra, ha a bejövő számlaadatok tartalmazzanak alap ÁFA analitikához szükséges információkat is.

Ennek jegyében a számlasorokba új, opcionális minősítő tag került, a neve lineNatureIndicator. A tag értékkészlete alapján az egyes számlasorokban szereplő tételeket minősíteni kell a szerint, hogy a tétel termékértékesítésnek, szolgáltatás nyújtásnak vagy egyéb besorolásúnak minősül-e.

Ezen felül, minden QuantityType és MonetaryType típusú elemben, az érték kifejezhető volt a számla pénznemében és forintban is, a nevezett elemek kiegészítésre kerültek a megfelelő párjukkal, az alábbiak szerint:

- egységár forintban (line/unitPriceHUF)
- tétel nettó összege forintban (invoiceLines/line/lineAmountsNormal/lineNetAmountData/lineNetAmountHUF)
- tétel bruttó összege forintban (invoiceLines/line/lineAmountsNormal/lineGrossAmountData/lineGrossAmountNormalHUF)
- tétel bruttó összege forintban (invoiceLines/line/lineAmountsSimplified/lineGrossAmountSimplifiedHUF)
- számla bruttó összege forintban (invoiceSummary/summaryGrossData/invoiceGrossAmountHUF)
- számla nettó összege forintban (invoiceSummary/summaryNormal/invoiceNetAmountHUF)
- számla adótartalom bruttó összege forintban (invoiceSummary/summarySimplified/vatContentGrossAmountHUF)
- adómérték nettó összege forintban (invoiceSummary/summaryNormal/summaryByVatRate/vatRateNetData/vatRateNetAmountHUF)
- adómérték bruttó összege forintban (invoiceSummary/summaryNormal/summaryByVatRate/vatRateGrossData/vatRateGrossAmountHUF)

Az elemek kötelezősége öröklődik a sémában már korábban definiált párjukból.

4) Változások 2020.02.06-án

### 4.1) API sémaleíró

- a requestVersion és headerVersion tagekről lekerültek a default értékek, mostantól nem lehet önbezárt taget küldeni ezeknél
- a /queryTaxpayer válaszában új elemek kerültek (infoDate, taxNumberDetail)
- a /queryTaxpayer válaszában a címadat listatípusra módosult (taxpayerAddressItem), a címadatok mellett a taxpayerAddressType elemben a cím típusa (székhely, telephely, fióktelep)
- a /queryInvoiceDigest kérésében az OriginalInvoiceNumberQueryType típus törlésre került, a mandatoryQueryParams/originalInvoiceNumber útvonal mostantól element, nem komplex típus
- a /queryInvoiceDigest kérésében az additionalQueryParams típus bővítésre került a taxNumber paraméterrel, amely a megtalált eredményeket szűkíti a megadott adószámmal (a keresett mező a keresés irányától függ ugyan úgy, ahogy a csomópont többi eleménél). Ezt kell használni a korábbi originalInvoiceNumber/supplierTaxNumber szűrés megadására vevő oldali lekérdezéskor.
- bevezetésre került a /queryInvoiceChainDigest operáció és a hozzá kapcsolódó request/response elemek
- bevezetésre került a /queryTransactionList operáció és a hozzá kapcsolódó request/response elemek

### 4.2) serviceMetrics sémaleíró

- a rendszer publikus működési metrikáit és a lekérdezések válaszstruktúráját tartalmazó séma publikálásra került

--------------------------------------------------------------------------------------------------------------------------------------------

# Changelog 2.0

Comprehensive changes from 1.1 to 2.0.

## 1) Reasons for the change in general

There were two reasons for introducing the new 2.0 interface. First, many new opportunities and demands presented themselves since the actual launch of the system in July 2018, enough to justify considering a new major version for the invoicing interface. Second, Version 1.0 has a number of inconsistencies and shortcomings, and it is time to move beyond them. In other words, Version 2.0 contains both a new
functionality and refactorations. As this is a major version and it involves breaking changes on both the server and client sides, the 2.0 schema will receive its own namespace. It is also quite probable that the system will use a different URL to receive 2.0 messages than the one used currently.

### 1.1) Main problems requiring solutions:

- Technical annulment shares the /manageInvoice operation in the API structure used for submitting invoices (which allows it to be submitted in a compressed format, for example, even though doing so is completely pointless) while the internal content also depends on Data XSD. Linking results in unnecessary tags (e.g. boolean for technicalAnnulment) or other inconsistencies (e.g. ANNUL can be used as an operational parameter when querying invoices, but will never find any matches) and will make it impossible to extend the /manageInvoice operation in a modular fashion in the long run.
- The /queryInvoiceData operation used for querying invoices has both the input and output as choice. This is antipattern, and means that it is impossible to guarantee the POST body size limit for the response, as defined in the submissions.
- The response to the /queryInvoiceStatus operation used for querying the processing status does not show whether the invoice has been saved, meaning that it is impossible to know from when data can be provided regarding the modifying invoice. This can result in paradoxical cases, where the transaction is not yet fully processed, but the individual invoices can nonetheless be queried as saved invoices, either on the screen or through the interface.
- The approval status of technical annulment cannot be queried using the API, meaning that neither resubmission nor other processes conditional on resubmission can be automated on the client-side.
- Invoices cannot be queried through the API customer-side.
- While it is possible to sort the individual modifying invoices based on modificationTimestamp, it is not possible to verify uniqueness or sequentiality.
- It is not possible to modify multiple invoices using a single invoice.
- The issue date of the invoice and the issue date of the modifying document are two separate tags in the schema definition, which causes unnecessary complications in the queries.
- The equality, smaller-than and greater-than relations used in searches done through the frontend and through the API are not consistent. In addition, the XML structure for the smaller-than and larger-than search fields is unnecessarily verbose.
- The typing is incorrect for a number of request and response elements in the API XSD, meaning that it is not always possible to generate the objects correctly.
- The CRC32 checksum code needs to be replaced with a cryptographic hash function capable of fully protecting the business-related content.
- There is no metric query option regarding the system’s operation.

### 1.2) Planned solutions

- Technical annulment will be separated from the /manageInvoic operation, and will receive its own operation named /manageAnnulment. A token will need to be requested for the operation in advance, the same way as for submitting an invoice. Just as before, it will be possible to submit a technical annulment using the operation, up to 100 indexes. The operation associated with individual items will be given a separate singleType, currently with a single assignable value, which is the ANNUL enum. Transactions containing operations other than ANNUL will be rejected with a synchronous ERROR message, meaning that the transactions must still remain homogenous (purely data exchange, or purely technical annulment.) Annulment data must be sent coded in base64, just as before. A separate schema definition, invoiceAnnulment.XSD will be used for technical annulment data, meaning that it will be independent of the data schema definition. The server will still respond to the request with a transaction identifier, as before. In addition, the technicalAnnulment boolean will be deleted from the /manageInvoice request, meaning that only CREATE, MODIFY, and STORNO can be entered as operations, ANNUL is no longer valid.
- Parametric search will be separated from the previously homogeneous /queryInvoiceData operation. In the current schema, /queryInvoiceData only accepts an invoice number as a search parameter. In addition, there is a new invoiceDirection tag, in which it is mandatory to specify whether you wish to query the invoice as an issuer or a customer. This also settles one half of customer-side invoice queries. The response will only include the total data content of the queried invoice number, plus the previous nodes (auditData, invoiceReference, compressedContentIndicator). 
> Please note that the interface will also only allow customer-side invoice queries if the customer’s tax number is filled out in the data exchange. Invoices without customer tax numbers cannot be queried!
- There will be a new operation for parametric queries, /queryInvoiceDigest. As this request also includes invoiceDirection, it will also be possible to perform a customer-side parametric query of the invoices. Search parameters can be structured completely differently. Mandatory, optional, relational and transactional parameters will all have different nodes. The operation will only provide a digest as a response, never the full content any more. If you need the full content for one of the invoices from the list, you can use the /queryInvoiceData operation to query it.
> Please note that the interface will also only allow customer-side invoice queries if the customer’s tax number is filled out in the data exchange. Invoices without customer tax numbers cannot be queried!
- You must provide either an issue or insert date range or the invoice number of the base document as a mandatory search parameter.
- When providing an issue data range, both the operation and the response will work the same way as they have before, whereas when providing the invoice number of the base document, all invoices referencing the provided invoice number will be returned.
- The optional search parameters now include the tax number for the VAT group member and the name of the taxpayer, which can also be entered either as a customer or as an issuer.
- The relational search parameters will now list the search values that could be searched previously in smaller-than and greater-than relations. What is new is that you can now select a relational operator from a list of five, in addition to the value searched (equal, smaller, greater, smaller-than, greater-than.) The current structure also makes future extensions much simpler.
- Relational query for invoice payment date is now possible.
- The paging parameter of the digest query is increased from 10 to 100 items.
- In addition to the reorganisation, the only change made to the transactional search parameters is that much like the way the ANNUL value was removed from the /manageInvoice operation, only the CREATE, MODIFY, STORNO values can now be provided as search parameters here as well.
- The response for /queryInvoiceStatus can optionally include an annulmentVerificationStatus tag containing the approval status of the technical annulment, as long as the transactionId provided in the request references a transaction containing a technical annulment.
- The invoiceStatus tag value set returned in the /queryInvoiceStatus operation now has a new element, the value SAVED. The SAVED status is sequentially between PROCESSING and DONE, when the processing of the transaction is not yet completed, but the invoice for the index provided has already been saved, meaning that its data can be queried, and it is ready for data exchange modifying or cancelling it.
> Please note that a returned status of SAVED does not mean that data exchange obligations have been met, DONE is still the only status signifying that! Accordingly, the processing status queries should continue until all line items for the transaction are DONE or ABORTED.
- All requests and responses in the API schema definition have been assigned their own type.
- Modifying invoices will be handled differently in the data schema definition. The modificationIssueDate, modificationTimestamp and lastModificationReference tags will be deleted from the invoiceReference node. Of these, modificationIssueDate will be conceptually merged with the invoiceIssueDate tag, meaning that as of Version 2.0, it will refer to the issue date of the invoice OR the modifying document. The other deleted tags will no longer need to be submitted in Version 2.0. There will also be a new element, modificationIndex, which should be filled out client-side to define the sequence of the modification. The value of the tag starts with 1, meaning that logically, the first modifying or cancelling invoice should be assigned a modificationIndex of 1. Uniqueness verification will definitely return an ERROR server-side, we are still investigating the possibility of sequentiality verification. If sequentiality verification is implemented, we may introduce a delay into server-side processing. In this case – provided that the system has already received the base invoice – there would be a predefined time frame available for the data exchange for the individual modification document, and sequentiality would only be tested after the expiry of the delay time. The /queryInvoiceDigest response will return the modificationIndex if several modifying documents were issued for the base document from different invoicing systems, and the correct next value in the sequence is not known at the time of modification.
- Along with the deleted tags, the WARNING messages used for investigating these relationships will also be deleted.
- It will be possible to modify multiple invoices using a single invoice.
- To do this, the first step is to move the invoice number and the invoice issue date from their current location, invoiceHead/invoiceData, and to include them right at the top, after the root element. This will also mean that the logic will not permit modifying an incorrect issue date. Therefore, to allow for doing so, we have introduced a new technical annulment reason, named ERRATIC_INVOICE_ISSUE_DATE.
- After moving the invoice number and the invoice issue date up to the top, the schema includes a choice defining whether it is a “singular” or a batch data exchange. The structure for a singular data exchange is identical to that in Version 1.1. Batch data exchanges can be created under the name batchInvoice. Under a batchInvoice element, you can repeat the “singular” data exchange structure any number of times in a non-enumerated list, by generating a batchIndex.
- In internal content, the batchInvoice tag should, of course, only be generated for MODIFY or STORNO operations. There is an additional    restriction: data exchanges of this nature can only have 1 external index in the API XML, meaning that in these cases, the request can only include 1 data exchange instead of the usual 100. A new asynchronous ERROR will be introduced for both cases. 
> We do not currently see a justification for introducing 2 new operations to handle these cases (e.g. BATCH_MODIFY and BATCH_STORNO), but this could change in the future.
- To allow for searching batch processing error messages more easily, the response to /queryInvoiceStatus will return both index and batchIndex. In addition, all of the pointers will also include the referenced invoice number, originalInvoiceNumber.
- Batch modifications will be processed according to the rules of technical annulment, i.e. if any batchIndex item fails due to an ERROR, the entire data exchange will fail verification.
- The /queryTaxpayer operation used for taxpayer query now will return the VAT group membership of the taxpayer, and now contains the shortened name on schema level.
- The frequency of client-side queries will be restricted on the application side. When designing this function, the cost of returning metrics will also be an aspect, therefore we cannot provide an precise value yet in the absence thereof.
> The response of the operation is currently base64Binary, and we will provide a separate XSD for the content as soon as we know exactly what will be included in it. This functionality will not necessarily be ready in time to make it into Version 2.0, but we would already like to create an option in the schema for it.
- The API will include a new operation, /queryInvoiceCheck. Using the service will require basic authentication, much like in the case of token requests. If successful, the system will return the fact of existence of the invoice, provided the invoice is valid and the tax number of the querying entity is present on the invoice as supplier or customer.

### 1.3) The calculation of requestSignature, as of Version 2.0

As of Version 2.0, the value of requestSignature should be calculated using SHA3-512 instead of SHA2-512 for all operations.

The calculation of requestSignature for the /manageInvoice and /manageAnnulment operations is still an exception. The difference, in
comparison to Version 1.1, is that the indexwise partial values for these operations should be calculated using SHA3-512 instead of CRC32,
and the full content of the operation and invoice tags under the individual indexes should be concatenated before calculating. For example, the partial hash value for index 1, given the following

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

will be

```java
'CREATE' + 'QWJjZDEyMzQ=' -> SHA3-512(CREATEQWJjZDEyMzQ=) -> 4317798460962869bc67f07c48ea7e4a3afa301513ceb87b8eb94ecf92bc220a89c480f87f0860e85e29a3b6c0463d4f29712c5ad48104a6486ce839dc2f24cb
```

The partial hashes should be concatenated in ascending order of index, then the requestSignate value for the resulting requestId+timestamp+signkey+concatenated partial hashes should be calculated.
> We will provide example XMLs as well in a future pull request.

## 2) Other changes

In addition to the main problems listed, we have received a number of taxpayer requests for extending the length or value set of various
fields, and a number of other refactored elements will also be included in the schema definitions, as per the following. To check all the
changes in detail, simply DIFF Version 2.0 and Version 1.1 in the pull request.

### 2.1) API schema definition

- software data must be sent in all requests, and there are new mandatory requirements for the internal elements
- the queryInvoiceData request has been changed to a completely new type (InvoiceQueryType)
- InvoiceResultType renamed -> InvoiceDataResultType
- InvoiceQueryParamsType was extended with the tax number for the group member (groupMemberTaxNumber)
- the tax number parameter was deleted from InvoiceQueryParamsType
- InvoiceQueryResultType was deleted
- TaxpayerAddressType was deleted, the response to tax number queries will use data:DetailedAddressType for address data (if the tagged address cannot be returned due to the requirements described by the response, we will fill out the missing tags with a uniform signifier defined in the specification, e.g. N/A)
- insert date is now changed to data:TimestampType and the value is converted to UTC time in the response
- batchIndex may be optionaly provided in single invoice queries
- the previous /queryInvoiceStatus operation is now renamed to /queryTransactionStatus operation due to naming convention
- the cardinality of the processingRequest list element in the response of /queryTransactionStatus operation is now unbounded

### 2.2) DATA schema definition

- the lineExpressionIndicator tag is mandatory, the LINE_EXPRESSION_INDICATOR_MISSING verification can be deleted in Version 2.0
- the default value of the lineExpressionIndicator tag is false
- PostalCodeType will be given a new pattern: from now on, space and hyphen characters are allowed (as long as they are not at the start or the end of a string), and the minimum length has now been reduced to 3 characters
- the lineDescription tag is extended to a length of 512 characters
- ProductCodeCategoryType now has a new enum value (TESZOR), the length of the new pattern is extended from 5 to 6
- the productCodeOwnValue tag is extended to a length of 255 characters
- invoiceDeliveryDate tag is mandatory, the MANDATORY_CONTENT_MISSING verification can be deleted in Version 2.0
- InvoiceDataType renamed -> InvoiceDetailType
- IndexType was removed from the API, and is now a part of Data
- LineNumberType is now restricted to 20 digits length
- QuantityType is extended from 6 to 10 fracture digits
- ExchangeRateType lower bound is changed from 0.00 to 0
- Fraction seconds in TimestampType are now optional
- For margin scheme taxation, consideration and net amount is now treated as identical notions in the schema

### 3) VAT analytics preparation

In order to have adequate base interval data it is necessary to have the incoming data to carry basic VAT analytics related data even before interface version 3.0.  

In light of this endeavour, line data has been supplemented with a new optional qualifier tag named lineNatureIndicator. Based on its values, all lines need to be classified whether they are supply of goods, supply of services or other .

Furthermore, every QuantityType és MonetaryType element - where it is possible to express the value in the currency of the invoice or in HUF - have been supplemented with their corresponding pair as follows:

- unit price in HUF (line/unitPriceHUF)
- line net amount in HUF (invoiceLines/line/lineAmountsNormal/lineNetAmountData/lineNetAmountHUF)
- line gross amount in HUF (invoiceLines/line/lineAmountsNormal/lineGrossAmountData/lineGrossAmountNormalHUF)
- line gross amount in HUF (invoiceLines/line/lineAmountsSimplified/lineGrossAmountSimplifiedHUF)
- summary gross amount in HUF (invoiceSummary/summaryGrossData/invoiceGrossAmountHUF)
- summary net amount in HUF (invoiceSummary/summaryNormal/invoiceNetAmountHUF)
- gross content summary in HUF (invoiceSummary/summarySimplified/vatContentGrossAmountHUF)
- VAT rate net amount in HUF (invoiceSummary/summaryNormal/summaryByVatRate/vatRateNetData/vatRateNetAmountHUF)
- VAT rate gross amount in HUF (invoiceSummary/summaryNormal/summaryByVatRate/vatRateGrossData/vatRateGrossAmountHUF)

Cardinality of the new elements is inherited from their pair previously defined in the schema.

4) Changes on 06.02.2020

### 4.1) API schema definition

- default values of requestVersion and headerVersion elements are discarded, it not possible from now on to send self-closed tags
- the response of /queryTaxpayer now contains new elements (infoDate, taxNumberDetail)
- the address element in the response of /queryTaxpayer is now a list type (taxpayerAddressItem), and the qualifier of the addresses (hq, branch, site) is also returned
- OriginalInvoiceNumberQueryType in the request of /queryInvoiceDigest is deleted, the mandatoryQueryParams/originalInvoiceNumber path is now an element, not a complex type
- the additionalQueryParams type in the request of /queryInvoiceDigest is now extended with the taxNumber param, which filters the found invoices with the provided tax number. (the searched field is depending on the direction of the query, just like in case of the other similar elements) This tag must be used for setting the previous originalInvoiceNumber/supplierTaxNumber serch criteria in case of an inbound query.
- /queryInvoiceChainDigest operation is now added, along with it's own request/response elements
- /queryTransactionList operation is now added, along with it's own request/response elements

### 4.2) serviceMetrics schema definition

- the schema containing the response structure of the system's public operational metrics is now public