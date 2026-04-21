# Hints

Start with the issue description on GitHub. Try to find and fix the problem yourself before opening this file. Each issue has three hint levels — stop as soon as you have enough to proceed.

Issues are grouped by language. Use the label on each GitHub issue (`python`, `r`, or `markdown`) to find hints for the issues you are working on.

## Python issues

### P1 — Hardcoded output path defined but never used

**Hint 1 — where to look**

Near the top of `src/analyse_spacewalks.py`, just after the data loading block, look for a variable called `output_dir`. Then search the rest of the script for where figures are saved.

**Hint 2 — what to change**

First, make `output_dir` a relative path:

```python
output_dir = 'results/figures/'
```

Then update each `plt.savefig()` call to use it. You'll need `os.path.join()` to combine the directory with the filename safely:

```python
import os
plt.savefig(os.path.join(output_dir, 'fig_cumulative_hours.png'), dpi=150)
```

**Hint 3 — why this matters**

Right now the script works by coincidence — the `savefig()` calls happen to use the correct relative path. But `output_dir` is misleading: it points to an absolute path that doesn't exist on any other machine. A collaborator reading the script would trust `output_dir` as the source of truth, then be confused when figures appear somewhere else. Using a single variable consistently means there's one place to change if the output location ever moves.

### P2 — `parse_duration()` never called

**Hint 1 — where to look**

Find the function definition for `parseDuration` in the script. Look at the lines immediately below it.

**Hint 2 — what to uncomment**

There is a commented-out line just below the function definition:

```python
# df['duration_hrs'] = df['duration'].apply(parseDuration)
```

Remove the `#` to uncomment it. Note that this line sits above the figure blocks — it creates `duration_hrs` once for the whole script, but the figure blocks currently recalculate duration themselves. After uncommenting, the figure blocks will use the pre-calculated column instead.

**Hint 3 — also fix the function name**

The function is named `parseDuration` (camelCase), but Python convention (PEP 8) is `snake_case`. Rename it:

```python
def parse_duration(d):
    ...
```

Then update the call site to match:

```python
df['duration_hrs'] = df['duration'].apply(parse_duration)
```

### P3 — Axis labels missing

**Hint 1 — where to look**

Each of the three figure blocks (`fig1`, `fig2`, `fig3`) has mmissing label lines.

**Hint 2 — what to uncomment**

In each figure block, add the `set_xlabel()` and `set_ylabel()` lines. For example in Figure 1:

```python
ax1.set_xlabel('Year')
ax1.set_ylabel('Cumulative EVA time (hours)')
```

**Hint 3 — check consistency with the manuscript**

Once added, check that the axis label wording matches the corresponding figure caption in `manuscript/manuscript.qmd`. The caption describes what the axes represent — the labels should be consistent.

### P4 — Inaccessible colour palette

**Hint 1 — where to look**

Near the top of the script, after the data loading block, find the lines that define `colour_usa` and `colour_russia`.

**Hint 2 — what to change**

Replace the current red/green values with a colourblind-safe alternative:

```python
colour_usa    = '#377eb8'   # blue
colour_russia = '#ff7f00'   # orange
```

**Hint 3 — check all figures**

Both colours are used in the duration distribution figure (Figure 2) and the top astronauts figure (Figure 3). After changing the values, re-run the script and check both figures look correct and the legend is still accurate.

### P5 — Unused import

**Hint 1 — where to look**

Look at the import block at the top of the script (the first ~10 lines).

**Hint 2 — what to remove**

The `csv` module is imported but never used. Remove this line:

```python
import csv
```

**Hint 3 — check for others**

Scan the rest of the import block. Is every imported module actually used somewhere in the script? A quick way to check: search the file for the module name (e.g. `numpy` → search for `np.`). Remove any that have no usage.

### P6 — Missing docstrings and comments

**Hint 1 — where to start**

Find the `parse_duration` function definition. It currently has one inline comment but no docstring. A docstring is the first thing inside the function body, wrapped in triple quotes.

**Hint 2 — what a good docstring looks like**

A minimal but useful docstring for `parse_duration` might look like:

```python
def parse_duration(d):
    """
    Convert a duration string in H:MM format to decimal hours.

    Parameters
    ----------
    d : str
        Duration string, e.g. '6:30'

    Returns
    -------
    float or None
        Duration in decimal hours, or None if the string cannot be parsed.
    """
```

**Hint 3 — meaningful comments elsewhere**

Once the docstring is in place, scan the rest of the script for places where a reader might ask "why is this done?". The crew-splitting loop is a good candidate — a one-line comment explaining that crew names are semicolon-delimited and need expanding to one row per astronaut adds real value. Contrast this with a comment like `# Load data` above an `open()` call — that adds nothing and should be removed.


## R issues

### R1 — Hardcoded output path defined but never used

**Hint 1 — where to look**

Near the top of `src/analyse_spacewalks.R`, just before the data loading line, look for a variable called `output_dir`. Then search the rest of the script for where figures are saved with `ggsave()`.

