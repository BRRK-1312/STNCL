# STNCL — Geruzaniztun Plantilla Estudioa

```
███████╗████████╗███╗   ██╗ ██████╗██╗
██╔════╝╚══██╔══╝████╗  ██║██╔════╝██║
███████╗   ██║   ██╔██╗ ██║██║     ██║
╚════██║   ██║   ██║╚██╗██║██║     ██║
███████║   ██║   ██║ ╚████║╚██████╗███████╗
╚══════╝   ╚═╝   ╚═╝  ╚═══╝ ╚═════╝╚══════╝
```

**Nabigatzailean oinarritutako plantilla sortzaile aurreratua** — zerbitzaririk gabe, instalaziorik gabe, graffitirako eta arte grafikorako pentsatuta.

Fitxategi HTML bakarra. Irekitzen da. Funtzionatzen du.

---

## Zer da STNCL?

STNCL irudi bat hartu eta inprimatu daitezkeen geruzaniztuneko plantiletan bihurtzeko tresna bat da. Algoritmo guztiak — K-means segmentazioa, bridge detekzioa, irekidura morfologikoa, bektorizazioa, DXF/HPGL eraiketa — nabigatzaileko JavaScript hutsa dira. Ez du backend-ik behar, ez datu-transferentziarik, ez dependentziarik.

---

## Ezaugarriak

### Segmentazioa — K-means LAB espazio pertzeptuala
- **K-means++ hasieraketa** koloreen klusterizaziorako
- **CIELAB kolorean** egiten da kalkulua: ΔE76 distantzia pertzeptuala erabiltzen du RGB ordez. Azal-tonuetan, gradienteetan eta argi/itzal eskualdetan kluster uniformeagoak ematen ditu
- **2–6 geruza** hautatu (luminantziaren arabera ordenatuta automatikoki)
- **Geruza bakoitzeko atalasa** (UMBRAL): ratio-oinarritutako threshold-a
  - `1.0` = K-means exactua
  - `> 1.0` = hedatu (geruza handitu)
  - `< 1.0` = uzkurtu (geruza txikitu)

### Irekidura Morfologikoa (OPEN)
- **Higadura + Dilatazioa** sekuentzia, geruza bakoitzean
- Tinta-fragmentu meheak, pixel isolatuak eta teksura-zarata desagerrarazten ditu
- Isleta txiki gehienak ezabatzen ditu bridge-ak behartu aurretik
- **Filtro separagarria**: O(n×r) konplexutasuna — 1200×900 irudian r=4 izanda ~4.3M eragiketa soilik
- Slider: 0–8px, geruza bakoitzeko, laranja kolorez

### Bridge Motor v4 — Kateatutako Zubia
Irudi baten "islak" paper-zatiak dira tintaz inguratuta, paper nagusitik isolatuta. Estantzila fisiko batean, isla hauek erori egingo lirateke; bridge-ek eusten dituzte.

**Algoritmoa:**
```
1. BFS (4-konekt.) → kanpoko paper konektatua markatu
2. Irlak bildu → handienak lehenik prozesatu (katea bridging)
3. override.skip? → saltu (erabiltzaileak erabakita)
4. 8-konekt. BFS distantzia-mapa result[] erabiliz
   (lehenago egindako korridoreak ikusten ditu → katea bridging)
5. Muga-pixelak bilatu → dist[] kontsulta O(1)
6. Bidea berreraiki prev[] bidez → [x,y] koordinatuak
7. Angelu-banaketa → N zubia kokatzen ditu (area-ren arabera)
8. Chaikin (2 iterazio) → zubia leundu
9. Korridore zirkularra marraztu + isConn markatu
   → hurrengo irlen erreferentzia bihurtzen da
```

**Bridge kontrolak:**
- **Ancho del puente**: 0.5–8mm, estantzilaren lodieraren araberakoa
- **Isla mínima**: irla txikiak tintan bete (ruido ezabatu), geruza bakoitzeko
- **FILL>**: irla handiak tintan bete (% bidez), geruza bakoitzeko
- **OPEN**: irekidura morfologikoa bridge aurretik

### Bridge Edizio Modua ⊕

⊕ botoia sakatu geruza batean → B&N aurrebista agertzen da bridges aplikatuta.

