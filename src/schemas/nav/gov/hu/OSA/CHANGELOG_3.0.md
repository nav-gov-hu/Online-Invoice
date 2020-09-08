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
- További változás a VatRateType esetében, hogy az ÁFA bevallások pontosításához az ÁFA mentesség(vatExemption) és a hatályon kívüliség (vatOutOfScope) eseteiben az API külön kódot és indokolást fog elvárni. A kódot (pl: TAM, AAM stb) az interfész dokumentáció fogja tartalmazni és blokkoló validáció fogja kikényszeríteni a helyességét. Az indokolás pedig szabadon kitölthető ahogy a számlán az szerepel, a megadható hosszat pedig megduplázzuk.
- Az ÁFA bevallások készítéséhez szükség van az előlegszámla-végszámla kérdés rendezésére is. A 3.0-ás sémában sorszinten átalakul az előleg jelzés kezelése, és egy új, advanceData csomópontban lehetőség lesz megadni végszámla esetén - sorszinten - az előleget tartalmazó számla sorszámát, a teljesítés időpontját, valamint az alkalmazott árfolyamot. 
- A közszolgáltatói elszámolószámlákra vonatkozó speciális szabályok miatt a sémába bekerült egy új, utilitySettlementIndicator nevű jelölő. Ha a tag értéke true, akkor az előzmény nélküli módosítások (a modifyWithoutMaster tag értéke true) feldolgozása akkor is megtörténik, ha egyébként a hivatkozott alapszámla a rendszerben már létezik.
- Azért, hogy a számlák automatikus feldolgozása minél univerzálisabb lehessen, számlafej és számlasor szintre is bekerül egy conventionalInvoiceInfo nevű csomópont. A 3.0-ás séma ez alatt a piaci gyakorlat szerint használt leggyakoribb egyezményes egyéb adatokat tartalmazza (pl: megrendelés számok, szállítólevél számok, szerződésszámok, főkönyvi számlaszámok, vállalat kódok, költséghelyek stb) amiknek az egyezményes névvel ellátása segítheti a gépi feldolgozás elterjedését.
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
- új ERROR: ha az advancePayment értéke true, akkor az advancePaymentData csomópont megadása kötelező
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
- Ha a programod riportálja az elektronikus számlák ellenőrző hash értékét, akkor ezt a 3.0-ban már nem belül a Data XML-ben, hanem kívül, az API XML-ben kell megadnod. Az ehhez szükséges tag neve változatlanul electronicInvoiceHash, de a helye már a ManageInvoiceRequest/invoiceOperations/invoiceOperation/electronicInvoiceHash Xpath útvonalon található. Ne feledd, hogy ehhez a taghoz is tartozik cryptoType attribútum, de hogy mit írsz bele és a hash érték micsoda teljesen rajtad múlik akkor, ha ez nem a 3.0 újdonsága szerinti elektronikus számla. (completenessIndicator=false)
- Ha használni szeretné a programod a 3.0 újdonsága szerinti elektronikus számlázást (completenessIndicator=true) akkor tanulmányozd az interfész dokumentációt, hogy ebben az esetben a electronicInvoiceHash tagnak és a cryptoType attribútumnak hogyan kell értéket adni.
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
- A számlafej adatok között a utilitySettlementIndicator megadása kötelező. A tag helye az exchangeRate tag után következik.
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
- Ha a programod kezel ÁFA mentes (vatExemption) és ÁFA hatályon kívüli (vatOutOfScope) számlákat, akkor a különböző esetekre az interfész dokumentáció által meghatározott értékkészlet alapján kínálj fel listás beviteli lehetőséget a UI-on, vagy a felhasználó által transzparensen feleltesd meg a saját értékeid ennek az értékkészletnek, ha erre lehetőséged van. Az enumerált értéket a case tagban, a felhasználó által a számlára felvitt értéket pedig a reason tagba helyezd el az XML-en belül mind számlasor, mind számlaösszesítő szinten.
- Ha a programod kezeli az adóalap és a felszámított adó eltérésének - interfész dokumentáció szerint felsorolt - eseteit, akkor gondoskodj a vatAmountMismatch tag megfelelő töltéséről mind számlasor, mind számlaösszesítő szinten. Amennyiben az átváltási árfolyam ezen esetek miatt nem számítható ki, akkor az exchangeRate tag értékét állítsd 0-ra.
- Ha a programod kezel egyszerűsített számlákat, akkor az ÁFA tartalmat (vatContent) számla sor szinten már új Xpath alatt, a lineAmountsSimplified/lineVatRate alatt tudod megadni. A módosítás a számlaösszesítőben is megjelenik, itt az új útvonal a summarySimplified/vatRate alatt található. Hasonlóképp itt tudod megadni az összes olyan mentességet, amelyeket eddig csak nem egyszerűsített számlánál lehetett megadni. A mentességek helyes kezelése azért fontos, mert a 2.0-ig a vatContent lehetett 0, azonban a 3.0-ban ez a lehetőség már megszűnik, az új ellenőrzés már ERROR-t ad erre az esetre! Az említett módosításokat vezesd át magadnál. Figyelj rá, hogy a programodban egyszerűsített számla esetében adó mértéket (vatPercentage) sem számlasor, szem számlaösszesítő szinten ne lehessen megadni, mert ez ilyen adatszolgáltatások ERROR miatt bukni fognak!
- Ha a programod töltötte számlasor szinten a tételhez tartozó EKÁER számokat, akkor ezt a taget már új Xpath alatt, a line/conventionalInvoiceInfo találod meg. EKÁER szám a 3.0-tól megadható számlafej szinten is, nem csak számlasorban.
- Ha az ügyfeleidnél van igény az adatmodellben megjelenő új, ügylethez tartozó azonosítók töltésére akár kiállítói, akár vevői oldalról (megrendelésszám, szerződésszám, szállítólevélszám stb) akkor ezekre az adatokra biztosíts a UI-on bevitelt, illetve helyezd el a bevitt adatokat az XML-ben számlafej vagy számlasor szintjén a /conventionalInvoiceInfo csomópont alatt. Minden tag kardinalitása korlátlan, tehát bármiből bárhány adat felírható.
- Ha a programod kezeli azt az esetet, amikor az adatszolgáltatás túl nagy méretű (HTTP content-length >= 10.485.760 bájt), akkor az interfész dokumentáció által meghatározott logika szerint vond össze a számlasorokat tétel-szolgáltatás szinten és aggregáld a megfelelő számszaki értékeket! Az összevonás tényét az adatszolgáltatásban jelezd a számlasor szint felett, a mergedItemIndicator=true kifejezéssel. Figyelj rá, hogy ha ez a tag a számlaláncban bárhol true lett, az onnantól következő módosításokban (következő modificationIndexek alatt) már sosem lehet false! (ellenkező esetben az adatszolgáltatás WARNING-ot kap) Felhívjuk a figyelmet, hogy nem felel meg a törvényi szabályozásnak az a megoldás, ami ezeket a számla adatszolgáltatásokat kiemeli kézi feldolgozásba, mindenképpen a számlázóprogramnak kell az összevonási logikát implementálni.
- Ha a programod kezel közüzemi elszámolószámlákat, akkor ha a számlafejben található utilitySettlementIndicator tag értéke true, akkor is küldhetsz előzmény nélküli módosításról szóló adatszolgáltatást (modifyWithoutMaster=true), ha egyébként a hivatkozott alapszámla a rendszerben már létezik.

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

# Changelog 2.0

Comprehensive changes from 2.0 to 3.0.




Translation pending.