**Hint 2 — what to change**

First, make `output_dir` a relative path:

```r
output_dir <- 'results/figures/'
```

Then update each `ggsave()` call to use it with `file.path()`:

```r
ggsave(file.path(output_dir, 'fig_evas_per_year.png'), plot = p1,
       width = 8, height = 4, dpi = 150)
```

**Hint 3 — why this matters**

Right now the script works by coincidence — the `ggsave()` calls happen to use the correct relative path. But `output_dir` is misleading: it points to an absolute path that doesn't exist on any other machine. A collaborator reading the script would trust `output_dir` as the source of truth, then be confused when figures appear somewhere else. Using a single variable consistently means there's one place to change if the output location ever moves.

### R2 — `parse_duration()` never called

**Hint 1 — where to look**

Find the `parse_duration` function definition near the top of the script. Look at the line immediately below it.

**Hint 2 — what to uncomment**

Remove the `#` from the commented-out line:

```r
# df$duration_mins <- sapply(df$duration, parse_duration)
```

becomes:

```r
df$duration_mins <- sapply(df$duration, parse_duration)
```

**Hint 3 — simplify Figure 3**

The Figure 3 block currently recalculates duration inline using a `rowwise()` approach. Now that `df$duration_mins` exists, simplify the block:

```r
df_scatter <- df |>
  filter(!is.na(date), !is.na(duration_mins),
         country %in% c("USA", "Russia")) |>
  mutate(duration_hrs = duration_mins / 60)
```

This removes the duplicate duration-parsing logic.

### R3 — Axis labels missing

**Hint 1 — where to look**

In each of the three figure blocks (`p1`, `p2`, `p3`), axis labels are missing.

**Hint 2 — what to uncomment**

Add the `labs()` line in each figure. For example in Figure 1:

```r
labs(x = "Year", y = "Number of EVAs", fill = "Country") +
```

Make sure the `+` at the end of the line above `labs()` is present, and that `labs()` itself also ends with `+` before `theme_minimal()`.

**Hint 3 — check consistency with the manuscript**

Confirm that the axis label wording is consistent with the corresponding figure captions in `manuscript/manuscript.qmd`.

### R4 — Inaccessible colour palette

**Hint 1 — where to look**

Find the `colour_map` variable near the top of the script, after the data loading block.

**Hint 2 — what to change**

Replace the current red/green values with a colourblind-safe alternative:

```r
colour_map <- c("USA" = "#377eb8", "Russia" = "#ff7f00")
```

**Hint 3 — check all figures**

`colour_map` is passed to `scale_fill_manual()` in Figure 1 and `scale_colour_manual()` in Figures 2 and 3. Because it is defined once at the top, changing it here updates all three figures automatically. Re-run the script and verify.

### R5 — Unused library

**Hint 1 — where to look**

Look at the `library()` calls at the top of the script.

**Hint 2 — what to remove**

`library(knitr)` loads a package used for rendering R Markdown documents. It is not needed in a standalone analysis script. Remove it:

```r
library(knitr)   # delete this line
```

**Hint 3 — check for others**

Go through the remaining `library()` calls. Search the script for functions from each package to confirm they are actually used. For example, `tidyr` is loaded — is any `tidyr` function called?

### R6 — Missing function documentation and comments

**Hint 1 — where to start**

Find the `parse_duration` function definition. In R scripts (as opposed to packages), the convention is a structured comment block directly above the function rather than inside it.

**Hint 2 — what a good comment header looks like**

A minimal but useful header for `parse_duration` might look like:

```r
# parse_duration
# Converts a duration string in H:MM format to total minutes.
#
# Args:
#   d (character): Duration string, e.g. "6:30"
#
# Returns:
#   integer: Total duration in minutes, or NA if the string cannot be parsed.
parse_duration <- function(d) {
  ...
}
```

**Hint 3 — meaningful comments elsewhere**

Once the function header is in place, scan the rest of the script for places where a reader might ask "why is this done?". The Figure 3 block is a good candidate — it uses `rowwise()` with an inline duration calculation that duplicates logic defined elsewhere. A comment explaining why this approach was taken (or a note that it should be refactored once Issue R2 is resolved) adds real value. Contrast this with a comment like `# Load data` above an obvious `fromJSON()` call — that adds nothing and should be removed.

## Manuscript issues

### M1 — Broken figure cross-reference

**Hint 1 — where to look**

Open `manuscript/manuscript.qmd`. Search for `fig-duration-over_time` (note the underscore).

**Hint 2 — what to change**

There are two places to fix:

1. The image embed line — change `{#fig-duration-over_time}` to `{#fig-duration-over-time}` (hyphen, not underscore)
2. The in-text cross-reference — change `@fig-duration-over_time` to `@fig-duration-over-time`

**Hint 3 — check all labels**

Scan all six `{#fig-...}` labels and their `@fig-...` references in the manuscript. They should all use hyphens consistently. Fix any others that mix hyphens and underscores.

### M2 — Uninformative figure caption

**Hint 1 — where to look**

