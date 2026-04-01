# STNCL — Geruzaniztun Plantilla Estudioa

```
███████╗████████╗███╗   ██╗ ██████╗██╗
██╔════╝╚══██╔══╝████╗  ██║██╔════╝██║
███████╗   ██║   ██╔██╗ ██║██║     ██║
╚════██║   ██║   ██║╚██╗██║██║     ██║
███████║   ██║   ██║ ╚████║╚██████╗███████╗
╚══════╝   ╚═╝   ╚═╝  ╚═══╝ ╚═════╝╚══════╝
```

**Nabigatzailean oinarritutako plantilla sortzailea** — zerbitzaririk gabe, instalaziorik gabe, graffitirako eta arte grafikorako pentsatuta.

---

## Zer da STNCL?

STNCL irudi bat hartu eta inprimatu daitezkeen geruzaniztuneko plantiletan bihurtzeko tresna bat da. HTML fitxategi bakarra da, nabigatzailean zuzenean exekutatzen dena. Ez du backend-ik behar, ez datuak igotzen, ez dependentziarik instalatu behar.

Algoritmo guztiak (K-means, bridge detekzioa, bektorizazioa, DXF eraiketa) nabigatzaileko JavaScript hutsa da.

---

## Ezaugarriak

### Segmentazioa
- **K-means++ kolorea** — 2–6 geruzetan banatzen du irudia automatikoki, koloreak klusterizatuz
- **Geruza bakoitzeko atalasa** — ratio-oinarritutako threshold-a: `1.0` = K-means exactua, `>1.0` hedatzen du, `<1.0` uzkurtzen du
- **Luminantziaren araberako ordenatzea** — geruza argienak lehenik aplikatzen dira (fondo), ilunagoak azkenean (frente)

### Zubia Motor (Bridge Engine v4)
- **8-konektaturiko BFS** — diagonal bideak, ez eskailera-efekturik
- **Chaikin kurba leuntzea** — zubia leuna eta naturalagoa
- **Katea zubia (chain bridging)** — irla batek zubia jasotzen duenean, hurrengoak haren bidez bideratu daiteke
- **Banaketa angeluarra** — irla handietan hainbat zubia kokatzen ditu angelu optimoetan
- **MAX_BRIDGE muga** — sakonegi dauden islak ez dira bridgeatu nahi gabe

### Irla Kontrola (geruza bakoitzeko)
- **MIN irla** — irla txikiak tintan betetzen ditu (K-means zarata kenduz)
- **FILL>** — irla handiak tintan betetzen ditu (% bidez, irudiaren arearen arabera)

### Bridge Edizio Modua ⊕
- **Geruza bakoitzeko ikuspegi B&N** — bridges aplikatuta ikusten dira zuzenean
- **Irlan klik** — egoera ziklatzen du:
  - `●` urdina → automatikoa (algoritmoak erabakitzen du)
  - `✕` gorria → zubirik ez (irla flotatzen geratzen da edo FILL>-rekin betetzen da)
  - `+` berdea → behartu zubia (MAX_BRIDGE × 3 limitea)
- **Eskuin klik** → irla automatikora bueltatzen du
- **↺ RESET** → geruza honetako override guztiak ezabatzen ditu

### Bektorizazioa
- **Directed half-edge tracer** — CDN dependentzirik gabe, nabigatzailean bertan
- Konturrak `fill-rule="evenodd"` bidez — zuloak automatikoki kudeatzen dira
- **Ramer-Douglas-Peucker** leuntzea — 1–10 slider bidez

### Exportazioa
- **SVG** — `fill-rule="evenodd"` eta erregistro-markak
- **DXF** — LWPOLYLINE (milimetrotan, Y-ardatza irauli), ploter eta cutter-entzat
- **PNG** — B&N, erregistro-markak barne
- **ZIP** — geruza guztiak formatu guztiekin
- **Erregistro-markak** — 4 izkina, crosshair 5mm, margen 8mm

---

## Erabileraren gida

### Lan-fluxu tipikoa

```
1. Irudia kargatu
      ↓
2. ANALIZAR sakatu (K-means exekutatzen da)
      ↓
3. Geruzak doitu:
   • Kolorea aldatu (color picker)
   • Atalasa doitu (UMBRAL slider)
   • Ordena aldatu (↑↓ botoiak edo drag & drop)
   • Ikusgarritasuna toggle (● / ○)
      ↓
4. Bridge parametroak doitu:
   • Zubiaren zabalera (mm)
   • MIN irla (px²) — geruza bakoitzeko
   • FILL> (%) — geruza bakoitzeko
      ↓
5. VER BRIDGES sakatu → aurrebista
      ↓
6. [Aukerakoa] ⊕ botoia → Bridge Edizio Modua
   • Klik isletan egoera aldatzeko
      ↓
7. Formatua hautatu (SVG / DXF / PNG)
      ↓
8. EXPORTAR ZIP
```

