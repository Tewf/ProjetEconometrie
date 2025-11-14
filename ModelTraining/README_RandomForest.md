# ModelTraining (RandomForest.ipynb) — Forêt Aléatoire

## Résumé

Ce notebook implémente la **modélisation prédictive des prix immobiliers** par Random Forest (Forêt Aléatoire), un algorithme d'apprentissage automatique non-paramétrique. Il complète l'approche économétrique classique (régression linéaire) en capturant automatiquement les non-linéarités et interactions entre variables.

### Le but dans le Projet

Le Random Forest représente l'**approche Machine Learning** de la modélisation des prix. Ce notebook :

1. **Charge** les mêmes données que LinearRegression.ipynb
2. **Prépare** les variantes du dataset (original, log, scaled)
3. **Entraîne** des modèles RandomForestRegressor (scikit-learn)
4. **Évalue** les performances (RMSE, R², MAE)
5. **Visualise** les prédictions et l'importance des variables
6. **Compare** avec les résultats OLS

### Pourquoi C'est Utile ?

- **Performance prédictive** : Généralement RMSE inférieur à OLS
- **Non-paramétrique** : Pas d'hypothèse sur la forme fonctionnelle
- **Interactions** : Capture automatiquement les interactions entre variables
- **Robustesse** : Résistant aux outliers et multicolinéarité
- **Importance des variables** : Ranking automatique des features

### Comment Il Participe à la Démarche Économétrique ?

```
[Phase 1] DataPreparation.ipynb → df_grenoble_vente.csv
                                           ↓
                        ┌──────────────────┴───────────────────┐
                        ▼                                      ▼
[Phase 2a]      LinearRegression.ipynb            RandomForest.ipynb    [Phase 2b]
                (Paramétrique, OLS)                (Non-paramétrique, ML)
                        │                                      │
                        └──────────────────┬───────────────────┘
                                           ▼
                                  Comparaison des modèles
                                  Trade-off : Interprétabilité vs Performance
```

##  Contenu du Notebook

### Structure (10 Sections) :

1. **Importations et Configuration** - Bibliothèques Python
2. **Chargement des Données** - df_grenoble_vente.csv
3. **Préparation des Sous-Ensembles** - Séparation X/y
4. **Fonctions Utilitaires** - Logs et standardisation
5. **Aides de Visualisation** - Fonctions plot
6. **Création des Variantes** - Datasets log/scaled
7. **Définition du Modèle** - RandomForestRegressor
8. **Entraînement Multi-Datasets** - Boucle sur variantes
9. **Visualisations** - Observé vs prédit, importances
10. **Conclusion** - Synthèse

##  Random Forest : Principes

### Algorithme

Le Random Forest est un **ensemble de arbres de décision** (CART) construits par :

1. **Bootstrap Aggregating (Bagging)** :
   - Créer B échantillons bootstrap (avec remplacement) du dataset → On crée plusieurs copies du dataset, en tirant les observations au hasard avec remplacement.
(Donc certaines observations sont répétées, d’autres absentes.)
   - Entraîner un arbre sur chaque échantillon

2. **Sélection Aléatoire de Features** :
  → À chaque division dans un arbre, le modèle ne regarde qu’un petit sous-ensemble de variables.
Cela fait que chaque arbre “pense différemment”, ce qui rend la forêt plus stable.

3. **Prédiction par Vote/Moyenne** :
   → Chaque arbre donne une prédiction du prix.
