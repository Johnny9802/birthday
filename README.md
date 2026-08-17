# Happy (fake) birthday, Kelly

Biglietto d'auguri in una pagina sola. Quattro atti in sequenza automatica: l'ammissione
di colpa, il panda che prova a fare una torta, la torta vera in SVG con le candeline, gli auguri.

Parte da solo al caricamento. L'unica interazione prevista è un bottone **Play** di riserva,
che compare solo se il browser rifiuta l'autoplay del video (capita su alcuni iOS).

**Durata totale: ~44 secondi** (~18 secondi con `prefers-reduced-motion: reduce`, che salta
il video, tiene le fiamme ferme e accorcia la timeline).

---

## Struttura

```
kelly-birthday/
├── index.html          44 KB   tutto inline: CSS, JS, torta in SVG
├── README.md
└── assets/
    ├── panda.mp4      1,9 MB   H.264 main / yuv420p / faststart / senza audio
    ├── panda.webm     1,6 MB   VP9 / senza audio
    ├── poster.jpg      72 KB   fotogramma finale, usato come poster e og:image
    └── fonts/         108 KB   Cormorant Garamond (roman + italic) + Jost, subset latin
```

**Totale: 3,7 MB.** Tutti i percorsi sono relativi: funziona in qualsiasi sottocartella,
e funziona anche col doppio clic su `index.html` senza server.

Font self-hostati, nessuna chiamata a Google Fonts o a qualsiasi altro host esterno.
Sono i file variabili ufficiali, subset latin, dichiarati con `@font-face` locali e
`font-display: swap`: la tipografia regge anche dalla Cina continentale.

---

## Video

Sorgente: `../video/mi_generi_un_video_tipo_carto.mp4` — 1280×720, 24 fps, 10,0 s, **2,4 MB**
(H.264 + traccia audio AAC).

Dieci secondi sono già nell'intervallo ideale, quindi non è stato tagliato niente.
Era già a 720p e a 24 fps, quindi risoluzione e framerate sono rimasti invariati.

| file | prima | dopo |
|---|---|---|
| MP4 (H.264) | 2,4 MB con audio | **1,9 MB** senza audio |
| WebM (VP9)  | —                | **1,6 MB** |
| poster.jpg  | —                | **72 KB** |

Comandi usati:

```sh
# MP4 — H.264 main profile, faststart, audio rimosso
ffmpeg -i mi_generi_un_video_tipo_carto.mp4 -an \
  -c:v libx264 -profile:v main -level 4.0 -preset veryslow -crf 24 \
  -pix_fmt yuv420p -vf "scale=1280:720:flags=lanczos" \
  -movflags +faststart assets/panda.mp4

# WebM — VP9
ffmpeg -i mi_generi_un_video_tipo_carto.mp4 -an \
  -c:v libvpx-vp9 -b:v 0 -crf 34 -row-mt 1 -tile-columns 2 \
  -deadline good -cpu-used 1 -pix_fmt yuv420p assets/panda.webm

# poster
ffmpeg -ss 9.45 -i mi_generi_un_video_tipo_carto.mp4 -frames:v 1 -q:v 4 assets/poster.jpg
```

L'MP4 è il primo `<source>`: Safari iOS lo prende subito senza scaricare il WebM.

---

## Anteprima su WhatsApp e Telegram

`og:url` e `og:image` sono già assoluti e puntano a GitHub Pages:

```html
<meta property="og:url"   content="https://johnny9802.github.io/birthday/">
<meta property="og:image" content="https://johnny9802.github.io/birthday/assets/poster.jpg">
```

WhatsApp pretende un URL assoluto per l'immagine di anteprima, quindi **se sposti il sito
altrove vanno cambiate tutte e due**, sostituendo solo l'host.

---

## Pubblicazione

Il sito è già pronto per `https://johnny9802.github.io/birthday/`.
Tutti i percorsi sono relativi, quindi funziona anche se lo sposti in una sottocartella:
l'unica cosa legata al dominio sono i due meta tag qui sopra.

