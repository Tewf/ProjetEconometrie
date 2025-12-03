# DVF Grenoble — Régression linéaire (OLS)

Modélisation hédonique des prix immobiliers à Grenoble via régression linéaire (statsmodels/sklearn).  
Objectifs: charger les données prétraitées, créer des variantes (log, standardisation), préparer jeux de données, ajuster OLS, diagnostiquer (hétéroscédasticité), visualiser observé vs prédit et importance des variables.

---

## Plan du notebook

1. Importations et configuration  
2. Installation des dépendances  
3. Chargement des données  
4. Préparation des sous-ensembles  
5. Analyse de corrélation  
6. Transformations logarithmiques  
7. Jeux de données dérivés  
8. Modélisation OLS (formule) et métriques  
9. Visualisation: observé vs prédit et importance des variables  
10. Conclusion et perspectives

---

## 1. Importations et configuration

Extrait de code (imports) :

```python
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from pathlib import Path
from sklearn.preprocessing import StandardScaler
```

---

## 3. Chargement des données

Lecture du CSV prétraité (`df_grenoble_vente.csv`). Le notebook cherche le fichier dans plusieurs emplacements relatifs.

Extrait de code important :

```python
from pathlib import Path
candidates = [
    Path('../DataPreprocessing/PreprocessedData/df_grenoble_vente.csv'),
    Path('DataPreprocessing/PreprocessedData/df_grenoble_vente.csv'),
    Path('projet_dvf/DataPreprocessing/PreprocessedData/df_grenoble_vente.csv'),
]
csv_path = next((p for p in candidates if (Path.cwd()/p).exists()), None)
if csv_path is None:
    tried = [str((Path.cwd()/p).resolve()) for p in candidates]
    raise FileNotFoundError(f'Missing file. Tried: {tried}')
df = pd.read_csv(csv_path)
print(f'Loaded {len(df)} rows and {len(df.columns)} columns from {csv_path}')
df.head()
```

Exécution (stdout):

```
Loaded 247 rows and 6 columns from ../DataPreprocessing/PreprocessedData/df_grenoble_vente.csv
```

Aperçu (extrait du DataFrame) :

|   | price     | type_local                                      | surface_bati | surface_terrain | nb_pieces | type_local_1234 |
|---:|----------:|:------------------------------------------------|-------------:|----------------:|----------:|----------------:|
| 0 | 112560.0 | Appartement                                     | 45.0        | 1.0             | 2         | 1               |
| 1 | 65000.0  | Appartement                                     | 29.0        | 1.0             | 1         | 1               |
| 2 | 9000.0   | Local industriel. commercial ou assimilé        | 12.0        | 1.0             | 0         | 3               |
| 3 | 133000.0 | Appartement                                     | 49.0        | 1.0             | 2         | 1               |
| 4 | 53000.0  | Appartement                                     | 14.0        | 1.0             | 1         | 1               |

---

## 4. Préparation des sous-ensembles

Création d’un sous-ensemble `Appartement` et suppression des colonnes non numériques pour la modélisation.

Extrait de code :

```python
df_copy = df.copy()

# Robust appartement subset (works even if 'type_local' is missing)
if 'type_local' in df_copy.columns:
    m_app = df_copy['type_local'].astype(str).str.contains('Appartement', case=False, na=False)
    df_appartement = df_copy.loc[m_app].copy()
else:
    df_appartement = df_copy.copy()  # fallback: use all rows

# Drop non-numeric identifiers if present
df_copy = df_copy.drop(columns=['type_local_1234'], errors='ignore')
df_appartement = df_appartement.drop(columns=['type_local'], errors='ignore')
```

---

## 5. Analyse de corrélation

Matrice de corrélations des variables numériques (sous-ensemble appartements) et heatmap.

Extrait de code :

```python
corr = df_appartement.corr(numeric_only=True)
sns.heatmap(corr, annot=True, cmap="YlGnBu")
plt.title("Correlation Heatmap of Grenoble_Vente Dataset")
plt.show()
```

