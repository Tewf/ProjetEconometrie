```markdown
# DVF Grenoble — Forêt aléatoire (RandomForest)

Modélisation prédictive des prix immobiliers à Grenoble avec RandomForestRegressor.

## 1. Importations et configuration

```python
# Standard imports
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from pathlib import Path

# Scikit-learn
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import r2_score, mean_squared_error
```

## 2. Chargement des données

Lecture du CSV prétraité (df_grenoble_vente_panel.csv). Le code recherche plusieurs chemins possibles et choisit le premier existant.

```python
candidates = [
    Path('../../DataPreprocessing/PreprocessedData/df_grenoble_vente_panel.csv'),
    Path('../DataPreprocessing/PreprocessedData/df_grenoble_vente_panel.csv'),
    Path('DataPreprocessing/PreprocessedData/df_grenoble_vente_panel.csv'),
]

csv_path = next((p for p in candidates if (Path.cwd() / p).exists()), None)

if csv_path is None:
    tried = [str((Path.cwd() / p).resolve()) for p in candidates]
    raise FileNotFoundError(f"Missing file. Tried: {tried}")

df = pd.read_csv(csv_path)
print(f"Loaded {len(df):,} rows and {len(df.columns)} columns from {csv_path}")
display(df.head())
```

Exemple (d'après la sortie lors de l'exécution) :
- Loaded 1,288 rows and 7 columns from ../../DataPreprocessing/PreprocessedData/df_grenoble_vente_panel.csv

Aperçu des colonnes présentes : `price`, `type_local`, `surface_bati`, `surface_terrain`, `date`, `nb_pieces`, `type_local_1234`.

## 3. Préparation des sous-ensembles

Création d’un sous-ensemble `Appartement` et suppression des colonnes non numériques avant modélisation.

```python
df_copy = df.copy()

# Robust appartement subset (works even if 'type_local' is missing)
if 'type_local' in df_copy.columns:
    m_app = df_copy['type_local'].astype(str).str.contains('Appartement', case=False, na=False)
    df_appartement = df_copy.loc[m_app].copy()
else:
    df_appartement = df_copy.copy()  # fallback: use all rows

# Drop non-numeric identifiers if present
drop_cols = ['type_local', 'date', 'type_local_1234']
df_copy = df_copy.drop(columns=[c for c in drop_cols if c in df_copy.columns], errors='ignore')
df_appartement = df_appartement.drop(columns=[c for c in drop_cols if c in df_appartement.columns], errors='ignore')
```

## 4. Aides de visualisation

Fonctions utilitaires pour afficher Observé vs Prédit et les importances des variables (en échelle prix si le modèle prédit log_price).

```python
def _to_price_scale(y, target_col):
    # If the model predicts log_price we convert back to price for plotting
    if isinstance(target_col, str) and target_col == 'log_price':
        return np.exp(y)
    return y

def plot_rf_results(name, y_true, y_pred, feature_names, importances, target_col, top_k=7):
    # convert to price scale for visualization
    y_true_p = _to_price_scale(y_true, target_col)
    y_pred_p = _to_price_scale(y_pred, target_col)
    rmse_price = float(np.sqrt(np.mean((y_true_p - y_pred_p) ** 2)))

    # Top-k features
    imp = np.array(importances)
    if len(imp) != len(feature_names):
        feat = feature_names
        vals = np.zeros(len(feat))
    else:
        idx = np.argsort(imp)[-top_k:]
        feat = [feature_names[i] for i in idx][::-1]
        vals = imp[idx][::-1]

    fig, axes = plt.subplots(1, 2, figsize=(12, 5))

    # Scatter Actual vs Predicted (price scale)
    ax = axes[0]
    ax.scatter(y_true_p, y_pred_p, s=8, alpha=0.6)
    lo = float(np.min([y_true_p.min(), y_pred_p.min()]))
    hi = float(np.max([y_true_p.max(), y_pred_p.max()]))
    ax.plot([lo, hi], [lo, hi], 'r--', linewidth=1)
    ax.set_xlabel('Prix observé')
    ax.set_ylabel('Prix prédit')
    ax.set_title(f'{name} — RMSE(price): {rmse_price:,.0f}')
    ax.grid(True, linewidth=0.3, alpha=0.6)

    # Feature importances
    ax2 = axes[1]
    ax2.barh(feat, vals)
    ax2.set_xlabel('Feature importance (Gini)')
    ax2.set_title(f'Top {len(feat)} features')
    ax2.grid(axis='x', linewidth=0.3, alpha=0.6)

    plt.tight_layout()
    plt.show()
```

## 5. Création des variantes et jeux de données

Assemblage des versions raw / (optionnellement) log / scaled pour comparaison empirique.

```python
datasets = {
    'raw_appartement': df_appartement,
    'raw_all': df_copy,
}

# Optionally create log_price and scaled versions (uncomment/use if desired)
for name, dfi in list(datasets.items()):
    dfi = dfi.copy()
    if 'price' in dfi.columns and 'log_price' not in dfi.columns:
        dfi['log_price'] = np.log(dfi['price'].replace({0: np.nan}))
    datasets[name] = dfi
