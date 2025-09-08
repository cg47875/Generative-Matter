# Generative-Matter-(120k)
LLM-drive material database

This repository contains **120,000 machine-readable synthesis recipes** for candidate inorganic materials intended for exploratory materials research, prototyping, and architectural studies. Each row links a **chemical composition** to a **model-generated recipe** in markdown form (with steps, notes, and parameters embedded in the text).

> ✅ For convenience, the dataset is provided as **two CSV files**. Use both together for the full 120,000 rows.

---

## Repository structure

```
data/
  Generative_Matter_Recipes_2parts_Part1.csv   # ~60k rows
  Generative_Matter_Recipes_2parts_Part2.csv   # ~60k rows
  # (ZIP versions may also be provided as .zip with a single CSV inside)
```

> **Note on large files:** If any CSV exceeds GitHub’s 100 MB file limit, use **Git LFS** or attach files under **GitHub Releases**. Zipped variants are also provided for easier download.

---

## Columns

The dataset columns:

- `Composition`
- `response_body_choices`

---

## How to open the data

### Python (pandas)

**Option 1: CSV files directly**
```python
import pandas as pd

# Single file
df = pd.read_csv("data/Generative_Matter_Recipes_2parts_Part1.csv")

# Combine both 2-part files
parts = [
    "data/Generative_Matter_Recipes_2parts_Part1.csv",
    "data/Generative_Matter_Recipes_2parts_Part2.csv",
]
df = pd.concat((pd.read_csv(p) for p in parts), ignore_index=True)
```

**Option 2: Read from ZIP (if using zipped downloads)**
```python
import pandas as pd

# Pandas can read a ZIP that contains a single CSV
df = pd.read_csv("data/Generative_Matter_Recipes_2parts_Part1.zip")
```

> Tip: If you encounter memory pressure, read in chunks (`chunksize=...`) and process incrementally.

### DuckDB (fast, SQL over CSV)

```sql
-- From a DuckDB shell or Python

-- Read each file
SELECT COUNT(*) FROM read_csv_auto('data/Generative_Matter_Recipes_2parts_Part1.csv');
SELECT COUNT(*) FROM read_csv_auto('data/Generative_Matter_Recipes_2parts_Part2.csv');

-- Or use a wildcard to read both
SELECT * FROM read_csv_auto('data/Generative_Matter_Recipes_2parts_Part*.csv') LIMIT 5;
```

---

## Parsing the recipe text

If your dataset includes the `response_body_choices` column, it is stored as a Python-style dict string (single quotes). First **parse it** and then extract the markdown recipe at `['message']['content']`.

```python
import ast
import pandas as pd

df = pd.read_csv("data/Generative_Matter_Recipes_2parts_Part1.csv")

# Safely parse the dict-like string into a Python dict
parsed = df["response_body_choices"].map(ast.literal_eval)

# Extract the recipe markdown text
df["recipe_md"] = parsed.map(lambda x: x["message"]["content"])

# Keep only what you need
recipes = df[["Composition", "recipe_md"]]
print(recipes.head(3))
```

---

## Typical workflows

- **Join parts:** Load both files and `pd.concat(...)` to get the full dataset.
- **Filter by element(s):** Filter the `Composition` string (e.g., rows containing `'Si'`), or parse to element–count pairs with a simple parser if you need structured stoichiometry.
- **Extract structured parameters:** The recipe text is markdown; you can regex for temperatures, times, solvents, etc., or run an additional parser/LLM pass to normalize steps into a schema.

---

## Example: combine two parts and extract recipes

```python
import ast, pandas as pd

parts = [
    "data/Generative_Matter_Recipes_2parts_Part1.csv",
    "data/Generative_Matter_Recipes_2parts_Part2.csv",
]
df = pd.concat((pd.read_csv(p) for p in parts), ignore_index=True)

# If your dataset includes 'response_body_choices' as a Python-dict-like string:
if "response_body_choices" in df.columns:
    parsed = df["response_body_choices"].map(ast.literal_eval)
    df["recipe_md"] = parsed.map(lambda x: x["message"]["content"])
    # Save a lightweight version (composition + recipe only)
    df[["Composition", "recipe_md"]].to_csv("recipes_compact.csv", index=False)
```