**⬡ ISLA modua** — Irla bakoitzean klik eginez egoera ziklatu:

| Egoera | Marka | Portaera |
|---|---|---|
| auto | ● urdina | Algoritmoak erabakitzen du |
| ✕ gabe | ✕ gorria | Bridge-rik ez, irla flotatzen |
| + behartu | + berdea | Bridge behartu (MAX×3 limitea) |
| ⊗ ezabatu | ⊗ laranja | Irla tintan bete, desagertu |

- **Eskuin klik** → irla automatikora bueltatzen du
- **↺ RESET** → geruza honetako aldaketa guztiak ezabatu

**╱ LÍNEA modua** — Eskuzko bridge-ak marraztu:
- **1. klik**: hasiera-puntua finkatu (puntu urre bat agertzen da)
- **Sagua mugitu**: aurrebista denbora errealean (lerro etetena)
- **2. klik**: korridorea berretsi eta marraztu
- **Eskuin klik**: lerro pendentea bertan behera utzi edo gertukoa ezabatu
- **✕ BORRAR**: azpi-moduan klik eginez lerro manuala ezabatu
- **Ø slider**: korridorearen zabalera (1–40px)

### Bektorizazioa
- **Directed half-edge tracer**: CDN dependentzirik gabe
- **Chaikin closed-loop smoothing**: pixel-eskailera kurbatu bihurtzen du, 2 iterazio
- **Ramer-Douglas-Peucker**: leuntzea (tolerantzia 0.3–7.95px)
- **fill-rule="evenodd"**: zuloak automatikoki kudeatzen dira SVG-an
- Slider: 1 (xehetasun handia) → 10 (sinplifikazio handia)

### Zoom eta Pan
- **Saguaren gurpila** → zoom (15%–1200%), kursorea ardatz bezala
- **Botoi erdiko sagua** edo **Espazio + arrastatu** → pan librea
- **Klik bikoitza** → 1:1 eta pantaila-egokitua arteko txandaketa
- **Kontrol-botoiak**: −, %, +, ⊡ (egokitu), 1:1
- **Leihoa aldatu** → automatikoki egokitzen da

---

## Interfazea

### Ezkerreko panela
| Atala | Kontrolak |
|---|---|
| Irudia | Upload area + bereizmen hautatzailea |
| Segmentazioa | K-means geruza kopurua + ANALIZAR botoia |
| Puentes (islas) | Zabalera, MIN irla, VER BRIDGES |
| Vectorización | Simplifikazio slider |
| Dimensiones | Cm-tan, exportazioarentzat |
| Formato | SVG / DXF / PNG / HPGL toggle-ak + DPI slider |

### Eskuineko panela (geruzak)
Geruza bakoitzak du:
- **⠿⠿** drag & drop atxikitzaile
- **Kolore-hautatzaile** (clic)
- **Miniatura** 32×32px B&N
- **● / ○** ikusgarritasun toggle
- **↑ ↓** ordenatzeko botoiak
- **⊕** Bridge Edizio Modua aktibatu
- **UMBRAL** slider: threshold ratio (0.2–2.5)
- **OPEN** slider: irekidura morfologikoa (0–8px)
- **MIN isle** slider: irla minimoa (4–600px²)
- **FILL>** slider: irla maximoa (0–40% arearen)

Geruzen gainean: **luminantzia histograma** — pixel-banaketaren grafikoa K-means zentroeen lerro koloreztatuekin.

### Headereko botoiak
| Botoia | Funtzioa |
|---|---|
| EU / ES / EN | Interfaze-hizkuntza aldatu |
| ⬛ SIM | Simulatutako spray preview |
| ↩ UNDO | Azken aldaketa desegin (Ctrl+Z) |
| ↓ PROYECTO | Proiektua JSON gisa gorde |
| ↑ CARGAR | Proiektua JSON-etik kargatu |

---

## Lan-fluxua

### Fluxu tipikoa

