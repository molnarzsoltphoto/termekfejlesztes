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

A szerkesztőben nincs adat; a CSV-k csak a böngészőben dolgozódnak fel. Az **⬇ Export** gombbal (élő riportban is) PowerPoint (.pptx) vagy PNG készíthető az aktuális szűrésről.