---

## Companion files

### Generative Matter Recipe Book.pdf
A curated PDF that presents selected compositions from this dataset alongside formatted, human-readable recipes and visual panels to support quick review and presentation. Open with any standard PDF viewer (Preview, Acrobat, browser). Useful for sharing snapshots of the project without loading large CSVs.

**Suggested use**
- Browse visually to shortlist candidate compositions.
- Cross-reference the **Composition** values with the CSV parts in `data/`.
- If the PDF is large, consider hosting it under **GitHub Releases** for reliable downloads.

### Filtered Materials_300_descriptions.txt
A plain-text list of **300** filtered compositions, each followed by a one‑line descriptive summary (e.g., crystal structure, appearance, texture, reflectivity). This file is convenient metadata for quick filtering, plotting labels, or seeding visualization pipelines.

**Format**
Each line is a single record:
```
<Composition>, <short description>
```

**Python quickload**
```python
import pandas as pd

records = []
with open("Filtered Materials_300_descriptions.txt", "r", encoding="utf-8") as f:
    for line in f:
        line = line.strip()
        if not line:
            continue
        comp, desc = line.split(",", 1)  # split on the first comma
        records.append({"Composition": comp.strip(), "description": desc.strip()})

df_desc = pd.DataFrame(records)

# Example: join with the recipes (after loading/concatenating the CSV parts)
# df_recipes = ...  # DataFrame containing 'Composition' and recipe text
# df_merged = df_recipes.merge(df_desc, on="Composition", how="left")
```

**Typical workflows**
- Join `df_desc` with your main recipes DataFrame on `Composition` to enrich records with descriptors.
- Filter by keywords in the description (e.g., "layered", "rhombohedral", "telluride").
- Use descriptors to drive figure captions or downstream selection heuristics.

---

## Data quality & caveats

- Recipes are **model-generated** and intended for research and prototyping, **not** for unsupervised laboratory execution. Use expert judgment and follow lab safety protocols.
- The recipe content column can contain long texts (markdown). If a spreadsheet viewer appears to freeze, use pandas/DuckDB instead.

---

## Attribution

**Material compositions** in the `Composition` column were extracted from the **Material Discovery** (GNoME) work:

```
@article{merchant2023scaling,
  title={Scaling deep learning for materials discovery},
  author={Amil Merchant and Simon Batzner and Samuel S. Schoenholz and Muratahan Aykol and Gowoon Cheon and Ekin Dogus Cubuk},
  journal={Nature},
  year={2023},
  doi={10.1038/s41586-023-06735-9},
  href={https://www.nature.com/articles/s41586-023-06735-9},
}
```

If you use this dataset, please cite both this repository and the above paper.

---

## License

- **Code**: MIT License © 2025 Catherine Graubard.
- **Data (CSV files)**: Creative Commons Attribution 4.0 International (CC BY 4.0) © 2025 Catherine Graubard.

### How to attribute
If you use the dataset, please credit:
> *Catherine Graubard. “Generative Matter Recipes (120k).” 2025. GitHub repository: <repo URL>.*

In addition, please cite the source of the **Composition** column (Merchant et al., 2023) as shown above.

### Notes
- **MIT** applies to all scripts, notebooks, and code in this repo. It excludes composition column which its license by the reference mentioned above.
- **CC BY 4.0** applies to the dataset files under `data/` (CSV/ZIP).
- For CC BY 4.0, include attribution to **Catherine Graubard** and link back to this repository when redistributing or adapting the data.

---

## Contact / Issues

Open an issue in this repository with questions, problems parsing the files, or suggestions for improvements.
Contributions (e.g., parsers, schema normalizers, or notebooks) are welcome.

*Last updated: 2025-09-08*
