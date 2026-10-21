# PsyInsomnia — ghid tehnic (folderul `dev/` este opțional)

## Structura (toate fișierele în același folder, pentru GitHub Pages)
| Fișier | Rol | Adresă pe site |
|---|---|---|
| `index.html` | Site-ul principal (HTML, CSS, JS, fonturi și Chart.js incluse) | `/` |
| `isi.html` | Chestionarul ISI — ascuns (`noindex`, nelegat de nicăieri) | `/isi` |
| `va.html` | VA Master Sleep Diary Calculator — ascuns | `/va` |
| `confidentialitate.html`, `termeni.html` | Pagini juridice | `/confidentialitate`, `/termeni` |
| `404.html` | Pagina de eroare (GitHub Pages o folosește automat) | — |
| `.nojekyll` | Oprește procesarea Jekyll (fișier gol, trebuie să rămână) | — |
| `dev/` | Teste și unelte | — |

Toate paginile sunt autonome: fără scripturi, stiluri sau fonturi de la terți. Singura cerere externă este formată din cele 5 miniaturi YouTube din „Resurse". Datele utilizatorului rămân în browser, iar politica CSP interzice trimiterea lor în afară (`connect-src 'none'`).

## Publicare pe GitHub Pages
1. Creează un depozit și încarcă fișierele de mai sus la rădăcină. **Nu încărca `dev/`** dacă nu vrei ca testele să fie publice (nu conțin nimic sensibil, doar zgomot).
2. *Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)`*.
3. Domeniu propriu (recomandat): *Settings → Pages → Custom domain*, apoi bifează *Enforce HTTPS*. GitHub creează singur fișierul `CNAME`.
4. Instrumentele ascunse sunt la `adresa-ta/isi` și `adresa-ta/va`, **fără slash la final**. `adresa-ta/isi/` dă 404 pe GitHub Pages.

**Limitări GitHub Pages**
- Nu poți seta antete HTTP. CSP funcționează prin tag-ul `<meta>` din pagini, iar `noindex` este tot în pagini. Nu se pot aplica `frame-ancestors` (protecție împotriva încadrării site-ului în alte pagini) și `X-Robots-Tag`; impactul este mic. Dacă muți site-ul pe Netlify, Cloudflare Pages sau un server propriu, adaugă antetele `Content-Security-Policy: frame-ancestors 'none'`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin` și (doar pe HTTPS) `Strict-Transport-Security`.
- `404.html` folosește linkuri relative la rădăcină (`/`). Este corect cu domeniu propriu sau cu un site de tip `utilizator.github.io`. Dacă folosești adresa de proiect `utilizator.github.io/depozit/`, înlocuiește în `404.html` `href="/` cu `href="/depozit/`.

## Rulare teste (opțional, local)
```bash
cd dev
npm install        # o singură dată
npm test           # toate verificările (≈1 minut), inclusiv Chromium real
npm run test:fast  # fără browser real
```
`npm test` pornește singur un server local care se comportă ca GitHub Pages (`/isi` → `isi.html`, `404.html` pentru adrese inexistente) și rulează: teste de comportament și de valori calculate (jsdom), verificări statice (linkuri interne, ancore, id-uri, afirmațiile din politica de confidențialitate comparate cu codul), validare HTML, verificarea hash-urilor CSP și teste în Chromium real (mobil și desktop, tastatură, descărcări JSON/CSV, import, grafic, CSP activ). Codul de ieșire este nenul dacă ceva eșuează.

## Reguli la modificarea paginilor
1. **După orice modificare a unui `<script>` inline, rulează `npm run csp`** (din `dev/`), care recalculează hash-urile din meta CSP. Fără asta, browserul blochează scriptul modificat și instrumentele nu mai merg. `npm test` te avertizează.
2. Nu adăuga scripturi, fonturi sau imagini de la alte domenii și nu folosi atribute inline de tip `onclick=`: CSP le blochează. Pentru o imagine externă nouă, adaugă domeniul în `img-src` din `dev/tools/finalize_csp.py`.
3. Cheile din localStorage încep cu `cbti_` (și `psyinsomnia_notice_ok`). Exportul JSON are `version: 1`; importul refuză versiuni mai noi.
4. Linkurile dintre pagini folosesc `.html` (`termeni.html`) și `./` pentru pagina principală, deci merg și la deschiderea locală sau pe alt server.

## Lista de verificare înainte de lansare
- [ ] Completează în `confidentialitate.html` și `termeni.html` cele 3 câmpuri galbene „[de completat: …]" (forma juridică, CUI/CIF, adresa), apoi rulează `npm run csp` și `npm test`.
- [ ] Verifică adresele de contact din footer și din politică.
- [ ] Adaugă `<link rel="canonical">`, `og:url` și `og:image` când ai domeniul final și o imagine de previzualizare, apoi rulează `npm run csp`.
- [ ] Deschide site-ul publicat pe un telefon real (iOS Safari și Android Chrome): jurnal, calculator, export, `/isi`, `/va`.
- [ ] Dă paginile juridice la citit unui jurist.
