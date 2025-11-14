# ModelTraining (LinearRegression.ipynb) — Régression Linéaire 

##  Résumé 

Ce notebook implémente la **modélisation hédonique des prix immobiliers** par régression linéaire (OLS - Ordinary Least Squares) en utilisant les données prétraitées de Grenoble. Il constitue l'approche économétrique classique pour estimer les déterminants du prix en fonction des caractéristiques des biens.

### Le but dans le Projet

La régression linéaire est le **modèle de référence** en économétrie des prix hédoniques. Ce notebook :

1. **Charge** les données nettoyées (`df_grenoble_vente.csv`)
2. **Prépare** différentes variantes du dataset (échelle originale, logarithmique, standardisée)
3. **Estime** plusieurs spécifications de modèles OLS
4. **Teste** les hypothèses économétriques (significativité, robustesse)
5. **Visualise** les résultats (coefficients, prédictions, diagnostics)
6. **Compare** les performances entre spécifications

### Pourquoi C'est Utile ?

- **Interprétabilité** : Les coefficients β ont une signification économique directe (prix implicite)
- **Tests statistiques** : Significativité individuelle (t-test), globale (F-test), diagnostics (hétéroscédasticité)
- **Référence** : Permet de comparer les modèles ML (Random Forest) à l'approche paramétrique classique
- **Élasticités** : Les modèles log-log fournissent directement des élasticités-prix

### Comment Il Participe à la Démarche Économétrique ?

Ce notebook représente la **Phase 2** du pipeline après le preprocessing :

```
[Phase 1] DataPreparation.ipynb → df_grenoble_vente.csv
                                           ↓
[Phase 2a] LinearRegression.ipynb ────→ Modèles OLS (paramétrique)
                                           ↓
                                  Estimation des coefficients
                                  Tests statistiques
                                  Interprétation économique
```

---



##  Contenu du Notebook

### Structure (11 Sections)

1. **Importations et Configuration** - Bibliothèques (pandas, statsmodels, sklearn)
2. **Installation** - Installation de statsmodels si nécessaire
3. **Chargement** - Lecture de df_grenoble_vente.csv
4. **Préparation** - Séparation X/y
5. **Corrélation** - Matrice et heatmap
6. **Transformations Log** - log_price, log_surface_bati
7. **Standardisation** - StandardScaler (z-score)
8. **Datasets Dérivés** - df_log, df_scaled, etc.
9. **Modélisation OLS** - Estimation via statsmodels.formula.api
10. **Visualisation** - Observé vs Prédit, importance variables
11. **Conclusion** - Synthèse

##  Modèles Estimés

### Spécifications

1. **Niv-Niv** : `price ~ surface_bati + nb_pieces + date + surface_terrain + C(type_local)`
2. **Log-Niv** : `log(price) ~ surface_bati + ...`
3. **Log-Log** : `log(price) ~ log(surface_bati) + ...`
4. **Standardisé** : Variables normalisées (z-score)

### Interprétation des Coefficients

- **Niv-Niv** : β = effet marginal en euros
- **Modèle log-Niv** : β×100 ≈ effet en %
- **Modèle log-log** : β = élasticité (constant)

##  Packages Utilisés

- **statsmodels** : Estimation OLS, tests statistiques, formules R-style
- **sklearn** : StandardScaler pour normalisation
- **pandas, numpy** : Manipulation de données
- **matplotlib, seaborn** : Visualisation

##  Instructions d'Exécution

### Prérequis

1. Données préparées : `../DataPreprocessing/PreprocessedData/df_grenoble_vente.csv`
2. Packages : `pip install pandas numpy statsmodels scikit-learn matplotlib seaborn`

### Exécution

```bash
cd ModelTraining
jupyter notebook LinearRegression.ipynb
# Exécuter toutes les cellules (Run All)
```

**Output attendu** : Plusieurs modèles OLS avec R² entre 0.65-0.80

## Résultats Attendus

- **R²** : 0.70-0.78 (selon spécification)
- **RMSE** : 50,000-60,000 € 
- **Variables significatives** : surface_bati (***), nb_pieces (***), type_local[Maison] (***)
- **Meilleur modèle** : Généralement Log-Log (R² max, RMSE min)

##  Concepts Économétriques

- **Méthode hédonique** : Décomposition du prix en attributs
- **régression linéaire** : Estimateur BLUE sous hypothèses Gauss-Markov
- **Tests** : t-test (significativité), F-test (global), Durbin-Watson
- **Diagnostics** : R², R²_adj, RMSE, AIC
- **Élasticité** : Variation % de y pour 1% de variation de x

---

**Version** : 1.0  
**Dernière mise à jour** : 14 novembre 2025
