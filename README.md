# termekfejlesztes
## Hozzáférés és adatvédelem (2026-11)

- **Nyilvános tárolóban nincs riportadat.** A `havi-riport.html` csak a programkódot tartalmazza; az adat a Firestore-ban van (`riport/meta`, `riport/c_<verzió>_<n>` darabok, `riport/summary`), és csak bejelentkezés után töltődik be.
- **Bejelentkezés:** e-mail + jelszó (Firebase Authentication). A TEKNO főoldala, a havi riport, a szerkesztő és Abigél ugyanazt a bejelentkezést használja (közös token, egyszer kell belépni).
- **Hozzáférési lista:** Firestore `access` gyűjtemény, dokumentumazonosító = kisbetűs e-mail-cím, mező: `role` = `admin` | `szerkeszto` | `olvaso`. Olvasni minden listán lévő, írni (szolgáltatások, riportfeltöltés) csak `szerkeszto`/`admin`, a listát kezelni csak `admin` tud.
- **Szabályok:** `firestore.rules` → Firebase konzol → Firestore Database → Rules → Publish. Névtelen bejelentkezéssel semmi nem olvasható és írható.
- **Küldhető riport:** a szerkesztő „🔒 Titkosított HTML mentése” gombja jelszóval titkosított fájlt ad (AES-256-GCM, PBKDF2 600 000 iteráció). A fájl jelszó nélkül nem nyitható; a jelszót külön csatornán küldd. Ezt a fájlt **ne** töltsd fel a GitHubra.

### Firebase-beállítás (egyszeri)

1. Authentication → Sign-in method: **Email/Password** be, **Anonymous** ki.
2. Authentication → Settings → Authorized domains: `molnarzsoltphoto.github.io` szerepeljen.
3. Authentication → Users → Add user: felhasználók felvétele (jelszó-visszaállítás a belépőablak „Elfelejtett jelszó” linkjével).
4. Firestore → `access` gyűjtemény: felhasználónként egy dokumentum (azonosító a kisbetűs e-mail, `role` mező). Az első `admin` dokumentumot a konzolban kell létrehozni.
5. Rules → a `firestore.rules` tartalmának beillesztése → Publish.

## Havi riport frissítése

1. Nyisd meg a `riport-szerkeszto.html`-t (TEKNO → Havi riport → „Adatfrissítés (szerkesztő)”), jelentkezz be (`szerkeszto` vagy `admin` szerep).
2. **☁ Betöltés a TEKNO-ból**: a legutóbb feltöltött riport betöltése (beállításokkal együtt), vagy **📂 CSV betöltés**: töltsd be **mindkét** fájlt (járati + napijegy CSV). A járati CSV a teljes időszakot fedje le, mert a betöltés lecseréli az adatot (előző év / YoY ehhez kell).
Régebbi riportból: **📄 Riport-HTML betöltése** – egy korábbi, adatot tartalmazó `havi-riport.html` vagy titkosított küldhető fájl (jelszóval) adatai a szerkesztőbe kerülnek.
3. A további fájlok (bisztró, kerékpár, menetrendszerűség stb.) betöltése.
4. **☁ Feltöltés a TEKNO-ba**: a riport, a beállítások és az Abigél-összesítő a Firestore-ba kerül; a `havi-riport.html` azonnal az új adatot mutatja. GitHub-feltöltés nem kell.

Opcionálisan egy 3. fájl is betölthető: **viszonylati km-tábla** (`vonatszám;km` vagy `induló;cél;km`) – ezzel km-súlyozott kihasználtság és utaskm jelenik meg. A 3. fül beállításai (cél-kihasználtság, alap egységméret, helyjegyár) a mentett riportba is bekerülnek.

További opcionális fájlok:
- **Bisztró bevétel és tranzakció** (`.xlsx`, az eredeti munkafüzet, pl. `bisztro_bev_tra_20261120.xlsx`): vonatszám és nap szerint a prémium adatok mellé kerül a 2026-os tranzakciószám (KPI, járattáblázat, grafikon-tooltip). Kihagyott cellák: 300 tranzakció/vonat fölött vagy 200 Ft alatti átlagkosárnál (elcsúszott cella). A több vonatra rögzített fordulótételek (pl. `808.801`) nem bonthatók vonatra.
- **Indulási idők** (`vonatszám;indulás`, CSV vagy .xlsx). Nem kell, ha a járati CSV-ben van „Indulási idő” oszlop. A vonatok a vonatneveken belül ez szerint rendeződnek; enélkül vonatszám szerint.

