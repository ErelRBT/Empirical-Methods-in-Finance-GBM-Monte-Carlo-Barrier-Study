# Empirical Methods in Finance — GBM Monte Carlo Barrier Study

This repository contains a Jupyter Notebook for the **Empirical Methods in Finance**.

The objective is to use historical total return index data, calibrate a **Geometric Brownian Motion (GBM)** model, and run **Monte Carlo simulations** to estimate the probability that an asset reaches a **30% gain at least once within one year**.

A success is defined as:

max          (St/S0) ≥ 1.3.
0≤t≤1 year


The project compares two cases:

1. the **S&P 500 Total Return Index**;
2. a simple **60/40 buy-and-hold portfolio**, invested 60% in the S&P 500 Total Return Index and 40% in a U.S. Treasury Total Return Index.

---

## Repository structure

```text
.
├── Empirical_Methods_In_Finance.ipynb
├── S&P500_TR.csv
├── S&P500_US_Treasury_TR.csv
└── README.md
```

## Assignment overview

The notebook follows three main parts.

### Part A — Data preparation and returns

The notebook:

* loads the two CSV datasets with `pandas`;
* parses the date column;
* converts index levels to numeric values;
* drops missing observations;
* sorts observations by date;
* removes duplicate dates, if any;
* aligns the two series on common dates using an inner join;
* normalises each index to start at 1;
* computes daily log-returns.

The daily log-return is defined as:


r_t = log(S_t/S_{t-1})


The notebook then reports summary statistics for the return series, including mean, standard deviation, minimum and maximum.

### Part B — GBM calibration

For each return series, the notebook estimates annualised GBM parameters from daily log-returns.

The annualised volatility is computed as:

STD_Ann = data["DLR"].std() * np.sqrt(252)

The annualised drift is computed using the GBM relationship:

Ann_Drift = 252 * data["DLR"].mean() + 0.5 * STD_Ann**2


The notebook estimates these parameters for:

* the S&P 500 Total Return Index;
* the U.S. Treasury Total Return Index;
* the 60/40 portfolio used later in the Monte Carlo study.

### Part C — Monte Carlo barrier study

The notebook implements a function:

```python
gbm_simulate(mu, sigma, initial, n_paths, time_step, n_periods, rng)
```

This function simulates weekly GBM paths over a one-year horizon.

The Monte Carlo experiment uses:

* `M = 10_000` simulated paths;
* 52 weekly time steps;
* an initial value equal to 1;
* a barrier level equal to 1.3;
* a NumPy random generator initialised with seed 123.

For both the S&P 500 index and the 60/40 portfolio, the notebook computes:

1. the percentage of simulated paths that reach the 30% barrier at least once;
2. the relationship between expected return ( \mu ) and success probability, holding volatility fixed;
3. the average time to success in weeks, conditional on success;
4. visualisations of simulated paths and barrier probabilities.

## Portfolio construction

The 60/40 portfolio is constructed as a **buy-and-hold portfolio**:


V_t = 0.6 S^{eq}_t + 0.4 S^{bond}_t


where:

* (S^{eq}_t) is the normalised S&P 500 Total Return Index;
* (S^{bond}_t) is the normalised U.S. Treasury Total Return Index.

The portfolio is not rebalanced. This means that the portfolio weights are allowed to drift over time.

Daily portfolio log-returns are then computed from the portfolio level and used to estimate GBM parameters for the portfolio.

## Main outputs

The notebook produces:

* cleaned and aligned time series;
* normalised index levels;
* daily log-return series;
* summary statistics tables;
* annualised GBM parameter estimates;
* Monte Carlo estimates of barrier-hit probabilities;
* plots of success probability against expected return;
* simulated GBM path visualisations.

## Requirements

The notebook uses the following Python libraries:

```bash
pip install numpy pandas matplotlib
```

Required packages:

* `numpy`
* `pandas`
* `matplotlib`
* `math` from the Python standard library.

## How to run the project

1. Clone this repository:

```bash
git clone <repository-url>
cd <repository-name>
```

2. Place the two CSV files in the same folder as the notebook:

```text
S&P500_TR.csv
S&P500_US_Treasury_TR.csv
```

3. Open the notebook:

```bash
jupyter notebook Empirical_Methods_In_Finance.ipynb
```

or with JupyterLab:

```bash
jupyter lab Empirical_Methods_In_Finance.ipynb
```

4. Run the notebook from top to bottom.

## Reproducibility

All Monte Carlo simulations use a NumPy random number generator with seed 123:

```python
rng = np.random.default_rng(seed=123)
```

This ensures that the simulation results can be reproduced.

## Methodological comments

This project uses a GBM model for simplicity and clarity. However, the model has several limitations:

* volatility is assumed to be constant;
* returns are assumed to be normally distributed in log terms;
* shocks are assumed to be independent over time;
* fat tails and volatility clustering are ignored;
* the 60/40 portfolio is treated as a single GBM asset, even though it is built from two underlying assets;
* correlations between equity and bond returns are not explicitly modelled in the single-asset GBM approximation.

A more structural extension would simulate equity and bond returns jointly using a multivariate process with estimated correlation, then construct the portfolio path from the simulated asset paths.