### GitHub Pages — repo `Johnny9802/birthday`

> **Il repo deve essere pubblico.** Su un account gratuito GitHub Pages non pubblica da un
> repo privato: la voce *Pages* nelle impostazioni resta disattivata. Il codice non è
> sensibile, quindi renderlo pubblico va benissimo — ma va fatto, altrimenti il sito non esce.

```sh
cd kelly-birthday
git init -b main
git add .
git commit -m "Happy (fake) birthday, Kelly"
git remote add origin https://github.com/Johnny9802/birthday.git
git push -u origin main

# rendi pubblico il repo e attiva Pages da main / root
gh repo edit Johnny9802/birthday --visibility public --accept-visibility-change-consequences
gh api -X POST repos/Johnny9802/birthday/pages -f 'source[branch]=main' -f 'source[path]=/'
```

A mano, senza `gh`: *Settings → General → Change visibility → Public*, poi
*Settings → Pages → Source: Deploy from a branch → main / (root)*.

Il primo build richiede uno o due minuti. Poi: **https://johnny9802.github.io/birthday/**

- **Limite dimensione file:** 100 MB per file (oltre i 50 MB Git avvisa), 1 GB consigliato
  per il repo, ~100 GB/mese di banda. Il nostro file più grosso è 1,9 MB.
- **Da Hong Kong:** sì, `github.io` è servito da Fastly e si raggiunge senza problemi.
  Dalla Cina continentale è spesso irraggiungibile — vedi sotto.

---

## Le altre due opzioni

Restano valide se GitHub Pages non ti convince. In particolare **Cloudflare Pages è
sensibilmente più veloce da Hong Kong** (ha un POP proprio lì, contro il POP Fastly che
serve `github.io`), ed è l'opzione tecnicamente migliore per questo caso d'uso.

### Netlify Drop — link in trenta secondi, senza account

Vai su **https://app.netlify.com/drop** e trascina dentro la cartella `kelly-birthday`.
Il link (`https://qualcosa-a-caso-123456.netlify.app`) è attivo appena finisce l'upload.

Senza account il sito resta online circa un'ora; registrandoti gratis (anche dopo, dallo
stesso schermo, con il bottone *Claim your site*) diventa permanente e puoi rinominare il
sottodominio.

Da riga di comando, se preferisci:

```sh
npm install -g netlify-cli
cd kelly-birthday
netlify deploy --dir=. --prod
```

- **Limite dimensione file:** 25 MB per singolo file. Il nostro file più grosso è 1,9 MB.
- **Da Hong Kong:** sì, `*.netlify.app` è raggiungibile senza problemi, con POP asiatici.
  Dalla Cina continentale funziona ma è lento e a tratti instabile.

### Cloudflare Pages — la migliore come latenza dall'Asia

```sh
cd kelly-birthday
npx wrangler login
npx wrangler pages project create birthday --production-branch main
npx wrangler pages deploy . --project-name birthday
```

Il link è `https://birthday.pages.dev`.

In alternativa, senza riga di comando: dashboard Cloudflare → **Workers & Pages** →
*Create* → *Pages* → *Upload assets*, e trascini la cartella.

- **Limite dimensione file:** 25 MB per singolo file, 20.000 file per deploy.
- **Da Hong Kong:** sì, ed è la più veloce delle tre. `*.pages.dev` dalla Cina continentale
  è spesso bloccato: se serve anche lì, ci vuole un dominio proprio.

Se passi di qui, ricordati di aggiornare `og:url` e `og:image` in `index.html`.

---

## Provarlo in locale

Doppio clic su `index.html` basta. Se vuoi controllare i percorsi esattamente come li
vedrà il server:

```sh
cd kelly-birthday
python3 -m http.server 8777
# http://127.0.0.1:8777/index.html
```