A CSV-k csak a böngészőben dolgozódnak fel; a szerver csak a feltöltött, összesített riportot kapja meg. Az **⬇ Export** gombbal (élő riportban is) PowerPoint (.pptx) vagy PNG készíthető az aktuális szűrésről.

## Előrejelzés és kocsiszám-javaslat (3. fül)

- **Eseménynaptár** (`EVENT_CALENDAR` a riport kódjában): iskolai szünetek (tanév rendje rendeletek), ünnepek és munkanap-áthelyezések (NGM-rendeletek), fesztiválok (rendezői közlemények). „becsült” = még nem kihirdetett időpont – évente érdemes frissíteni.
- **Eseményelőrejelzés:** a legutóbbi azonos esemény tényleges vonatonkénti foglalása × csillapított trend (utolsó 12 hét YoY fele, max ±10%), összevetve a tavalyi tényleges és a szezonális normál összeállítással.
- **Egység:** vonatonként a kapacitás lépcsőiből (2. o. 55–90, 1. o. 40–70 hely), különben az alapérték.
- **Normál hét:** az utolsó 8 hét, eseménynapok nélkül, hétnaponként; bővítés és leépíthető egység.

## Kínálatfejlesztési jelzések (4. fül)

Időszak (utolsó 12 hónap vagy naptári év) és évszak szerint szűrhető; páros/páratlan vonatszám, rendezés vonatnév + indulás vagy érintettség szerint.

- **Bisztró-penetráció:** bisztrótranzakció / 100 foglalt utas vonatonként (2026-tól, a bisztró-xlsx alapján). „Sok utas, kevés vásárlás”: a vonatnév mediánjánál ≥25%-kal kisebb penetráció legalább medián utasszám mellett.
- **Osztályok közti eltolódás:** napok, amikor a 2. o. ≥90% és az 1. o. ≤50% (állítható). Javaslat: deklasszálás (≥30% napon és ≥30 üres 1. o. hely) vagy upgrade-akció.
- **Kerékpár-radar:** megtelt kerékpárhelyek vonatonként, havi eloszlással; szezonális / állandó / csúcsidőszaki bővítési javaslat.
- **Busz–vonat:** helyjegyes helyközi buszok és helyjegyköteles vonatok azonos végpontú viszonylatokon (település szerint). Közbenső megállók nincsenek az adatban. A buszadat 2024. augusztus elejétől teljes.

## Havi összesítő más alkalmazásoknak (Abigél)

A szerkesztő feltöltéskor egy kis JSON-összesítőt is ír a Firestore `riport/summary` dokumentumába: havonta, szekciónként (vonatnév, vonatnem, helyjegyes busz viszonylat, napijegy) foglalás, kapacitás, járatszám, kerékpár és bisztró-tranzakció. Abigél üzleti eset paneljét („Mért utasszám a TEKNO havi riportból”) ez táplálja (bejelentkezés után), így a havi frissítés után ott is automatikusan az új számok látszanak.

## Kerékpárjegy-térkép, időszakszűrés, Excel-export (2026. november)

- **Kerékpárjegy-eladások** (szerkesztő, 6. mező, `.xlsx`): a szolgálati hely szerinti havi kimutatás. A hónapot a kimutatás szűrő-szövegéből olvassa, több hónap egymás után betölthető. A 4. fül kerékpár-részében térképen jelenik meg (ország / Budapest környéke). Az állomáskoordináták a `mav-stations` adatcsomagból (ISC licenc) vannak beépítve; az ügyfélszolgálati irodák és buszállomások a településük állomásához kerülnek. Az online (E-Ticket) eladás helyhez nem köthető.
- **Kínálatfejlesztési jelzések** időszaka: utolsó 12 hónap, év, hónap, hét vagy nap. Rövid időszaknál a küszöbök arányosan kisebbek.
- **Excel-export**: az Export ablakban „Excel (.xlsx)” formátum. Minden kijelölt elem külön munkalap, a táblázatok teljes terjedelemben, a grafikonok mögötti adatsorokkal; a számok számként kerülnek át.

## Kerékpáros viszonylatok és menetrendszerűség