```
1. Irudia kargatu (arrastatu edo klik)
         ↓
2. Bereizmena hautatu (1200px lehenetsita)
         ↓
3. ANALIZAR sakatu → K-means LAB espazioean
         ↓
4. Geruzak doitu:
   • UMBRAL → kolore-esleipena doitu
   • OPEN → zarata eta fragmentu meheak kendu
   • Kolorea aldatu (color picker)
   • ● / ○ → ikusgarritasuna toggle
   • ↑↓ edo arrastatu → ordena aldatu
         ↓
5. Bridge parametroak doitu (panel ezkerrean):
   • Ancho del puente (mm)
   • VER BRIDGES → aurrebista ikusi
         ↓
6. [Aukerakoa] ⊕ → Bridge Edizio Modua:
   • Isletan klik → auto / ✕ / + / ⊗
   • ╱ LÍNEA → eskuzko korridoreak marraztu
   • Ctrl+Z → akatsak desegin
         ↓
7. Formatua hautatu: SVG / DXF / PNG / HPGL
         ↓
8. DPI hautatu (PNG-rako)
         ↓
9. EXPORTAR ZIP → fitxategiak deskargatu
         ↓
10. [Aukerakoa] ↓ PROYECTO → saioa gorde
```

### Bridge-en parametroen gida praktikoa

| Egoera | Konponbidea |
|---|---|
| Irla txiki ugari (K-means zarata) | OPEN ↑ 2–4px → MIN isle ↑ |
| Aurpegia irla handi gisa (geruza ilunean) | FILL> %10–15 geruza horretan |
| Bridge-ak eskailera-formarekin | v4 Chaikin-ek ebazten du automatikoki |
| Irla sakon bat, bridge-rik gabe | ⊕ → klik bi aldiz (+) → behartu |
| Bridge posizioa egokia ez | ⊕ → (✕) + eskuzko lerroa (╱) |
| Geruza osoa oso konplexua | SIM preview-an egiaztatu, OPEN igo |

---

## Exportazioa

### SVG
- `fill-rule="evenodd"` — zuloek eta konturoak bat egiten dute automatikoki
- Erregistro-markak 4 izkinetan (crosshair 5mm, margen 8mm)
- `width/height` cm-tan, `viewBox` pixel-tan
- Konturoak Chaikin + RDP bidez leunduta

### DXF
- `$INSUNITS = 4` (milimetroak)
- `LWPOLYLINE` flag=1 (itxita) konturo bakoitzeko
- Y-ardatza irauli: SVG y-behera → DXF y-gora
- `LINE` entitateak erregistro-markentzat (`REG` geruza)
- AutoCAD, LibreCAD, Inkscape-rekin bateragarria

### PNG
- B&N (tinta=beltza, papera=zuria)
- Erregistro-markak canvas-ean marraztuta
- **pHYs txunk** injektatuta DPI metadatuekin
- 72–600 DPI slider bidez konfiguratuta
- Tamaina fisikoa zuzen interpretatzen dute inprimatze-programek

### HPGL
- Silhouette Cameo, Cricut, Roland ploterrentzat natibo
- 40 unitate/mm (0.025mm/unitate)
- `IN;SP1;...PU...PD...SP0;IN;` egitura estandarra
- Erregistro-markak barne

### ZIP egitura
```
stncl_export.zip
├── capa_01_a2a2a2.svg
├── capa_01_a2a2a2.dxf
├── capa_01_a2a2a2.png
├── capa_01_a2a2a2.hpgl
├── capa_02_2d2d2d.svg
├── capa_02_2d2d2d.dxf
├── capa_02_2d2d2d.png
└── capa_02_2d2d2d.hpgl
```

Fitxategi-izenak: `capa_NN_RRGGBB.ext` — kolore hexadezimala izenean.

---

## Proiektua gorde eta kargatu

**↓ PROYECTO** → `stncl_proyecto.stncl.json` deskargatu

JSON-ak hauek gordetzen ditu:
- Irudia base64 formatuan
- K-means esleipen guztiak (`kmAssign`)
- Zentroide LAB balioak (`kmCentroids`)
- Geruza konfigurazio osoa (atalasa, open, minIsle, maxIsle, kolorea, ordena)
- Bridge overrides (skip/force/delete)
- Eskuzko lerroak
- Ikusgarritasuna

**↑ CARGAR** → `stncl_proyecto.stncl.json` aukeratu → egoera osoa berrezarri

Saioa itxi eta berriro ireki dezakezu galdu gabe.

---

## Lasterbidesak

