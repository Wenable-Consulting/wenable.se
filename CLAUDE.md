# Wenable.se – Claude-kontext
> Starta Claude Code i denna katalog (`~/Utveckling/Wenable.se`) för allt arbete med webbplatsen.
> Detta är ett **eget projekt**, skilt från Home Assistant-kontexten.

Publik marknadsföringssajt för **Wenable Consulting AB** (specialistkonsulter inom AI & cybersäkerhet, Göteborg). Statisk HTML/CSS/JS byggd med Claude Design, med admin-inloggning + uppdragshantering via Supabase.

---

## Drift & hosting

| Del | Detalj |
|-----|--------|
| **Repo** | https://github.com/Wenable-Consulting/wenable.se (publikt, organisationen Wenable-Consulting), remote `origin`, gren `main` |
| **Hosting** | GitHub Pages (Deploy from branch: `main` / root). **Ingen byggsteg** – ren statisk HTML |
| **Domän** | wenable.se, registrerad hos **STRATO** (DNS hanteras i STRATOs panel) |
| **HTTPS** | Let's Encrypt, utfärdas automatiskt av GitHub. Enforce HTTPS på |
| **Deploy** | `git push origin main` → Pages bygger om automatiskt (~1 min). Inget mer behövs |

### DNS hos STRATO (så här är det satt)
| Typ | Namn | Värde |
|-----|------|-------|
| A | `@` | `185.199.108.153` (GitHub Pages) |
| AAAA | `@` | `2606:50c0:8000::153` |
| CNAME | `www` | `wenable-consulting.github.io.` |
| CNAME | `autodiscover` | `autodiscover.outlook.com.` — **Microsoft 365-e-post, RÖR EJ** |

Filen `CNAME` i repot (`wenable.se`) binder domänen till Pages – ta inte bort den.

---

## Struktur

v2 (ny design, portad från Lovable till ren HTML/CSS – **ingen JavaScript**):

```
index.html                 Startsida (CSS-animerad rubrik "We enable your …")
expertise.html             Expertis, utvalda uppdrag, FAQ
for-specialists.html       För specialister
about.html                 Teamet + Trust & commitments (#trust)
assignments.html           Uppdrag (statisk HTML, en brief per referensnummer, se nedan)
assets/v2.css              All CSS för v2-sidorna
assets/icons.svg           Ikon-sprite (Lucide), används via <use href="assets/icons.svg#namn">
assets/img/                v2-bilder som WebP (assets/img/team/ för porträtt), hero-dots.svg
assets/fonts/              Självhostad Mona Sans (samma filer som Google Fonts, latin + latin-ext)
sitemap.xml, robots.txt    SEO
For Clients.html, For Consultants.html, Wenable are us.html
                           Omdirigeringar (meta refresh) till de nya sidorna
```

- Header, kontakt och footer är kopierade i varje v2-sida. Ändra dem på alla fyra sidorna.
- Nya bilder: konvertera till WebP i ungefär 2x visningsstorlek, t.ex.
  `cwebp -q 80 -m 6 -sharp_yuv -resize 1200 0 bild.png -o bild.webp` (logotyper: `-lossless`).
- Interaktivitet utan JS: `<details>` (FAQ), radioknappar (How we work), `popover` (mobilmeny),
  CSS scroll-driven animations (panel-lyft, sektionsintoning). Lägg inte till `<script>` –
  v2-sidornas CSP har `script-src 'none'` och inga `style=`-attribut tillåts.

v1 (finns kvar):

```
Wenable CV.html            CV-mall (demo: Erik Lindqvist)
assets/
  wenable.css, site.css    v1-sidstilar (mycket CSS ligger även inline i <style> per sida)
  admin.js                 Login + uppdrag-CRUD mot Supabase (se nedan)
  i18n.js                  Språkväxling SV/EN (se nedan)
  bg-network.js            Animerad bakgrund (canvas)
  vendor/supabase.js       Supabase-JS, VENDORAT lokalt (global window.supabase)
CNAME, DEPLOY.md           Pages-domän + driftsättningsguide
```

---

## Uppdrag (statisk HTML) – så lägger du till, ändrar och tar bort

Uppdragen ligger direkt i HTML:en. Det finns ingen inloggning och ingen databas på sajten.
Ändra i repot och pusha, så är ändringen live.

Två ställen:

1. `assignments.html` – alla öppna uppdrag, senaste överst.
2. `for-specialists.html` – förhandsvisning av de **3 senaste**, plus raden
   "+N more assignment(s)". Håll den i synk med `assignments.html`.

