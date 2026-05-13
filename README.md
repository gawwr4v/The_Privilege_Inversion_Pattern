# LCD Simulation Artifact

This folder is ready to be pushed to GitHub as the public simulation artifact for *Privilege Inversion Pattern: A Cross-Stage Framework for Data Contamination in Agentic AI*.

## Contents

- `lcd_simulation_notebook.ipynb` — the main runnable notebook.
- `lcd_simulation_detailed_report.md` — a plain-language walkthrough of the notebook and its limits.
- `requirements.txt` — the Python packages needed to run the notebook.
- `simulation_figures/section_a_simulation.png` — alignment / preference-poisoning figure.
- `simulation_figures/section_b_simulation.png` — sleeper-persistence figure.
- `simulation_figures/section_c_simulation.png` — deployment-time trust-boundary figure.
- `simulation_figures/section_d_simulation.png` — multi-agent cascade figure.

## What Readers Can Do

Readers can open the notebook in Jupyter, run it end-to-end, and reproduce the main simulation panels used in the paper. The notebook is intended for mechanism-level inspection and reproduction, not as a production benchmark or a claim about universal poisoning thresholds.

## Quick Start

1. Install the packages in `requirements.txt`.
2. Open `lcd_simulation_notebook.ipynb` in Jupyter.
3. Run the notebook from top to bottom.
4. Compare the regenerated outputs with the PNGs in `simulation_figures/`.

## Scope Note

This artifact is a supplementary research package. It helps readers inspect the paper's synthetic simulations, but it does not validate the full LCD architecture in production settings.