(Image de la heatmap incluse dans le notebook — voir la section Visualisation.)

---

## 6. Transformations logarithmiques

Construction des variables en log pour un modèle hédonique multiplicatif (log-log).

Fonction pour calculer les variables log (extrait) :

```python
# Keep strictly positive values for log transform
def compute_log_vars(df_in):
    mask = (df_in['price'] > 0) & (df_in['surface_bati'] > 0) & (df_in['surface_terrain'] >= 0)
    df_out = df_in.loc[mask].copy()
    df_out['log_price'] = np.log(df_out['price'])
    df_out['log_surface_bati'] = np.log(df_out['surface_bati'])
    df_out['log_surface_terrain'] = np.log(df_out['surface_terrain'].replace(0, 1))  # example handling
    # Also keep a duplicate name for any legacy cells using 'log_price'
    drop_cols = ['price', 'surface_bati', 'surface_terrain']
    df_out = df_out.drop(columns=drop_cols, errors='ignore')
    return df_out
```

Aperçu de `df_appartement_log.head()` :

|   | nb_pieces | log_price  | log_surface_bati | log_surface_terrain |
|---:|----------:|-----------:|-----------------:|--------------------:|
| 0 | 2         | 11.631242  | 3.806662         | 0.0                 |
| 1 | 1         | 11.082143  | 3.367296         | 0.0                 |
| 3 | 2         | 11.798104  | 3.891820         | 0.0                 |
| 4 | 1         | 10.878047  | 2.639057         | 0.0                 |
| 5 | 4         | 11.225243  | 4.110874         | 0.0                 |

---

## 8. Jeux de données dérivés

Création des versions `raw/log/scaled` et assemblage dans un dictionnaire pour itérer les modèles.

```python
datasets = {
    "raw_appartement": df_appartement,
    "raw_appartement_log": df_appartement_log,
    "raw_all": df_copy,
    "raw_all_log": df_copy_log
}
```

---

## 9. Modélisation OLS (formule) et métriques

Ajustement de modèles OLS pour chaque dataset et calcul des métriques en échantillon (R², RMSE). Le notebook construit des formules automatiquement (en rendant les noms de colonnes "safe" pour la formule), puis exécute `smf.ols(...).fit()` pour chaque dataset.

Extrait de code principal (résumé) :

```python
import numpy as np
import statsmodels.formula.api as smf
from sklearn.metrics import r2_score, mean_squared_error

models = {}
metrics = {}

for name, df_in in datasets.items():
    df = df_in.copy()
    # Make all column names formula-safe
    df.columns = df.columns.str.replace('[^0-9a-zA-Z_]+', '_', regex=True)
    target = 'log_price' if 'log_price' in df.columns else 'price'
    if target not in df.columns:
        print(f"Skipping {name} (no target)")
        continue
    predictors = [c for c in df.columns if c != target]
    formula = f"{target} ~ " + " + ".join(predictors)
    model = smf.ols(formula=formula, data=df).fit()
    models[name] = model

    # In-sample metrics
    y_true = df[target]
    y_pred = model.predict(df)
    r2 = r2_score(y_true, y_pred)
    rmse = float(np.sqrt(mean_squared_error(y_true, y_pred)))
    metrics[name] = {
        "formula": formula,
        "target": target,
        "r2": float(r2),
        "rmse": rmse,
        "n_obs": int(len(df)),
        "predictors": predictors
    }

    print(f"\n=== {name} ===")
    print(formula)
    print(model.summary().as_text())
    print(f"R2={r2:.3f} RMSE={rmse})")
```

Sorties textuelles (résumés OLS clairement imprimés) — extraits complets ci-dessous.

---

### === raw_appartement ===
Formula:
```
price ~ surface_bati + surface_terrain + nb_pieces
```

