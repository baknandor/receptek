# Receptgyűjtemény

Egyfájlos, offline is működő receptgyűjtemény (`index.html`). Nincs hozzá szerver, adatbázis
és build-lépés: a receptek magában a HTML-fájlban vannak, a `<script id="data">` blokk
`const RECIPES = [...]` tömbjében.

**Funkciók:** keresés, kategóriaszűrés, offline receptek szűrése, receptek szerkesztése,
új receptek felvitele, a módosítások **feltöltése egy gombbal** a tárhelyre, illetve kimentése fájlba.

---

## Szerkesztés és új recept felvitele

| Művelet | Hol találod |
|---|---|
| Meglévő recept módosítása | Nyisd meg a receptet → **✏️ Szerkesztés** |
| Új recept felvitele | A lista fölötti **➕ Új recept** gomb |
| Saját recept törlése | A saját recept szerkesztő paneljén → **🗑 Recept törlése** |
| **Közzététel egy gombbal** | **☁ Feltöltés a tárhelyre** (csak ha van módosítás) |
| GitHub token megadása/törlése | **🔑** gomb a feltöltés mellett |
| Módosítások kimentése fájlba | **⬇ Mentés fájlba** (csak ha van módosítás) |
| Helyi módosítások eldobása | **🧹 Módosítások eldobása** |

Szerkeszthető mezők: cím, kategória (új kategória is felvehető), hozzávalók, elkészítés,
megjegyzés, eredeti link. A hozzávalókat és a lépéseket **soronként egy tétel** formában
kell megadni. Ha egy recepthez bekerül hozzávaló vagy lépés, automatikusan „✓ Offline"
jelölést kap. A saját recepteket a **★ Saját** címke jelzi.

Csak a saját receptek törölhetők — a beépítettek nem, azokat legfeljebb átírni lehet.

---

## Hogyan kerül fel a módosítás a tárhelyre?

### A rövid válasz

A szerkesztés először **csak a böngésződben** él: a módosításokat a `localStorage` őrzi,
két kulcs alatt (`receptek_overrides_v1` = meglévő receptek módosításai,
`receptek_uj_receptek_v1` = a felvitt új receptek). Ez addig azt jelenti, hogy más eszközön
nem látszik, és a böngésző adatainak törlésével elveszne.

Innen kétféleképpen lehet közzétenni:

| | **☁ Feltöltés a tárhelyre** | **⬇ Mentés fájlba** |
|---|---|---|
| Mit csinál | egyenesen a repóba commitol a GitHub API-val | letölt egy kész `index.html`-t |
| Hány lépés | egy koppintás | letöltés + kézi feltöltés |
| Mit kér egyszer | egy GitHub tokent (lásd lentebb) | semmit |
| Kell hozzá net | igen | csak a feltöltéshez |

A két út ugyanazt a fájlt állítja elő — a **⬇ Mentés fájlba** végig megmarad tartaléknak,
például ha nincs kéznél a token, vagy ha nem GitHub Pages-re publikálsz.

### A teljes útvonal

```
böngésző (localStorage)  →  index.html  →  git repó  →  élő oldal
  szerkesztés / új recept                               GitHub Pages

  ☁ Feltöltés:      [───────── egy gomb ─────────]  →  [automatikus]
  ⬇ Mentés fájlba:  [ kézi ]     [ kézi feltöltés ]  →  [automatikus]
```

Az utolsó lépés — repó → élő oldal — **automatikus**, ha az oldalt GitHub Pages szolgálja ki:
minden `main`-re érkező push után pár tíz másodpercen belül frissül az oldal.

Ha az oldal nem GitHub Pages-en, hanem hagyományos webtárhelyen fut, a ☁ gomb a repót
frissíti, de az élő oldalt nem — oda a letöltött `index.html`-t kell kitenni (FTP-vel vagy a
szolgáltató fájlkezelőjével).

### A ☁ feltöltés beállítása (egyszeri)

1. GitHubon: **Settings → Developer settings → Personal access tokens →
   Fine-grained tokens → Generate new token**
2. **Repository access:** *Only select repositories* → `receptek` — semmi más.
3. **Permissions → Repository permissions → Contents:** *Read and write*.
   Több jog nem kell.