- **Kerékpárjegyek jegy-szinten (R155 CSV, szerkesztő 7. mező):** egy fájlban több év is lehet (a hónap az értékesítés időpontjából jön; a fájlban lévő hónapokat felülírja). A böngésző darabonként olvassa, így a 900 MB-os, 2024 óta tartó export is betölthető, `.zip`-be vagy `.gz`-be tömörítve is. Excelbe mentve ne: az 1 048 576 sornál levág. A térkép időszaka: hónap, naptári év, utolsó 12 hónap vagy az összes, és követi a 4. fül időszakszűrőjét. Település kiválasztható (keresővel vagy a térképen a pontra kattintva): „Innen hova” / „Honnan ide” nézetben a célt vagy indulást település szerint mutatja, a település állomásainak megoszlásával. Az állomások települését a KSH-településhatárok adják (pl. Kiskundorozsma → Szeged, Kőbánya-Kispest → Budapest). Viszonylatokká összesítve (indulási–célállomás, mindkét irány együtt) a kerékpár-térkép „Utazási viszonylatok” rétegén, alatta a 20 legnépszerűbb viszonylat táblázata, oldalt az értékesítési csatornák. Sztornózott és visszatérített jegyek nélkül. A térkép vonalai az indulási és célállomást kötik össze, nem az útvonalat.
- **Menetrendszerűség (xlsx, szerkesztő 8. mező):** a 4. fül 5. kártyáján a 25 legrosszabb menetrendszerűségű vonat (szegmens- és minimális utasszám-szűrővel), vonatszám szerint összekötve a járati adattal (vonatnév, reláció, kihasználtság, kerékpárhely, bisztró), valamint vonatnév szerinti összesítő. A kártyán saját kereső van (vonatnév, vonatszám vagy állomás): kereséskor az összes találat látszik, nem csak a 25 legrosszabb. Az 1. fül táblázatában „Menetrendszerűség” oszlop, ha a szűrt időszakban van menetrendszerűségi adat.

## Lenyitható blokkok

Minden tartalmi blokk a címére kattintva becsukható / kinyitható (▾ / ▸). Az állapotot a böngésző megjegyzi; exportkor a becsukott blokkok is bekerülnek.

## Menetrendszerűség és kereslet (4. fül, 6. kártya)

- **Pontdiagram:** vonatnevenként az utasszámmal súlyozott menetrendszerűség és a foglalás / közlekedő vonat változása az előző év azonos hónapjaihoz (a szezonalitás és a járatszám-változás így kiesik; az éjszakai vonatoknál a fekvő- és hálóhely-foglalással együtt), a betöltött menetrendszerűségi hónapokra. Korrelációs együttható és lineáris illesztés; narancs: a hálózatnál rosszabb menetrendszerűségű és gyengébb keresletű vonatnév.
- **Havi bontás:** a kiválasztott vonatnév menetrendszerűsége és keresletváltozása közös időtengelyen, a hálózati értékkel.
- **Tartósan romló menetrendszerűség:** legalább 3 hónap adatból lineáris trend vonatnevenként. A kereslet értékelése: „csökkenő”, ha a foglalás / vonat az időszakban kevesebb, mint egy évvel korábban (vagy a végén már csökken és lefelé tart); „lassuló ütemben nő”, ha még nő, de egyre kevésbé; külön jelzés, ha vonatonként nem csökken, de kevesebb közlekedés miatt az összes foglalás igen.
- Több hónap menetrendszerűségi kimutatása a szerkesztő 8. mezőjébe egyszerre is kijelölhető. Együttmozgást mutat, nem bizonyított ok-okozatot.

## Figyelőlista (5. fül)

- Vonatszám, vonatnév, reláció (vonatok és helyjegyes buszok végállomásai szerint) vagy kerékpáros viszonylat vehető fel, döntés / megjegyzés és hatályba lépés hónapja megadásával.
- Kártyánként: utolsó hónap foglalása, előző év, foglalás / vonat változása, kihasználtság, menetrendszerűség, 24 havi grafikon az előző évvel; hatálybalépés esetén a döntés előtti 3 és utáni legfeljebb 6 hónap előző évhez mért változásának összevetése.
- A lista a Firestore `riport/watch` dokumentumában van: mindenki látja, `szerkeszto` / `admin` szerkeszti (a riport oldaláról is, bejelentkezve). A titkosított küldhető fájlba a mentéskori lista kerül.

## Vonatnév-elemzés (6. fül)

