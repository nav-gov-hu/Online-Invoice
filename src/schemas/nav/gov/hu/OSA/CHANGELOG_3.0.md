# Changelog 3.0

`scroll down for English version`

Változtatások 2.0-ról 3.0-ra.

## 1) A módosítás igénye általánosságban

A módosítási igények - ahogy a 2.0 esetében is voltak - ezúttal is többrétűek. Egyrészről szükséges biztosítani azt, hogy 2021.01.01-től a rendszer képes legyen fogadni API-n keresztül a magánszemélyeknek kiállított, közösségi és az export számlákat is. Másrészről az Online számla projekt elérkezett ahhoz a szakaszához, ahol már - tervezetten - nem akarunk a 3.0-ás állapothoz képest új XML API verziót kiadni (csak jogszabályi változás vagy nagyobb horderejű igény jövőbeni megjelenése esetén, és akkor sem biztos hogy ez nagy verzió lesz), ezért fenntarthatósági és hosszú távon fontos egyéb szempontokat is figyelembe akartunk venni. Ennek okán megkíséreltük kisimítani az összes olyan problémát az API-ból, ami eddig üzletileg vagy technikailag problémaként felmerült és olyan megoldást találni rá, amely mindkét oldal számára kielégítő. Legvégül de nem utolsó sorban pedig szükség van azon hiányzó adatok pótlására vagy nevesítésére, amelyek birtokában az ÁFA bevallási tervezetek pontosabban elkészíthetők.

### 1.1) A tervezett megoldások