4. Adj neki lejárati időt (pl. 90 nap), és másold ki a tokent — csak egyszer mutatja meg.
5. Az oldalon szerkessz valamit, majd koppints a **🔑** gombra, és illeszd be a tokent.
   Ettől kezdve a **☁ Feltöltés a tárhelyre** magától commitol.

A token **ennek a böngészőnek** a `localStorage`-ában marad, és soha nem kerül bele a
feltöltött `index.html`-be (a fájlt a program a DOM-ból építi, a token pedig nincs a DOM-ban).
Viszont aki hozzáfér a feloldott telefonodhoz, ki tudja olvasni — ezért fontos a szűk
jogosultság és a lejárati idő. A tokent a **🔑** gombbal bármikor lecserélheted; ha üresen
hagyod a mezőt, törlődik. Lejárt tokennél a program magától törli, és szól, hogy adj újat.

Feltöltés után az oldal kb. egy perc múlva frissül. Ha újratöltöd, a program észreveszi,
hogy a helyi módosítások már bekerültek a fájlba, és **magától kipucolja a böngésző
tárolóját** — ezért tűnnek el a gombok. Duplikáció nem keletkezik.

### A kézi út (ha nincs token, vagy nem GitHub Pages a tárhely)

**Telefonról:**

1. Koppints a **⬇ Mentés fájlba** gombra → letöltődik egy `index.html`.
2. Nyisd meg a repót: `github.com/baknandor/receptek`
3. **Add file → Upload files**, húzd/válaszd be a letöltött `index.html`-t.
4. **Commit changes.** A régi fájlt felülírja — ugyanaz a név, ezért nem lesz kettő belőle.
5. Fél perc múlva töltsd újra az élő oldalt.

A **🧹 Módosítások eldobása** gombra itt sincs szükség: újratöltéskor a program felismeri,
hogy a helyi másolatok már a fájlban vannak, és magától elhagyja őket — a receptek nem
duplázódnak, a jelzés pedig eltűnik. A 🧹 arra való, ha *el akarod dobni* a helyi
módosításokat anélkül, hogy közzétennéd őket.

**Gépről:**

```bash
# a letöltött index.html-t a repó gyökerébe másolva:
git add index.html
git commit -m "Receptek frissítése"
git push
```

---

## Miért kell token, és mi lenne még lehetséges?

Statikus fájl önmagában nem tud a tárhelyre írni: ahhoz kell egy hitelesítés. Azt viszont
**nem lehet magába a publikus `index.html`-be beírni** — aki megnyitja az oldalt,
elolvashatná, és a GitHub az így kiszivárgott tokent amúgy is azonnal visszavonja. Ezért
kerül a token a böngésző tárolójába, ahol csak te férsz hozzá.

| Megoldás | Hogyan | Mit kér | Kinek jó |
|---|---|---|---|
| **A. Kézi feltöltés** | ⬇ mentés → upload | semmit | ha ritkán módosítasz, vagy nincs kéznél token |
| **B. Token a böngészőben** ✅ *ez van beépítve* | ☁ gomb, GitHub Contents API | fine-grained token, csak erre a repóra, `Contents: write` | ha egyedül szerkeszted, a saját eszközödön |
| **C. Köztes szolgáltatás** | egy Cloudflare Worker / Netlify Function tartja a tokent, az oldal csak jelszót küld neki | ~30 sor plusz kód és egy ingyenes fiók | ha többen szerkesztitek, és nem akartok fejenként tokent |
| **D. Adatbázis** (Supabase, Firebase) | a receptek nem a fájlban, hanem adatbázisban élnek | átírt adatkezelés | ha valós idejű közös szerkesztés kell |

A **B** azért lett a választás, mert nem sérti az oldal legjobb tulajdonságát: a receptek
továbbra is magában a fájlban vannak, tehát internet nélkül is megnyílnak. A **D** ezt
elvenné, mert a receptekért hálózaton kellene menni.

A **C** akkor éri meg, ha többen szerkesztenétek: ott a token egy szerveren ül, a
szerkesztők csak jelszót kapnak, és visszavonni is elég egy helyen. Szólj, ha erre lenne
szükség.

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