| Tekla | Ekintza |
|---|---|
| `Ctrl+Z` / `Cmd+Z` | Desegitea (bridge edizio moduan) |
| `Espazio + arrastatu` | Panoramika (pan) |
| `Botoi erdikoa + arrastatu` | Panoramika (pan) |
| `Gurpila` | Zoom kurtsorea ardatz bezala |
| `Klik bikoitza` | 1:1 ↔ pantaila-egokitua |

---

## Teknikaren xehetasunak

### CIELAB bihurketa
```
sRGB → linear (gamma kendu) → XYZ D65 → CIE LAB
Alderantziz: LAB → XYZ → linear → sRGB (gamma jarri)
Distantzia: ΔE76 = √(ΔL²+Δa²+Δb²)
```

### Irekidura morfologikoa — Filtro separagarria
```
Higadura horizontala (H) → Higadura bertikala (V) → Higadutako maska
Dilatazioa horizontala (H) → Dilatazioa bertikala (V) → Emaitza
Konplexutasuna: O(n×r) ordez O(n×r²)
```

### Chaikin closed-loop smoothing
```
Iterazio bakoitzean puntu berri biak sortu:
  P'₀ = 0.75·Pᵢ + 0.25·Pᵢ₊₁
  P'₁ = 0.25·Pᵢ + 0.75·Pᵢ₊₁
Ondorio: B-spline kuadratikoa, ertzetik ≈ 25% barrura
```

### Bridge Engine v4 — Katea bridging mekanismoa
```
result[] array erabiltzen du mask[] ordez dist-mapan.
Honela, zubia egindako korridore batek isConn bihurtzen ditu
bere pixelak, eta hurrengo irlak korridore hori ikusi dezake
iturrira iristeko → katea efektua
```

### DXF pHYs txunk injekzioa
```
PNG egitura: Sinadura(8) + IHDR(25) + [pHYs(21)] + IDAT + IEND
pHYs: 4B luzera + 'pHYs' + 4B X(ppm) + 4B Y(ppm) + 1B unitate + 4B CRC32
300 DPI = 11811 pixel/metro
CRC32 kalkulatuta fitxategiak baliozko PNG izaten jarraitzen du
```

### Vektorizadore — Directed half-edge tracer
```
Pixel bakoitzak 4 ertz orientatu ditu (tinta eskuinera).
Konturoak ibilbide itxietan jarraitzen ditu: left-turn lehentasuna.
Konturoak: fill-rule="evenodd" → kanpoko CW eta barneko CCW automatikoki.
```

---

## Dependentziak

| Liburutegia | Bertsioa | Erabilera |
|---|---|---|
| [JSZip](https://stuk.github.io/jszip/) | 3.10.1 | ZIP fitxategiak sortzeko |
| [Bebas Neue](https://fonts.google.com/specimen/Bebas+Neue) | — | Display tipografia |
| [JetBrains Mono](https://www.jetbrains.com/lp/mono/) | — | Monospace tipografia |

K-means (LAB), bridge motor, morfologia, bektorizazioa, DXF, HPGL, PNG (DPI), zoom, undo, proiektua gorde/kargatu — guztiak JavaScript hutsa dira, CDN gabe.

---

## Hizkuntza

| Kodea | Hizkuntza |
|---|---|
| `EU` | Euskara (lehenetsia) |
| `ES` | Gaztelania |
| `EN` | Ingelesa |

Interfazeko testu guztiak itzulita: sekzio-izenburuak, botoiak, laguntzak, geruza-etiketa guztiak.

---

## Mugak

- K-means eta bridge-ak **main thread-ean** exekutatzen dira — irudi handiekin pantaila denbora laburrean izoztuko da (Web Worker etorkizunerako)
- DXF irteerak **LWPOLYLINE** erabiltzen du (bezier kurba lauak, ez kurba perfektuak) — ploter gehienentzat nahikoa
- Bridge edizio moduan egin aldaketak **undo-pila batean** gordetzen dira, baina maskak (K-means emaitza) **ezin dira desegin** — horretarako ANALIZAR berriro egin
- **Proiektua kargatzean** `kmPixels` LAB bihurketatik berreraikitzen da, ez JSON-etik — pixka bat mantsoago

---

## Lizentzia

MIT — Erabili, aldatu, banatu.

---

*STNCL v3 — B_RR_K_ · Euskal Herria*