Le modèle final prend la moyenne de toutes les prédictions.
(Comme demander l'avis à 100 personnes et faire la moyenne.)

### Formule

```
ŷ = (1/B) × Σ(b=1 to B) T_b(x)

où :
  B = nombre d'arbres (ex: 100)
  T_b = prédiction de l'arbre b
  x = vecteur de features
```
#### Explication :
  - On construit B arbres (par exemple 100).
  - Chaque arbre fait une prédiction T_b(x)
  - Le modèle final = la moyenne de toutes les prédictions.

C’est exactement comme prendre 100 estimations de prix et calculer la moyenne pour avoir la valeur finale.

### Hyperparamètres Clés

| Paramètre | Description | Valeur typique | Impact |
|-----------|-------------|----------------|--------|
| `n_estimators` | Nombre d'arbres B | 100 | Plus = meilleur (rendements décroissants) |
| `max_depth` | Profondeur maximale | None | None = croissance complète |
| `min_samples_split` | Min observations pour split | 2 | Plus = arbres plus simples |
| `min_samples_leaf` | Min observations par feuille | 1 | Plus = lissage |
| `max_features` | Features à considérer par split | 'sqrt' | Régression: 'sqrt' ou 'log2' |
| `random_state` | Seed aléatoire | 42 | Reproductibilité |


## Schéma Explicatif : Construction d'un Random Forest

```
┌────────────────────────────────────────────────────────────────┐
│          CONSTRUCTION D'UN RANDOM FOREST (B=3 arbres)          │
└────────────────────────────────────────────────────────────────┘

Dataset Original (n=1288 observations)
[Obs1, Obs2, Obs3, ..., Obs1288]
         │
         │ Bootstrap Sampling (avec remplacement)
         │
    ┌────┴────┬──────────────┐
    ▼         ▼              ▼        
Bootstrap1  Bootstrap2  Bootstrap3
(n=1288)    (n=1288)    (n=1288)
    │         │          │
    │ Certaines obs répétées, d'autres absentes (~37% OOB)
    │
    ├─────────┼──────────┤
    ▼         ▼          ▼
┌───────┐ ┌───────┐  ┌───────┐
│ Arbre1│ │ Arbre2│  │ Arbre3│
│       │ │       │  │       │
│   •   │ │   •   │  │   •   │ ← Nœud racine
│  / \  │ │  / \  │  │  / \  │
│ •   • │ │ •   • │  │ •   • │ ← Nœuds internes
│/ \ / \│ │/ \ / \│  │/ \ / \│   (splits sur features aléatoires)
│• • • •│ │• • • •│  │• • • •│ ← Feuilles (prédictions)
└───────┘ └───────┘  └───────┘

    │         │          │
    │ Prédiction pour une nouvelle observation x_new
    │
    ▼         ▼          ▼
  250k€     280k€      265k€
    │         │          │
    └────┬────┴────┬─────┘
         │         │
         ▼         ▼
    Moyenne : (250+280+265)/3 = 265k€
              ────────────────────
                Prédiction finale
```



##  Workflow (Entrée → Traitement → Sortie)

```
INPUT
df_grenoble_vente.csv (1288 obs × 7 vars)
         │
         │ Séparation X/y
         ▼
X: [surface_bati, nb_pieces, date, surface_terrain, type_local_1234]
y: [price]
         │
         ├──────────────┬──────────────┬──────────────┐
         ▼              ▼              ▼              ▼
   Dataset ORIG   Dataset LOG    Dataset SCALED  Dataset LOG+SCALED
         │              │              │              │
         │ train_test_split(test_size=0.2, random_state=42)
         ▼
   X_train (80%)   X_test (20%)
   y_train         y_test
         │              │
         │ RandomForestRegressor(n_estimators=100, random_state=42)
         │
         ▼
   model.fit(X_train, y_train)
         │
         │ Entraînement : Construction de 100 arbres CART
         │                 • Bootstrap sampling
         │                 • Feature randomization
         │                 • Croissance jusqu'à pureté
         ▼
   Modèle entraîné
         │
         │ model.predict(X_test)
         ▼
   y_pred (prédictions sur test set)
         │
         │ Évaluation
         ▼
┌────────────────────────────────┐
│ Métriques                      │
│ ────────                       │
│ • R² = 1 - RSS/TSS             │
│ • RMSE = √(MSE)                │
│ • MAE = mean(|y - ŷ|)          │
│                                │
│ • Feature importances          │
│   (Gini importance)            │
└────────────────────────────────┘
         │
         ▼
OUTPUT
Tableau comparatif + Graphiques
```

##  Importance des Variables (Gini Importance)

### Calcul

Pour chaque feature j :
```
Importance(j) = Σ(tous les arbres t) Σ(tous les nœuds n utilisant j) 
                  [Réduction de l'impureté au nœud n]
                ─────────────────────────────────────────────
                Nombre total d'arbres
```

**Impureté** : MSE (Mean Squared Error) pour la régression
```
MSE(nœud) = (1/N) × Σ(i dans nœud) (y_i - ȳ_nœud)²
```

**Réduction d'impureté** lors d'un split :
```
ΔImpureté = MSE(parent) - [p_left × MSE(left) + p_right × MSE(right)]

où p_left = proportion d'échantillons dans le fils gauche
```

### Interprétation

- **Valeur** : Entre 0 et 1, somme = 1
- **Plus élevée = plus importante** pour la prédiction
- **Attention** : 
  - Biais vers variables continues (plus de splits possibles)
  - Pas d'inférence statistique (pas de p-value)
  - Importance relative, pas absolue

### Exemple

```
Feature Importances :
  surface_bati     0.52  ████████████████████
  type_local_1234  0.21  ████████
  nb_pieces        0.15  ██████
  surface_terrain  0.08  ███
  date             0.04  █
```

Interprétation :
- `surface_bati` explique 52% de la réduction totale d'impureté
- C'est la variable la plus prédictive
- `date` est peu informative (4%)

##  Résultats Attendus

### Performance Typique

| Métrique | OLS Linéaire | OLS Log-Log | Random Forest |
|----------|--------------|-------------|---------------|
| **R²** | 0.70-0.75 | 0.75-0.78 | 0.78-0.85 |
| **RMSE** | 55,000-60,000€ | 50,000-55,000€ | 45,000-52,000€ |
| **MAE** | 40,000-45,000€ | 36,000-40,000€ | 32,000-38,000€ |

### ✅ Avantages Random Forest

 **Meilleure précision** : RMSE typiquement 5-15% inférieur à OLS  
 **Capture non-linéarités** : Effet de seuil, rendements décroissants  
 **Interactions automatiques** : surface × type_local sans spécification manuelle  
 **Robustesse** : Peu sensible aux outliers  
 **Pas de multicolinéarité** : Gère surface et nb_pieces corrélés

### ❌ Inconvénients Random Forest

 **Boîte noire** : Difficile d'interpréter un effet marginal précis  
 **Pas de tests statistiques** : Pas de p-value, pas d'intervalle de confiance  
 **Extrapolation** : Ne prédit pas hors de la plage d'entraînement  
 **Overfitting** : Si mal paramétré (arbres trop profonds, peu d'arbres)  
 **Computationnellement coûteux** : Entraînement plus lent que regression lineaire

##  Comparaison avec OLS

### Quand Utiliser OLS ?

- **Objectif** : Comprendre les déterminants, tester des hypothèses
- **Interprétation** : Besoin d'élasticités, effets marginaux
- **Publication** : Article académique (tests requis)
- **Données** : Relation linéaire plausible
- **Taille** : Petit dataset (n < 100)

### Quand Utiliser Random Forest ?

- **Objectif** : Maximiser la précision de prédiction
- **Complexité** : Relations non-linéaires, interactions multiples
- **Robustesse** : Données bruitées, outliers
- **Taille** : Large dataset (n > 500)
- **Application** : Outil opérationnel (estimation automatique)

### Approche Hybride

```
1. Exploration : Random Forest
   → Identifier les variables importantes
   → Détecter les interactions significatives

2. Confirmation : OLS
   → Tester ces variables/interactions spécifiquement
   → Quantifier les effets marginaux
   → Publier les résultats interprétables
```



##  Concepts Machine Learning

### Bias-Variance Tradeoff

**Erreur totale** = Biais² + Variance + Erreur irréductible

- **Biais** : Erreur due à des hypothèses simplificatrices (ex: linéarité en OLS)
- **Variance** : Sensibilité aux fluctuations des données d'entraînement
- **Random Forest** : Réduit la variance (bagging) sans augmenter le biais (arbres profonds)

```
┌──────────────────────────────────────────────────────┐
│        BIAS-VARIANCE TRADEOFF                         │
└──────────────────────────────────────────────────────┘

 Erreur
   ^
   │                     
   │        •            Arbre unique (CART)
   │       / \           Bias faible, Variance ÉLEVÉE
   │      /   \          → Overfitting
   │     /  RF \
   │    /  ●    \        Random Forest
   │   /         \       Bias faible, Variance RÉDUITE
   │  /  OLS      \      → Bon équilibre
   │ /      ●      \
   │/_______________\__  OLS simple
   │                    Bias ÉLEVÉ, Variance faible
   │                    → Underfitting
   └──────────────────────────────> Complexité du modèle
     Simple                        Complexe
```

### Bagging (Bootstrap Aggregating)

**Principe** : Réduire la variance en moyennant plusieurs modèles

1. Créer B échantillons bootstrap (tirage avec remplacement)
2. Entraîner un modèle sur chaque échantillon
3. Prédire en moyennant les B prédictions

**Mathématiquement** :
```
Var(moyenne de B modèles) ≈ Var(modèle unique) / B

Si les modèles sont indépendants (décorrélation par feature randomization)
```

### Out-of-Bag (OOB) Error

Estimation de l'erreur de généralisation **sans validation croisée** :

- Chaque arbre est entraîné sur ~63% des données (bootstrap)
- Les 37% restants = OOB samples pour cet arbre
- Prédire chaque observation avec les arbres qui ne l'ont pas vue
- Calculer l'erreur sur ces prédictions OOB

```python
model = RandomForestRegressor(oob_score=True)
model.fit(X, y)
print(f"OOB R²: {model.oob_score_:.3f}")
```



## Packages Utilisés

### scikit-learn

```python
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score, mean_absolute_error
from sklearn.preprocessing import StandardScaler
```

**Fonctions clés** :
- `RandomForestRegressor()` : Instanciation du modèle
- `.fit(X_train, y_train)` : Entraînement
- `.predict(X_test)` : Prédictions
- `.feature_importances_` : Importance des variables (Gini importance)
- `train_test_split()` : Division train/test (80/20)
- Métriques : RMSE, R², MAE
---
## Instructions d'Exécution

### Prérequis

Identiques à LinearRegression.ipynb :
1. Données : `../DataPreprocessing/PreprocessedData/df_grenoble_vente.csv`
2. Packages : `pip install pandas numpy scikit-learn matplotlib seaborn`

### Exécution

```bash
cd ModelTraining
jupyter notebook RandomForest.ipynb
# Exécuter toutes les cellules
```


### Validation

```python
# Vérifier les modèles entraînés
for name, model in models.items():
    r2 = metrics[name]['r2_test']
    rmse = metrics[name]['rmse_test']
    print(f"{name}: R²={r2:.3f}, RMSE={rmse:,.0f}€")

# Attendu : R² > 0.75, RMSE < 55,000€
```

**Version** : 1.0  
**Dernière mise à jour** : 14 novembre 2025  
