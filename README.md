# Receptgyűjtemény

Egyfájlos, offline is működő receptgyűjtemény (`index.html`). Nincs hozzá szerver, adatbázis
és build-lépés: a receptek magában a HTML-fájlban vannak, a `<script id="data">` blokk
`const RECIPES = [...]` tömbjében.

**Funkciók:** keresés, kategóriaszűrés, offline receptek szűrése, receptek szerkesztése,
új receptek felvitele, a módosítások kimentése fájlba.

---

## Szerkesztés és új recept felvitele

| Művelet | Hol találod |
|---|---|
| Meglévő recept módosítása | Nyisd meg a receptet → **✏️ Szerkesztés** |
| Új recept felvitele | A lista fölötti **➕ Új recept** gomb |
| Saját recept törlése | A saját recept szerkesztő paneljén → **🗑 Recept törlése** |
| Módosítások kimentése | **⬇ Mentés fájlba** (csak ha van módosítás) |
| Helyi módosítások eldobása | **🧹 Módosítások eldobása** |

Szerkeszthető mezők: cím, kategória (új kategória is felvehető), hozzávalók, elkészítés,
megjegyzés, eredeti link. A hozzávalókat és a lépéseket **soronként egy tétel** formában
kell megadni. Ha egy recepthez bekerül hozzávaló vagy lépés, automatikusan „✓ Offline"
jelölést kap. A saját recepteket a **★ Saját** címke jelzi.

Csak a saját receptek törölhetők — a beépítettek nem, azokat legfeljebb átírni lehet.

---

## Hogyan kerül fel a módosítás a tárhelyre?

### A rövid válasz

**Magától nem kerül fel.** Ez egy statikus HTML-fájl, ami a böngészőben fut, szerver nélkül.
A módosításokat a böngésző saját tárolója (`localStorage`) őrzi, két kulcs alatt:

- `receptek_overrides_v1` — a meglévő receptek módosításai
- `receptek_uj_receptek_v1` — a felvitt új receptek

Ez azt jelenti, hogy:

- a módosítás **csak abban a böngészőben** látszik, ahol beírtad (a telefonodon felvitt recept
  a laptopodon nem lesz ott),
- ha törlöd a böngésző adatait vagy privát ablakot használsz, **elveszik**,
- **más nem látja**, aki megnyitja az oldalt a tárhelyről.

Ezért a szerkesztésnek van egy záró lépése: a **⬇ Mentés fájlba** gomb legenerál
egy új `index.html`-t, amiben a módosítások már bele vannak írva a `RECIPES` tömbbe. Ezt a
fájlt kell feltölteni a tárhelyre — onnantól mindenki, minden eszközön azt látja.

### A teljes útvonal

```
böngésző (localStorage)          →  index.html fájl        →  git repó       →  élő oldal
  szerkesztés / új recept           ⬇ mentés fájlba           feltöltés         GitHub Pages
        [kézi]                          [kézi]                  [kézi]        [ez automatikus]
```

Az utolsó lépés — repó → élő oldal — **automatikus**, ha az oldalt GitHub Pages szolgálja ki:
minden `main`-re érkező push után pár tíz másodpercen belül frissül az oldal, nincs vele
további dolgod. Ami *nem* automatikus, az a böngésződ és a repó közötti két lépés.

Ha az oldal nem GitHub Pages-en, hanem hagyományos webtárhelyen fut, akkor az utolsó lépés
is kézi: a letöltött `index.html`-t oda kell feltölteni (FTP-vel vagy a szolgáltató
fájlkezelőjével), a repó ilyenkor csak a változások nyilvántartása.

### Feltöltés a gyakorlatban

**Telefonról (a legegyszerűbb út):**

