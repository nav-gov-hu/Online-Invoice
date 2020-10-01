# Hozzájárulás a projekthez

`scroll down for English version`

A projekthez bárki szabadon hozzájárulhat tudástár (wiki) vagy fejlesztési ötlet, javaslat hozzáadásával akár szöveges megjegyzés (issue) akár kód (pull request) formájában. A hozzájáruláskor a hozzájáruló automatikusan elfogadja a projekt licensz (MIT) által meghatározott feltételeket.

## 1) Hogyan tudom 2 eltérő API verzió változásait tételesen megnézni? (diff)

A fájlok közötti különbség megtekintésére a GitHub több nézetet is felkínál. A legegyszerűbb 2 commit összehasonlítása, de lehetőség van URL param alapján is különbségeket vizsgálni. 

Például, az 1.1 és a 2.0 közötti különbséget az alábbi URL-en lehet megnézni: https://github.com/nav-gov-hu/Online-Invoice/commit/0da32edd71c68da2e529d704f06825317767245e?diff=split

A változásokat javasoljuk összeolvasni minden esetben az aktuális changelog [CHANGELOG 2.0.md](https://github.com/nav-gov-hu/Online-Invoice/blob/master/src/schemas/nav/gov/hu/OSA/CHANGELOG_2.0.md), illetve [CHANGELOG 3.0.md](https://github.com/nav-gov-hu/Online-Invoice/blob/master/src/schemas/nav/gov/hu/OSA/CHANGELOG_3.0.md) fájllal, ami segít az áttekintésben.

## 2) Fejlesztési ötletem vagy kérdésem van (issue)

Az API **koncepcionális** működésére vonatozó kérdés vagy ötlet esetén issue-t lehet feladni a fejlesztőknek: https://github.com/nav-gov-hu/Online-Invoice/issues

Minden XSD módosításhoz készül külön CHANGELOG az adott branchen, ezt minden esetben kérjük az issue feladása előtt előzetesen tanulmányozni! Minden feladott issue-t megválaszolunk, de kérjük, hogy lehetőség szerint kerüljük a duplikációkat. Minden issue nyilvános, a kérdést és a választ is mindenki láthatja. Amennyiben az issue javaslat és annak tartalmával egyetértünk, úgy a javaslatot befogadjuk és a projekt kanban táblájára is felkerül mint feladat. A módosítást a későbbiekben új commit alatt hozzá fogjuk adni a projekthez.

A projekt alatt lehetőség van az interfész dokumentáció hibáinak jelzésére is.

Kérjük a tárgynak megfelelő sablon használatát az issue-k alatt kiválasztani, a következők alapján:

- kérdés esetén: Kérdés-válasz / Q&A issue
- javaslat esetén: Fejlesztési kérés / Feature request
- dokumentációs hiba jelzése esetén: Dokumentációs hiba / Documentation error

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

--------------------------------------------------------------------------------------------------------------------------------------------

# Contributing to the project

Anyone is free to contribute to the project by posting a knowledge base (wiki) entry, or a development idea or suggestion, either in the form of a text comment (issue) or as code (pull request). All contributors will be considered to have automatically accepted the terms of the project license (MIT).

## 1) How can I view the changes in 2 different API versions in detail? (diff)

GitHub offers multiple views for reviewing differences between files. The simplest way is to compare 2 commits, but you can also analyse
differences using URL params. For example, the differences between Versions 1.1 and 2.0 can be viewed at the following URL:
https://github.com/nav-gov-hu/Online-Invoice/commit/0da32edd71c68da2e529d704f06825317767245e?diff=split

We recommend always reviewing changes alongside the latest [CHANGELOG 2.0.md](https://github.com/nav-gov-hu/Online-Invoice/blob/master/src/schemas/nav/gov/hu/OSA/CHANGELOG_2.0.md) or [CHANGELOG 3.0.md](https://github.com/nav-gov-hu/Online-Invoice/blob/master/src/schemas/nav/gov/hu/OSA/CHANGELOG_3.0.md) file, which will help with the review.

## 2) I have a development idea or question (issue)

If you have a question or idea regarding the **conceptual** operation of the API, you can submit an issue to the developers: https://github.com/nav-gov-hu/Online-Invoice/issues

A separate CHANGELOG is created for each XSD modification on a given branch. Please always make sure to consult the changelog before
submitting an issue! We will respond to all submitted issues, but please avoid duplications wherever possible. All issues are public: both
the question and the response are visible to everyone. If the issue is a proposal and we agree with the content thereof, we will accept the
proposal and list it on the project Kanban board as a task. The change will then be added to the project as a new commit.

In the project it is possible to indicate the errors of the interface documentation.

Please select the use of the template suitable for the subject from the issues, based on the following:

  - for questions: Kérdés-válasz / Q&A issue
  - for proposals: Fejlesztési kérés / Feature request
  - for documentation errors: Dokumentációs hiba / Documentation error

Please fill out the part after the [] label in the subject field of the template according to what the issue is referencing.

Please note that GitHub is not a help desk, and we will not be able to respond to queries about specific issues or problems relating to production systems on this platform. The options listed in Section 7.2 of the interface documentation continue to be available for the resolution of any such issues. In addition, please note that for purely business- or invoicing-related issues, within the GitHub framework we will at best only be able to provide a response to the extent required for correct and accurate data exchange. Issues that do not meet the above conditions will be deleted.

## 3) I would like to contribute to the knowledge base (wiki)

NAV supports community knowledge sharing, as the dedicated and high standard work provided by invoicing software developers is essential to
the success of the Online Invoicing System. Accordingly, we use GitHub to provide a framework and an opportunity for creating a knowledge base open to any developer interested in contributing. Contributions may include business logic, design patterns, or implementations for a
specific part or function in a given program language (among others), essentially anything that may prove useful to those joining the project.

Accessing the wiki requires being logged into your GitHub account: https://github.com/nav-gov-en/Online-Invoice/wiki

The content and structure of the wiki is completely up to the community itself, in a self-organising fashion. As part of this, the community
shall be responsible for adhering to and enforcing the moderation principles set out in README.md. NAV cannot be held responsible for the
technical accuracy of the wiki’s content. It will remain the responsibility of the contributor and the community.

Currently, the project wiki is freely editable without requiring group membership; however, this may change over time.

## 4) I would like to submit my own code (pull request)

As the repository contains only the XSD and its drafts, you can currently only send pull requests for the schema definition and the
associated sample XML files. We will reject all PRs that (also) include files other than the aforementioned. As the scope of the repository is extended, other types of PRs will become available for submission.

If we agree with the content of the pull request, we will merge the change, and if the change requires server-side changes as well, we will
also post the task on the project Kanban board. If the pull request is incorrect or incomplete, we will ask you to correct or supplement it.

Making a pull request requires being logged into your GitHub account: https://github.com/nav-gov-hu/Online-Invoice/pulls

### 4.1) Pull request submission workflow

1.  Set up your own client-side development environment (IDE, GIT, etc.).
2.  Fork the Online-Invoice repository.
3.  Clone the forked repository to your own device.
4.  Create a new branch, adhering to the naming convention.
5.  Modify, verify and test the codes. Having regard to the fact that currently you can only send pull requests for a non-functional code,
    we will only require static code analysis (i.e. the schema should be well-formatted and valid), unit and development testing is not yet a requirement for approval.
6.  Commit your change(s) to your own local branch.
7.  Push your committed changes to your own repository fork.
8.  Send a pull request on the GitHub screen. The pull request should be sent to the appropriate branch for the main version of the XSD to
    which the change request applies (during the review period, this is always the ‘master’ branch.). The project supports both merge commit
    and squash, but squash is preferred for multiple commits. (https://github.blog/2016-04-01-squash-your-commits/)

### 4.2) Management of pull requests

  - All pull requests must have a description. PRs without a description will be rejected.
  - Pull request descriptions should be as clear and concise as possible, and they should show what issue was addressed by the PR in
    question.
  - A pull request can only be merged if reviewers find it appropriate.
  - If the pull request is incomplete or incorrect, we will ask you to correct or complete it during the review.

### 4.3) Pull request naming conventions

Always name the pull requests according to the following naming convention: `[type]/[short description of changes]`.

The `[type]/` prefix should be given one of the following values:

  - `feature/` = when adding new functionality
  - `try/` = for proposals, implemented on an experimental basis
  - `fix/` = correction, clarification

The `[short description of changes]` postfix should contain the business need to be met by the change. Example:

  - `[feature]/[adding modifyWithoutMaster search parameter to the /queryInvoiceDigest operation]`
  - `[fix]/[correcting an annotation typo in ProductCodeCategoryType]`

Formulations that are ambiguous or too general (e.g. “modifying the /queryInvoiceDigest operation”), are incorrect and should be avoided if
possible.
