# Wenable.se – Claude-kontext
> Starta Claude Code i denna katalog (`~/Utveckling/wenable.se`) för allt arbete med webbplatsen.
> Detta är ett **eget projekt**, skilt från Home Assistant-kontexten.

Publik marknadsföringssajt för **Wenable Consulting AB** (specialistkonsulter inom AI, cybersäkerhet och mjukvaruutveckling, Göteborg). Ren statisk HTML/CSS – **ingen JavaScript, ingen backend, ingen databas**.

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

```
index.html                 Startsida (CSS-animerad rubrik "We enable your …")
expertise.html             Expertis, utvalda uppdrag, FAQ
for-specialists.html       För specialister
about.html                 Teamet + Trust & commitments (#trust)
assignments.html           Uppdrag (statisk HTML, en brief per referensnummer, se nedan)
assets/v2.css              All CSS
assets/icons.svg           Ikon-sprite (Lucide), används via <use href="assets/icons.svg#namn">
assets/img/                Bilder som WebP (assets/img/team/ för porträtt), hero-dots.svg, og-image.png
assets/fonts/              Självhostad Mona Sans (latin + latin-ext)
assets/favicon/, favicon.ico   Ikoner
sitemap.xml, robots.txt    SEO
For Clients.html, For Consultants.html, Wenable are us.html
                           Omdirigeringar (meta refresh) från gamla adresser till de nya sidorna
CNAME                      Pages-domän
```

- Header, kontakt och footer är kopierade i varje sida. Ändra dem på alla fyra sidorna.
- Nya bilder: konvertera till WebP i ungefär 2x visningsstorlek, t.ex.
  `cwebp -q 80 -m 6 -sharp_yuv -resize 1200 0 bild.png -o bild.webp` (logotyper: `-lossless`).
- Interaktivitet utan JS: `<details>` (FAQ), radioknappar (How we work), `popover` (mobilmeny),
  CSS scroll-driven animations (panel-lyft, sektionsintoning). Lägg inte till `<script>`.
- Sajten är **enbart på engelska** (enstaka uppdrag kan vara på svenska med `lang="sv"`).

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
- Referensnummer: `WEN-<år>-<löpnr, 3 siffror>`. Nästa nummer = högsta befintliga löpnr för året + 1
  (senaste: WEN-2026-103).

---

## CSP (Content Security Policy)

Alla sidor har samma CSP i `<meta>`:
`default-src 'self'; script-src 'none'; style-src 'self'; img-src 'self' data:; font-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'`

- Inga `<script>`, inga `style=`-attribut, inga externa resurser (CDN, teckensnitt, API:er) –
  allt hostas lokalt. Lägger du till något externt måste CSP:n uppdateras på alla sidor, annars
  blockeras det tyst.

---

## Designprincip (uttalad av ägaren)

Så mycket **statisk HTML/CSS som möjligt**, minimal JS → mindre attackyta. Sajten har nu ingen JS alls.

---

## Fakta som återkommer

- **Telefon:** +46 70 635 25 24 (`tel:+46706352524`) – på alla sidor, håll i synk.
- **E-post:** `hello@wenable.se` (allmänt), `management@wenable.se` (uppdragsansökningar), `network@wenable.se` (konsultnätverket).
- **Adress / org:** Göteborg · Lindholmspiren 5A · Org.nr 559571-3164. Bolaget är **Göteborg**-baserat (ej Stockholm).
- **LinkedIn:** https://www.linkedin.com/company/wenable-consulting-ab/
- **E-post går via Microsoft 365** – `autodiscover`-CNAME:n hos STRATO ska vara kvar.

---

## Vanliga uppgifter

### Lokal förhandsvisning
```bash
cd ~/Utveckling/wenable.se && python3 -m http.server 8000
```
Öppna `http://localhost:8000/…`. **OBS:** webbläsaren cachar hårt – gör hård omladdning (Cmd+Shift+R) efter CSS-ändringar.

### Deploy
```bash
git add -A && git commit -m "…" && git push origin main
```
Pages bygger om automatiskt (~1 min). Verifiera live med `curl -sI https://wenable.se`.

---

## Arkiv: v1-sajten

Den gamla sajten (statisk HTML + JS med SV/EN-växlare och admin-inloggning mot Supabase) finns
kvar i git som taggen **`v1-archive`**: `git checkout v1-archive` för att titta på den.

- Supabase-projektet (`kvhfqedtjtlzrgmajuqq.supabase.co`, tabell `public.assignments`) finns kvar
  men används inte av sajten sedan 2026-10-26. Uppdragen flyttades därifrån till HTML.
- En lokal kopia av det tidigare Lovable-utkastet kan finnas i `~/Utveckling/wenable.se v2`
  (inte under versionshantering, inte källan till den publicerade sajten).

---

## Fallgropar

- **Filnamn med mellanslag** (`For Clients.html`, `Wenable are us.html`) – citera alltid i shell.
- **Ändra inte** `autodiscover`-CNAME (e-post) eller `CNAME`-filen (domän).
- **`git` + iCloud krockar** – repot ligger därför i `~/Utveckling`, inte i iCloud-mappen. Håll det så.
- **Allt i repots rot serveras publikt** av GitHub Pages (även `CLAUDE.md`). Lägg aldrig hemligheter i repot.
- Header/kontakt/footer är kopierade per sida – missa inte någon sida vid ändring.
