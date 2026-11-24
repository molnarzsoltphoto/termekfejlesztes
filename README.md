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

- **Kerékpárjegyek jegy-szinten (R155 CSV, szerkesztő 7. mező):** egy fájlban több év is lehet (a hónap az értékesítés időpontjából jön; a fájlban lévő hónapokat felülírja). A böngésző darabonként olvassa, így a 900 MB-os, 2024 óta tartó export is betölthető, `.zip`-be vagy `.gz`-be tömörítve is. Excelbe mentve ne: az 1 048 576 sornál levág. A térkép időszaka: hónap, naptári év, utolsó 12 hónap vagy az összes, és követi a 4. fül időszakszűrőjét. Viszonylatokká összesítve (indulási–célállomás, mindkét irány együtt) a kerékpár-térkép „Utazási viszonylatok” rétegén, alatta a 20 legnépszerűbb viszonylat táblázata, oldalt az értékesítési csatornák. Sztornózott és visszatérített jegyek nélkül. A térkép vonalai az indulási és célállomást kötik össze, nem az útvonalat.
- **Menetrendszerűség (xlsx, szerkesztő 8. mező):** a 4. fül 5. kártyáján a 25 legrosszabb pontosságú vonat (szegmens- és minimális utasszám-szűrővel), vonatszám szerint összekötve a járati adattal (vonatnév, reláció, kihasználtság, kerékpárhely, bisztró), valamint vonatnév szerinti összesítő. Az 1. fül táblázatában „Pontosság” oszlop, ha a szűrt időszakban van menetrendszerűségi adat.