1. Koppints a **⬇ Mentés fájlba** gombra → letöltődik egy `index.html`.
2. Nyisd meg a repót: `github.com/baknandor/receptek`
3. **Add file → Upload files**, húzd/válaszd be a letöltött `index.html`-t.
4. **Commit changes.** A régi fájlt felülírja — ugyanaz a név, ezért nem lesz kettő belőle.
5. Fél perc múlva töltsd újra az élő oldalt.
6. Ha minden a helyén van, a **🧹 Módosítások eldobása** gombbal kipucolhatod a
   böngésző tárolóját — a receptek megmaradnak, hiszen már a fájlban vannak.

A 6. lépés nem kötelező: a program felismeri, ha egy helyben tárolt recept már bekerült a
feltöltött fájlba, és ilyenkor a helyi másolatot elhagyja, tehát **nem duplázódnak** a receptek.
A 🧹 csak azért hasznos, hogy a „van mentendő módosításod" jelzés eltűnjön.

**Gépről:**

```bash
# a letöltött index.html-t a repó gyökerébe másolva:
git add index.html
git commit -m "Receptek frissítése"
git push
```

---

## Lehet ezt teljesen automatikussá tenni?

Igen, de mindegyik megoldás hoz magával valamit. Statikus fájl önmagában nem tud a
tárhelyre írni: ahhoz kell egy hitelesítés (token), azt pedig **nem lehet magába a publikus
`index.html`-be beírni** — aki megnyitja az oldalt, elolvashatja, és a GitHub az ilyen
tokent amúgy is azonnal visszavonja.

| Megoldás | Hogyan | Mit kér | Kinek jó |
|---|---|---|---|
| **A. Kézi feltöltés** (a mostani) | ⬇ mentés → upload | semmit | ha ritkán módosítasz |
| **B. Token a böngészőben** | egyszer beírsz egy GitHub tokent, utána egy „☁ Feltöltés" gomb a GitHub API-val magától commitol | fine-grained token, csak erre a repóra, `contents: write` | ha egyedül szerkeszted, a saját telefonodon/gépeden |
| **C. Köztes szolgáltatás** | egy Cloudflare Worker / Netlify Function tartja a tokent, az oldal csak jelszót küld neki | ~30 sor plusz kód és egy ingyenes fiók | ha többen szerkesztitek |
| **D. Adatbázis** (Supabase, Firebase) | a receptek nem a fájlban, hanem adatbázisban élnek | átírt adatkezelés | ha valós idejű közös szerkesztés kell |

**Amit javaslok:** ha egyedül és a saját eszközödön szerkesztesz, a **B** a jó választás —
attól még megmarad a kézi mentés is tartaléknak, és az oldal offline jellege sem sérül.
A **D** ezzel szemben elvenné a gyűjtemény legjobb tulajdonságát: hogy internet nélkül is
megnyílik, mert a receptekért hálózaton kellene menni.

A **B** változatnál érdemes tudni: a token a böngésző `localStorage`-ában marad, tehát a
publikus oldal HTML-jébe soha nem kerül bele, viszont aki hozzáér a feloldott telefonodhoz,
ki tudja olvasni. Ezért kell szűk jogosultságú (csak erre az egy repóra írási jogot adó),
lejárati idővel ellátott tokent használni.

Ha kell, ez a „☁ Feltöltés" gomb megírható — szólj, és megcsinálom.

---

## Az adatszerkezet

Egy recept a `RECIPES` tömbben:

```js
{
  id: 10000,                       // egyedi szám; a saját recepteké 10000-től indul
  title: "Almás pite",
  category: "Édességek, sütemények",
  url: "",                         // eredeti oldal linkje, ha van
  source: "Saját recept",
  ingredients: ["500 g liszt"],    // tömb, soronként egy tétel
  steps: ["180 fokon 30 perc."],   // tömb, soronként egy lépés
  image: "",                       // kép URL-je, ha van (megjelenítéshez net kell)
  note: "",
  status: "full",                  // "full" = teljes szöveg, "stub"/"fetch" = csak link
  offline: true                    // van-e teljes szöveg net nélkül
}
```

A saját receptek azért kapnak 10000-től induló `id`-t, hogy soha ne akadjanak össze a
beépített receptek (1–128) számaival.
