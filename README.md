# termekfejlesztes
## Biztonság (2026-11)

- `firestore.rules`: olvasás nyilvános, írás csak bejelentkezett kliensnek. Beillesztés: Firebase konzol → Firestore Database → Rules → Publish.
- Az `index.html` megnyitáskor névtelenül bejelentkezik (`signInAnonymously`); ehhez Authentication → Sign-in method → **Anonymous** engedélyezése kell. Amíg nincs bekapcsolva, az app auth nélkül fut tovább, de a szabály publikálása után írni nem tud.
- Ez a kíváncsiskodó REST-írást zárja ki; aki az oldalt megnyitja, továbbra is írhat. Személyhez kötött jogosultsághoz e-mail/jelszó auth kell.
