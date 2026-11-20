# termekfejlesztes
## Biztonság (2026-11)

- `firestore.rules`: olvasás nyilvános, írás csak bejelentkezett kliensnek. Beillesztés: Firebase konzol → Firestore Database → Rules → Publish.
- Az `index.html` megnyitáskor névtelenül bejelentkezik (`signInAnonymously`); ehhez Authentication → Sign-in method → **Anonymous** engedélyezése kell. Amíg nincs bekapcsolva, az app auth nélkül fut tovább, de a szabály publikálása után írni nem tud.
- Ez a kíváncsiskodó REST-írást zárja ki; aki az oldalt megnyitja, továbbra is írhat. Személyhez kötött jogosultsághoz e-mail/jelszó auth kell.

## Havi riport frissítése

1. Nyisd meg a `riport-szerkeszto.html`-t (TEKNO → Havi riport → „Adatfrissítés (szerkesztő)”).
2. **📂 CSV betöltés**: töltsd be **mindkét** fájlt (járati + napijegy CSV). A járati CSV a teljes időszakot fedje le, mert a betöltés lecseréli az adatot (előző év / YoY ehhez kell).
3. **💾 Küldhető HTML mentése** → a letöltött fájlt nevezd át `havi-riport.html`-re.
4. GitHub: Add file → Upload files → húzd be → Commit changes. A Pages 1–2 percen belül frissül.

Opcionálisan egy 3. fájl is betölthető: **viszonylati km-tábla** (`vonatszám;km` vagy `induló;cél;km`) – ezzel km-súlyozott kihasználtság és utaskm jelenik meg. A 3. fül beállításai (cél-kihasználtság, alap egységméret, helyjegyár) a mentett riportba is bekerülnek.

A szerkesztőben nincs adat; a CSV-k csak a böngészőben dolgozódnak fel. Az **⬇ Export** gombbal (élő riportban is) PowerPoint (.pptx) vagy PNG készíthető az aktuális szűrésről.

## Előrejelzés és kocsiszám-javaslat (3. fül)

- **Eseménynaptár** (`EVENT_CALENDAR` a riport kódjában): iskolai szünetek (tanév rendje rendeletek), ünnepek és munkanap-áthelyezések (NGM-rendeletek), fesztiválok (rendezői közlemények). „becsült” = még nem kihirdetett időpont – évente érdemes frissíteni.
- **Eseményelőrejelzés:** a legutóbbi azonos esemény tényleges vonatonkénti foglalása × csillapított trend (utolsó 12 hét YoY fele, max ±10%), összevetve a tavalyi tényleges és a szezonális normál összeállítással.
- **Egység:** vonatonként a kapacitás lépcsőiből (2. o. 55–90, 1. o. 40–70 hely), különben az alapérték.
- **Normál hét:** az utolsó 8 hét, eseménynapok nélkül, hétnaponként; bővítés és leépíthető egység.