Kiválasztott vonatnévre és időszakra (utolsó 12 / 3 hónap, naptári év, egy hónap, egy hét vagy egy nap; hétnél és napnál az előző év azonos hétnapjaihoz mérve) automatikus, szabályalapú szöveges elemzés grafikonokkal: összefoglaló, forgalom (foglalás / vonat havonta az előző évvel), kihasználtság osztályonként, a hét napjai, vonatonként (90% fölötti közlekedések), csúcsnapok, kerékpár, bisztró, menetrendszerűség, végül következtetések és javaslatok. Nyomtatható (A4 PDF) és PowerPointba menthető. A prémium, a fekvőhely és a hálóhely külön szerepel (kihasználtság, előző év, 90% fölötti közlekedések, javaslat); az ülőhely-kihasználtság az 1., 2. osztály és a prémium együtt. Küszöbök: telítettség ≥90%, rendszeres = a közlekedések ≥15%-a (legalább 3 alkalom), alacsony kihasználtság = Ø <35%.

## Balatoni szezon (7. fül)

- Elő-, fő- és utószezon külön, a MÁV balatoni menetrendi időszakai szerint (2024: máj. 13. – jún. 21. / jún. 22. – szept. 1. / szept. 2–22.; 2025: máj. 12. – jún. 20. / jún. 21. – aug. 31. / szept. 1–21.; 2026: máj. 1. – jún. 19. / jún. 20. – aug. 30. / aug. 31. – szept. 20.; 2027: ápr. 26. – jún. 18. / jún. 19. – aug. 29. / aug. 30. – szept. 19.). Új évnél a `BAL_SEASONS` táblát kell bővíteni.
- Balatoni vonatok: balatoni végállomású vagy onnan induló helyjegyes vonatok, valamint a déli parton átmenő Budapest/Székesfehérvár – Nagykanizsa, Zágráb vonatok; vonatnevenként szűrhető.
- Szezononkénti táblázat (foglalás/nap, foglalás/vonat, kihasználtság, osztályok, ≥90%-os közlekedés, kerékpár, Balaton24 napijegy havonta – a havi adat a szezonnal átfedő napokkal súlyozva) az előző év azonos szezonjához; napi grafikon (7 napos átlag, előző év azonos hétnapjai); évek összevetése; vonatnevenként; javaslatok. Nyomtatható és PowerPointba menthető.
- Prémium (foglalás/nap, kihasználtság, ≥90%) és kerékpár (kerékpárhely-foglalás/nap, kihasználtság, megtelt) külön sorokban, grafikonnal és javaslattal; a 2. osztály a prémium nélkül.
- A vonatnév-elemzésben (6. fül) a balatoni vonatneveknél külön „Balatoni szezon” fejezet.

## Hibrid InterCityk és bisztró-KPI

- Hibrid (nem helyjegyköteles) InterCityk: NAPFÉNY, KRESZ GÉZA, SOMOGY, RIPPL-RÓNAI, BAKONY, GÖCSEJ, BÉKÉS, valamint a balatoni elő-, fő- és utószezonon kívül a BALATON és a TÓPART. A foglaltsági adatok náluk csak az IC vonatrészre vonatkoznak. Jelölés („hibrid IC”) az 1. fül táblázatában és figyelmeztetés a KPI-k alatt, a 4. fül vonatcelláiban, a vonatnév-elemzésben (a kapacitás-csökkentési javaslat helyett figyelmeztetéssel) és a figyelőlistán. Lista: `HYB_ALWAYS` / `HYB_OFFSEASON` a kódban.
- 1. fül: külön „Bisztró eladások” KPI-kártya (tranzakció, bevétel, átlagkosár) a szűrésre, ha a bisztró-munkafüzet be van töltve.

## Kapacitás: összes hely vagy ülőhely

A fejlécben (a fülek alatt) kapcsoló: **Összes hely** (alapértelmezés: 1., 2. osztály, prémium, fekvő- és hálóhely) vagy **Ülőhely** (1., 2. osztály és prémium). Minden foglalási és kihasználtsági mutató ehhez számol (1. fül KPI-k és táblázat, 4. fül, menetrendszerűségi kártya járati adatai, figyelőlista, vonatnév-elemzés, balatoni szezon). A választást a böngésző megjegyzi. A menetrendszerűség–kereslet elemzés mindig az összes hellyel számol; az osztályonkénti bontások (1. o., 2. o., prémium, fekvő, háló) változatlanok.

## Menetrendszerűség az 1. fül grafikonján