Varje uppdrag består av två delar i `.assignment-board`, **i samma ordning**:

```html
<!-- 1. Fliken i listan (inuti <nav class="assignment-list">) -->
<a class="assignment-tab" href="#WEN-2026-104">Titel</a>

<!-- 2. Briefen (efter .assignment-sidebar). id = referensnumret. -->
<article class="assignment-detail" id="WEN-2026-104"><div class="assignment-detail-content">
  <span class="assignment-example">WEN-2026-104 · IT</span>   <!-- IT | AI | CyberSecurity -->
  <h3>Titel</h3>
  <p>Ingress (en eller flera <p>).</p>
  <dl><div><dt>Stack</dt><dd>…</dd></div><div><dt>Start</dt><dd>…</dd></div><div><dt>Location</dt><dd>…</dd></div></dl>
  <div class="assignment-full-brief">            <!-- bara i assignments.html -->
    <div class="assignment-brief-section"><h4>Rubrik</h4><p>…</p><ul><li>…</li></ul></div>
  </div>
  <div class="assignment-actions">
    <a href="mailto:management@wenable.se?subject=WEN-2026-104" class="action-link primary">Apply / Discuss …</a>
  </div>
</div></article>
```

- Första uppdraget visas tills någon klickar. Klick sätter `#REF` i adressen (`:target`), så
  `assignments.html#WEN-2026-104` är en direktlänk.
- CSS:en (`.assignment-board:has(...:nth-of-type(k):target)`) räcker för **högst 20** uppdrag.
- Svensk brief: lägg `lang="sv"` på `<article>`.
- Tillsatt uppdrag: ta bort både fliken och briefen. Uppdatera förhandsvisningen.
- Uppdragen flyttades hit från Supabase 2026-10-26 (4 öppna: WEN-2026-103, -092, -021, -102).

## Tvåspråkighet (i18n) – VIKTIGT arbetsmönster

Engelska är **standardtexten** i HTML:en. Svenskan ligger i `data-sv`-attribut och växlas av `i18n.js` när användaren klickar SV/EN.

```html
<h2 data-sv="Svensk text">English text</h2>
<input data-sv-attr-placeholder="Svensk platshållare" placeholder="English placeholder">
```

- **Ändra svenska** → redigera `data-sv`-attributet.
- **Ändra engelska** → redigera elementets synliga text (och kontrollera att `data-sv` fortfarande stämmer).
- Håll siffror/fakta i synk mellan SV och EN. Bolaget är **Göteborg**-baserat (ej Stockholm – en sådan felöversättning har redan rättats en gång).
- Undvik "calque" (direktöversatt engelska) i svenskan – skriv idiomatiskt.

---

## Admin-login + uppdrag (Supabase) – v1, används inte längre av sajten

> Sedan v2 (2026-10-26) läser ingen sida från Supabase. Avsnittet beskriver v1 och projektet finns kvar.
> `assets/admin.js`, `i18n.js`, `bg-network.js`, `vendor/supabase.js`, `wenable.css` och `site.css` används inte av v2-sidorna.


Statisk sajt kan inte ha egen backend, så **Supabase** sköter inloggning + delad databas. All logik ligger i `assets/admin.js`.

### Supabase-projekt
- **URL:** `https://kvhfqedtjtlzrgmajuqq.supabase.co`
- **Nyckel i koden:** `sb_publishable_…` (publishable/anon) – ligger överst i `assets/admin.js`. **Säker att vara publik** eftersom RLS är på; ger bara läsning av uppdrag + skrivning som inloggad. Använd **ALDRIG** `service_role`/`sb_secret_…` i frontend.
- **Tabell:** `public.assignments`
  - kolumner: `id uuid`, `title`, `category` (`ai`|`cyber`|`it`), `status` (`open`|`filled`), `location`, `start`, `lede`, `work`, `ref`, `tags text[]`, `created_at`
  - **OBS:** kolumnerna `duration` och `rate` finns kvar i DB men är **borttagna ur UI** (formulär + visning). Oanvända, ofarliga.
- **RLS-policies:** public `select`; `authenticated` för `insert`/`update`/`delete`.
- **Auth:** en admin-användare (e-post + lösenord), självregistrering avstängd.

### Admin-flöde
Login-knapp i menyn → logga in med Supabase-admin (e-post + lösenord) → på `assignments.html` visas **Lägg till / Redigera / Ta bort**. Uppdrag sparas i Supabase och syns direkt för alla. Uppdrag hanteras via **live-sajten**, inte i kod.

