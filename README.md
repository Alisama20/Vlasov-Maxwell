# Vlasov-Maxwell: Semi-Lagrangian and Particle-in-Cell Methods

High-order numerical simulation of collisionless plasma dynamics via **semi-Lagrangian schemes** and **Particle-in-Cell (PIC) Monte Carlo methods**. Two implementations demonstrate complementary approaches to solving the Vlasov-Maxwell system.

> **Note:** The semi-Lagrangian scheme requires characteristic curves to be well-defined.
> The model focuses on collisionless plasmas; collisional effects are not included.

---

## Repository Structure

```
.
├── scripts/
│   ├── 1D1V.py              # Semi-Lagrangian Vlasov-Poisson (electrostatic two-stream)
│   ├── 1D2V.py              # Semi-Lagrangian 1D2V (electromagnetic Weibel instability)
│   ├── 1D1V_PIC.py          # Particle-in-Cell Monte Carlo (two-stream)
│   └── translate_memoria.py # Translation utility (Spanish → English)
├── figures/                 # Output figures
│   ├── weibel_campos.png    # Weibel: field evolution
│   ├── weibel_fpx.png       # Weibel: marginal f(x,px)
│   ├── weibel_fpy.png       # Weibel: marginal f(x,py)
│   ├── twostream_campos.png # PIC: field evolution
│   └── twostream_fase.png   # PIC: phase space
├── latex/
│   ├── memroria.tex         # Spanish thesis (source)
│   ├── memoria_en.tex       # English thesis (source)
│   ├── memoria_en.pdf       # English thesis (compiled)
│   ├── memroria.pdf         # Spanish thesis (compiled)
│   ├── bibliografia.bib     # Bibliography
│   ├── escudoUGRmonocromo.png # UGR logo
│   └── *.png                # Thesis figures
├── MemoryEN.pdf             # English thesis (copy in root)
├── MemoriaES.pdf            # Spanish thesis (copy in root)
├── README.md                # This file
├── LICENSE                  # MIT License
└── .gitignore               # Git configuration
```

---

## Report

The full academic report is available in two languages:

| File | Description |
|------|-------------|
| [MemoryEN.pdf](MemoryEN.pdf) | English version — semi-Lagrangian scheme, PIC method, Weibel and two-stream instability results |
| [MemoriaES.pdf](MemoriaES.pdf) | Spanish version (original) |

The LaTeX sources are in the [](latex/) folder together with the bibliography.

---

## Physical Model

The system is governed by the **Vlasov-Maxwell equations**

$$
\frac{\partial f}{\partial t} + \frac{p}{\gamma}\frac{\partial f}{\partial x} + F(t,x)\frac{\partial f}{\partial p} = 0 \quad \text{(Vlassov equation)}
$$

$$
\frac{\partial^2 A}{\partial t^2} - \frac{\partial^2 A}{\partial x^2} = -J(t,x)
\quad \text{(Wave equation)}
$$

$$
\frac{\partial E}{\partial x} = \rho(t,x) - 1
\quad \text{(Poisson)}
$$

| Symbol | Meaning |
|--------|---------|
| $f(t,x,p)$ | phase-space distribution of electrons |
| $\gamma = \sqrt{1 + p^2}$ | relativistic Lorentz factor |
| $F(t,x)$ | Lorentz force on particle |
| $A(t,x)$ | transverse vector potential |
| $\rho(t,x)$ | electron charge density |
| $J(t,x)$ | electron current density |

The system captures **electromagnetic wave coupling** absent in purely electrostatic models,
enabling study of instabilities like Weibel (magnetic) and two-stream (electrostatic).
---
## Methods

### Semi-Lagrangian Scheme — 1D1V & 1D2V

**Principle:** Transport $f$ backward along characteristics in phase space, then interpolate.

The method uses **Strang splitting** (order $\mathcal{O}(\Delta t^2)$):

1. **Half-step advection in momentum** via cubic Catmull-Rom interpolation
2. **Full-step advection in space** using periodic boundary conditions  
3. **Update fields** (Poisson solver via FFT, wave equation via Störmer-Verlet)
4. **Half-step advection in momentum** with centered fields (symmetric palindrome)

**Advantages:**
- CFL-free: stable for any $\Delta t$ (no time-step restriction)
- Preserves compact support and positivity
- High accuracy in phase space via cubic interpolation

**Disadvantages:**
- Requires fine spatial grids for smooth interpolation
- More complex than finite-difference schemes

### Particle-in-Cell (PIC) Monte Carlo — 1D1V

**Principle:** Represent $f$ as weighted **macroparticles**; compute fields on fixed grid.

$$f(t,x,p) \approx \sum_{p=1}^{N} w_p \, \delta(x - x_p(t)) \, \delta(p - p_p(t))$$

Each time step:

1. **Deposit charge** — CIC (Cloud-In-Cell) weighting to grid
2. **Solve for $E$** — FFT-based Poisson solver  
3. **Interpolate $E$** back to particle positions (CIC)
4. **Push particles** — Leapfrog integrator (Boris method)

**Advantages:**
- Error $\propto 1/\sqrt{N}$ independent of phase-space dimension
- Naturally captures **BGK vortices** (nonlinear trapping structures)
- Lower memory for high-dimensional phase spaces
- Intuitive physical interpretation