```

Note: scaling souvent n'aide pas beaucoup les Random Forests, mais on peut garder des variantes pour comparaison empirique.

## 6. Définition du modèle RandomForest et métriques

Fonction d’entraînement avec split train/test et calcul des métriques (R², RMSE, RMSE en échelle prix si le target est log_price).

```python
def fit_random_forest(df, target_col=None, test_size=0.2, random_state=42,
                      n_estimators=500, max_depth=None, verbose=False):
    """Fits a RandomForestRegressor on df and returns (model, metrics)."""
    df = df.copy()

    # Auto-detect target
    if target_col is None:
        if 'log_price' in df.columns:
            target_col = 'log_price'
        elif 'price' in df.columns:
            target_col = 'price'
        else:
            raise ValueError("No 'price' or 'log_price' column found in dataframe")

    # Drop rows with missing target
    df = df.dropna(subset=[target_col])

    # Features: numeric only
    X = df.drop(columns=[target_col]).select_dtypes(include=[np.number])
    if X.empty:
        raise ValueError('No numeric predictors found in the DataFrame')

    y = df[target_col]

    # Train/test split
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=test_size, random_state=random_state
    )

    # Model
    model = RandomForestRegressor(
        n_estimators=n_estimators,
        max_depth=max_depth,
        random_state=random_state,
        n_jobs=-1
    )
    model.fit(X_train, y_train)

    # Evaluate (target scale)
    y_pred = model.predict(X_test)
    r2 = float(r2_score(y_test, y_pred))
    rmse = float(np.sqrt(mean_squared_error(y_test, y_pred)))

    # RMSE on price scale for log targets
    if target_col == 'log_price':
        y_true_p = np.exp(y_test)
        y_pred_p = np.exp(y_pred)
        rmse_price = float(np.sqrt(mean_squared_error(y_true_p, y_pred_p)))
    else:
        rmse_price = rmse

    metrics = {
        'target': target_col,
        'r2': r2,
        'rmse': rmse,
        'rmse_price': rmse_price,
        'n_train': int(len(X_train)),
        'n_test': int(len(X_test)),
        'features': list(X.columns),
        'y_test': y_test,
        'y_pred': y_pred,
    }

    if verbose:
        print(f'Random Forest — target: {target_col}')
        print(f'R²: {r2:.4f}')
        print(f'RMSE: {rmse:.4f}  |  RMSE(price): {int(round(rmse_price)):,}')

    return model, metrics
```

## 7. Entraînement multi-datasets et tableau récapitulatif

Ajustement d’un modèle par dataset et construction d’un tableau comparatif des métriques.

```python
models = {}
metrics = {}

for name, dfi in datasets.items():
    print(f'\n--- Fitting model for: {name} ---')
    model, metr = fit_random_forest(dfi, verbose=True)
    models[name] = model
    metrics[name] = metr

# Summary table
rows = []
for name, m in metrics.items():
    rows.append({
        'dataset': name,
        'target': m['target'],
        'r2': m['r2'],
        'rmse': m['rmse'],
        'rmse_price': m['rmse_price'],
        'n_test': m['n_test'],
        'n_train': m['n_train'],
    })

summary_df = pd.DataFrame(rows).sort_values(['target', 'rmse_price']).reset_index(drop=True)
display(summary_df)
```

Extrait des sorties lors d'une exécution antérieure :

```
--- Fitting model for: raw_appartement ---
Random Forest — target: price
R²: 0.8261
RMSE: 72061.3868  |  RMSE(price): 72,061

--- Fitting model for: raw_all ---
Random Forest — target: price
R²: 0.7250
RMSE: 76770.4234  |  RMSE(price): 76,770
```

Résumé (tableau) — résultat d'exécution montré dans le notebook :

| dataset         | target | r2      | rmse         | rmse_price   | n_test | n_train |
|-----------------|--------|---------|--------------|--------------|--------|---------|
| raw_appartement | price  | 0.82614 | 72061.386784 | 72061.386784 | 213    | 849     |
| raw_all         | price  | 0.72503 | 76770.423442 | 76770.423442 | 258    | 1030    |

## 8. Visualisations: observé vs prédit et importances

Tracés des performances en échelle prix et des importances Gini.

```python
for name, model in models.items():
    m = metrics[name]
    y_test = m['y_test']
    y_pred = m['y_pred']
    feat_names = m['features']
    importances = getattr(model, 'feature_importances_', np.zeros(len(feat_names)))
    plot_rf_results(name, y_test, y_pred, feat_names, importances, m['target'], top_k=7)
```

(Le notebook affichait deux figures par modèle : scatter Observé vs Prédit et barre horizontale des importances.)

Exemples d'images générées dans le rendu HTML original :
- RandomForest_files/figure-html/cell-9-output-1.png
- RandomForest_files/figure-html/cell-9-output-2.png

## 9. Conclusion

Le notebook met en place une chaîne RandomForest : préparation des données, variantes log/scaled (optionnelles), entraînement et diagnostics (graphiques et tableau synthétique). Les résultats préliminaires montrent un R² autour de 0.83 pour le sous-ensemble appartements et ~0.73 sur l'ensemble brut, avec des RMSE autour de 72k–77k euros en sortie.

```