OLS Regression Results (extrait texte complet):

```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                  price   R-squared:                       0.096
Model:                            OLS   Adj. R-squared:                  0.082
Method:                 Least Squares   F-statistic:                     6.754
Date:                Wed, 26 Nov 2025   Prob (F-statistic):           0.000237
Time:                        16:47:55   Log-Likelihood:                -2705.0
No. Observations:                 194   AIC:                             5418.
Df Residuals:                     190   BIC:                             5431.
Df Model:                           3                                         
Covariance Type:            nonrobust                                         
===================================================================================
                      coef    std err          t      P>|t|      [0.025      0.975]
-----------------------------------------------------------------------------------
Intercept        1.318e+05   4.96e+04      2.657      0.009     3.4e+04     2.3e+05
surface_bati      404.0749   1551.004      0.261      0.795   -2655.324    3463.474
surface_terrain   622.9559    144.466      4.312      0.000     337.993     907.919
nb_pieces        2.652e+04   3.68e+04      0.721      0.472   -4.61e+04    9.91e+04
==============================================================================
Omnibus:                      161.590   Durbin-Watson:                   0.326
Prob(Omnibus):                  0.000   Jarque-Bera (JB):             1321.464
Skew:                           3.449   Prob(JB):                    1.12e-287
Kurtosis:                      13.766   Cond. No.                         369.
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
R2=0.096 RMSE=275006.941)
```

---

### === raw_appartement_log ===
Formula:
```
log_price ~ nb_pieces + log_surface_bati + log_surface_terrain
```

OLS Regression Results (extrait texte complet):

```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:              log_price   R-squared:                       0.275
Model:                            OLS   Adj. R-squared:                  0.264
Method:                 Least Squares   F-statistic:                     24.06
Date:                Wed, 26 Nov 2025   Prob (F-statistic):           3.03e-13
Time:                        16:47:55   Log-Likelihood:                -195.99
No. Observations:                 194   AIC:                             400.0
Df Residuals:                     190   BIC:                             413.1
Df Model:                           3                                         
Covariance Type:            nonrobust                                         
=======================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
---------------------------------------------------------------------------------------
Intercept              10.0119      0.526     19.020      0.000       8.974      11.050
nb_pieces               0.0712      0.077      0.928      0.355      -0.080       0.223
log_surface_bati        0.4408      0.170      2.590      0.010       0.105       0.777
log_surface_terrain     0.2642      0.038      6.867      0.000       0.188       0.340
==============================================================================
Omnibus:                       80.486   Durbin-Watson:                   0.833
Prob(Omnibus):                  0.000   Jarque-Bera (JB):              236.811
Skew:                           1.775   Prob(JB):                     3.78e-52
Kurtosis:                       7.085   Cond. No.                         55.7
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
R2=0.275 RMSE=0.665)
```

---

### === raw_all ===
Formula:
```
price ~ type_local + surface_bati + surface_terrain + nb_pieces
```

OLS Regression Results (extrait texte complet):