- A 3.0-ás API egyik fő sarokköve az elektronikus számlázás katalizálása és használhatóbbá tétele. Ennek érdekében az API XML-ben megadható lesz az electronicInvoiceHash elem, amely az elektronikus számlák hash lenyomatát tartalmazza. Ez a módosítás még csak azt biztosítja, hogy a számla kiállítás és a számla módosítás során az üzleti adatok és a hash lenyomatok elválaszthatók lesznek egymástól. A módosítással az is kimondásra került, hogy az API nem támogatja azt a használati esetet, amikor csak az eredeti számla hash helyesbítése miatt érkezne be módosító adatszolgáltatás, ezért erre nem is biztosítunk beviteli lehetőséget. Az elektronikus hash miatt hibás adatszolgáltatások csak technikai érvénytelenítéssel javíthatók, ez miatt új érvénytelenítési ok került az Annulment sémába.
- Az elektronikus számlázás támogatását érintő másik fő változás, hogy a sémába bekerül egy olyan kötelező, completenessIndicator nevű jelölő, amelyben nyilatkozni lehet arról, hogy a 3.0-ás adatszolgáltatás keretében - az adózó saját döntése alapján - maga az elektronikus számla lesz beküldve, a séma szerinti XML formátumban, teljes adattartalommal. Ily módon a vevő már az adatszolgáltatás birtokában befogadhatja és kontírozhatja a számlát, nincs szükség további kézbesítésre, sem pedig az adatszolgáltatás és a számla későbbi (akár manuális) összehasonlítására. A jogszabályi környezet igazodni fog ehhez a változáshoz, a NAV ki fogja kényszeríteni ezen adatszolgáltatások esetében hogy az adatok integritása ne sérülhessen. Ezen számlák esetében az electronicInvoiceHash elemet az interfész dokumentációban meghatározott módon kell majd számolni és annak helyesnek is kell lennie. Ezen adatszolgáltatásokat továbbá nem lehet majd technikailag érvényteleníteni, tekintettel arra hogy a számla és az adatszolgáltatás között nem lehet eltérés, mivel az adatszolgáltatás maga a számla. Az ilyen számlákból készült számlaláncokra a NAV ezt a szolgáltatást csak bizonyos megkötésekkel biztosítja. (ld. ERROR módosítások fejezet) A funkciót csak 2021.01.01-től lehet igénybe venni, ezt megelőzően a feldolgozás átmeneti hibakóddal el lesz utasítva.
- A projekt indulása óta érdemben megoldatlan az ún. nagyméretű adatszolgáltatások problémája, amikor a POST body size meghaladja a 10 MB-t. Ez jellemzően olyan számlakiállítók esetében fordul elő kis számban, akik szektorális jogszabály alapján nagyon részletes számlát kell hogy kiállítsanak, pl telekommunikációs és közüzemi szolgáltatók. Ugyanakkor az ÁFA törvény messze nem vár el ilyen részletezettségű számlaadatokat, és az adatszolgáltatási kötelezettség is kizárólag az ÁFA törvény által előírt kötelező adattartalomra vonatkozik. Az interfész dokumentációt bővítjük egy olyan módszertani útmutatóval, amely segítségével ezeket az adatszolgáltatásokat termék és szolgáltatás alapján össze lehet vonni, miáltal a méret lecsökkenthető 10 Mb alá, a NAV számára releváns üzleti adat elvesztése nélkül. Az ilyen adatszolgáltatásokat a számlasorok felett egy új, mergedItemIndicator nevű kötelező jelölővel kell ellátni.
- 2021.01.01-től kötelező adatot szolgáltatni a közösségi export számlákról, illetve a nem ÁFA alanyok számára kiállított számlákról is. Ezeknek a fogadásához a customerInfo csomópont átalakul, ennek segítségével elkülöníthetők a belföldi, közösségi valamint harmadik országos számlák illetve a magánszemély vevőknek szóló számlák is. Szintén változás, hogy a 3.0-ban a magyar, közösségi és harmadik országos adószámok nem írhatók fel egymás mellé, kizárólag egyet lehet közülük megadni. (technikailag a korábbi sequence choice stuktúrává alakul) A magánszemélynek (ide nem értve az adószámos magánszemélyt, illetve az egyéni vállalkozót) kiállított számla adatszolgáltatása nem tartalmazhat név-és címadatokat, ezért a sémában ezen elemek opcionálissá váltak. A fenti szabálynak nem megfelelő adatszolgáltatásokat a rendszer blokkoló validációval elutasítja. (a nem magánszemélynek szóló adatszolgáltatásokon pedig továbbra is kötelező elem a név és a cím)
- Az exchangeRate tagban a 3.0-tól kezdődően megadható 0 értékű árfolyam, mert azon devizás ügyletek esetében amelyek felszámított adót nem tartalmaznak az árfolyam nem számítható ki helyesen. A rendszer WARNING-ot fog visszaadni azon esetekre, ahol az árfolyam = 0 de a számla pénznemében számított ÁFA összege bárhol nagyobb mint 0. Ezzel a módosítással egyidejűleg a korábbi ún. "6-os choice", hivatalosabb nevén VatRateType, amely az áthárított adó összegét (vagy az ÁFA mentesség okát, hatályon kívüliségét stb) tartalmazta kibővül egy 7-ik esettel, melynek a neve vatAmountMismatch. A vatAmountMismatch elemet akkor lehet megadni, amikor van adóalap, de nincs ÁFA összeg (vagy fordítva), például az ingyenes ügyletek esetében. Ugyancsak változás, hogy a vatRate csomópont megadható lesz egyszerűsített számlák esetén is.
- Az egyszerűsített számlák adótartalma mellett taxatíve megadhatók ugyan azok a speciális esetek is (mentesség, hatályon kívül, fordított adózás stb.) amelyek a normál és a gyűjtőszámlák esetében is használhatók. Az egységesítés miatt az egyszerűsített számlákban megadható adótartalom a VatRateType részévé válik, ezzel az elem "8-as choice" értékűre bővül. A rendszer blokkoló validációval el fogja utasítani azon adatszolgáltatásokat, amelyeknél a normál és gyűjtőszámlákban használt adómérték (vatPercentage) és az egyszerűsített számlákban használt adótartalom (vatContent) keveredik. A módosítás számlasor és számla összesítő szintjén is megjelenik.
- További változás a VatRateType esetében, hogy az ÁFA bevallások pontosításához az ÁFA mentesség(vatExemption) és a hatályon kívüliség (vatOutOfScope), valamint a vatAmountMismatch eseteiben az API külön kódot és indokolást fog elvárni. A kódot (pl: TAM, AAM stb) az interfész dokumentáció fogja tartalmazni és blokkoló validáció fogja kikényszeríteni a helyességét. Az indokolás pedig szabadon kitölthető ahogy a számlán az szerepel, a megadható hosszat pedig megduplázzuk.
- Az ÁFA bevallások készítéséhez szükség van az előlegszámla-végszámla kérdés rendezésére is. A 3.0-ás sémában sorszinten átalakul az előleg jelzés kezelése, és egy új, advanceData csomópontban lehetőség lesz megadni végszámla esetén - sorszinten - az előleget tartalmazó számla sorszámát, a teljesítés időpontját, valamint az alkalmazott árfolyamot. 
- A közszolgáltatói elszámolószámlákra vonatkozó speciális szabályok miatt a sémába bekerült egy új, utilitySettlementIndicator nevű opcionális jelölő. A tag működését az interfész dokumentáció fogja tartalmazni.
- Azért, hogy a számlák automatikus feldolgozása minél univerzálisabb lehessen, számlafej szintre bekerül egy conventionalInvoiceInfo nevű, míg számla sor szintre egy conventionalLineInfo nevű csomópont. A 3.0-ás séma ez alatt a piaci gyakorlat szerint használt leggyakoribb egyezményes egyéb adatokat tartalmazza (pl: megrendelés számok, szállítólevél számok, szerződésszámok, főkönyvi számlaszámok, vállalat kódok, költséghelyek, cikkszámok stb) amiknek az egyezményes névvel ellátása segítheti a gépi feldolgozás elterjedését.
- Figyelemmel arra, hogy a 2020.07.01 óta történt értékhatár eltörlés miatt az Online számla API egy olyan közös nyelv és kommunikációs platform lett, amelyet országosan minden jogszabályok szerint működő számlázó szoftvernek ismernie kell, a NAV informatikailag továbbviszi ezt a koncepciót. Az Online számla saját sémáiból kiemelésre kerülnek azon generikus, üzleti katalógus jellegű, valamint a kommunikációt leíró típusok, amelyek más projektben is felhasználhatóak és átkerülnek egy új common XSD-be. A common XSD-nek saját névtere van és külön is verziózzuk, ami okán a Githubon is külön projektben található. (https://github.com/nav-gov-hu/Common) A kiemelés rengeteg namespace változással és átnevezéssel is jár az Online számla sémáiban, azonban a kiemelés azt eredményezi, hogy más, jövőben készülő NAV-os XML API-hoz nem kell új technikai felhasználókat létrehozni az adózóknak, az Online számla alatt regisztrált technikai felhasználók ugyan azokkal az authentikációs adatokkal és kulcsokkal, ugyan azon alap XML szerkezetben, ugyan azon (vagy hasonló) kriptográfiai metódusokkal képesnek lesznek más projekt API-t is megszólítani. Példaként a folyamatban lévő e-ÁFA projekt XML API-ját lehetne felhozni, amely képes lesz ezen működés támogatására.
- Tekintettel arra, hogy a common XSD esetében az Online számla projekt sémái már olyan névteret is importálnak amely már nem a projekt része, bevezetésre kerül a catalog XSD mint technológia (https://www.oasis-open.org/committees/download.php/14809/xml-catalogs.html#s.using.catalogs). Ez azt eredményezi, hogy minden <xs:import> tag elveszíti a "schemaLocation" attribútumát, az importok helyét NAV által szerkesztett sémában a catalog határozza meg. A legtöbb XML processzor az importált sémákat ugyan azon a filepath-on keresi mint ahol a feldolgozandó séma definíció is van, ezért minden fejlesztőnek el kell dönteni, hogy vagy visszaírja a "schemaLocation" értékeket a NAV-tól letöltött sémába saját magánál, vagy catalogot használ. Mindkét megoldás elfogadható. A common XSD esetében a catalog használatát azért javasoljuk mindenkinek, mert ha már több saját projektben fogja a common-os sémát használni akkor elég lesz csak egy helyen frissíteni ha a fenti séma változik. A catalogra adunk template fájlt, amit fel lehet majd használni. A template-ben lesz uri name és publicId támogatás is, valamint működni fog lokálból és webes resource eléréssel is a Github repón keresztül.
- Rendezésre kerül az XSD hierarchia, mi által megszűnik az invoiceData elsődlegessége az importok tekintetében. Ezt úgy lehet elérni, hogy az Online számla rendszerre nézve több sémában felhasznált azon típusok, amelyek túl speciálisak ahhoz hogy a common XSD-be kerüljenek kikerülnek egy új, invoiceBase nevű sémába. Az Online számla rendszer 3.0-ás sémái (Api, Data, Annulment, Metrics) már csak a common-t és az invoiceBase sémát importálják, ez által a függési struktúra tisztul.
- A hosszútávú fenntarthatóság érdekében a commonban bevezetésre kerül a CryptoType nevű típus. A típus lehetővé teszi, hogy az API alatt úgy lehessen hash és kriptográfiai algoritmust váltani, hogy ahhoz a sémát nem kell megváltoztatni. A cryptoType nevű attribútumot minden hash értéket tartalmazó elemnél kötelező lesz feltüntetni (passwordHash, requestSignature, electronicInvoiceHash). Az attribútum használható értékeit az interfész dokumentáció fogja tartalmazni. A hash értéket tartalmazó elemek hosszát jelentősen megnöveltük azért, hogy a későbbi algoritmusok kimenetei is elférjenek bennük, a validációs patterneket pedig hasonló okból fellazítottuk. (már nem csak hexadecimális érték adható meg)
- Az API szolgáltatásait bővítjük a Githubon beérkezett javaslatok alapján, az adózó lekérdező válasza visszaadja a lekérdezett adószám típusát (cég vagy egyéni vállalkozó), illetve a tranzakció listázó válasza visszaadja a listában lévő tranzakciók feldolgozási státuszát, ennek segítségével API-n keresztül is ellenőrizhető, hogy van-e az adózónak még nem lekérdezett adatszolgáltatási tranzakciója.
- INFO szintre átkerülnek azok a WARNING-ok, ahol a javítás az adózó részéről esetleges módosítás során sem lehetséges.

## 1.2) Átállás, fejlesztői támogatás

- Úgy becsüljük, hogy a 3.0-ás XML API-t tartalmazó fejlesztés szeptember végére lesz elérhető a teszt környezeten, a hozzá kapcsolódó magyar nyelvű interfész dokumentációval együtt.
- Okulva a 2.0-ás átállás tapasztalataiból felkészítettük a belső feldolgozó rendszereinket arra, hogy egyszerre kaphatnak éles adatot a 2.0-ás és a 3.0-ás számla adatokkal is. Így a 3.0-ás XML API már nem csak teszt rendszeren, hanem rögtön az éles környezetben is fogadhat adatot. Ettől azt reméljük, hogy akit verziókezelési okokból blokkol ha a NAV API nem működik az éles környezetben, azok tudni fognak haladni az átállással. A hardveres környezet elkészül a 3.0 élesítésére, ezért azonos performanciával fogjuk tudni kiszolgálni mindkét verziójú API kéréseit.
- Régóta tervezett fejlesztésünk valósul meg az által, hogy most már az XML API-hoz online elérhető openapi dokumentációt és swagger UI-t tudunk biztosítani. A swagger URL hamarosan elérhető lesz a teszt és éles rendszereken, de a Github readme is tartalmazni fogja amint a fejlesztés kikerül a publikus környezetekre. Az API definíció elérhető lesz a 2.0-ra és a 3.0-ra is egyaránt. A swaggerben a try-out funkciót is aktiváljuk, így felületről, kézzel összerakott XML beküldését ki is lehet majd próbálni. Az openapi dokumentáció generálását CI tool végzi automatikusan a release részeként, ezért garantálni tudjuk hogy mindig naprakész információ lesz elérhető. A funkció élesedéséről külön hírt fogunk kitenni az Online számla felületen.
- Az interfész dokumentáció a Githubra is felkerül, ez által gyorsabban - és aki követi a projektet, az automatikusan is, email útján - értesülhet arról, ha a dokumentáció frissült.
- Továbbra is biztosítjuk a gyors fejlesztői támogatást azon ticketekre, amelyeket DEV supportként adtok fel számunkra a Githubon.
- A séma véleményezésére, XML API-val kapcsolatos javaslatok adására érdemben a NAV oldali fejlesztés ideje alatt van lehetőség, utána már (szeptember végén elindul a műszaki notifikáció) csak korlátozottan, ezért ha van még nem kezelt eset vagy fejlesztési kérés, akkor kérjük azokat mielőbb tudassátok velünk, minden észrevételt szívesen fogadunk!

FELHÍVJUK a figyelmet, hogy az Online számla felületről a 3.0-ás séma zipként, egyben lesz elérhető, de a Githubon a common XSD-t verziókezelési okok miatt nem tudjuk az Online számla projekt részévé tenni. Ezért aki Githubos forrás alapján akar dolgozni, annak külön kell letölteni a két sémát! (A tartalom természetesen ugyan az lesz, letöltési forrástól függetlenül.)

## 2) Egyéb Módosítások

## 2.1) Common és base sémaleíró (új sémák)

- A commonban új típusként megjelenik az AtomicStringType, a SimpleTextNotBlank típusok ebből öröklődnek. Hasonlóan új típus a GenericDecimalType, amelyből a lebegőpontos értékek származnak.
- Minden Online számla sémában (Api, Data, Annulment, Metrics) egységesen a primitív xs:string típusok kivezetésre kerültek. Ezek a típusok már a common:AtomicStringType megfelelő hosszúságú típusait használják. Hasonlóképp a xs:decimal típusok is változnak GenericDecimalType-ra.
- A commonban használt request struktúrákban nincsenek software adatok, mivel azok Online számlához köthető, specifikus típusok. Azonban a software adatok megadása a 3.0-ban is kötelezők, azokat az API sémaleíróban öröklődéssel a BasicOnlineInvoiceRequestType típus terjeszti ki.
- A commonban a headerVersion, requestVersion elemek régi, 2.0-ás típusai törlése kerültek. Az elemek új típusa AtomicString15, és nincsenek enumjaik. (nem biztos hogy minden NAV projekt úgy fog/akar verziózni, ahogy az Online számla teszi) A verzió értékeket az intefész dokumentáció fogja tartalmazni, és a helyes értékeket a rendszer ki fogja kényszeríteni. (lényegi változás nincs kliens oldalon, az ellenőrzés a sémából átkerül szolgáltatás szintre, ez azoknak fontos aki generált XML-t használ mert ott már nem lesz automatikusan requestVersion 3.0)
- A BasicResultType egy új opcionális, notification nevű válaszelemmel bővült. Ezt a NAV a jövőben egyéb, API hívásokban értelmezhető tájékoztató üzenetek közlésére fogja használni.
- A base XSD-ben az alábbi típusok nevei megváltoznak, mivel túlságosan általánosak:
DateType -> InvoiceDateType
TimestampType -> InvoiceTimestampType
IndexType -> InvoiceIndexType
UnboundedIndexType -> InvoiceUnboundedIndexType

### 2.2) API sémaleíró

- TaxPayerDataType új elemmel bővül: incorporation, amely megmondja hogy az adószám gazdasági társaság vagy egyéni vállalkozó-e
- TransactionType kibővül a tranzakció státuszával (requestStatus) és a technikai érvénytelenítés (technicalAnnulment) tényével
- A /queryInvoiceDigest operáció válasza visszaadja a completenessIndicator értékét
- A /queryInvoiceDigest operáció válaszában pontosításra kerültek az ÁFA csoport tagok adószámait tartalmazó elemek nevei: supplierGroupTaxNumber -> supplierGroupMemberTaxNumber, customerGroupTaxNumber -> customerGroupMemberTaxNumber. Ezen kívül a válasz opcionálisan tartalmazza a beküldött electronicInvoiceHash tag értékét is.

## 2.3) Annulment sémaleíró

- az AnnulmentCodeType típus kibővült egy új technikai érvénytelenítési okkal: ERRATIC_ELECTRONIC_HASH_VALUE

### 2.4) DATA sémaleíró

- data:RegNumType -> common:PlateNumberType, és új a patternje, az EKÁER-rel azonosan (ÖÜŐ is megengedett)
- ekaerId-t átmozgatásra kerül az új egyezményes (conventionalInvoiceInfo) adatok közé, ez által fej és tétel szinten is megadható

### 2.5) ERROR módosítások

- új ERROR: a requestVersion és (ha meg van adva akkor) a headerVersion tag értéke csak az interfész dokumentációban engedélyezett lehet
- új ERROR: a timestamp értéke nem lehet kisebb mint 2010-01-01T00:00:00Z
- új ERROR: ha az alapszámlában a completenessIndicator értéke false, akkor egy módosító okiraté sem lehet true (az API nem támogatja az ilyen irányba történő váltást a számlaláncon belül, de a másik irányba történő váltás megengedett)
- új ERROR: ha privatePersonIndicator értéke false, akkor a customerName ÉS a customerAddress megadása kötelező (B2B számla)
- új ERROR: ha privatePersonIndicator értéke true, akkor sem a customerName sem a customerAddress, sem a customerVatData nem adható meg (B2C számla)
- Új ERROR: ha a completenessIndicator értéke true, az invoiceAppearance értéke csak ELECTRONIC lehet
- új ERROR: az electronicInvoicehash megadása kötelező, ha a completenessIndicator értéke true
- új ERROR, ha az electronicInvoicehash értéke helytelen és a completenessIndicator értéke true
- Új ERROR: ha a completenessIndicator értéke true akkor a mergedItemIndicator értéke nem lehet true
- Új ERROR: ha a completenessIndicator értéke true akkor a vevő nem lehet magánszemély
- új ERROR: NORMAL vagy AGGREGATE számlának nem lehet lineAmountsSimplified sora vagy SIMPLIFIED számlának nem lehet lineAmountsNormal sora
- új ERROR: NORMAL vagy AGGREGATE számlában nem lehet vatContent értéket adni vagy SIMPLIFIED számlában nem lehet vatPercentage értéket adni a VatRateType típuson belül
- új ERROR: egy általános átmeneti hibakód is bekerül a rendszerbe, melyet első alkalommal a rendszer akkor fog visszaadni, amikor a elektronikus számlával egyenértékű 3.0-ás adatszolgáltatás érkezik 2021.01.01. előtt
- vatContent értéke már nem lehet 0, mivel a mentességi okok a VatRateType típuson belül taxatíve megadhatók

### 2.6) WARNING módosítások

- új WARNING: az exchangeRate értéke nem lehet 0, ha van bárhol (fej vagy sorszinten) olyan ÁFA összeg a számla pénznemében, aminek az értéke != 0
- új WARNING: ha az mergedItemIndicator a számlaláncon belül bárhol true lett, onnantól végig true értéket kell kapjon a további módosítások során
- INFO szintre történő átsorolások felülvizsgálata folyamatban van
- a meglévő WARNING logika felülvizsgálata folyamatban van, ma leglévő észrevételek mellett az új felvetéseket is várjuk a Githubon

### 2.7) Uppercase konverzió megszüntetése

A 3.0-ás adatszolgáltatások feldolgozása során minden korábbi nagybetűsítés megszűnik a rendszerben. Minden string típus úgy kerül mentésre ahogy az az adatszolgáltatásban beérkezett.

## 3) 3.0 átállási útmutató lépésről lépésre

### 3.1) API

#### 3.1.1) Kötelező API módosítások

- Minden hívott URL-ben vezesd át a főverzió változást, '/v2/' helyett '/v3/' legyen mindenhol.
- Minden root elementnél emeld 3.0-ra az API-s séma namespace értékét, illetve kösd be a common XSD-t. Egy lehetséges példa: 'xmlns="http://schemas.nav.gov.hu/OSA/3.0/api" xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common"'.
- Minden requestVersion tagban emeld a verzió értéket 3.0-ra.
- A header és user csomópontokban mindenhová (nyitó és zárótagekbe, illetve a gyermek tagekbe is) tedd bele a common XSD-re általad definiált namespace taget. Ha a példa szerinti ns-t használod, akkor ez a 'common:' lesz.
- A user/passwordHash tagba tedd bele a 'cryptoType="SHA2-512"' attribútumot.
- A user/requestSignature tagba tedd bele a 'cryptoType="SHA3-512"' attribútumot.

#### 3.1.2) Használatfüggő API módosítások

- Ha használsz DTO generálást a projektedben, akkor az új sémákkal a projekt már nem fog fordulni. Vagy használj catalog XSD-t a common és a base sémák importálására és ezt kösd be a projekthez, vagy írd be a letöltött sémákba a schemaLocation attribútumot olyan filepath értékkel, ami neked megfelelő. (egy 2.0 szerinti séma deklarációs részét meg tudod nézni, hogyan szerepelt ez az attribútum pontosan a sémában korábban, ha szükséged van rá) A common XSD projektet itt éred el: https://github.com/nav-gov-hu/Common
- Ha használsz response validációt a projektedben, akkor ha szükséges készülj fel arra, hogy az API üzleti válaszait nem minden esetben (pl token kérésnél az encodedExchangeToken, számlabeküldésnél a transactionId vagy a lekérdezéseknél a válasz adatok) a default, hanem esetlegesen más namespace alatt fogod visszakapni mint ahogy az korábban a 2.0 alatt történt. Ez a common XSD importálásának egyik következménye.
- Ha a programod riportálja az elektronikus számlák ellenőrző hash értékét, akkor ezt a 3.0-ban már nem belül a Data XML-ben, hanem kívül, az API XML-ben kell megadnod. Az ehhez szükséges tag neve változatlanul electronicInvoiceHash, de a helye már a ManageInvoiceRequest/invoiceOperations/invoiceOperation/electronicInvoiceHash Xpath útvonalon található. Ne feledd, hogy ehhez a taghoz is tartozik cryptoType attribútum. A hash típusa completenessIndicator=false esetben csak SHA3-512 vagy SHA-256 lehet, de a hash értéke ekkor nem történik szerver oldali validáció. 
- Ha használni szeretné a programod a 3.0 újdonsága szerinti elektronikus számlázást (completenessIndicator=true) akkor a cryptoType attribútumban a hash típusa csak SHA3-512 lehet, az értékét pedig az API XML-ben szereplő invoiceData objektumból kell kiszámolni, melynek helyességére a szerver validál.
- Ha a programod használja az adózó lekérdező /queryTaxpayer szolgáltatást, akkor készülj fel a válaszban az incorporation tag feldolgozására.
- Ha a programod használja a tranzakció listázó /queryTransactionList szolgáltatást, akkor készülj fel a válaszban a requestStatus és a technicalAnnulment tagek feldolgozására.
- Ha a programod használja a teljes adattartalmú számla lekérdező /queryInvoiceData szolgáltatást, akkor készülj fel hogy a válaszban már visszakaphatsz 3.0-ás számla XML-t is. Az ehhez kapcsolódó parsolási, üzleti, megjelenítési és egyéb program módosításokat kezeld le magadnál. Szintén készülj fel arra, hogy az operáció visszaadhatja az electronicInvoiceHash értékét is, amikor az ki volt töltve a beküldésnél.
- Ha a programod használja a kivonatos számla lekérdező /queryInvoiceDigest szolgáltatást, akkor készülj fel a válaszban a completenessIndicator tag feldolgozására, illetve készülj fel, hogy az eladó és a vevő csoportos adószámát már más néven fogod visszakapni. (ld: 2.2 fejezet)
- Javasoljuk, hogy készülj fel a válaszokban opcionálisan érkező notification tag értékének feldolgozására és megjelenítésére a felhasználók felé! A szerver még nem ad vissza a tagban értéket (a használatba vétel külön fejlesztés lesz), unit teszttel vagy debugban tudod kipróbálni hogy a program jól működik-e.
- Gondold végig, hogy az uppercase konverzió megszüntetése okoz-e bármiféle törést vagy nem várt változást a programodban (pl /queryInvoiceData és /queryInvoiceDigest válaszának változásai miatt) és ha szükséges akkor kezeld le őket!

### 3.2 Data 

#### 3.2.1) Kötelező Data módosítások

- az InvoiceData root elementnél emeld 3.0-ra az Data séma namespace értékét, illetve kösd be a Base XSD-t. Egy lehetséges példa: 'xmlns="http://schemas.nav.gov.hu/OSA/3.0/data" xmlns:base="http://schemas.nav.gov.hu/OSA/3.0/base"'
- A számla felső szintű adatainál meg kell adnod a completenessIndicator tag értékét, ez határozza meg hogy az adatszolgáltatás egyenértékű-e a kibocsátott elektronikus számlával. Az új tag helye az invoiceIssueDate tag után következik. Javasoljuk, hogy egyelőre mindenki használja default false értékkel, mivel 2021.01.01-ig a szerver nem fogadhat el adatszolgáltatásokat true értékkel. Az üzleti funkció használatát pedig javasoljuk valamilyen dinamikusan változtatható üzleti vagy konfigurációs paraméterhez kötni, hogy csak az aktiválásához ne kelljen külön kliens oldali release-t kiadni, amikor majd itt az idő.
- Három komplex adattípus esetében át kell vezetned a Base XSD namespace értékét a gyermek tagekben. Ezek a magyar adószámot leíró TaxNumberType (törzsszám, ÁFA kód, megyekód) illetve a címtípusok, függetlenül attól hogy egyszerűsített (SimpleAddressType) vagy tagolt (DetailedAddressType) címekről van-e szó. Ezt a módosítást minden olyan helyen át kell vezetned, ahol a felsorolt típusok a Data XML-ben előfordulhatnak. Figyelj rá, hogy ez nem olyan namespace módosítás mint ami az API-nál van, itt a szülő tag nem kaphat új namespace értéket! Tehát, amíg az API esetében így néz ki a helyes ns:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TokenExchangeRequest xmlns="http://schemas.nav.gov.hu/OSA/3.0/api" xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common">
	<common:header>
		<common:requestId>String</common:requestId>
		<common:timestamp>2020-09-06T12:09:24.901Z</common:timestamp>
		<common:requestVersion>3.0</common:requestVersion>
		<common:headerVersion>1.0</common:headerVersion>
	</common:header>
```
addig a Data esetében, például a kiállító adatainál már így (feltéve hogy a példa szerinti ns-t, a 'base:' értéket használod):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<InvoiceData xmlns="http://schemas.nav.gov.hu/OSA/3.0/data" xmlns:base="http://schemas.nav.gov.hu/OSA/3.0/base" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://schemas.nav.gov.hu/OSA/3.0/data invoiceData.xsd">
	<invoiceNumber>2021/1</invoiceNumber>
	<invoiceIssueDate>2020-09-06</invoiceIssueDate>
	<completenessIndicator>false</completenessIndicator>
	<invoiceMain>
		<invoice>
			<invoiceHead>
				<supplierInfo>
					<supplierTaxNumber>
						<base:taxpayerId>88888888</base:taxpayerId>
						<base:vatCode>5</base:vatCode>
						<base:countyCode>41</base:countyCode>
					</supplierTaxNumber>
					<groupMemberTaxNumber>
						<base:taxpayerId>44444444</base:taxpayerId>
						<base:vatCode>4</base:vatCode>
						<base:countyCode>42</base:countyCode>
					</groupMemberTaxNumber>
					<supplierName>Supplier Ltd.</supplierName>
					<supplierAddress>
						<base:simpleAddress>
							<base:countryCode>HU</base:countryCode>
							<base:postalCode>1031</base:postalCode>
							<base:city>Budapest</base:city>
							<base:additionalAddressDetail>Example street 1.</base:additionalAddressDetail>
						</base:simpleAddress>
					</supplierAddress>
				</supplierInfo>
```

Látható, hogy amíg az API-ban a header tag is új ns-t kapott (nem csak az alatta lévő gyermek tagek), addig a supplierTaxNumber és a supplierAddress tagek maradtak a default namespace alatt, csak a gyermek tagek kaptak új ns értéket.
- A vevő adatainak felírását már úgy kell kezdened, hogy a privatePersonIndicator tagban megadot hogy a számla vevője magánszemély-e. A tag helye sorrendben legelől van a customerInfo tagen belül. Javasoljuk, hogy ennek az információnak a bevitelére legyen UI beviteli funkció (amennyiben más üzleti logika alapján ez nem derül ki) és ezt a számlát készítő felhasználó adja meg, mert a vevő adószámának a hiánya nem feltétlen jelenti azt hogy a vevő egyben magánszemély is, nehéz az algoritmizáció. (pl egyéni vállalkozó vevők számlái)
- Minden nem magánszemélyes számlánál a vevői adószámot új Xpath alatt, a customerInfo/customerVatData alatt lehet megadni, ezt kezeld le magadnál. Figyelj rá, hogy nem lesz feltétlenül minden nem magánszemélyes számlánál adószám (pl. gazdasági tevékenységet nem folytató társasház, egyesület stb.)
- Építs kitöltési logikát a vevői adatok XML-ben történő szerepeltetésére:
	- ha a vevő nem magánszemély akkor
		- a magyar (plusz opcionálisan az ÁFA csoport) adószám (customerTaxNumber+groupMemberTaxNumber), a közösségi adószám (communityVatNumber) és a harmadik országos adószám (thirdStateTaxId) közül kizárólag egyet lehet megadni
	- ha a vevő magánszemély akkor
		- sem név (customerName), sem cím (customerAddress), sem pedig vevő adószámot (customerVatData) tartalmazó csomópont nem adható meg

FIGYELEM: a kitöltési logika azt jelenti, hogy magánszemély esetén a névnek és címadatnak a számlán rajta kell lennie, de az adatszolgáltatásban használt XML-ben már nem!
- Ha az adatszolgáltatás tartalmaz számlasort, akkor a mergedItemIndicator megadása kötelező. A tag helye még a sorok felsorolása előtti szinten (invoiceLines) található, sorrendben legelől.
- Ha a számlasor előleg adotokat tartalmaz, akkor azokat új Xpath alatt, a line/advanceData alatt adhatod meg. Figyelj rá, hogy ha van előleg adat (advanceIndicator=true) akkor az advancePaymentData csomópont alatt kötelező megadnod az előleg számla sorszámát (advanceOriginalInvoice), az előleg fizetés időpontját (advancePaymentDate) és az alkalmazott árfolyamot (advanceExchangeRate). A félreértések elkerülése érdekében hangsúlyozzuk, hogy az advanceOriginalInvoice tagba az előleget tartalmazó számla invoiceNumber értéket kell szerepeltetni, tehát ez nem számlasor szintű hivatkozás, hanem számla szintű!
- Figyelj rá, hogy nem egyszerűsített számla esetén (NORMAL és AGGREGATE típusokban) se számlasor, se számlaösszesítő szinten a VatRateType által leírt csomópontokban ne lehessen megadni a számlázóprogramban az egyszerűsített számlákban használt ÁFA tartalmat (vatContent), mert ez ilyen adatszolgáltatások ERROR miatt bukni fognak!

#### 3.2.1) Használatfüggő Data módosítások

- Ha használni szeretné a programod a 3.0 újdonsága szerinti elektronikus számlázást, akkor:
	- ne engedj olyan elektronikus (completenessIndicator=true) számlát (CREATE) vagy módosító okiratot (MODIFY, STORNO) kiállítani akkor, ha
		- a számla vagy módosító okirat magánszemélynek szól (privatePersonIndicator=true),
		- a számla vagy módosító okirat méretcsökkentés miatt összevont soradatokat tartalmaz (mergedItemIndicator=true),
		- a számla vagy módosító okirat megjelenési formája nem elektronikus (invoiceAppearance != ELECTRONIC)
		- a számla vagy módosító okirat kiállítási dátuma (invoiceIssueDate) 2021.01.01 előtti 
		- sysdate 2021.01.01 előtti vagy az erre a célra létrehozott üzleti paraméter állása a funkció használatát nem engedi
	- fenti szabályokon felül ne engedj olyan elektronikus (completenessIndicator=true) módosító okiratot kiállítani (MODIFY, STORNO), amelynek az alapszámlája létezik a rendszerben (modifyWithoutMaster=false) és abban a completenessIndicator tag értéke false
	- az API XML-ben tüntesd fel az electronicInvoiceHash taget helyes cryptoType és hash értékkel, az interfész dokumentáció szerint
Ha a fenti szabályokat a programod nem tartja be, akkor az adatszolgáltatás ERROR miatt bukni fog, ami jelen esetben meghiúsítja az elektronikus számla vagy a módosító okirat kibocsátását is!
- Ha a programod kezel ÁFA mentes (vatExemption) és ÁFA hatályon kívüli (vatOutOfScope) számlákat, akkor a különböző esetekre az interfész dokumentáció által meghatározott értékkészlet alapján kínálj fel listás beviteli lehetőséget a UI-on, vagy a felhasználó által transzparensen feleltesd meg a saját értékeid ennek az értékkészletnek, ha erre lehetőséged van. Az enumerált értéket a case tagban, a felhasználó által a számlára felvitt értéket pedig a reason tagba helyezd el az XML-en belül mind számlasor, mind számlaösszesítő szinten. (értékkészlethez ld: PDF 126. oldal)
- Ha a programod kezeli az adóalap és a felszámított adó eltérésének - interfész dokumentáció szerint felsorolt - eseteit, akkor gondoskodj a vatAmountMismatch tag megfelelő töltéséről mind számlasor, mind számlaösszesítő szinten. Amennyiben az átváltási árfolyam ezen esetek miatt nem számítható ki, akkor az exchangeRate tag értékét állítsd 0-ra. (értékkészlethez ld: PDF 127. oldal)
- Ha a programod kezel egyszerűsített számlákat, akkor az ÁFA tartalmat (vatContent) számla sor szinten már új Xpath alatt, a lineAmountsSimplified/lineVatRate alatt tudod megadni. A módosítás a számlaösszesítőben is megjelenik, itt az új útvonal a summarySimplified/vatRate alatt található. Hasonlóképp itt tudod megadni az összes olyan mentességet, amelyeket eddig csak nem egyszerűsített számlánál lehetett megadni. A mentességek helyes kezelése azért fontos, mert a 2.0-ig a vatContent lehetett 0, azonban a 3.0-ban ez a lehetőség már megszűnik, az új ellenőrzés már ERROR-t ad erre az esetre! Az említett módosításokat vezesd át magadnál. Figyelj rá, hogy a programodban egyszerűsített számla esetében adó mértéket (vatPercentage) sem számlasor, szem számlaösszesítő szinten ne lehessen megadni, mert ez ilyen adatszolgáltatások ERROR miatt bukni fognak!
- Ha a programod töltötte számlasor szinten a tételhez tartozó EKÁER számokat, akkor ezt a taget már új Xpath alatt, a line/conventionalInvoiceInfo találod meg. EKÁER szám a 3.0-tól megadható számlafej szinten is, nem csak számlasorban.
- Ha az ügyfeleidnél van igény az adatmodellben megjelenő új, ügylethez tartozó azonosítók töltésére akár kiállítói, akár vevői oldalról (megrendelésszám, szerződésszám, szállítólevélszám stb) akkor ezekre az adatokra biztosíts a UI-on bevitelt, illetve helyezd el a bevitt adatokat az XML-ben számlafej vagy számlasor szintjén a /conventionalInvoiceInfo csomópont alatt. Minden tag kardinalitása korlátlan, tehát bármiből bárhány adat felírható.
- Ha a programod kezeli azt az esetet, amikor az adatszolgáltatás túl nagy méretű (HTTP content-length >= 10.485.760 bájt), akkor az interfész dokumentáció által meghatározott logika szerint vond össze a számlasorokat tétel-szolgáltatás szinten és aggregáld a megfelelő számszaki értékeket! Az összevonás tényét az adatszolgáltatásban jelezd a számlasor szint felett, a mergedItemIndicator=true kifejezéssel. Figyelj rá, hogy ha ez a tag a számlaláncban bárhol true lett, az onnantól következő módosításokban (következő modificationIndexek alatt) már sosem lehet false! (ellenkező esetben az adatszolgáltatás WARNING-ot kap) Felhívjuk a figyelmet, hogy nem felel meg a törvényi szabályozásnak az a megoldás, ami ezeket a számla adatszolgáltatásokat kiemeli kézi feldolgozásba, mindenképpen a számlázóprogramnak kell az összevonási logikát implementálni.
- Ha a programod kezel közüzemi elszámolószámlákat, akkor kezeld a utilitySettlementIndicator tag megfelelő értékadását.

### 3.3 Annulment 

#### 3.3.1) Kötelező Annulment módosítások

N/A

#### 3.3.1) Használatfüggő Annulment módosítások

- Ha a programod használja a technikai érvénytelenítési funkcióját az API-nak, akkor a belső XML-ben az InvoiceAnnulment root elementnél javítsd a namespace értékét: 'xmlns="http://schemas.nav.gov.hu/OSA/3.0/annul"'
- Ha a programod riportálja az elektronikus számlák ellenőrző hash értékét, akkor helytelen hash érték esetén már nincs lehetőség módosítani az adatszolgáltatást, technikai érvénytelenítést kell kezdeményezni. Ehhez a használati esethez új annulmentCode érték tartozik, ezért a technikai érvénytelenítő kérésbe az 'ERRATIC_ELECTRONIC_HASH_VALUE' értéket tedd. Fontos kihangsúlyozni, hogy ez a kód csak completenessIndicator=false esetén használható, jelen bejegyzés is csak ezzel a logikai esettel foglalkozik. (completenessIndicator=true esetén nincs technikai érvénytelenítési lehetőség, noha az is igaz, hogy ott helytelen hash érték sem lehet, mert a szerver a hibás hashel érkező ilyen adatszolgáltatásokat elutasítja)

### 3.4 ServiceMetrics 

#### 3.4.1) Kötelező ServiceMetrics módosítások

N/A

#### 3.4.1) Használatfüggő ServiceMetrics módosítások

- Ha a programod használja a metrika lekérdező funkcióit az API-nak, akkor ld: 3.1.2 fejezet DTO generálással és response validációval foglalkozó bejegyzések.
--------------------------------------------------------------------------------------------------------------------------------------------

# Changelog 3.0

Comprehensive changes from 2.0 to 3.0.

## 1) Overview of reasons for changes

Similarly to version 2.0, change requests of multiple kinds have been incorporated. First, the system’s API shall be capable of receiving EU invoices, invoices for export transactions and invoices issued to natural persons from 01.01.2021. On the other hand, the Online Invoicing project reached a phase where no new versions (compared to version 3.0) of the XML API are scheduled for release according to our plans, except for legislation changes and for eventual significant future demand. Even if changes will be made, those may be incorporated in a minor version instead of a major one. For all of these reasons, sustainability and other relevant aspects have also been considered in this release. We have, therefore, tried to eliminate all business and technical issues found in the API, and find solutions satisfying both parties. At last, but not at least, data missing for creating more accurate VAT return report drafts can now be identified and provided.

### 1.1) Planned solutions

- A cornerstone of v3.0 of the API is to catalyse electronic invoicing and it’s usability. For this reason, you’ll be able to specify the electronicInvoiceHash element in the API XML to contain the hash fingerprint of an e-invoice. This change only ensures the separation of business data and fingerprints during invoice creation and amendment. The change incorporated the decision to not support use cases when corrective data exchange (amendment) is received solely for the purpose of correcting the fingerprint of the original invoice, and therefore no feature is provided to do so. Data exchange treated deficient for a wrong fingerprint can only be corrected in the form of technical invalidation (annulment). The Annulment schema has been extended with a new annulment reason to support this.
- Another key change in the support of e-invoicing is the new required flag, completenessIndicator, added to the schema. You can use completenessIndicator to declare the e-invoice itself will be submitted as the means of performing v3.0 data exchange based on the sole decision of the taxpayer, using the XML format based on the schema and containing all data. Purchasing parties can then receive and process invoices in their books right after receiving the data exchange, without the need for further invoice delivery and later manual or automated comparison of supplied data and invoice data. Legislation will be adjusted to accommodate this change, and NTCA will enforce the integrity of data in such data exchange scenarios. The electronicInvoiceHash element for such invoices shall be computed as specified in the interface documentation, and the fingerprint shall be correct. Furthermore, you will not be able to perform technical annulment on such instances of data exchange, because the contents of the data exchange will practically form and incarnate the invoice itself, and therefore the two shall be identical. NAV imposes certain restrictions on invoice chains formed of such invoices (see section ERROR changes). The feature can only be used from 01.01.2021. Attempts to use it sooner will result in the rejection of the processing request in the company of a temporary error code.
- The issue of large-sized data exchanges with a POST body size larger than 10MB was practically left unaddressed since the beginning of the project. The issue typically affects a small amount of invoices issued by invoicing parties required to create very detailed invoices to conform to industry legislation, such as telecommunication and public utility service providers. The VAT act, however, is far from expecting such detailed invoice data, and the scope of data exchange obligation only includes data classified as mandatory in the VAT act. The interface documentation have been extended with a methodology guide to help you consolidate such data exchange on the basis of products and services sold in order to create a package for NAV that is less than 10MB, without losing any relevant business data. Data exchanges of this kind shall be indicated using the mandatory mergedItemIndicator flag inserted above the invoice item lines.
- Effective from 01.01.2021, data is required to be supplied about EU export invoices, and invoices issued to parties not considered VAT tax payers. In order to accommodate this, the customerInfo node has been changed to allow separation of invoices submitted to domestic parties, EU parties and parties in other foreign countries, as well as invoices issued to natural persons. Just another change in v3.0 is that you cannot display all of the Hungarian VAT ID, the EU VAT ID and the VAT ID from another country’s system next to each other, instead you can only pick one. At implementation level, the previously used sequence element is converted to a choice construct. Data exchange about invoices issued to natural persons (not including natural persons holding a VAT ID and individual entrepreneurs) shall not contain name and address data, and therefore the schema elements to hold such data have been made optional. Data exchanges not conforming to this rule are rejected in the form of a blocking validation. Note that name and address are still mandatory for invoices issued to others than natural persons.
- An exchange rate of 0 can be specified in the exchangeRate tag from version 3.0, since the exchange rate for transactions in foreign currency without incorporated tax cannot be calculated properly. You will receive a WARNING if the exchange rate is 0, and the VAT amount expressed in the currency of the invoice is greater than 0 anywhere. In concert with this change, VatRateType, the choice with 6 options and containing the amount of shifted taxes (or the reason of exemption or out-of-scope status) has been extended with a 7th option, called vatAmountMismatch. You can specify the vatAmountMismatch element when the transaction contains a tax base, however there’s no VAT amount (or vice versa), such as in case of free transactions. Yet another change is that you can add the vatRate node to simplified invoices, too.
- In addition to the tax amount for simplified invoices, you can also specify all the special cases (exemption, out of scope, reverse charge and so on) that you can specify for normal and summary invoices. As a consequence of this homogenization effort, the tax amount you can specify for simplified invoices becomes part of VatRateType, so it becomes a choice of 8 options. The system will apply blocking validation to reject data exchange attempts containing a mix of the VAT rate (vatPercentage) used in normal and aggregate invoices, and the VAT amount used in simplified invoices (vatContent). The change has been implemented at item and invoice summary levels as well.
- Another change affecting VatRateType is that the API expects a separate code and reason for vatExemption, vatOutOfScope and vatAmountMismatch to refine VAT return reports. The documentation of the interface discloses the codes (such as TAM for exempt by transaction, or AAM for not subject to VAT), and will be enforced using blocking validation. The reason is just a free text, and you can use the same text that is shown on the invoice. The length of the attribute have been doubled.
- The issues about the relation of advance payment invoices and final invoices had to be settled to be able to create VAT returns. Handling of advance payments have been restructured at item level in version 3.0 of the schema, and you can use a new item-level node in final invoices, called advanceData, to include the number of the corresponding advance payment invoice, the date of performance and the exchange rate used. 
- A new, optional flag named utilitySettlementIndicator has been added to the schema to respond to special rules governing public utility settlement invoices. Please see the interface documentation for the rules of usage.
- In an effort to unify automated processing of invoices as much as possible, a node named conventionalInvoiceInfo to invoice level, and a node named conventionalLineInfo to invoice line level has been added. Version 3.0 of the schema supports most frequently used common data, such as PO numbers, delivery note numbers, agreement IDs, G/L account codes, company codes, cost centre codes, item numbers etc. in the faith that giving a common and conventional name to such data can help spreading automated processing.
- With respect to that after abolishing the VAT value limit for invoice riporting with the effect of 01.07.2020, the Online Invoicing API became a common language and communication platform that all invoicing solutions across the country conforming to legislation have to support, NTCA takes this IT concept further. Generic types, as well as types of a business catalogue nature and those describing communications which can be used in other projects have been extracted and refactored into a new common XSD from the schemas of Online Invoicing. The common XSD received its own namespace and versioning schema, and is therefore stored in a separate GitHub project at https://github.com/nav-gov-hu/Common. Extraction came at the cost of a substantial amount of namespace changes, however taxpayers will not need to create new technical users for future NTCA XML APIs in exchange, since the technical users registered in Online Invoicing will be able to use the same authentication data and keys, the same base XML structure and the same or similar cryptographic methods to invoke services of APIs of other projects. A great example is the XML API of the ongoing e-VAT project, which will support this platform and mechanism.
- Since schemas of the Online Invoicing project import a namespace to support the common XSD that is no longer part of the project, the catalogue XSD technology (https://www.oasis-open.org/committees/download.php/14809/xml-catalogs.html#s.using.catalogs) has been implemented. This causes all <xs:import> tags to lose their schemaLocation attributes, and the catalogue defines the location of imports in the schema maintained by NTCA. Most XML processors look for imported schemas in the same file path where the schema definition to process is stored, therefore all developer parties shall decide to either reinsert the schemaLocation tag to the schema downloaded from NTCA, or switch to using the catalogue. Both solutions are adequate. Using the catalogue for the common XSD is generally recommended for the reason that once you use the common schema in multiple projects of yours, you only need to make updates in one location in response to schema changes. A template file is provided for the catalogue for convenience. The template will support URI name and publicId, too, and you will be able to use it with local and web resource access as well, via the GitHub repo.
- The XSD hierarchy has been cleared, and invoiceData is no longer preferred in the sequence of imports. To achieve this, types used in multiple schemas of the Online Invoicing System, yet too specific to be included in the common XSD, are extracted to a new schema named invoiceBase. Version 3.0 schemas of the Online Invoicing System (API, Data, Annulment, Metrics) all import the common XSD and the invoiceBase schema only, clearing the dependency hierarchy.
- A new type named CryptoType has been implemented for long-term sustainability in the common XSD. This enables changing hash and cryptographic algorithms for the API without the need to change the schema. The cryptoType attribute is mandatory for elements containing hash values. These include passwordHash, requestSignature and electronicInvoiceHash. The set of values valid for the attribute are disclosed in the interface documentation. The max length of elements containing hash values have been extended significantly to accommodate room for the values generated by future algorithms, and the validation patterns have been weakened for the same reason. From now on, not only hexadecimal values can be specified.
- The set and features of API services have been improved in response to feature requests received on GitHub: the response to taxpayer query contains the type of queried VAT ID (business or private entrepreneur); furthermore the response to getting the list of transactions contains the processing status of enlisted transactions, so you can use the API to check if the taxpayer has any data exchange transaction not yet queried.
- The level of severity have been changed from WARNING to INFO for all validation errors which cannot be corrected by the taxpayer, even by submitting amendments.

## 1.2) Transition and developer support

- Developments containing the XML API 3.0 are expected to be ready by the end of September in the test environment, in the company of the interface documentation in Hungarian.
- Based on our experiences with upgrading to 2.0, our internal processing systems have been prepared to receive production data using v2.0 and v3.0 format in parallel. For this reason, XML API 3.0 can not only receive data in the test environment, but in the production environment, too, immediately. It is expected that parties who would otherwise be blocked by versioning issues if the NTCA API would not work in production environment can still proceed with their upgrade processes. The hardware environment will be ready by the time 3.0 is deployed, therefore requests conforming to either version of the API will be served at the same performance.
- A long-desired development has come to fruition since online OpenAPI documentation and Swagger UI are both provided for the XML API. Swagger’s URL will soon be disclosed in the test and the production environments and, in addition, GitHub’s Readme will also publish it once the development is released to the public. The API definition will be available for both 2.0 and 3.0. Swagger’s try-out feature will be activated, so you will be able to try to submit XML payloads assembled manually on the user interface. OpenAPI documentation is generated automatically by our CI tool as part of the release process, making availability of up-to-date information guaranteed. Go-live of this feature will be announced on Online Invoicing’s website.
- Interface documentation will also be published on GitHub, so people, especially those tracking the project, can receive email notifications more quickly and automatically about updates to the documentation.
- Rapid developer support is kept to be continued for tickets submitted to DEV support on GitHub.
- You have the option to review the schema and provide feedback for improving the XML API until NTCA finishes this development phase. Afterwards (with the technical notification scheduled to the end of September) such options become really limited, so we kindly ask you to let us know as soon as possible if you find any unhandled cases or unaddressed feature requests. We appreciate all of your feedbacks.

NOTE that version 3.0 of the schema will be published on the website of Online Invoicing as a single ZIP file. For versioning issues, however, we cannot add the common XSD to the Online Invoicing project on GitHub. Those who would like to use the GitHub source need to download both schemas separately. (The contents of the schemas are identical, obviously, independently of the download source.)

## 2) Other changes

## 2.1) Common and base schema definitions (new schemas)

- A new type, AtomicStringType, is implemented in the common XSD, and SimpleTextNotBlank types are inherited from it. GenericDecimalType is another new type of the similar pattern for floating point types.
- Primitive xs:string types have been retired uniformly in all Online Invoicing schemas (API, Data, Annulment, Metrics), and have been replaced by the subtypes of the corresponding length variant of the common:AtomicStringType type family. Similarly, xs:decimal types have been changed to GenericDecimalType.
- Request structures used in the common schema do not contain software information, since such information is considered specific to Online Invoicing. Specifying software information is still required in version 3.0 as well, and are implemented in the BasicOnlineInvoiceRequestType subtype in the API schema definition.
- Legacy 2.0 types of the headerVersion and requestVersion elements have been deleted from the common schema. The new type of these elements is AtomicString15, and these have no enums defined (because other NTCA projects may want to employ a different versioning scheme than that used by Online Invoicing). Version values can be found in the interface documentation, and valid values will be enforced by the system. (There’s no material change on the client side with regards to this, only that validation is relocated from the schema to the service layer. This is only important for those using generated XML payload, since it will not automatically contain 3.0 as the value of requestVersion.)
- The type BasicResultType has been extended with a new optional response element, ‘notification’. NTCA will use this to convey informational messages via API calls in the future.
- The following type names have been changed to make them less generic:
DateType -> InvoiceDateType
TimestampType -> InvoiceTimestampType
IndexType -> InvoiceIndexType
UnboundedIndexType -> InvoiceUnboundedIndexType

### 2.2) API schema definition

- A new element, incorporation, has been added to TaxPayerDataType to tell whether the VAT ID belongs to a business or a private entrepreneur.
- TransactionType have been extended by information on transaction status (requestStatus) and the indication of technical annulment (technicalAnnulment).
- From now on, the /queryInvoiceDigest operation returns the value of completenessIndicator.
- Names of elements containing VAT IDs of tax group members in the response of the /queryInvoiceDigest operation have been refined: supplierGroupTaxNumber -> supplierGroupMemberTaxNumber, customerGroupTaxNumber -> customerGroupMemberTaxNumber. The response may optionally include the value of the electronicInvoiceHash tag submitted, too.

## 2.3) Annulment schema definition

- A new technical annulment reason has been added to the AnnulmentCodeType type: ERRATIC_ELECTRONIC_HASH_VALUE

### 2.4) DATA schema definition

- As a result of switching from data:RegNumType to common:PlateNumberType and of using the according new pattern, you can also use ÖÜŐ characters as well, just as in EKÁER.
- ekaerId has been moved to the common information section (conventionalInvoiceInfo), and therefore you can specify it at both header and item level.

### 2.5) ERROR changes

- New ERROR: The value of requestVersion and headerVersion (if specified) tags shall fit the range disclosed in the interface documentation.
- New ERROR: Timestamps shall not be less than 2010-01-01T00:00:00Z.
- New ERROR: If the value of completenessIndicator is false in a base invoice, it cannot be true in amending documents (since the API does not support this kind of change within an invoice chain, however it is supported the other way around).
- New ERROR: If privatePersonIndicator is false, customerName AND customerAddress becomes required (B2B invoices).
- New ERROR: If privatePersonIndicator is true, you are not allowed to specify customerName, customerAddress and customerVatData (B2C invoices).
- New ERROR: If completenessIndicator is set to true, the only valid value for invoiceAppearance is ELECTRONIC.
- New ERROR: Specifying a value for electronicInvoicehash becomes required if completenessIndicator is set to true.
- New ERROR: If electronicInvoicehash has an invalid value, and completenessIndicator is set to true.
- New ERROR: If completenessIndicator is set to true, mergedItemIndicator shall not be true.
- New ERROR: If completenessIndicator is set to true, the purchasing party shall not be a natural person.
- New ERROR: NORMAL and AGGREGATE (consolidated) invoices shall not contain lines of the lineAmountsSimplified type, just as SIMPLIFIED invoices shall not contain lines of the lineAmountsNormal type.
- New ERROR: You shall not specify a vatContent value for NORMAL and AGGREGATE (consolidated) invoices, just as you shall not specify a vatPercentage value within the VatRateType type for SIMPLIFIED invoices.
- New ERROR: A new generic temporary error code has been added to be returned the first time when a v3.0 data exchange equivalent to an electronic invoice is received before 01.01.2021.
- The value of vatContent must not be 0 from now on, since exemption reasons can be specified taxonomically using VatRateType.

### 2.6) WARNING changes

- New WARNING: The value of exchangeRate shall not be 0 if there’s a VAT amount anywhere in the invoice (either in the header or at item level) with a non-zero value, expressed in the currency of the invoice.
- New WARNING: If mergedItemIndicator has been set to true anywhere in an invoice chain, it shall keep this value from that point on during later amendments.
- Moving some items down to INFO level is in progress.
- Review of the current WARNING logic is in progress. We encourage you to submit your feedbacks and suggestions on GitHub in the course of this review process.

### 2.7) Retiring uppercase conversion

No data is converted to uppercase while processing data supplies conforming to version 3.0. String values are persisted in the exact form they had in data exchange records.

## 3) Step by step developer guide for v3.0

### 3.1) API

#### 3.1.1) Mandatory changes related to the API schema

- Increase the version tag in all invoked URLs. Make sure you replace all instances of ‘/v2/’ with ‘/v3/’.
- Increase the version number in the namespace the API schema reference to 3.0 in all root elements, and make a reference to the common XSD. Example: 'xmlns="http://schemas.nav.gov.hu/OSA/3.0/api" xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common"'.
- Increase the value of ‘version’ to 3.0 in all requestVersion tags.
- Include the namespace tag you defined for the common XSD in all opening, closing and child tags of the header and user nodes. If you use the example namespace we provided, it will be 'common:'.
- Include the 'cryptoType="SHA2-512"' attribute in the user/passwordHash tag.
- Include the 'cryptoType="SHA3-512"' attribute in the user/requestSignature tag.

#### 3.1.2) Scenario-specific changes related to the API schema

- If you employ DTO generation in your project, you will not be able to compile/build it with the new schemas. You can either use catalog XSD to import the common and the base schemas, and reference them in your project; or re-insert a schemaLocation attribute to the downloaded schemas with a filepath value of your choice. (You can simply view the declaration of a v2.0 schema to learn how the attribute was used previously.) You can access the common XSD’s project on page https://github.com/nav-gov-hu/Common.
- If you validate responses in your project, be prepared, if necessary, that business responses from the API will not always be placed in the default namespace, since it may be placed in a different one than in v2.0. For example, encodedExchangeToken for tokenExchange, transactionId for manageInvoice response and other query results will be returned in a different namespace. This is a consequence of importing the common XSD.
- If your software reports the verification hash of the electronic invoices, you have to specify it in API XML instead of Data XML in v3.0. The name of the tag is still electronicInvoiceHash, however it is now located on XPath ManageInvoiceRequest/invoiceOperations/invoiceOperation/electronicInvoiceHash. Do not forget that even this tag has a cryptoType attribute. When completenessIndicator is false, the hash method needs to be SHA3-512 or SHA-256. Hovewer, there is no server side validation for the hash value in this case.
- If you want your software to use the new v3.0 electronic invoicing feature set completenessIndicator to true, provide SHA3-512 for the method of cryptoType and calculate the correct hash value based on the input of the invoiceData object found in the API XML. The server validates the correct hash value in this case.
- If your software uses the /queryTaxpayer service to retrieve taxpayer information, prepare it for processing the ‘incorporation’ tag contained in the response.
- If your software uses the /queryTransactionList service to list transactions, prepare it for processing the requestStatus and the technicalAnnulment tags contained in the response.
- If your software uses the /queryInvoiceData service to retrieve complete invoice data, prepare it to accept v3.0 invoice XML payloads in the responses. Be sure to properly adjust parser logic, business logic, presentation and other components in your software. Be also prepared that the operation may return the value of the electronicInvoiceHash, if it was specified in the submission.
- If your software uses the /queryInvoiceDigest service to get key invoice information, prepare it for processing the completenessIndicator tag contained in the response. Be also prepared that the group VAT ID of sellers and customers will be returned under new names (see section 2.2).
- It is recommended to prepare your software to process and display the value of the ‘notification’ tag optionally included in responses. The server will not return any values in the tag yet (populating it with values is subject to a different development effort), so you may want to use unit tests or debug mode to check the soundness of your software.
- Examine whether retiring uppercase conversion causes any break or unexpected change in your software (for the changes of the responses of the /queryInvoiceData and /queryInvoiceDigest services, for example), and adjust your software if necessary.

### 3.2 Data 

#### 3.2.1) Mandatory changes related to the Data schema

- Increase the version number in the namespace of the Data schema reference to 3.0 in root element InvoiceData, and reference the base XSD. Example: 'xmlns="http://schemas.nav.gov.hu/OSA/3.0/data" xmlns:base="http://schemas.nav.gov.hu/OSA/3.0/base"'.
- Top level invoice data shall include the value of the completenessIndicator tag, which decides whether a data exchange instance is equivalent to an issued electronic invoice or not. You have to specify the new tag after the invoiceIssueDate tag. It is recommended to specify the default false value, since the server shall not accept data exchange submissions with this tag set to true before 01.01.2021. It is recommended to govern the behaviour of the related business logic via a business or configuration parameter that can be changed dynamically so that you do not need to publish a new version of the client just for activating this feature when time is up.
- You have to implement the changes made to the value of the base XSD namespace in the child tags for three complex data types. This includes TaxNumberType (tax number, VAT code, county code) describing Hungarian VAT numbers, as well as address types, namely SimpleAddressType for simple addressing and DetailedAddressType for more structured address information. Make sure to implement these changes in every location where these types may occur in the Data XML. Be warned that this change to the namespace is different from that made to the API schema, since the parent tag in this case is not allowed to get a new namespace value. The sound namespace for the API:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<TokenExchangeRequest xmlns="http://schemas.nav.gov.hu/OSA/3.0/api" xmlns:common="http://schemas.nav.gov.hu/NTCA/1.0/common">
	<common:header>
		<common:requestId>String</common:requestId>
		<common:timestamp>2020-09-06T12:09:24.901Z</common:timestamp>
		<common:requestVersion>3.0</common:requestVersion>
		<common:headerVersion>1.0</common:headerVersion>
	</common:header>
```
For the Data schema, you have to specify supplier data like this (provided that you use the namespace name from the example, 'base:'):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<InvoiceData xmlns="http://schemas.nav.gov.hu/OSA/3.0/data" xmlns:base="http://schemas.nav.gov.hu/OSA/3.0/base" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://schemas.nav.gov.hu/OSA/3.0/data invoiceData.xsd">
	<invoiceNumber>2021/1</invoiceNumber>
	<invoiceIssueDate>2020-09-06</invoiceIssueDate>
	<completenessIndicator>false</completenessIndicator>
	<invoiceMain>
		<invoice>
			<invoiceHead>
				<supplierInfo>
					<supplierTaxNumber>
						<base:taxpayerId>88888888</base:taxpayerId>
						<base:vatCode>5</base:vatCode>
						<base:countyCode>41</base:countyCode>
					</supplierTaxNumber>
					<groupMemberTaxNumber>
						<base:taxpayerId>44444444</base:taxpayerId>
						<base:vatCode>4</base:vatCode>
						<base:countyCode>42</base:countyCode>
					</groupMemberTaxNumber>
					<supplierName>Supplier Ltd.</supplierName>
					<supplierAddress>
						<base:simpleAddress>
							<base:countryCode>HU</base:countryCode>
							<base:postalCode>1031</base:postalCode>
							<base:city>Budapest</base:city>
							<base:additionalAddressDetail>Example street 1.</base:additionalAddressDetail>
						</base:simpleAddress>
					</supplierAddress>
				</supplierInfo>
```

While the header tag also received a new namespace in addition to its children in case of the API schema, the supplierTaxNumber and the supplierAddress tag remained in the default namespace, and the new namespace was only added to their child tags.
- When you provide customer data you need to start by specifying in the privatePersonIndicator tag whether the customer in the invoice is a private person or not. This tag shall come first within the customerInfo tag. Unless this information can be computed using some business logic, it is recommended to add an option for the users to mandatorily specify it on the UI, since the absence of the customer’s VAT ID on an invoice does not necessarily imply that the customer is a private person (just think of invoices issued to private entrepreneurs). This is hard to tell apart algorithmically.
- For invoices issued to others than private persons the customer VAT ID shall be specified on a new XPath, customerInfo/customerVatData. You have to prepare your software for this. Do not forget that some invoices will not include a VAT ID even if the particular invoice is issued to someone else than a private person (such as to a block of flats or an association not engaged in any economical or business activity).
- Add business rules to properly specify and validate customer data in XML:
	- if the customer is not a private person
		- only one of the following VAT IDs can be specified: Hungarian VAT ID (optionally including the VAT group, customerTaxNumber+groupMemberTaxNumber), the EU VAT ID (communityVatNumber) and the VAT ID of other countries (thirdStateTaxId)
	- if the customer is a private person
		- you cannot specify any nodes containing a name (customerName), an address (customerAddress) or a customer VAT ID (customerVatData)

WARNING: According to the business rules, name and address information shall be included in the invoice for private persons, however such data must not be included in the XML submitted as the means of data exchange.
- If the data exchange payload includes an invoice item, you must specify mergedItemIndicator. The tag shall be placed in front of item data (invoiceLines), and it must be the first one.
- If the invoice contains advance payment information, you have to specify such information on a new XPath, line/advanceData. In the presence of advance payment information (advanceIndicator=true), you must specify the number of the original invoice for the advance payment (advanceOriginalInvoice), the date of the advance payment (advancePaymentDate) and the exchange rate used (advanceExchangeRate) under node advancePaymentData. Pay attention that you must include the value of invoiceNumber of the invoice containing the advance payment in the advanceOriginalInvoice tag, that is, this reference is invoice-level information, not an item-level one.
- Keep in mind that your software shall not allow specifying VAT amount (vatContent) used in simplified invoices on invoice or item level (i.e., in nodes described by VatRateType) in invoices of other types (NORMAL and AGGREGATE types), or your data exchange will fail for an ERROR.

#### 3.2.1) Scenario-specific changes related to the Data schema

- If you want your software to use the new v3.0 electronic invoicing feature, consider the followings:
	- do not allow issuing (CREATE operation) electronic invoices (with completenessIndicator=true) or corrective financial documents (MODIFY operation for amendments, STORNO operation for void invoices), when:
		- the invoice or the corrective financial document is issued to private persons (privatePersonIndicator=true),
		- the invoice or the corrective financial document contains item data aggregated for size decrease (mergedItemIndicator=true),
		- the type of the invoice or the corrective financial document is not an electronic invoice (invoiceAppearance != ELECTRONIC),
		- the issue date (invoiceIssueDate) of the invoice or the corrective financial document is before 01.01.2021, 
		- the system date is before 01.01.2021, or a business parameter created for this purpose disables the feature
	- in addition to the rules above, make sure to disable creating corrective electronic financial documents (with completenessIndicator=true) with MODIFY and STORNO operations for amendments and voids, respectively, with base invoices existing in the system (modifyWithoutMaster=false) with completenessIndicator set to false
	- include the electronicInvoiceHash tag with the proper cryptoType and fingerprint (hash value) in the API XML, according to the interface documentation.
Software not conforming to these rules will fail to supply data for an ERROR, and therefore it may not be able to issue the electronic invoice or the amending financial document.
- If your software supports invoices with parties exempt to VAT (vatExemption) and invoices issued to regions out of the territory of the VAT act (vatOutOfScope), offer a list to the users on the UI for the various scenarios, based on the valid values found in the interface documentation, or map your values to these values behind the scenes, transparently to the user. Place enumeration value in the case tag, and the value specified by the user on the invoice in the reason tag within the XML, both on item and invoice summary level. (we will update the changelog with PDF page number when translations are completed)
- If your software manages the cases to handle differences of tax base and accumulated tax enlisted in the interface documentation, make sure to properly specify the value of the vatAmountMismatch tag, both on item and invoice summary level. If the exchange rate cannot be computed in any of these cases, set the value of the exchangeRate tag to 0. (we will update the changelog with PDF page number when translations are completed)
- If your software supports simplified invoices, the amount of VAT (vatContent) for items shall be specified on a new XPath, lineAmountsSimplified/lineVatRate. The summary section is also affected, and the new XPath for summary VAT amount information is summarySimplified/vatRate. This is also the location to specify all reasons of exempt which you could only specify for non-simplified invoices in the past. Proper management of exempts is especially important, since vatContent could have a value of 0 until v2.0 of the API, however this option is retired in v3.0, and the new validation will trigger an error for this value. Implement all the changes in your software. Make sure to disable specifying VAT rate (vatPercentage) on item and summary level for simplified invoices, or your data exchange will fail due to an ERROR.
- If your software included EKÁER codes on item level, you have to include this data on a new XPath, line/conventionalInvoiceInfo. From v3.0, EKÁER codes can be included in the invoice header, too, not only in item lines.
- If you want to support specifying additional transaction information just introduced to the data model (such as PO number, agreement ID, delivery note number etc.) on invoices either for customer or supplier role, create the corresponding data entry UIs, and include the specified data at invoice header level or at item level in the XML file, under the /conventionalInvoiceInfo node. Cardinality of the tags is unbounded, so you can repeat tags as many times as necessary.
- If your software supports oversize data exchange (when HTTP content-length >= 10,485,760 bytes), merge items lines at item/service level according to the interface documentation, and aggregate the corresponding values. Do not forget to indicate the fact of merging by adding mergedItemIndicator=true above the item level in the data exchange payload. Note that once this item becomes true anywhere in an invoice chain, it cannot be reverted to false any more in subsequent corrective documents (with higher modificationIndex values), or the data exchange will receive a WARNING. Be warned that escalating data exchange regarding such documents for manual processing is against the law, and the invoicing software must implement the merging logic itself.
- If your software supports public utility settlement invoices, and make sure to set utilitySettlementIndicator the correct value.

### 3.3 Annulment 

#### 3.3.1) Mandatory changes related to the Annulment schema

N/A

#### 3.3.1) Scenario-specific changes related to the Annulment schema

- If your software supports the technical annulment feature of the API, change the namespace URI of the InvoiceAnnulment root element to 'xmlns="http://schemas.nav.gov.hu/OSA/3.0/annul"' in the inner XML.
- If your software reports verification hash values of electronic invoices, presence of an invalid hash prevents further correction of the data exchange, and therefore you have to submit a technical annulment. You have to use a new annulmentCode value in this scenario, so be sure to add the 'ERRATIC_ELECTRONIC_HASH_VALUE' value to the technical annulment request. Note that this code can only be used when completenessIndicator=false, and therefore this note also regards this precious case only. (If completenessIndicator is set to true, there is no possibility for technical annulment. On the other side, the hash cannot be invalid in that case, since the server rejects data supplies of this kind if the hash is invalid.)

### 3.4 ServiceMetrics 

#### 3.4.1) Mandatory changes related to the ServiceMetrics schema

N/A

#### 3.4.1) Scenario-specific changes related to the ServiceMetrics schema

- If your software uses the metrics query feature of the API, please refer to section 3.1.2 for notes on DTO generation and response validation.