**Disadvantages:**
- Statistical noise (Monte Carlo error)
- More particles needed for accuracy than fine grid
---
## Results

### Weibel Instability — 1D2V Semi-Lagrangian

**Setup:** Bi-Maxwellian with temperature anisotropy ($T_y > T_x$) seeded with transverse magnetic field.

**Physics:** Anisotropic velocity distribution drives exponential growth of $B_z$ via gyro-resonance.

<p align="center">
<img src="figures/weibel_campos.png" width="90%">
<br>
<em>Left: Transverse magnetic field $\max|B_z(t)|$ grows exponentially with rate $\gamma_{\rm sim}$, 
compared to theoretical prediction (red dashed). 
Right: Longitudinal electric field $E_x$ grows secondarily via nonlinear coupling.</em>
</p>

<p align="center">
<img src="figures/weibel_fpx.png" width="90%">
<br>
<em>Marginal distribution $\Delta f(x,p_x)$ showing spatial modulation at wavelength $2\pi/k_0$. 
The divergent colormap highlights instability-driven structure.</em>
</p>

<p align="center">
<img src="figures/weibel_fpy.png" width="90%">
<br>
<em>Marginal distribution $\Delta f(x,p_y)$ demonstrates anisotropy: 
the temperature-driving direction $(p_y)$ shows stronger modulation than $(p_x)$.</em>
</p>

**Key observation:** Electromagnetic mode structure emerges from initial seed, saturating via particle trapping.
---
### Two-Stream Instability — 1D1V PIC Monte Carlo

**Setup:** Two counter-propagating Maxwellian beams ($v_0 = \pm 2$) with small spatial density perturbation.

**Physics:** Symmetric counterstreaming drives exponential growth of longitudinal electric field.

<p align="center">
<img src="figures/twostream_campos.png" width="90%">
<br>
<em>Left: Electric field $\max|E(t)|$ grows exponentially with rate from PIC simulation (blue) 
fitted against theoretical two-stream dispersion relation (red dashed). 
Right: Electrostatic energy $\tfrac{1}{2}\int E^2 dx$ accumulates during linear phase, 
then plateaus at saturation.</em>
</p>

<p align="center">
<img src="figures/twostream_fase.png" width="90%">
<br>
<em>Phase space $(x,v)$ at $t=10,20,30,40$ shows formation of characteristic 
<strong>BGK vortices</strong> (closed orbits in $(x,v)$ plane). 
These represent nonlinear trapping: particles become trapped in potential wells of the 
self-consistent electric field.</em>
</p>

**Key observation:** PIC captures the transition from exponential growth (linear regime) 
to vortex saturation (nonlinear phase) without grid refinement in velocity space.
---
## Requirements

```
numpy  numba  matplotlib  tqdm
```

Install via pip:

```bash
pip install numpy numba matplotlib tqdm
```

All scripts are tested on Python 3.8+.
---
## Usage

Run scripts from the repository root to ensure relative paths to `figures/` work correctly:

```bash
# Semi-Lagrangian solvers
python scripts/1D1V.py       # Two-stream (grid-based)
python scripts/1D2V.py       # Weibel instability (semi-Lagrangian 1D2V)

# Particle-in-Cell solver
python scripts/1D1V_PIC.py   # Two-stream (Monte Carlo PIC)
```

Each script generates PNG figures automatically in `figures/`.

### Key Parameters

**`scripts/1D2V.py` — Weibel Instability**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `Tx` | 1.0 | Temperature parallel to $B$ |
| `Ty` | 4.0 | Temperature perpendicular to $B$ (anisotropy driver) |
| `k0` | 0.5 | Wavenumber of EM seed |
| `eps` | 0.01 | Amplitude of seed $A_y$ |
| `nx, np_x, np_y` | 128, 64, 64 | Grid resolution |
| `dt` | 0.02 | Time step |
| `tmax` | 50 | Simulation duration |

**`scripts/1D1V_PIC.py` — Two-Stream Instability**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `v0` | 2.0 | Beam velocity ($\pm v_0$) |
| `vT` | 0.3 | Thermal width |
| `k0` | 0.5 | Wavenumber of density perturbation |
| `alpha` | 0.05 | Amplitude of density seed |
| `N` | 200,000 | Number of macroparticles |
| `nx` | 128 | Number of spatial grid cells |
| `dt` | 0.05 | Time step |
| `tmax` | 40 | Simulation duration |
---
## Physical Insights

### Why Two Methods?

1. **Semi-Lagrangian (Grid):** 
   - Best for smooth distributions and high accuracy
   - Deterministic; no statistical noise
   - Ideal for parameter studies and high-dimensional phase spaces

2. **PIC (Particle):**
   - Excels at capturing discrete structures (vortices, filaments)
   - Scales favorably for very high-dimensional problems
   - Industry standard for realistic plasma simulations

Both methods agree on the linear growth rates and nonlinear saturation,
validating each approach.

### Connection to Astrophysics

These instabilities occur in:
- **Collisionless shocks** (solar wind, supernova remnants)
- **Accretion disk jets** (black holes, active galactic nuclei)
- **Laser-plasma interactions** (inertial confinement fusion, laboratory astrophysics)

## Author

**A. S. Amari Rabah**

Developed as part of the coursework for *Transport Pdes in Kinetic Theory and Fluid Mechanics* —
Master's Degree in Physics and Mathematics (Fisymat),
University of Granada, Spain.