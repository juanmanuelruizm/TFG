<div align="center">

# Un mètode lagrangià per a la identificació de situacions de bloqueig atmosfèric

**Treball de Fi de Grau (TFG) · Grau en Física · Universitat d'Alacant**

*Identificar bloquejos atmosfèrics a partir de l'anàlisi massiva de retro-trajectòries de partícules d'aire, en lloc dels camps termodinàmics clàssics.*

[![Camp](https://img.shields.io/badge/Camp-F%C3%ADsica%20de%20l'Atmosfera-0b7285)](#)
[![Mètode](https://img.shields.io/badge/Enfocament-Lagrangi%C3%A0-1864ab)](#)
[![Model](https://img.shields.io/badge/Traject%C3%B2ries-HYSPLIT-2b8a3e)](#)
[![Dades](https://img.shields.io/badge/Rean%C3%A0lisi-ERA5-862e9c)](#)
[![Fet amb](https://img.shields.io/badge/Constru%C3%ABt%20amb-Python-3776AB?logo=python&logoColor=white)](#)
[![Llicència](https://img.shields.io/badge/Llic%C3%A8ncia-CC%20BY%204.0-lightgrey)](../LICENSE)

**🌐 Idioma:** [English](../README.md) · [Español](README.es.md) · **Català**

<br>

<img src="../figures/fig_clusters_dbscan.gif" width="70%" alt="Regió de bloqueig detectada sobre l'Atlàntic Nord / Europa, classificada amb DBSCAN sobre l'altura geopotencial a 500 hPa">

<sub><i>Regió de bloqueig detectada (clúster roig) evolucionant sobre el sector euro-atlàntic, aïllada amb DBSCAN sobre el camp d'altura geopotencial a 500 hPa — onada de calor d'agost de 2003.</i></sub>

</div>

---

## 📑 Índex

- [Resum](#-resum)
- [Per què és important](#-per-què-és-important)
- [La idea en una imatge](#-la-idea-en-una-imatge)
- [Metodologia](#-metodologia)
- [Resultats](#-resultats)
- [Competències i stack tècnic](#-competències-i-stack-tècnic)
- [Estructura del repositori](#-estructura-del-repositori)
- [Llegir la memòria completa](#-llegir-la-memòria-completa)
- [Com citar](#-com-citar)
- [Autor](#-autor)
- [Llicència](#-llicència)

---

## 📄 Resum

Els bloquejos atmosfèrics són patrons de pressió quasi-estacionaris de gran escala que
interrompen l'avanç normal d'oest a est dels sistemes meteorològics. Poden provocar **onades de
calor a l'estiu** i **onades de fred a l'hivern**, amb impactes ben documentats sobre la salut
pública, l'agricultura, la disponibilitat d'aigua i el risc d'incendis. En un clima canviant, la
freqüència d'aquests fenòmens extrems ha augmentat, cosa que fa cada cop més rellevant la seua
identificació i caracterització precises.

Tradicionalment, els bloquejos s'han detectat mitjançant diagnòstics **eulerians** basats en camps
termodinàmics (per exemple, anomalies d'altura geopotencial o gradients meridionals). Aquest
treball proposa una **alternativa lagrangiana innovadora**: en lloc de mirar els camps, analitza el
*moviment* del mateix aire. Utilitzant el model **HYSPLIT** alimentat amb dades de reanàlisi
**ERA5**, es calculen milions de **retro-trajectòries** de partícules virtuals, i les regions on
aquestes trajectòries *s'estanquen* es marquen com a zones candidates a bloqueig. Això proporciona
una identificació explícita i geogràficament localitzada dels bloquejos, juntament amb la seua
**localització, extensió, durada i intensitat** i la seua **evolució temporal**.

> *"Comprendre i predir el bloqueig atmosfèric continua sent un dels grans reptes de la
> meteorologia."* — Michael E. Mann

**Paraules clau:** bloquejos atmosfèrics · mètode lagrangià · retro-trajectòries · HYSPLIT · ERA5 · DBSCAN · canvi climàtic

---

## 🌍 Per què és important

| Impacte | Conseqüència d'un bloqueig persistent |
| --- | --- |
| 🔥 Onades de calor | Excés de mortalitat, sobretot en regions no preparades per a la calor extrema |
| ❄️ Onades de fred | Condicions de gelades prolongades a l'hivern |
| 🌵 Sequeres | Pressió sobre l'agricultura, el subministrament d'aigua i els ecosistemes |
| 🌲 Incendis | Major risc d'incendis per condicions seques i estancades |
| ⚡ Energia | Pics de demanda (refrigeració/calefacció) i més emissions |

Millors mètodes de detecció alimenten directament la **predicció a llarg termini** i la **gestió
del risc climàtic** — just el que aquest enfocament lagrangià busca millorar.

---

## 💡 La idea en una imatge

Un bloqueig és, físicament, una regió on l'atmosfera **deixa de fluir**. Si alliberem partícules
virtuals i n'integrem les trajectòries *cap enrere* en el temps, les partícules dins d'un bloqueig
amb prou feines es desplacen: la seua **distància end-to-end** i la **longitud de la trajectòria**
s'enfonsen davant del flux zonal ràpid del *jet stream* circumdant.

<div align="center">
<table>
<tr>
<td align="center"><b>Distància end-to-end</b><br><sub>Com de lluny acaba cada partícula del seu origen</sub><br><img src="../figures/fig20_end-to-end.gif" width="100%"></td>
<td align="center"><b>Longitud de la trajectòria</b><br><sub>Longitud d'arc total recorreguda per cada partícula</sub><br><img src="../figures/fig21_trajectory-length.gif" width="100%"></td>
</tr>
</table>
<sub><i>Colors freds = trajectòries curtes i estancades → candidat a bloqueig. Colors càlids = trajectòries llargues i mòbils → flux zonal sense bloqueig. Sector euro-atlàntic, estiu de 2003.</i></sub>
</div>

---

## 🔬 Metodologia

El flux de treball transforma un conjunt brut de trajectòries d'HYSPLIT en una regió de bloqueig
delimitada i amb evolució temporal:

```
Reanàlisi ERA5 ─▶ retro-trajectòries HYSPLIT ─▶ mètriques per partícula ─▶ filtratge temporal ─▶ clustering DBSCAN ─▶ regió de bloqueig
```

1. **Dades i domini.** Reanàlisi ERA5 sobre el sector euro-atlàntic, analitzat al nivell de
   **500 hPa** — la superfície de referència per a la dinàmica de bloqueig a la troposfera mitjana.
2. **Mètriques de trajectòria.** Per a cada retro-trajectòria es calculen tres descriptors
   lagrangians:
   - **Distància end-to-end** i **longitud de la trajectòria**, amb la fórmula de **Haversine** sobre l'esfera.
   - **Projecció zonal** i **meridional** (extensió est–oest / nord–sud de cada trajectòria).
3. **Reducció estadística.** Se segueixen la mitjana, la mediana i la desviació estàndard de cada
   mètrica, tant en l'espai (perfils en latitud/longitud) com en el **temps** (evolució temporal
   global).
4. **Filtre de bloqueig.** Un llindar basat en percentils i adaptatiu a la intensitat marca els
   punts de malla amb trajectòries anòmalament curtes → una **matriu de bloqueig** binària per
   instant.
5. **Delimitació de la regió.** Els punts candidats s'agrupen amb **DBSCAN** (clustering basat en
   densitat) per a aïllar la regió de bloqueig coherent i descartar el soroll.

> Als apèndixs de la memòria s'inclou una introducció als conceptes de suport — vent geostròfic i
> de gradient, altura geopotencial, l'algorisme DBSCAN i la mètrica del taxista.

---

## 📊 Resultats

El mètode es va validar sobre dos episodis de bloqueig europeus ben coneguts: les onades de calor
d'**agost de 2003** i **juny de 2019**. Les animacions següents són les eixides reals del flux de
treball.

### Empremta del bloqueig en les mètriques de trajectòria

Els perfils espacials de la distància end-to-end revelen una depressió clara sobre la regió
bloquejada (banda ombrejada = ±1 desviació estàndard):

<div align="center">
<table>
<tr>
<td><img src="../figures/fig23_top-left.gif" width="100%"></td>
<td><img src="../figures/fig23_top-right.gif" width="100%"></td>
</tr>
<tr>
<td><img src="../figures/fig23_bottom-left.gif" width="100%"></td>
<td><img src="../figures/fig23_bottom-right.gif" width="100%"></td>
</tr>
</table>
<sub><i>Perfils de distància end-to-end (esquerra) i longitud de trajectòria (dreta), vistos en longituds (dalt) i en latituds (baix).</i></sub>
</div>

### Projeccions zonal i meridional

<div align="center">
<table>
<tr>
<td align="center"><b>Projecció zonal</b><br><img src="../figures/fig25_left.gif" width="100%"></td>
<td align="center"><b>Projecció meridional</b><br><img src="../figures/fig25_right.gif" width="100%"></td>
</tr>
</table>
</div>

### Matrius de bloqueig

El pas de filtratge produeix una màscara binària per instant — blanc (`1`) on es compleix el
criteri de bloqueig, negre (`0`) a la resta:

<div align="center">
<table>
<tr>
<td align="center"><b>Projecció meridional</b><br><img src="../figures/fig27_left.gif" width="100%"></td>
<td align="center"><b>Projecció zonal</b><br><img src="../figures/fig27_right.gif" width="100%"></td>
</tr>
</table>
</div>

### Regió delimitada final (DBSCAN)

Després del clustering, la regió de bloqueig emergeix com un clúster coherent (roig) sobre el camp
d'altura geopotencial a 500 hPa — vegeu l'[animació de la capçalera](#un-mètode-lagrangià-per-a-la-identificació-de-situacions-de-bloqueig-atmosfèric).

**Idees clau**

- ✅ Un detector purament **lagrangià** reprodueix episodis de bloqueig coneguts — una mirada
  genuïnament diferent als diagnòstics eulerians estàndard.
- ✅ La regió detectada **s'adapta en el temps**, cosa que permet estudiar el **cicle de vida**
  (inici, maduresa, declivi) d'un bloqueig.
- 🔭 Reptes oberts (treball futur): **generalitzar** el mètode més enllà dels casos estudiats i
  classificar automàticament els bloquejos en tipus **Rex** i **Omega** i els seus centres d'acció.

---

## 🧰 Competències i stack tècnic

Aquest projecte se situa en la intersecció entre la **física de l'atmosfera**, la **computació
científica** i la **ciència de dades**:

- **Llenguatges i eines:** Python, anàlisi numèrica, tractament de dades geoespacials.
- **Modelatge:** model lagrangià de trajectòries HYSPLIT; ingesta de reanàlisi ERA5.
- **Matemàtiques i mètodes:** geometria esfèrica (Haversine), estadística descriptiva, llindars per
  percentils, clustering per densitat **DBSCAN**, nocions de topologia (mètrica del taxista).
- **Domini:** meteorologia sinòptica — vent geostròfic/de gradient, força de Coriolis, altura
  geopotencial, mapes d'isohipses/isòbares.
- **Comunicació:** figures reproduïbles, visualització científica animada, informe trilingüe.

---

## 🗂 Estructura del repositori

```
.
├── README.md                  # Versió en anglés (principal)
├── docs/
│   ├── README.es.md           # Español
│   └── README.ca.md           # Ets ací (Català)
├── thesis/
│   └── TFG_metodo_lagrangiano_bloqueo_atmosferico.pdf   # Memòria completa (75 pp.)
├── figures/
│   ├── *.gif                  # Resultats animats usats en aquest README
│   └── source/                # Figures vectorials originals (PDF)
├── CITATION.cff               # Metadades de citació llegibles per màquina
└── LICENSE                    # CC BY 4.0
```

---

## 📚 Llegir la memòria completa

El document complet (75 pàgines, en castellà) — incloent-hi el marc teòric, la derivació completa
del mètode, la discussió i els apèndixs — està disponible ací:

👉 **[`thesis/TFG_metodo_lagrangiano_bloqueo_atmosferico.pdf`](../thesis/TFG_metodo_lagrangiano_bloqueo_atmosferico.pdf)**

---

## 🔖 Com citar

Si fas referència a aquest treball, cita'l (vegeu [`CITATION.cff`](../CITATION.cff)):

```bibtex
@thesis{RuizMunoz2024Lagrangian,
  author = {Ruiz Muñoz, Juan Manuel},
  title  = {Un método lagrangiano para la identificación de situaciones de bloqueo atmosférico},
  school = {Universitat d'Alacant, Facultat de Ciències},
  type   = {Treball de Fi de Grau},
  year   = {2024}
}
```

---

## 👤 Autor

**Juan Manuel Ruiz Muñoz** — Grau en Física, Universitat d'Alacant (2023–2024).

[![GitHub](https://img.shields.io/badge/GitHub-JuanManuelRM7-181717?logo=github)](https://github.com/JuanManuelRM7)

---

## 📜 Llicència

Els continguts d'aquest repositori (memòria, figures i animacions) es publiquen sota la llicència
**[Creative Commons Reconeixement 4.0 Internacional (CC BY 4.0)](../LICENSE)**. Ets lliure de
compartir i adaptar el material per a qualsevol propòsit, sempre que en dones el crèdit adequat.
