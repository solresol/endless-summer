# IMPROVEMENTS.md

*Analysis date: 2026-07-11*

**endless-summer** is a small data-science repo of Jupyter analyses of Sydney (and other Australian cities') weather, built on Bureau of Meteorology daily temperature and rainfall CSVs. It contains three notebooks (`cities-of-summer.ipynb`, `farewell-winter-endless-summer.ipynb`, `rainfall.ipynb`), one derived dataset (`sydney-yearly-data.csv`), and a two-line README. The last substantive work was the end-of-2018 analysis (`last_full_year = 2018`); the repo has been dormant since. Working tree is clean; there is no remote-visible TODO list, no environment spec, and no tests. No committed secrets were found.

## Bugs & Fixes

1. **Missing input data breaks reproducibility.** All three notebooks read files that are not in the repo:
   - `farewell-winter-endless-summer.ipynb` reads `data/weather/IDCJAC0011_066062_1800_Data.csv` and `IDCJAC0010_066062_1800_Data.csv` — no `data/` directory exists.
   - `rainfall.ipynb` and `cities-of-summer.ipynb` open BoM zip files (`IDCJAC0009_066062_1800.zip` etc.) that were removed in commit 5b05f10 ("Remove some files that never should have been here"). Add a small download script (or documented `curl` commands against the BoM data service) so `Run All` works on a fresh clone, or check in the zips via Git LFS if licensing allows.
2. **Deprecated pandas API.** `parse_dates=[['Year','Month','Day']]` (column-combining form) was deprecated in pandas 2.0 and removed since; the notebooks will not run on a modern pandas. Replace with an explicit `pd.to_datetime(df[['Year','Month','Day']])` step in each notebook.
3. **Hard-coded `last_full_year`.** `farewell-winter-endless-summer.ipynb` still says `last_full_year = 2016` with a comment telling the reader to change it; `cities-of-summer.ipynb` hard-codes 2018. Compute it (`datetime.now().year - 1`) or set it once in a shared cell at the top.
4. **`rainfall.ipynb` extracts to the repo root.** `rainfall_zip.extract(...)` litters the working tree with `*_Data.csv`; extract into a gitignored `data/` directory (there is currently no `.gitignore` at all).

## Improvements

- **Refactor shared code.** `get_temperature_ranges`/the min-max merge logic is duplicated between `farewell-winter-endless-summer.ipynb` and `cities-of-summer.ipynb`. Move it into a small module (e.g. `bom.py`) imported by all notebooks.
- **Update the analysis.** Seven-plus additional years of data (2019–2025, including the Black Summer of 2019–20 and the 2021–22 La Niña floods) exist since the last run — rerunning would materially change the trend lines and is the most interesting single improvement.
- **Regenerate `sydney-yearly-data.csv`** at the same time; it currently ends around 2016/2018.

## Testing

- Add a smoke test that executes each notebook headlessly (`uv run --with jupyter,nbconvert jupyter nbconvert --execute`) so data-path and pandas-API breakage is caught. Even a single CI-less `make test` target would help.
- Unit-test `season()`/`binary_season()` and the extremes-counting logic once extracted to `bom.py`.

## Documentation

- README.md is two lines. Add: what each notebook shows (with one headline chart), where the BoM data comes from (station 066062, Observatory Hill; product codes IDCJAC0009/0010/0011), how to fetch it, and how to run (`uv run jupyter lab`).
- State a data license/attribution note for BoM data.

## Security

- Nothing sensitive found; data is public weather data. Keep raw BoM downloads out of git via `.gitignore` to avoid re-committing bulk data (the reason for commit 5b05f10).

## Housekeeping / Modernization

- **Adopt uv with a `pyproject.toml`**: `uv init && uv add pandas matplotlib seaborn scikit-learn jupyter`, then run everything via `uv run`. There is currently no dependency spec of any kind (and do not add a requirements.txt — pyproject + uv.lock only).
- Add `.gitignore` (`data/`, `*.zip`, `*_Data.csv`, `.ipynb_checkpoints/`).
- Consider `nbstripout` or jupytext pairing: the notebooks are 1.1 MB and 2.4 MB mostly because of embedded output images, which makes diffs unreviewable.
- `cities-of-summer.ipynb` imports `matplotlib` but uses seaborn/pandas plotting; tidy imports while refactoring.

## Quick Wins

1. Create `.gitignore` and `pyproject.toml` (10 minutes).
2. Fix the deprecated `parse_dates` usage and hard-coded years.
3. Write a `fetch_data.sh`/`fetch_data.py` for the BoM zips so notebooks run from a fresh clone.
4. Rerun `farewell-winter-endless-summer.ipynb` with data through 2025 and refresh `sydney-yearly-data.csv`.
