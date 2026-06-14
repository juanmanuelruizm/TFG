<div align="center">

# A Lagrangian Method for the Identification of Atmospheric Blocking Situations

**Bachelor's Thesis (TFG) · Degree in Physics · University of Alicante**

*Identifying atmospheric blockings from the massive analysis of air-parcel back-trajectories, instead of the classical thermodynamic fields.*

[![Field](https://img.shields.io/badge/Field-Atmospheric%20Physics-0b7285)](#)
[![Method](https://img.shields.io/badge/Approach-Lagrangian-1864ab)](#)
[![Model](https://img.shields.io/badge/Trajectories-HYSPLIT-2b8a3e)](#)
[![Data](https://img.shields.io/badge/Reanalysis-ERA5-862e9c)](#)
[![Made with](https://img.shields.io/badge/Built%20with-Python-3776AB?logo=python&logoColor=white)](#)
[![License](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey)](LICENSE)

**🌐 Language:** **English** · [Español](docs/README.es.md) · [Català](docs/README.ca.md)

<br>

<img src="figures/fig_clusters_dbscan.gif" width="70%" alt="Detected blocking region over the North Atlantic / Europe, classified with DBSCAN over 500 hPa geopotential height">

<sub><i>Detected blocking region (red cluster) evolving over the Euro-Atlantic sector, isolated with DBSCAN over the 500 hPa geopotential height field — August 2003 heatwave.</i></sub>

</div>

---

## 📑 Table of contents

- [Abstract](#-abstract)
- [Why it matters](#-why-it-matters)
- [The idea in one picture](#-the-idea-in-one-picture)
- [Methodology](#-methodology)
- [Results](#-results)
- [Skills & tech stack](#-skills--tech-stack)
- [Repository structure](#-repository-structure)
- [Read the full thesis](#-read-the-full-thesis)
- [How to cite](#-how-to-cite)
- [Author](#-author)
- [License](#-license)

---

## 📄 Abstract

Atmospheric blockings are large-scale, quasi-stationary pressure patterns that interrupt the
normal west-to-east progression of weather systems. They can trigger **summer heatwaves** and
**winter cold spells**, with well-documented impacts on public health, agriculture, water
availability and wildfire risk. In a warming climate, the frequency of these extreme events has
increased, which makes their accurate identification and characterization increasingly relevant.

Traditionally, blockings have been detected from **Eulerian** diagnostics based on thermodynamic
fields (e.g. geopotential height anomalies or meridional gradients). This thesis proposes an
**innovative Lagrangian alternative**: instead of looking at the fields, it analyses the *motion*
of the air itself. Using the **HYSPLIT** model fed with **ERA5** reanalysis data, millions of
virtual particle **back-trajectories** are computed, and the regions where these trajectories
*stagnate* are flagged as candidate blocking areas. This yields an explicit, geographically
localized identification of blockings, together with their **location, extent, duration and
intensity**, and their **evolution in time**.

> *"Understanding and predicting atmospheric blocking remains one of the grand challenges of
> meteorology."* — Michael E. Mann

**Keywords:** atmospheric blocking · Lagrangian method · back-trajectories · HYSPLIT · ERA5 · DBSCAN · climate change

---

## 🌍 Why it matters

| Impact | Consequence of a persistent blocking |
| --- | --- |
| 🔥 Heatwaves | Excess mortality, especially in regions unprepared for extreme heat |
| ❄️ Cold spells | Prolonged freezing conditions in winter |
| 🌵 Droughts | Pressure on agriculture, water supply and ecosystems |
| 🌲 Wildfires | Higher fire risk from dry, stagnant conditions |
| ⚡ Energy | Demand spikes (cooling/heating) and higher emissions |

Better detection methods feed directly into **long-range forecasting** and **climate-risk
management** — which is exactly what this Lagrangian approach aims to improve.

---

## 💡 The idea in one picture

A blocking is, physically, a region where the atmosphere **stops flowing**. If we release virtual
particles and integrate their trajectories *backwards* in time, parcels inside a blocking barely
travel: their **end-to-end distance** and **trajectory length** collapse compared to the fast,
zonal flow of the surrounding jet stream.

<div align="center">
<table>
<tr>
<td align="center"><b>End-to-end distance</b><br><sub>How far each parcel ends from where it started</sub><br><img src="figures/fig20_end-to-end.gif" width="100%"></td>
<td align="center"><b>Trajectory length</b><br><sub>Total arc length travelled by each parcel</sub><br><img src="figures/fig21_trajectory-length.gif" width="100%"></td>
</tr>
</table>
<sub><i>Cool colours = short, stagnant trajectories → candidate blocking. Warm colours = long, mobile trajectories → unblocked, zonal flow. Euro-Atlantic sector, summer 2003.</i></sub>
</div>

---

## 🔬 Methodology

The pipeline turns a raw set of HYSPLIT trajectories into a delineated, time-evolving blocking region:

```
ERA5 reanalysis ─▶ HYSPLIT back-trajectories ─▶ per-parcel metrics ─▶ temporal filtering ─▶ DBSCAN clustering ─▶ blocking region
```

1. **Data & domain.** ERA5 reanalysis over the Euro-Atlantic sector, analysed at the **500 hPa**
   level — the reference surface for mid-tropospheric blocking dynamics.
2. **Trajectory metrics.** For every back-trajectory three Lagrangian descriptors are computed:
   - **End-to-end distance** and **trajectory length**, using the **Haversine** formula on the sphere.
   - **Zonal** and **meridional projections** (east–west / north–south extent of each trajectory).
3. **Statistical reduction.** Mean, median and standard deviation of each metric are tracked,
   both in space (latitude/longitude profiles) and through **time** (global temporal evolution).
4. **Blocking filter.** A percentile-based, intensity-adaptive threshold flags the grid points
   whose trajectories are anomalously short → a binary **blocking matrix** per instant.
5. **Region delineation.** The candidate points are grouped with **DBSCAN** (density-based
   clustering) to isolate the coherent blocking region and discard noise.

> A short primer on the supporting concepts — geostrophic & gradient wind, geopotential height,
> the DBSCAN algorithm and the taxicab metric — is included in the thesis appendices.

---

## 📊 Results

The method was validated on two well-known European blocking episodes: the **August 2003** and
**June 2019** heatwaves. The animations below are the actual outputs of the pipeline.

### Blocking signature in the trajectory metrics

Spatial profiles of the end-to-end distance reveal a clear depression over the blocked region
(shaded band = ±1 standard deviation):

<div align="center">
<table>
<tr>
<td><img src="figures/fig23_top-left.gif" width="100%"></td>
<td><img src="figures/fig23_top-right.gif" width="100%"></td>
</tr>
<tr>
<td><img src="figures/fig23_bottom-left.gif" width="100%"></td>
<td><img src="figures/fig23_bottom-right.gif" width="100%"></td>
</tr>
</table>
<sub><i>End-to-end distance (left) and trajectory length (right) profiles, viewed across longitudes (top) and latitudes (bottom).</i></sub>
</div>

### Zonal & meridional projections

<div align="center">
<table>
<tr>
<td align="center"><b>Zonal projection</b><br><img src="figures/fig25_left.gif" width="100%"></td>
<td align="center"><b>Meridional projection</b><br><img src="figures/fig25_right.gif" width="100%"></td>
</tr>
</table>
</div>

### Blocking matrices

The filtering step produces a binary mask per instant — white (`1`) where the blocking criterion
is met, black (`0`) elsewhere:

<div align="center">
<table>
<tr>
<td align="center"><b>Meridional projection</b><br><img src="figures/fig27_left.gif" width="100%"></td>
<td align="center"><b>Zonal projection</b><br><img src="figures/fig27_right.gif" width="100%"></td>
</tr>
</table>
</div>

### Final delineated region (DBSCAN)

After clustering, the blocking region emerges as a coherent cluster (red) over the 500 hPa
geopotential height field — see the [animation at the top](#a-lagrangian-method-for-the-identification-of-atmospheric-blocking-situations).

**Key takeaways**

- ✅ A purely **Lagrangian** detector reproduces known blocking episodes — a genuinely different
  lens from the standard Eulerian diagnostics.
- ✅ The detected region **adapts in time**, enabling the study of the **life cycle** (onset,
  maturity, decay) of a blocking.
- 🔭 Open challenges (future work): **generalizing** the method beyond the studied cases, and
  automatically classifying blockings into **Rex** and **Omega** types and their centres of action.

---

## 🧰 Skills & tech stack

This project sits at the intersection of **atmospheric physics**, **scientific computing** and
**data science**:

- **Languages & tools:** Python, NumPy/SciPy-style numerical analysis, geospatial data handling.
- **Modelling:** HYSPLIT Lagrangian trajectory model; ERA5 reanalysis ingestion.
- **Maths & methods:** spherical geometry (Haversine), descriptive statistics, percentile-based
  thresholding, **DBSCAN** density clustering, basic topology (taxicab metric).
- **Domain:** synoptic meteorology — geostrophic/gradient wind, Coriolis force, geopotential
  height, isohypse/isobar maps.
- **Communication:** reproducible figures, animated scientific visualization, trilingual reporting.

---

## 🗂 Repository structure

```
.
├── README.md                  # You are here (English)
├── docs/
│   ├── README.es.md           # Español
│   └── README.ca.md           # Català
├── thesis/
│   └── TFG_metodo_lagrangiano_bloqueo_atmosferico.pdf   # Full document (75 pp.)
├── figures/
│   ├── *.gif                  # Animated results used in this README
│   └── source/                # Original vector (PDF) figures
├── CITATION.cff               # Machine-readable citation metadata
└── LICENSE                    # CC BY 4.0
```

---

## 📚 Read the full thesis

The complete document (75 pages, in Spanish) — including the theoretical background, the full
derivation of the method, the discussion and the appendices — is available here:

👉 **[`thesis/TFG_metodo_lagrangiano_bloqueo_atmosferico.pdf`](thesis/TFG_metodo_lagrangiano_bloqueo_atmosferico.pdf)**

---

## 🔖 How to cite

If you reference this work, please cite it (see [`CITATION.cff`](CITATION.cff)):

```bibtex
@thesis{RuizMunoz2024Lagrangian,
  author = {Ruiz Muñoz, Juan Manuel},
  title  = {Un método lagrangiano para la identificación de situaciones de bloqueo atmosférico},
  school = {Universidad de Alicante, Facultad de Ciencias},
  type   = {Bachelor's Thesis (Trabajo Fin de Grado)},
  year   = {2024}
}
```

---

## 👤 Author

**Juan Manuel Ruiz Muñoz** — Degree in Physics, University of Alicante (2023–2024).

[![GitHub](https://img.shields.io/badge/GitHub-JuanManuelRM7-181717?logo=github)](https://github.com/JuanManuelRM7)

---

## 📜 License

The contents of this repository (thesis document, figures and animations) are released under the
**[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE)** license. You are free to
share and adapt the material for any purpose, provided you give appropriate credit.
