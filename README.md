# Homework 3: Northwind Sales Analysis

## What this is

Homework 3 for 15.561 Digital Revolution (Fall 2026) by Alexandra Lopez. It analyzes the Northwind sample dataset to answer three business questions: where revenue growth comes from, how dependent the business is on its largest customers, and which products to stop carrying, push harder or reconsider. All of the analysis is in `homework3.ipynb`. The write-up is presented as a single-page website, `15561-fa26-hw3-alopez10/index.html`.

## Requirements

- **Python 3.9.** The notebook was last run with Python 3.9.6.
- **Packages imported by the notebook:**
  - `pandas`, version 2.3.3 was used.
  - `matplotlib`, version 3.9.4 was used (`pyplot`, `ticker`, `lines`, `patches`).
  - `IPython.display`, which comes with Jupyter.
  - The standard-library modules `re` and `pathlib`.
- **Jupyter:** Jupyter Notebook/Lab, or VS Code with the Jupyter extension, running a Python 3 kernel.
- **Network access:** the notebook downloads the data live from Google Sheets each time it runs, so it needs an internet connection. It does not read any local data file.

## Data source

- **Google Sheet ID:** `1mD4Hn4NZdJlXMC4EJcSp27vg-_jbxUYJ95vQvMbSpVM`
- **Tabs and their gids:**

  | Tab | gid |
  |---|---|
  | Categories | 2047671455 |
  | Customers | 667254992 |
  | Employees | 1244043231 |
  | Orders Details | 1881541474 |
  | Orders | 885169187 |
  | Shippers | 806775973 |
  | Product | 2082189938 |
  | Suppliers | 1298325637 |

- **How each tab is read:** via the CSV export URL `https://docs.google.com/spreadsheets/d/<SHEET_ID>/export?format=csv&gid=<gid>`, using `pd.read_csv(url, thousands=",")` (cell 3). The audit in cell 9 downloads every tab a second time as raw text (`dtype=str`) so it can check the values exactly as typed.
- **The sheet must be publicly readable** ("anyone with the link can view"), or every read fails.
- **Edits to the sheet change the results.** If anyone edits the sheet, the numbers, figures and tables will change the next time the notebook runs.

## How to reproduce

1. Install the packages: `python3 -m pip install "pandas==2.3.3" "matplotlib==3.9.4" jupyter`
2. Open `homework3.ipynb` with this folder as the working directory. Jupyter and VS Code do this by default. The figure path in cell 16 is relative to this folder.
3. Run **Kernel → Restart & Run All**. Always run the notebook from the top in one pass, not cell by cell.
   - **Why:** the cleaning cell (cell 13) is not idempotent. A second run of it fails: `Product.Cheap` is already True/False, so the `t`/`f` mapping turns every value into NaN and its check fails, and the two dropped columns no longer exist.
   - **Order matters:** the correction in cell 4 must also run before cell 13.
4. The run writes **Figures 1–4** to `15561-fa26-hw3-alopez10/figures/figure1.png` … `figure4.png` at 200 dpi, overwriting the previous files. A full run with the current sheet reproduces the published PNGs pixel for pixel.
5. **Tables 1 and 2** are printed in the outputs of cells 22 and 24. In `index.html`, their values and the write-up text are typed into the page, not generated. If the data changes, update them there by hand.

## Where each exhibit comes from

Cell numbers count every notebook cell, markdown included, starting from 0 at the top. Each code cell also starts with a comment label, given in brackets, to help you find it.

| Exhibit on the site | Notebook cell | What it shows |
|---|---|---|
| Figure 1 | 18 (`# 1b)`) | Like-for-like revenue totals and order counts: Jul 4 – Dec 31, 1996 vs 1997, and Jan – Apr 1997 vs Jan – Apr 1998 |
| Figure 2 | 23 (`# 3c)`) | Revenue, customers and orders by cohort (returning vs new customers), Jan – Apr 1997 vs Jan – Apr 1998 |
| Figure 3 | 22 (`# 3b)`) | Cumulative customer revenue concentration curves for both Jan – Apr windows, with the top 1/5/10 marked |
| Table 1 | 22 (`# 3b)`) | The 14 customers in the top 10 of either Jan – Apr window, with rank and revenue in each (the last table the cell prints) |
| Table 2 | 24 (`# 4)`) | Products by status (active above reorder level, active at/below, discontinued): product counts, units sold in each window, units in stock and on order (the "Products by status" table) |
| Figure 4 | 24 (`# 4)`) | Scatter of each product's Jan – Apr 1998 units sold per month vs units in stock, by status |

**Numbers in the text that no cell prints:** the Section 3 figures for discontinued products in Jan – Apr 1998 (revenue of $58,213, which is 13.8% of the window, and $33,683 for Thüringer Rostbratwurst). Reproduce them by running this snippet after a full run:

```python
w = in_window(sales, "1998-01-01", "1998-04-30")
disc = w[w["ProductID"].isin(products.loc[products["Discontinued"], "ProductID"])]
print(f"Discontinued revenue, Jan–Apr 1998: ${disc.Revenue.sum():,.2f} "
      f"({disc.Revenue.sum() / w.Revenue.sum():.1%} of the window)")
print(disc.groupby("ProductName").Revenue.sum().sort_values(ascending=False).round(2))
```

The other numbers in the text can be read or derived from the outputs of cells 18, 22, 23 and 24.

## Analysis decisions that affect every number

- **Revenue** = `Orders Details.UnitPrice × Quantity × (1 − Discount)`. It uses the price actually charged on each order line, not `Product.Price`, which is the current list price.
- **Discount is rounded to 2 decimals** before use, because the raw values carry float noise (e.g. `0.15000001`).
- **Thousands separators are parsed on read** (`thousands=","`), because at least one freight value is stored as `"1,007.64"`.
- **Comparisons use like-for-like windows:** Jan 1 – Apr 30, 1997 vs Jan 1 – Apr 30, 1998, inclusive. The data runs from Jul 4, 1996 to May 6, 1998, so whole years are uneven and May 1998 is a partial month.
- **Orders are dated by `OrderDate`**, parsed with the explicit format `%m/%d/%Y`. Shipped and required dates are not used.
- **Cohorts are based on each customer's first order in the data:** "New" means the first order falls inside the window, and "Returning" means the customer ordered before it. History before Jul 4, 1996 is not available.
- **Product status comes from the Product tab:** discontinued if `Discontinued` is True, otherwise "at/below reorder level" when `UnitsInStock ≤ ReorderLevel`. The Product tab is an undated snapshot, so stock is compared with recent (Jan – Apr 1998) sales.

## Manual data correction

`Product.Cheap` was `"B"` for ProductID 38 (Côte de Blaye, sheet row 53). Every other value in the column is `t` or `f`. Because the product is the most expensive in the table, it was corrected to `"f"` in the notebook only, in cell 4. The Google Sheet itself is unmodified. The audit in cell 9 reads the sheet directly, so it still reports the original `"B"`.

## Repository layout

```
homework 3/
├── README.md                     this file
├── homework3.ipynb               the full analysis: load, audit, clean, charts
├── 15561-fa26-hw3-alopez10/      the website
│   ├── index.html                single page, CSS inline, no JavaScript
│   └── figures/                  figure1.png … figure4.png, written by the notebook
├── data/northwind.xlsx           local copy of Northwind; not read by the notebook
├── SKILL (3).md                  docstring skill used to document the notebook
```