### Bridge parametroen gida praktikoa

| Egoera | Konponbidea |
|---|---|
| Irla txiki ugari (K-means zarata) | MIN irla ↑ geruza horretan |
| Aurpegi zuria irla handi gisa (geruza ilunean) | FILL> %10–15 geruza ilunean |
| Zubia lerro angeluzuzenetan | Bridge v4-k Chaikin leunduz ebazten du |
| Irla sakonegi, zubirik gabe | ⊕ → irlan klik bi aldiz (+) → behartu |
| Zubia posizio txarrean | ⊕ → irlan klik (✕) → FILL> edo utzi flotatzen |

---

## Fitxategi egitura

```
stncl.html              ← aplikazio osoa (fitxategi bakarra)
README.md               ← dokumentazio hau
```

Exportatutako ZIP-ak:

```
stncl_export.zip
├── capa_01_a2a2a2.svg  ← capa 1 (hex kolorea fitxategi izenean)
├── capa_01_a2a2a2.dxf
├── capa_01_a2a2a2.png
├── capa_02_2d2d2d.svg
├── capa_02_2d2d2d.dxf
└── capa_02_2d2d2d.png
```

---

## Teknikaren xehetasunak

### K-means++
Subsampling-a (8px-eko pausoa) zentroide kalkulurako; esleipen osoa pixelaren mailan. Maximoz 30 iterazio, konbergentzia automatikoa.

### Bridge Engine v4
```
1. BFS (4-konekt.) → kanpoko paper konektatua markatu
2. Irlak bildu → handienak lehenik (katea bridging-erako)
3. override.skip? → saltu
4. 8-konekt. BFS distantzia-mapa (result[] erabiliz, mask[] ez)
   → lehenago egindako korridoreak ikusten ditu (katea bridging)
5. Muga-pixelak bilatu → dist[] kontsulta O(1)
6. Bidea berreraikuntza prev[] bidez → [x,y] koordinatuak
7. Angelu-banaketa → N zubia kokatzen ditu (area-ren arabera)
8. Chaikin(2 iterazio) → zubia leundu
9. Korridore zirkularra marraztu + isConn markatu
   → hurrengo irlen erreferentzia-puntu bihurtzen da
```

### Vektorizadore (Directed half-edge tracer)
Pixel-mugen grafoa eraikitzen du: pixel bakoitzak 4 ertz orientatu ditu (tinta eskuinera). Konturrak ibilbide itxietan jarraitzen ditu. Gehienezko pauso-muga: `(w+2)*(h+2)`. Ondoren RDP leuntzea.

### DXF formatua
- `$INSUNITS = 4` (milimetroak)
- `LWPOLYLINE` flag=1 (itxita) bezier kurba bakoitzeko
- Y-ardatza irauli: SVG y-behera → DXF y-gora
- `LINE` entitateak erregistro-markentzat (`REG` geruza)

---

## Hizkuntza

STNCL hiru hizkuntzetan dago eskuragarri:

| Kodea | Hizkuntza |
|---|---|
| `EU` | Euskara |
| `ES` | Gaztelania |
| `EN` | Ingelesa |

Interfazeko testu guztiak itzulita daude. Hizkuntza-botoia goiburuan dago, eskuineko aldean.

---

## Dependentziak

| Liburutegia | Bertsioa | Erabilera |
|---|---|---|
| [JSZip](https://stuk.github.io/jszip/) | 3.10.1 | ZIP fitxategiak sortzeko |
| [Bebas Neue](https://fonts.google.com/specimen/Bebas+Neue) | — | Display tipografia |
| [JetBrains Mono](https://www.jetbrains.com/lp/mono/) | — | Monospace tipografia |

Bektorizatzailea, bridge motorra, K-means eta DXF eraiketa guztiak JavaScript hutsa dira, CDN gabe.

---

## Mugak

- Irudiak 1200px-ra **mozten dira** performantzia arrazoiengatik (K-means)
- DXF irteerak **LWPOLYLINE** erabiltzen du (bezier kurba lauak, ez kurba perfektuak)
- Bridge edizio modua **saioa itxi arte** mantentzen da (ez dago gordetze/kargatzeko aukerarik)
- Proiekturik ez — **analisia berriz egin** behar da irudia aldatzerakoan

---

## Lizentzia

MIT — Erabili, aldatu, banatu.

---

*STNCL — B_RR_K_ · Euskal Herria*