Find the first figure embed in the Results section. Its caption reads `Figure 1.`

**Hint 2 — what to change**

Replace the placeholder caption with something informative. A good caption describes: what the figure shows, the variables on each axis, and any grouping or colour coding. For example:

```
Annual EVA frequency by country, 1965–present. Bars represent the total number of spacewalks recorded per year, coloured by crewing agency (USA or Russia).
```

**Hint 3 — check other captions**

While you're here, review the other five figure captions. Are they all sufficiently informative? Would a reader encountering the figure in isolation understand what it shows?

### M3 — R version mismatch in Methods

**Hint 1 — where to look**

Search `manuscript/manuscript.qmd` for "version 4.3".

**Hint 2 — what to change**

Update the version number to match the R version specified in the GitHub Actions workflow. Open `.github/workflows/render.yml` and check the `r-version` field to confirm the correct version.

**Hint 3 — general principle**

The Methods section should describe the exact computational environment used to produce the results — it should match the `render.yml` workflow and ideally the `renv.lock` file too. Keeping these consistent is an important part of reproducibility.

### M4 — Placeholder author details *(good first issue)*

**Hint 1 — where to look**

The YAML front matter is at the very top of `manuscript/manuscript.qmd`, between the `---` lines.

**Hint 2 — what to change**

Replace the `name` and `affiliation` fields under `author:` with real names and institutions. The structure to follow is:

```yaml
author:
  - name: "Your Name"
    affiliation: "Your Institution"
```

**Hint 3 — you can add an ORCID**

Quarto supports ORCID identifiers in the author block:

```yaml
author:
  - name: "Your Name"
    affiliation: "Your Institution"
    orcid: "0000-0000-0000-0000"
```

### M5 — Unfilled placeholders in Data Availability

**Hint 1 — where to look**

Search `manuscript/manuscript.qmd` for `to be completed`.

**Hint 2 — what to fill in**

There are three placeholders to complete:

- `[repository URL — to be completed]` → add the GitHub URL for the forked repo
- `[licence — to be completed]` → add the licence you chose in Exercise 4.3
- `[DOI — to be completed]` → add the Zenodo DOI from Exercise 4.6

**Hint 3 — timing**

This issue can only be fully resolved after Session 4. The repository URL and licence can be added after Exercise 4.3; the DOI only after Exercise 4.6. It is good practice to open the issue now so it is not forgotten.


## General issues

### G1 — Separate analysis and visualisation

**Hint 1 — where to draw the line**

Read through the script and ask: does this line *produce* data, or does it *display* data? Everything up to and including duration parsing, crew splitting, and filtering belongs in a processing script. Everything that touches `plt` / `ggplot` belongs in a visualisation script.

**Hint 2 — how to share data between scripts**

The simplest approach is to save the processed dataframe as an intermediate file at the end of the processing script, then load it at the top of the visualisation script:

```python
# end of process_spacewalks.py
df.to_csv('results/spacewalks_processed.csv', index=False)

# top of visualise_spacewalks.py
import pandas as pd
df = pd.read_csv('results/spacewalks_processed.csv', parse_dates=['date'])
```

```r
# end of process_spacewalks.R
write.csv(df, 'results/spacewalks_processed.csv', row.names = FALSE)

# top of visualise_spacewalks.R
df <- read.csv('results/spacewalks_processed.csv')
df$date <- as.Date(df$date)
```

**Hint 3 — don't forget to update the workflow**

Once you have two scripts, the GitHub Actions workflow needs to run them in the correct order. In `render.yml`, update the run steps so the processing script runs before the visualisation script — otherwise the intermediate file won't exist when the visualisation script tries to load it.


### G2 — Raw data committed to the repository

**Hint 1 — where to look**

The `data/` directory is tracked by Git and contains `data.json`. Check your `.gitignore` — `data/` is not listed there, which is why it was committed in the first place.

**Hint 2 — how to download at runtime**

Once the data is deposited on Zenodo or Figshare, replace the local file read at the top of each script with a download step. Both approaches use only built-in language tools — no extra dependencies needed:

```python
# Python — replace the open() call with:
import urllib.request
data_url = 'https://zenodo.org/records/XXXXXXX/files/data.json'
urllib.request.urlretrieve(data_url, 'data.json')
with open('data.json', 'r') as data_f:
    raw = json.load(data_f)
```

```r
# R — replace the fromJSON() call with:
data_url <- 'https://zenodo.org/records/XXXXXXX/files/data.json'
download.file(data_url, 'data.json')
df <- fromJSON('data.json', flatten = TRUE)
```

**Hint 3 — removing committed data from Git history**

Adding `data/` to `.gitignore` stops Git tracking it going forward, but the files are still in the repository history. To remove them cleanly:

```bash
git rm --cached data/data.json
git commit -m "Remove raw data from version control"
```

The `--cached` flag removes the file from Git's index without deleting it from your local disk. After pushing, the file will no longer appear in the repository on GitHub — but it will still exist locally until you delete it manually or add it to `.gitignore`.