# Mini Project B – Pokémon Special Attack and Type Design

This project uses **LASSO regression** to analyze how Pokémon **Special Attack (Sp. Atk)** depends on core stats and type combinations. The goal is to identify **underpowered types** on Special Attack and recommend where Game Freak could introduce new offensive Pokémon to increase player excitement and sales.

This was originally completed as a UCLA Anderson mini-project and has been refactored into a reproducible R workflow.

---

## Motivation

Offensive Pokémon with high Special Attack often drive:

- Player hype and engagement,
- Competitive team-building,
- Demand for certain cards and in-game assets.

We wanted to know:

> Which Pokémon types are systematically underpowered in Special Attack after controlling for other stats?

If some types are consistently low on Special Attack, they are good candidates for **new rare, high–Special Attack designs**.

---

## Data

Source: A Pokémon stats dataset (e.g., Game Freak index via Kaggle) with ~1,200 Pokémon and the following fields:

- `Index` – Pokédex / internal index  
- `Name` – Pokémon name  
- `Type 1`, `Type 2` – primary and secondary types  
- `Total` – total base stats  
- `HP`, `Attack`, `Defense`, `SP. Atk.`, `SP. Def`, `Speed` – base stats  

For modeling, we:

- Renamed `Type 1` → `Type_1`, `Type 2` → `Type_2`  
- Replaced `NA` in `Type_2` with `"None"`  
- Created **dummy variables** for each type (both Type_1 and Type_2).

---

## Methods

### 1. Preprocessing

In R (`tidyverse` + `fastDummies`):

- Read the CSV:

  - `pokemon_df <- read_csv("data/pokemon.csv")`

- Clean type fields:

  - `Type_1` and `Type_2` as factors  
  - `Type_2` NAs replaced with `"None"`

- Created dummy variables:

  - `dummy_cols(select_columns = c("Type_1", "Type_2"), remove_selected_columns = FALSE)`

- Built a numeric matrix of predictors:

  - Kept numeric columns for base stats and type dummies  
  - Dropped `Index` and the response column when forming `X`

### 2. LASSO regression setup

We modeled:

- **Response:** `SP. Atk.` (Special Attack)  
- **Predictors:**  
  - Other base stats (`HP`, `Attack`, `Defense`, `SP. Def`, `Speed`, `Total`)  
  - Dummy variables for each **Type_1** and **Type_2**

Using `glmnet`:

- Standardized predictors internally (glmnet default)
- Ran cross-validated LASSO:

  - `cv.glmnet(X, y, alpha = 1)`

- Extracted coefficients at `lambda.min` and summarized:

  - Total predictors vs selected vs eliminated
  - Which **types** had negative vs positive coefficients on Special Attack

### 3. Visualization

To interpret type effects:

- Pulled out type-related coefficients from the LASSO model (e.g., `Type_1_Fighting`, `Type_1_Normal`, `Type_1_Ground`, etc.).
- Created a bar chart of **Special Attack coefficients by type**, highlighting types that were clearly underpowered.

---

## Key Findings

At the chosen LASSO penalty (`lambda.min`):

- The model retained a subset of stats and type dummies (e.g., 15 of 43 predictors).
- Core stats like **HP, Attack, Defense, SP. Def, Speed** had expected relationships with `SP. Atk.` (penalized but still informative).
- For type-specific coefficients (Type_1 dummies):

  - **Fighting**, **Normal**, and **Ground** types had **notably negative coefficients** on Special Attack.  
  - Types like **Psychic**, **Dragon (as Type_2)**, **Electric**, and **Fire** showed **positive coefficients**, aligning with intuition about “special” attackers.

Interpretation:

- After controlling for overall stat totals and other base stats, **Fighting, Normal, and Ground Pokémon are systematically underpowered on Special Attack**.
- From a design and product perspective, these types represent **opportunities** for new high–Special Attack Pokémon that would feel surprising and fresh within those archetypes.

---

## Recommendation

For future game and card design, we recommend:

- Introducing a **new generation of rare Fighting, Normal, and Ground-type Pokémon** with **high Special Attack**, while keeping their defensive and speed profiles consistent with type fantasy.
- Positioning these Pokémon as:
  - “Unexpected special sweepers” within historically physical or utility-focused types,
  - Potential anchors for competitive teams and collectible card lines.

This could:

- Increase **player interest** in underpowered types,
- Diversify type usage in games and TCG,
- Drive incremental engagement and revenue tied to these new designs.

---

## Repository Structure (example)


- `data/pokemon.csv` – Pokémon index with stats and types  
- `R/` – code   
- `notebooks/` – html
- `README.md` – this file

---

## How to Run

1. **Clone the repo** and open it in RStudio.

2. **Install packages** (if needed):

   ```r
   install.packages(c("tidyverse", "fastDummies", "glmnet", "ggplot2", "broom"))
