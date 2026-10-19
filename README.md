# Replication code: the illegal corporate service provider industry in the Netherlands

Code for:

> Garcia-Bernardo, J., Witteman, J. & Vlaanderen, M. (2022). Uncovering the size of the illegal corporate service provider industry in the Netherlands: a network approach. *EPJ Data Science* 11. https://doi.org/10.1140/epjds/s13688-022-00334-w

The notebooks build a network of Dutch companies, their directors, addresses and owners (Orbis, July 2020). They compute the features of every director with positions in at least three independent companies and flag directors that resemble licensed corporate service providers (CSPs), first with nearest neighbours and then with a penalized logistic regression. A manually annotated sample of the flagged directors is then used to estimate the size of the illegal CSP sector.

## Contents

| File | What it does | Paper |
|---|---|---|
| `1_download_clean_orbis_and_wtttc.ipynb` | Merges and cleans the Orbis exports (companies, current and previous addresses, directors, ownership), builds the list of licensed CSPs from the DNB register and reads the offshore leaks | 3.1.1, 3.1.2, 3.1.4, A.2, A.3 |
| `2_network_analysis.ipynb` | Network layers, licensed CSPs among the directors (DNB list and augmented list), features of each director | 3.1.2, 3.1.3, 3.2, A.4 |
| `3_analyze_samples.ipynb` | Nearest neighbours, robustness tests, samples for the manual annotation, logistic regression, size of the illegal sector, extrapolation to small CSPs. Includes the outputs (results and figures) of a run on the original data | 3.3, 4, A.7, A.9, A.10, Figures 3–7 |
| `Figures/` | Figures saved by notebook 3 in that run | Figures 3–7, A.7, A.9, A.10 |
| `data_raw_other/Internetregister-WTTTK.csv` | Public register of licensed trust offices (DNB, June 2020) | 3.1.2 |
| `data_raw_other/beleggingsondernemingen.csv` | Public register of investment firms (AFM, October 2020), only used in the annotation spreadsheets | 3.3.3 |
| `environment.yml` | Python environment | |

Run the notebooks in order from this folder. They create `data_processing/`, `data_final/` and `Figures/`.

## Data not included

- **Orbis** (Bureau van Dijk) data is licensed and cannot be shared, so you need your own Orbis access. Appendix A.2 of the paper lists the variables exported for all Dutch companies. Save the Excel exports in `data_raw_orbis/`. Notebook 1 recognises the four types of exports (addresses, legal events, directors, financials and ownership) by their columns.
- **ICIJ Offshore Leaks** (public): https://offshoreleaks.icij.org/pages/database. The paper used the csv files of the Bahamas Leaks, Offshore Leaks and Paradise Papers. Put each leak in its own folder in `data_raw_other/` with "offshore" in the folder name (e.g. `data_raw_other/offshoreleaks_paradise_papers/`), with "entity", "officer", "intermediary" and "address" in the names of the node files. If only the full database is available, keep these three leaks and adapt the last cell of notebook 1.
- **Manual annotations** of the flagged directors contain personal data and are not included. Notebook 3 reads them from `data_processing/`: `annotated_sample_nn.xlsx`, `annotated_sample_logit_1.xlsx` and `annotated_sample_logit_2.xlsx` (the spreadsheets created by `save_sample()`, with the columns added by the annotators and the final labels coded 1–5), and `annotated_sample_nn_raw.xlsx` (the nearest neighbours spreadsheet before coding the labels as numbers), used to select the positive examples of the logistic regression.
- The intermediate files in `data_processing/` and `data_final/` are derived from Orbis and contain personal data (names of directors). **Do not share them, or the outputs of notebooks 1 and 2.**

## Running

```bash
conda env create -f environment.yml
conda activate csp_replication
jupyter notebook
```

Environment variables:

- `GOOGLE_API_KEY`: Google geocoding API, used to find the postcodes of the postal addresses of licensed CSPs (notebook 1).
- `ORBIS_EMAIL` and `ORBIS_EMAIL_PASSWORD` (optional): mailbox that received the links to the Orbis exports, to download them automatically.

Some steps of notebook 2 are slow (they loop over all directors); set `RECOMPUTE = False` at its top to reuse the intermediate files of a previous run. Notebook 3 reuses the saved t-SNE embedding when there is one (set `RECOMPUTE = True` to compute it again). Set `CREATE_ANNOTATION_SAMPLES = True` in notebook 3 to create new spreadsheets for manual annotation (the codebook is in Appendix A.5).

## Names used in the code

- *RTO* (registered trust office): licensed CSP. `RTO_confidence`: 7 = licensed CSP from the DNB list (step i in Section 3.1.2), 6 = holds a position in a licensed CSP (step iia), 4 = added based on their network (step iib), 1 = other directors, 0 = directors not providing corporate services (Section 3.1.3).
- *GUO*: global ultimate owner. Companies with the same GUO are not independent.
- `bvd_id`: Orbis company ID. `dir_id`: Orbis director ID (starting with "C" for corporate directors).
- `ix`: flagged by the nearest neighbours. `ix_logit`: flagged by the logistic regression.

## Reproducing the numbers in the paper

- Orbis data changes over time, so a new download will not contain exactly the same companies and directors.
- t-SNE, the bootstrap, the robustness tests and the Monte Carlo draws are random, and the logistic regression can flag a few directors more or less depending on the version of scikit-learn. Expect small differences with the numbers in the paper.
