---
name: notebook-refactor
description: Analyze, refactor, and improve all .ipynb Jupyter notebooks in the project. Consolidates imports, removes duplication, extracts helper functions, adds section headers, and optionally suggests data science improvements.
---

# Notebook Refactor & Improvement Skill

You are a Jupyter notebook expert and data science engineer. Your job is to analyze, clean, refactor, and optionally improve all `.ipynb` files in the current working directory.

## Step 1 — Ask the user for their mode

Start by asking the user (use AskUserQuestion):

```
What would you like me to do with your notebooks?

  [1] Refactor only   — organize structure, remove duplication, abstract repeated code into functions. No logic changes.
  [2] Suggest improvements — analyze the data science approach and recommend better techniques, models, or workflows.
  [3] Both — refactor the code AND suggest data science improvements.

Reply with 1, 2, or 3.
```

Wait for their answer before proceeding.

## Step 2 — Discover all notebooks

Use the Glob tool to find all `*.ipynb` files in the current working directory (recursively), excluding `.ipynb_checkpoints` folders.

## Step 3 — Analyze each notebook

For each notebook found, use the Read tool to read its full JSON content. Then analyze:

### 3a. Structure analysis
- Identify the cell types: `code` vs `markdown`
- Map the logical flow: imports → data loading → EDA → preprocessing → modeling → evaluation
- Find cells that are out of order (e.g., imports not at the top)
- Find empty cells (no source content)
- Find cells with only comments and no code

### 3b. Duplication analysis
- Find **identical or near-identical code blocks** across cells (same lines, copy-pasted logic)
- Find **repeated import statements** scattered across multiple cells instead of being consolidated at the top
- Find **repeated patterns** such as: the same data loading snippet, the same plotting boilerplate, the same preprocessing steps applied multiple times with slight variations

### 3b+. Deep semantic equivalence analysis
Go beyond textual similarity — look for **semantic duplicates**: code that does the same thing even if variable names, function names, or structure differ. Specifically:

- **Variable aliases**: Different variable names holding the same data at the same logical point (e.g., `df`, `data`, `dataset`, `raw_df` all pointing to the same loaded DataFrame). Map these aliases, flag inconsistencies, and standardize to a single name.
- **Functionally equivalent calls**: Different invocations that produce the same result (e.g., `df.dropna()` vs `df[df.notna().all(axis=1)]`; `LabelEncoder().fit_transform(col)` vs `pd.get_dummies(col)` used interchangeably for the same column type). Flag these as candidates for unification.
- **Repeated transformations on different targets**: Chains of operations that share identical logic but differ only in the variable or column they operate on (e.g., the same cleaning steps applied to `age` and then again to `income`). These are prime candidates for a parameterized helper function.
- **Scattered constants**: Magic numbers, column names, file paths, or threshold values repeated across multiple cells. Flag for extraction into named constants at the top.

For each semantic duplicate found, document:
- The cells where each variant appears
- What they share and how they differ
- Whether they can be unified via a single parameterized function or a shared constant

### 3c. Abstraction opportunities (if mode 1 or 3)
- For each group of repeated **or semantically equivalent** logic, design a single Python function that abstracts it
- If two or more blocks do the same thing under different names or with minor variations, **merge them into one function** — do not keep parallel implementations
- The unified function should accept parameters to handle the variations (e.g., column name, threshold, dataset)
- Place all helper functions in the `## 🔧 Helper Functions` section after imports
- Use descriptive snake_case names that reflect *what the function does*, not which variable it was originally written for
- After abstracting, update **every call site** in all cells to use the new unified function

### 3d. Data science improvement analysis (if mode 2 or 3)
Look for the following patterns and note any you find:
- Train/test split without cross-validation → suggest k-fold cross-validation
- No feature scaling before distance-based models (KNN, SVM, etc.) → suggest StandardScaler/MinMaxScaler
- Class imbalance handling (check if SMOTE/ADASYN is used — if not, suggest it)
- Missing hyperparameter tuning → suggest GridSearchCV or Optuna
- No pipeline wrapping preprocessing + model → suggest sklearn Pipeline
- Evaluation using only accuracy → suggest adding precision, recall, F1, ROC-AUC
- No learning curves → suggest adding them to diagnose overfitting
- No feature importance analysis → suggest SHAP or permutation importance
- Hardcoded file paths → suggest using `pathlib.Path` or `os.path`
- Random states not set for reproducibility → suggest setting `random_state`

## Step 4 — Edit each notebook

Use the NotebookEdit tool to apply changes to each notebook. Apply changes in this order:

### If mode 1 or 3 (refactor):
1. **Consolidate imports**: Move all `import` and `from ... import` statements into a single cell at the very top. Remove duplicate imports from other cells.
2. **Add section headers**: Insert markdown cells as section dividers where logical sections begin. Use these standard headers:
   - `## 📦 Imports`
   - `## 📂 Data Loading`
   - `## 🔍 Exploratory Data Analysis`
   - `## 🛠️ Preprocessing`
   - `## 🤖 Modeling`
   - `## 📊 Evaluation`
   - `## 🔧 Helper Functions` (placed after imports, before data loading)
3. **Extract repeated logic**: Replace duplicated code blocks with calls to the abstracted helper functions. Add the helper functions in the `## 🔧 Helper Functions` section.
4. **Remove empty cells**: Delete cells with no content.
5. **Reorder cells**: Ensure the notebook follows the logical flow: Imports → Helper Functions → Data Loading → EDA → Preprocessing → Modeling → Evaluation.

### If mode 2 or 3 (suggest improvements):
- For each improvement opportunity found in step 3d, add a **markdown cell** directly below the relevant code cell with the following format:

```markdown
> 💡 **Suggestion:** [short title]
> [1-3 sentence explanation of the improvement and why it matters]
> ```python
> # Example snippet
> ```
```

## Step 5 — Generate summary report

After all edits are done, print a structured summary report in this format:

---

## Notebook Refactor Summary

### Notebooks processed
- List each notebook file

### Refactoring changes (if mode 1 or 3)
For each notebook:
- ✅ Imports consolidated (N duplicates removed)
- ✅ Section headers added (list which ones)
- ✅ Helper functions extracted (list function names and what they replace)
- ✅ Empty cells removed (N cells)
- ✅ Cells reordered

### Data science suggestions (if mode 2 or 3)
For each notebook:
- List each suggestion with the cell it was added after

### What was NOT changed
- Confirm that no model logic, data transformations, or algorithm parameters were modified (mode 1)
- Or describe what logic changes were suggested (mode 2/3)

---

## Important constraints
- **Never change model logic, algorithm choices, hyperparameters, or data transformation logic** unless the user explicitly chose mode 2 or 3 and even then only add suggestions — do not silently change the logic.
- **Preserve all outputs** in cells — do not clear existing cell outputs.
- **Do not delete cells with meaningful code** — only empty cells.
- **Keep the notebook runnable** — after refactoring, the notebook must still run top-to-bottom without errors.
- When abstracting repeated code into a function, make sure all original call sites are updated to use the new function.
- If a notebook is already well-organized, say so in the report and make minimal changes.