Kihasználtság % nézetben (osztályonkénti vagy vonatonkénti vonalaknál) külön vonal: a kiválasztott vonatok menetrendszerűsége, utasszámmal súlyozva, a betöltött menetrendszerűségi hónapokra (napi bontásban a havi érték). Ugyanazon a %-os tengelyen, az előző év (ha van adat) szaggatottan; a jelmagyarázatban ki-be kapcsolható. Darabszámos nézetben (foglalt utas, kapacitás) nem jelenik meg, mert ott a tengely nem %.

## Összeállítási eltérések (szerkesztő 9. mező)

Napi kimutatás vonatszám szerint (`Datum`, `Vonatszám`, `Eltérés`, `Kód`, `Esemény minősítése`, `Esemény`): kevesebb jármű / több jármű / egyéb. Új fájl betöltésekor a fájl időszakába eső napok lecserélődnek, a többi megmarad – elég az új hónapokat feltölteni.

- **4. fül, 7. kártya:** KPI-k (összes, minősítésenként, érintett vonatszám, helyjegyes vonatra eső rész) az előző évhez mérve; havi grafikon minősítésenként; okok táblája; a 25 vonat, amely a legtöbbször közlekedett kevesebb járművel (alapból a helyjegyes vonatok, kapcsolóval mind), a foglalási adattal: kihasználtság a kevesebb járműves napokon és a többi napon, ≥90%-os napok kevesebb járművel; hiányzó járműtípusok. Követi a 4. fül időszak-, kereső- és páros/páratlan szűrőjét.
- **Vonatnév-elemzés:** KPI-kártya, „Összeállítási eltérések” fejezet vonatszámonkénti grafikonnal, javaslatok (pl. ha a közlekedések ≥5%-ában kevesebb járművel ment, vagy telt vonat közlekedett kevesebb kocsival).
- **1. fül táblázat:** „Össz. eltérés” oszlop a szűrt időszakra (zárójelben a kevesebb járműves).
- A foglalási kapacitás a tervezett összeállítást tükrözi, ezért kevesebb kocsival a valós zsúfoltság nagyobb lehet, mint amit a kihasználtság mutat. A kimutatás sorainak kb. harmada köthető helyjegyes vonatszámhoz.

## Kerékpáros viszonylatok a vasúthálózaton (4. fül)

A viszonylat-térkép minden kerékpáros viszonylatot a vasúthálózatra terhel: az indulási és a célállomás közötti legrövidebb úton, a fővonalakat előnyben részesítve (mellékvonal 1,2×, csak teherforgalmú vagy megszűnt vonal 3× súllyal), így pl. Budapest–Szeged Cegléden át megy. A közös szakaszokon a jegyek összeadódnak; vonalvastagság és árnyalat: a szakaszon áthaladó jegyek (mindkét irány). A szakaszra mutatva látszik a vonal (és a közös szakaszon haladó további vonalak), valamint a legnagyobb áthaladó viszonylatok. A település kiválasztásakor ugyanígy, csak annak kapcsolataival. Alatta „Vonalak kerékpáros terhelése” táblázat (Ø terhelés, legnagyobb terhelés a saját szakaszon, jegy-km); Excel-exportban teljes terjedelemben.

A hálózat az OSM vonalrelációiból (`vasut-vonalak.geojson`) készül: a párhuzamos vágányok ~60 m-en belül összevonva, az állomások a legközelebbi vonalponthoz kötve. A legrövidebb út nem mindig egyezik a jegy tényleges útvonalával.

## Havi riport (8. fül)

Választható járatok (vonatnév vagy vonatszám, több is; alapból minden helyjegyes vonat) és hónapok (egy vagy több; gyorsgombok: utolsó teljes hónap, utolsó 3 hónap, az év eddig, előző év). KPI-kártyák: foglaltság és változása, kapacitás és változása, kihasználtság és változása (előző év azonos hónapjai), menetrendszerűség (utasszámmal súlyozva, változás az előző hónaphoz %p-ban), 1. osztály, 2. osztály, prémium és kerékpárhely kihasználtsága, bisztró bevétel (bevétel nélküli napokkal és vonatnapokkal) és változása az előző hónaphoz (több hónapnál az egy hónappal korábbi időszakhoz). A foglaltság és a kapacitás a fejléc kapacitás-kapcsolóját követi. Exportálható (PowerPoint, PNG, Excel).