### Återställa/bekräfta admin-konto (Supabase SQL Editor)
```sql
-- nytt lösenord (utan e-postutskick):
update auth.users set encrypted_password = crypt('nytt-lösenord', gen_salt('bf'))
where email = 'admin@…';
-- bekräfta konto (om "Email not confirmed"):
update auth.users set email_confirmed_at = now() where email = 'admin@…';
```

### admin.js – hur det hänger ihop
- Laddar vendorad `assets/vendor/supabase.js` (global `window.supabase.createClient`) och skapar klienten från `SUPABASE_URL` + `SUPABASE_ANON_KEY` överst i filen.
- Bygger login-modal (e-post/lösenord), nav-login-kontroll, och på uppdragssidan add/edit-modal + rendering av uppdrag i den befintliga `.assign-list` / `.assign-detail-stage`.
- **Tomtillstånd:** `body.wadm-has-assignments` (sätts av admin.js när det finns uppdrag) växlar mellan "Nya uppdrag kommer inom kort" (0 st) och "Välj ett uppdrag i listan" (>0). CSS styr vilket som visas.
- Redigering återanvänder add-modalen (läge växlas via `#wadm-add-mode`/`#wadm-add-submit`, `editingId`).

---

## CSP (Content Security Policy) – VIKTIGT

`For Clients.html`, `For Consultants.html` och `assignments.html` har en strikt CSP i `<meta>`:
`default-src 'self'; script-src 'self'; connect-src 'self' https://*.supabase.co wss://*.supabase.co; …`

- Supabase-libben är därför **vendorad lokalt** (`assets/vendor/supabase.js`) så `script-src 'self'` räcker.
- `connect-src` är öppnad för `*.supabase.co` (REST + auth + realtime).
- **Lägger du till externa resurser (CDN, teckensnitt, API:er) måste CSP:n uppdateras** på dessa sidor, annars blockeras de tyst.

---

## Designprincip (uttalad av ägaren)

Så mycket **statisk HTML/CSS som möjligt**, minimal JS → mindre attackyta. Den egentliga säkerhetsgränsen är **Supabase RLS** (serversidan), inte JS:en – därför är publik anon-nyckel + öppen källkod ofarligt. När en förändring kan göras i HTML/CSS i stället för JS, gör det.

---

## Fakta som återkommer

- **Telefon:** +46 70 635 25 24 (`tel:+46706352524`) – på flera sidor, håll i synk.
- **E-post:** `hello@wenable.se` (och `network@wenable.se` för konsultnätverket).
- **Adress / org:** Göteborg · Lindholmspiren 5A · Org.nr 559571-3164.
- **Personerna på "Wenable are us" är platshållare** – ska bytas mot riktiga.
- **E-post går via Microsoft 365** – `autodiscover`-CNAME:n hos STRATO ska vara kvar.

---

## Vanliga uppgifter

### Lokal förhandsvisning
```bash
cd ~/Utveckling/Wenable.se && python3 -m http.server 8000
```
Öppna `http://localhost:8000/…`. **OBS:** webbläsaren cachar hårt – lägg på `?v=N` som cache-buster när du testar CSS/JS-ändringar.

### Kolla JS-syntax utan Node (Node saknas ofta lokalt)
```bash
osascript -l JavaScript -e 'ObjC.import("Foundation"); \
  var s=$.NSString.alloc.initWithContentsOfFileEncodingError("'"$PWD"'/assets/admin.js",4,null).js; \
  try{ new Function(s); "PARSE_OK" }catch(e){ "ERR: "+e.message }'
```

### Deploy
```bash
git add -A && git commit -m "…" && git push origin main
```
Pages bygger om automatiskt (~1 min). Verifiera live med `curl -sI https://wenable.se`.

---

## Fallgropar

- **Filnamn med mellanslag** (`For Clients.html`, `Wenable are us.html`) – citera alltid i shell.
- **Browser-cache** vid lokal test – använd cache-buster (`?v=N`), inte bara reload.
- **Ändra inte** `autodiscover`-CNAME (e-post) eller `CNAME`-filen (domän).
- **`git` + iCloud krockar** – repot ligger därför i `~/Utveckling`, inte i iCloud-mappen. Håll det så.
- Uppdragen är statisk HTML. Se avsnittet "Uppdrag (statisk HTML)".