```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                  price   R-squared:                       0.157
Model:                            OLS   Adj. R-squared:                  0.136
Method:                 Least Squares   F-statistic:                     7.449
Date:                Wed, 26 Nov 2025   Prob (F-statistic):           2.50e-07
Time:                        16:47:55   Log-Likelihood:                -3435.1
No. Observations:                 247   AIC:                             6884.
Df Residuals:                     240   BIC:                             6909.
Df Model:                           6                                         
Covariance Type:            nonrobust                                         
==========================================================================================================================
                                                             coef    std err          t      P>|t|      [0.025      0.975]
--------------------------------------------------------------------------------------------------------------------------
Intercept                                               1.387e+05    4.6e+04      3.017      0.003    4.81e+04    2.29e+05
type_local[T.Dépendance]                               -2.543e+04   1.45e+05     -0.175      0.861   -3.12e+05    2.61e+05
type_local[T.Local industriel. commercial ou assimilé]   2.49e+05   6.72e+04      3.703      0.000    1.17e+05    3.81e+05
type_local[T.Maison]                                   -1.271e+05    1.2e+05     -1.056      0.292   -3.64e+05     1.1e+05
surface_bati                                            -320.9344    157.225     -2.041      0.042    -630.652     -11.216
surface_terrain                                          606.4685    113.190      5.358      0.000     383.495     829.442
nb_pieces                                               4.073e+04   1.71e+04      2.377      0.018    6974.177    7.45e+04
==============================================================================
Omnibus:                      178.289   Durbin-Watson:                   0.390
Prob(Omnibus):                  0.000   Jarque-Bera (JB):             1314.224
Skew:                           3.029   Prob(JB):                    4.17e-286
Kurtosis:                      12.539   Cond. No.                     1.51e+03
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 1.51e+03. This might indicate that there are
strong multicollinearity or other numerical problems.
R2=0.157 RMSE=265213.662)
```

---

### === raw_all_log ===
Formula:
```
log_price ~ nb_pieces + log_surface_bati + log_surface_terrain
```

OLS Regression Results (extrait texte complet):

```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:              log_price   R-squared:                       0.167
Model:                            OLS   Adj. R-squared:                  0.157
Method:                 Least Squares   F-statistic:                     15.98
Date:                Wed, 26 Nov 2025   Prob (F-statistic):           1.67e-09
Time:                        16:47:55   Log-Likelihood:                -283.11
No. Observations:                 243   AIC:                             574.2
Df Residuals:                     239   BIC:                             588.2
Df Model:                           3                                         
Covariance Type:            nonrobust                                         
=======================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
---------------------------------------------------------------------------------------
Intercept              11.6735      0.246     47.502      0.000      11.189      12.158
nb_pieces               0.0568      0.037      1.518      0.130      -0.017       0.131
log_surface_bati        0.0470      0.064      0.734      0.464      -0.079       0.173
log_surface_terrain     0.1998      0.030      6.693      0.000       0.141       0.259
==============================================================================
Omnibus:                       34.285   Durbin-Watson:                   0.885
Prob(Omnibus):                  0.000   Jarque-Bera (JB):               54.541
Skew:                           0.811   Prob(JB):                     1.43e-12
Kurtosis:                       4.660   Cond. No.                         23.8
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
R2=0.167 RMSE=0.776)
```

---

## 10. Visualisation: observé vs prédit et importance des variables

Le notebook contient une fonction `plot_preds_and_importance(models, metrics, top_k=12, importance="tstat")` qui :
- Récupère cibles/prédictions à l'échelle "price" (convertit `log_price` via exp si nécessaire)
- Affiche Observé vs Prédit (échelle prix) avec diagonale et RMSE en titre
- Affiche l'importance des variables (par |t-stat| ou par coefficients standardisés = betas standardisés)
- Calcule betas standardisés via la fonction `_std_betas(model)`

Extraits de code importants :

```python
def _price_scale(y, target_name):
    """Retourne y en unités 'price' (passe de log_price à price si besoin)."""
    if 'log_price' in target_name.lower():
        return np.exp(y)  # suppose un log naturel
    return y

def _std_betas(model):
    """Coefs standardisés (beta_j = coef_j * sd(X_j) / sd(y)). Intercept exclu."""
    y = model.model.endog
    X = model.model.exog
    names = model.model.exog_names
    params = model.params

    keep = [i for i,n in enumerate(names) if n.lower() not in ['intercept', 'const']]
    Xk = X[:, keep]
    names_k = [names[i] for i in keep]
    params_k = np.array([params[i] for i in keep])

    sd_y = np.std(y, ddof=0)
    sd_X = np.std(Xk, axis=0, ddof=0)
    sd_X = np.where(sd_X==0, 1.0, sd_X)
    betas = params_k * sd_X / (sd_y + 1e-9)
    return names_k, betas
```

