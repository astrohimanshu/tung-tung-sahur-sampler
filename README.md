<div align="center">

```
████████╗██╗   ██╗███╗   ██╗ ██████╗     ████████╗██╗   ██╗███╗   ██╗ ██████╗ 
╚══██╔══╝██║   ██║████╗  ██║██╔════╝        ██╔══╝██║   ██║████╗  ██║██╔════╝ 
   ██║   ██║   ██║██╔██╗ ██║██║  ███╗       ██║   ██║   ██║██╔██╗ ██║██║  ███╗
   ██║   ██║   ██║██║╚██╗██║██║   ██║       ██║   ██║   ██║██║╚██╗██║██║   ██║
   ██║   ╚██████╔╝██║ ╚████║╚██████╔╝       ██║   ╚██████╔╝██║ ╚████║╚██████╔╝
   ╚═╝    ╚═════╝ ╚═╝  ╚═══╝ ╚═════╝        ╚═╝    ╚═════╝ ╚═╝  ╚═══╝ ╚═════╝ 
                                                                                
              ███████╗ █████╗ ██╗  ██╗██╗   ██╗██████╗                         
              ██╔════╝██╔══██╗██║  ██║██║   ██║██╔══██╗                        
              ███████╗███████║███████║██║   ██║██████╔╝                        
              ╚════██║██╔══██║██╔══██║██║   ██║██╔══██╗                        
              ███████║██║  ██║██║  ██║╚██████╔╝██║  ██║                        
              ╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝ ╚═════╝ ╚═╝  ╚═╝                       
```

# 🥁 Tung Tung Tung Sahur — Monte Carlo Surface Sampler