La fonction `plot_preds_and_importance` produit plusieurs figures (observé vs prédit, barplot importance). Le notebook affiche plusieurs figures pour les différents jeux (`raw_appartement`, `raw_appartement_log`, ...).

Images générées (exemples) :
- Heatmap de corrélation
- Observé vs Prédit (plusieurs figures)
- Graphes d'importance (barh)
- Résidus diagnostics (residuals vs fitted, scale-location)

Les images présentes dans le HTML original pointent vers :
- https://raw.githubusercontent.com/Tewf/ProjetEconometrie/0b93e13c57cc352b84e6e23aa8c6b58a5bfd6656/ModelTraining/LinearRegression_files/figure-html/cell-5-output-1.png
- https://raw.githubusercontent.com/Tewf/ProjetEconometrie/0b93e13c57cc352b84e6e23aa8c6b58a5bfd6656/ModelTraining/LinearRegression_files/figure-html/cell-11-output-2.png
- (et d'autres images analogues pour les figures des cellules)

---

## 11. Conclusion

- Le notebook met en place une chaîne OLS complète : chargement et préparation des données, création de variantes (log), ajustement de modèles sur plusieurs jeux de données et diagnostics graphiques/textuels.
- Des tests d’hétéroscédasticité (Breusch–Pagan, White) et des diagnostics visuels sont effectués. Dans l'exécution fournie :
  - Pour `raw_all_log`, les tests détectent une hétéroscédasticité (p < 0.05), et le résumé est affiché avec erreurs-standards robustes HC3.
  - Pour les autres jeux, les tests n'étaient pas significatifs.
- Le modèle pour `appartement` recommandé dans le notebook suit la pratique : `log_price ~ nb_pieces + log_surface_bati + log_surface_terrain`.

Extrait du diagnostic d'hétéroscédasticité (stdout important) :

```
--- Heteroscedasticity diagnostics for: raw_appartement ---
Breusch-Pagan: LM stat = 2.163, p-value = 0.5393, F p-value = 0.5448
White test: LM stat = 6.859, p-value = 0.6518, F p-value = 0.6633
=> Tests non significatifs : pas d'évidence forte d'hétéroscédasticité (alpha=0.05).

--- Heteroscedasticity diagnostics for: raw_appartement_log ---
Breusch-Pagan: LM stat = 1.206, p-value = 0.7515, F p-value = 0.7558
White test: LM stat = 5.277, p-value = 0.8095, F p-value = 0.8193
=> Tests non significatifs : pas d'évidence forte d'hétéroscédasticité (alpha=0.05).

--- Heteroscedasticity diagnostics for: raw_all ---
Breusch-Pagan: LM stat = 2.317, p-value = 0.8884, F p-value = 0.8922
White test: LM stat = 10.037, p-value = 0.9307, F p-value = 0.9386
=> Tests non significatifs : pas d'évidence forte d'hétéroscédasticité (alpha=0.05).

--- Heteroscedasticity diagnostics for: raw_all_log ---
Breusch-Pagan: LM stat = 30.046, p-value = 0.0000, F p-value = 0.0000
White test: LM stat = 38.925, p-value = 0.0000, F p-value = 0.0000
=> Hétéroscédasticité détectée (p < 0.05). Résumé avec erreurs robustes HC3 :
[affichage du résumé OLS avec cov_type='HC3' pour raw_all_log]
```

---

Notes et suggestions rapides
- Si vous souhaitez que je génère un Markdown plus "narratif" (avec interprétations des coefficients, suggestions de transformations supplémentaires, ou un résumé court en français), dites-moi la longueur et le focus (p.ex. résumé technique, résumé pour non-spécialistes).
- Je peux aussi extraire et enregistrer ce Markdown dans le dépôt sous `ModelTraining/LinearRegression.md` (PR) si vous voulez que j'ajoute une version Markdown rendue dans le repo.