**Sampling the surface of a 3-D mesh using the Metropolis–Hastings algorithm**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat-square&logo=jupyter&logoColor=white)](https://jupyter.org/)
[![trimesh](https://img.shields.io/badge/trimesh-3.x-green?style=flat-square)](https://trimsh.org/)
[![Plotly](https://img.shields.io/badge/Plotly-Interactive_3D-3F4F75?style=flat-square&logo=plotly&logoColor=white)](https://plotly.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

</div>

---

## 🎯 What is this?

This project applies **Markov Chain Monte Carlo (MCMC)** methods — specifically the **Metropolis–Hastings algorithm** — to sample points directly on the surface of a 3-D mesh. Rather than using conventional UV-based or triangle-area-weighted approaches, the sampler learns the surface geometry purely through a **Signed Distance Function (SDF)** oracle.

The result: a dense red point cloud that hugs every curve and crevice of the *Tung Tung Tung Sahur* figure, generated from first principles of Bayesian sampling.

```
Start anywhere in 3-D space  →  Define a probability that peaks at the surface
→  Walk randomly, accepting surface-approaching moves  →  Collect the path
→  You have your point cloud 🎉
```

---

## 🧠 The Core Idea

We define an unnormalised **Boltzmann target distribution** over 3-D space:

$$p(\mathbf{x}) \propto \exp\!\left(-\beta \cdot |d(\mathbf{x})|\right)$$

where:
- $d(\mathbf{x})$ is the **signed distance** from point $\mathbf{x}$ to the nearest mesh surface point
- $\beta = 100$ is the inverse temperature — larger values concentrate the distribution more tightly on the surface

Points **on the surface** ($d \approx 0$) have probability $\approx 1$. Points **far away** have probability $\approx 0$. The Metropolis–Hastings chain then explores this landscape with a simple Gaussian random walk.

---

## 📁 Repository Structure

```
tung-tung-sampler/
│
├── 📓 tung-tung-sampler.ipynb   # Main documented notebook (this project)
│
├── 📂 data/
│   ├── tung_tung_sahur.glb      # Original 3-D model (binary glTF)
│   └── tung_tung_sahur_cropped.glb  # Cropped mesh (bat removed, optional)
│
└── 📄 README.md                 # You are here
```

---

## 🔬 Methodology

### Pipeline Overview

| Step | Description | Key Tool |
|------|-------------|----------|
| **1. Load** | Parse the `.glb` scene and concatenate all geometries | `trimesh.load` |
| **2. Crop** | Remove the baseball bat via triangle centroid masking ($x \leq 0.4$) | `mesh.submesh` |
| **3. Inspect** | Print vertices, faces, bounding box, centroid, extents | `trimesh.Trimesh` |
| **4. SDF** | Build a proximity query object backed by an R-tree spatial index | `trimesh.proximity.ProximityQuery` |
| **5. Sample** | Run 50 000 steps of Metropolis–Hastings with $\sigma = 0.05$ | Custom MCMC loop |
| **6. Visualise** | Interactive 3-D plots: point cloud, chain trajectory, ghost overlay, animation | `plotly.graph_objects` |

---

### The Metropolis–Hastings Algorithm

```python
x₀ ~ Uniform(bounding_box)          # start anywhere

for t = 1 … N:
    x* = xₜ + Normal(0, σ²)         # Gaussian random walk proposal

    log r = -β(|d(x*)| - |d(xₜ)|)   # log acceptance ratio
    r     = exp(min(0, log r))        # clamp to [0, 1]

    u ~ Uniform(0, 1)
    xₜ₊₁ = x*  if u < r             # accept → move
           xₜ  otherwise            # reject → stay
    record xₜ₊₁
```

The algorithm satisfies **detailed balance**, guaranteeing convergence to the target $p(\mathbf{x})$ regardless of starting point.

### Hyperparameter Choices

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| `σ` (proposal std) | `0.05` | ~5% of mesh extent; balances exploration vs. acceptance rate |
| `β` (inverse temperature) | `100` | Sharp enough to concentrate mass at $d \approx 0$ |
| `N` (samples) | `50 000` | Dense coverage with manageable runtime |
| Initialisation | `Uniform(bbox)` | Simple; chain self-corrects during burn-in |

---

## 🎨 Visualisations

The notebook produces four interactive 3-D visualisations using Plotly:

| View | Description |
|------|-------------|
| **Wireframe + Solid** | Brown mesh with blue triangle edges — reveals triangulation density |
| **Smooth Shaded Render** | Phong shading with ambient/diffuse/specular lighting — the full clay model |
| **Raw Point Cloud** | 50 000 red markers — confirms the sampler's surface concentration |
| **Chain Trajectory** | First 20 000 steps as a connected line — shows random walk structure |
| **Ghost Overlay** | Red points on 18%-opacity mesh — ground truth vs. sampled comparison |
| **Animated Walk** | Play button triggers real-time chain growth — burn-in + ergodicity visible |

---

## ⚙️ Installation

```bash
# Clone the repo
git clone https://github.com/himanshu/<repo-name>.git
cd <repo-name>

# Create a virtual environment (recommended)
python -m venv .venv
source .venv/bin/activate      # Linux/macOS
.venv\Scripts\activate         # Windows

# Install dependencies
pip install numpy trimesh rtree pyvista plotly jupyterlab

# Launch the notebook
jupyter lab tung-tung-sampler.ipynb
```

### Dependencies

| Package | Purpose |
|---------|---------|
| `numpy` | Numerical arrays & RNG |
| `trimesh` | 3-D mesh loading, SDF, geometry |
| `rtree` | Spatial indexing (used internally by trimesh) |
| `pyvista` | 3-D rendering backend |
| `plotly` | Interactive in-notebook 3-D visualisations |
| `jupyterlab` | Notebook environment |

> **Python ≥ 3.10** is recommended. Tested on the kernel `sampling-tung-tung-sahur (3.10.12)`.

---

## 🚀 Quick Start

```python
import numpy as np
import trimesh

# Load and concatenate mesh
mesh_or_scene = trimesh.load("data/tung_tung_sahur.glb")
mesh = trimesh.util.concatenate(tuple(mesh_or_scene.geometry.values()))

# Build SDF oracle
prox = trimesh.proximity.ProximityQuery(mesh)

# Run Metropolis–Hastings sampler
samples = metropolis_sampler(n_samples=50000, sigma=0.05)
# → shape: (50000, 3) — points living on the mesh surface
```

---

## 📐 Theory Primer

<details>
<summary><b>🎲 What is Monte Carlo Sampling?</b></summary>

Monte Carlo methods use repeated random sampling to approximate quantities that are hard to compute analytically:

$$\mathbb{E}_{x \sim p}[f(x)] \approx \frac{1}{N} \sum_{i=1}^{N} f(x_i), \quad x_i \sim p(x)$$

The challenge here is that our target $p(x)$ is concentrated on a 2-D surface embedded in 3-D space — a measure-zero set. Naive rejection sampling wastes virtually every sample. We need **MCMC**.

</details>

<details>
<summary><b>⛓️ What is MCMC?</b></summary>

Markov Chain Monte Carlo constructs a Markov chain whose **stationary distribution** equals the target $p(x)$. After a burn-in period, the chain's path is a sequence of (correlated) samples from $p(x)$.

The **Metropolis–Hastings** algorithm achieves this by satisfying **detailed balance**:

$$p(x) \, T(x \to x') = p(x') \, T(x' \to x)$$

This is guaranteed by the accept/reject step.

</details>

<details>
<summary><b>📏 What is a Signed Distance Function (SDF)?</b></summary>

For a closed surface $\mathcal{S}$, the SDF is:

$$d(\mathbf{x}) = \begin{cases} -\min_{\mathbf{s} \in \mathcal{S}} \|\mathbf{x} - \mathbf{s}\| & \mathbf{x} \text{ inside} \\ 0 & \mathbf{x} \in \mathcal{S} \\ +\min_{\mathbf{s} \in \mathcal{S}} \|\mathbf{x} - \mathbf{s}\| & \mathbf{x} \text{ outside} \end{cases}$$

The **zero level-set** $\{d = 0\}$ *is* the surface. We exploit this by making $|d(\mathbf{x})|$ the energy function in our Boltzmann distribution.

</details>

---

## 💡 Possible Extensions

- **Thinning** — keep every $k$-th sample to reduce chain autocorrelation and achieve more uniform surface coverage.
- **Parallel chains** — run multiple independent chains from different starting points and merge for faster global coverage.
- **Adaptive $\sigma$** — tune the proposal standard deviation online to maintain a target acceptance rate (typically 23–50%).
- **Hamiltonian Monte Carlo (HMC)** — use the SDF gradient to make directed, low-rejection proposals along the surface manifold.
- **Area-weighted direct sampling** — sample triangles proportionally to their area, then sample uniformly within each triangle (non-MCMC baseline for comparison).

---

## 📊 Results

| Metric | Value |
|--------|-------|
| Total samples | 50 000 |
| Chain length | 50 000 steps |
| Proposal std $\sigma$ | 0.05 |
| Inverse temperature $\beta$ | 100 |
| SDF queries | 50 000 |
| Output shape | `(50000, 3)` |

---

## 🛠️ Known Limitations

- **Autocorrelation**: Consecutive MCMC samples are correlated — the effective sample size (ESS) is lower than 50 000. Use thinning for downstream statistical analysis.
- **Mesh must be watertight**: The signed distance computation assumes a closed, manifold mesh for reliable inside/outside classification.
- **Single chain**: One chain may miss isolated surface regions if $\sigma$ is too small relative to the mesh scale. Consider multiple restarts.
- **No burn-in discard**: Early samples during the chain's snap-to-surface phase are included. For rigorous applications, discard the first ~500 steps.

---

## 📚 References

- Metropolis, N., et al. (1953). *Equation of state calculations by fast computing machines.* Journal of Chemical Physics, 21(6), 1087–1092.
- Hastings, W. K. (1970). *Monte Carlo sampling methods using Markov chains and their applications.* Biometrika, 57(1), 97–109.
- Trimesh documentation — [trimsh.org](https://trimsh.org/)
- Plotly Python documentation — [plotly.com/python](https://plotly.com/python/)

---

## 👨‍💻 Author

<div align="center">

**Himanshu**  
M.Tech — Computational Data Science  

*"Why sample uniformly when you can walk intelligently?"*

</div>

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

*Built with 🎲 randomness, ⛓️ Markov chains, and a deep respect for the Tung Tung Tung Sahur.*

</div>
