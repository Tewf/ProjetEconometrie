# Projet d'Économétrie : Modélisation Hédonique des Prix Immobiliers à Grenoble

## 📋 Résumé du Projet

Ce projet académique de niveau Master/Doctorat en économétrie applique les méthodes de **modélisation hédonique des prix** (*hedonic pricing*) au marché immobilier grenoblois. Il analyse les données DVF (Demandes de Valeurs Foncières) du premier semestre 2025 pour identifier les déterminants du prix des biens immobiliers et construire des modèles prédictifs robustes.

### Contexte Économétrique

La **méthode des prix hédoniques** (Rosen, 1974) décompose le prix d'un bien composite (ici, un bien immobilier) en fonction de ses caractéristiques individuelles. Le prix observé P d'un bien i s'exprime comme :

```
P_i = f(X_i) + ε_i
```

où :
- `X_i` = vecteur des caractéristiques du bien (surface, nombre de pièces, localisation, etc.)
- `f()` = fonction de prix hédonique (linéaire ou non-linéaire)
- `ε_i` = terme d'erreur aléatoire

### Objectifs du Projet

1. **Nettoyage et préparation** des données brutes DVF pour la commune de Grenoble
2. **Analyse exploratoire** des variables explicatives et de la variable dépendante (prix)
3. **Estimation économétrique** via régression linéaire (OLS) et ses variantes (log-log, semi-log)
4. **Modélisation prédictive** par algorithmes d'apprentissage automatique (Random Forest)
5. **Tests de robustesse** et diagnostics (hétéroscédasticité, multicolinéarité, outliers)
6. **Interprétation économique** des résultats et implications pour le marché immobilier

## 🗂️ Structure du Projet

```
ProjetEconometrie/
│
├── README.md                          # Ce fichier (vue d'ensemble)
│
├── DataPreprocessing/                 # Étape 1 : Préparation des données
│   ├── README.md                      # Documentation détaillée du preprocessing
│   ├── DataPreparation.ipynb          # Notebook de nettoyage et transformation
│   ├── valeursfoncieres-2025-s1.txt.zip  # Données brutes DVF (1,4M transactions)
│   ├── PreprocessedData/
│   │   └── df_grenoble_vente.csv      # Données nettoyées (1288 observations)
│   └── DataDocumentation/             # Documentation officielle DVF
│       ├── README.md                  # Guide des sources de données
│       ├── notice-descriptive-du-fichier-dvf-20221017.pdf
│       ├── faq-20221017.pdf
│       ├── conditions-generales-dutilisation-20201016.pdf
│       └── information-des-personnes-concernees-par-le-traitement-informatique-20201016.pdf
│
├── ModelTraining/                     # Étape 2 : Modélisation économétrique
│   ├── README.md                      # Documentation régression linéaire
│   ├── README_RandomForest.md         # Documentation Random Forest
│   ├── LinearRegression.ipynb         # Modèles OLS avec statsmodels
│   └── RandomForest.ipynb             # Modèles ML avec scikit-learn
│
└── RevueDeLitterature/                # Étape 0 : Fondements théoriques
    ├── README.md                      # Contexte académique
    └── HedonicHousingPriceIndexes.pdf # Article de référence
```

## 📊 Pipeline Méthodologique

```
┌─────────────────────────────────────────────────────────────────┐
│                    PIPELINE ÉCONOMÉTRIQUE                        │
└─────────────────────────────────────────────────────────────────┘

    [1] Données Brutes DVF                                 [Fichier : valeursfoncieres-2025-s1.txt.zip]
           │                                               [Taille : 1,387,077 transactions nationales]
           │                                               [Format : TXT délimité par |]
           ▼
    ┌──────────────────────────────┐
    │  PREPROCESSING               │                      [Notebook : DataPreparation.ipynb]
    │  ─────────────────            │
    │  • Filtrage géographique     │ ─────────────▶      Grenoble uniquement (INSEE: 38185)
    │    (commune de Grenoble)     │
    │  • Filtrage par nature       │ ─────────────▶      Ventes uniquement (pas de donations)
    │  • Gestion valeurs manquantes│ ─────────────▶      Suppression/imputation selon % missing
    │  • Détection outliers        │ ─────────────▶      Méthode IQR par type de bien
    │  • Ingénierie de features    │ ─────────────▶      Variables temporelles (mois, trimestre)
    │  • Encodage catégories       │ ─────────────▶      type_local → codes 1-4
    └──────────────────────────────┘
           │
           │ [Output : df_grenoble_vente.csv]
           │ [Observations : 1,288]
           │ [Variables : 7]
           ▼
    ┌──────────────────────────────┐
    │  EXPLORATION & DIAGNOSTICS   │                      [Notebooks : Linear + Random Forest]
    │  ─────────────────────────    │
    │  • Statistiques descriptives │
    │  • Matrice de corrélation    │
    │  • Tests de normalité        │
    │  • Analyse graphique         │
    └──────────────────────────────┘
           │
           ├─────────────────────────────────┬─────────────────────────────────┐
           ▼                                 ▼                                 ▼
    ┌──────────────────┐            ┌──────────────────┐            ┌──────────────────┐
    │ MODÈLE 1 : OLS   │            │ MODÈLE 2 : Log   │            │ MODÈLE 3 : RF    │
    │                  │            │                  │            │                  │
    │ P = β₀ + βX + ε  │            │ log(P) = βX + ε  │            │ Ensemble de 100  │
    │                  │            │                  │            │ arbres CART      │
    │ Interprétation : │            │ Interprétation : │            │                  │
    │ β = effet        │            │ β = élasticité % │            │ Non-paramétrique │
    │ marginal         │            │                  │            │                  │
    └──────────────────┘            └──────────────────┘            └──────────────────┘
           │                                 │                                 │
           └─────────────────────────────────┴─────────────────────────────────┘
                                             │
                                             ▼
                                    ┌──────────────────────────────┐
                                    │  ÉVALUATION & COMPARAISON    │
                                    │  ────────────────────────     │
                                    │  • R² (coefficient de        │
                                    │    détermination)            │
                                    │  • RMSE (erreur quadratique) │
                                    │  • MAE (erreur absolue)      │
                                    │  • Tests statistiques        │
                                    │    (F, t, DW, BP, White)     │
                                    └──────────────────────────────┘
                                             │
                                             ▼
                                    ┌──────────────────────────────┐
                                    │  INTERPRÉTATION ÉCONOMIQUE   │
                                    │  ────────────────────────     │
                                    │  • Élasticités-prix          │
                                    │  • Valeur marginale des      │
                                    │    caractéristiques          │
                                    │  • Prédiction de prix        │
                                    └──────────────────────────────┘
```

## 🎯 Données : DVF (Demandes de Valeurs Foncières)

### Source

Les données proviennent de la **base DVF** mise à disposition par la Direction Générale des Finances Publiques (DGFiP). Elles recensent l'ensemble des mutations immobilières (ventes, donations, échanges) enregistrées par les services de publicité foncière en France.

**Fichier utilisé** : `valeursfoncieres-2025-s1.txt.zip` (1er semestre 2025)

### Variables Clés Retenues

Après prétraitement, le jeu de données `df_grenoble_vente.csv` contient 7 variables :

| Variable | Type | Description | Unité |
|----------|------|-------------|-------|
| **price** | Continue | Prix de vente (valeur foncière) | Euros (€) |
| **type_local** | Catégorielle | Type de bien immobilier | {Appartement, Maison, Local industriel/commercial, Dépendance} |
| **surface_bati** | Continue | Surface bâtie (carrez) | Mètres carrés (m²) |
| **surface_terrain** | Continue | Surface de terrain associée | Mètres carrés (m²) |
| **date** | Discrète | Mois de la mutation | Entier 1-12 (janvier-décembre) |
| **nb_pieces** | Discrète | Nombre de pièces principales | Entier ≥ 0 |
| **type_local_1234** | Catégorielle encodée | Version numérique de type_local | {1=Appart, 2=Maison, 3=Local, 4=Dép} |

### Statistiques Descriptives (après nettoyage)

- **Observations** : 1 288 transactions de vente
- **Période** : 1er semestre 2025
- **Zone géographique** : Commune de Grenoble (INSEE 38185)
- **Prix moyen** : ~250 000 € (varie selon type de bien)
- **Surface moyenne** : ~60 m² (appartements dominants)

## 🔧 Installation et Exécution

### Prérequis

- **Python** ≥ 3.8 (testé avec Python 3.12.11)
- **Jupyter Notebook** ou **JupyterLab**
- **Gestionnaire de paquets** : pip ou conda

### Packages Requis

```bash
# Installation des dépendances
pip install pandas numpy scipy matplotlib seaborn statsmodels scikit-learn

# Versions recommandées (compatibilité testée)
# pandas >= 2.0
# numpy >= 1.22
# statsmodels >= 0.14
# scikit-learn >= 1.0
# matplotlib >= 3.5
# seaborn >= 0.12
```

### Ordre d'Exécution

#### Étape 1 : Préparation des Données

```bash
# 1. Extraire les données brutes
cd DataPreprocessing
unzip valeursfoncieres-2025-s1.txt.zip

# 2. Lancer le notebook de preprocessing
jupyter notebook DataPreparation.ipynb

# Exécuter toutes les cellules (Runtime > Run all)
# Output attendu : PreprocessedData/df_grenoble_vente.csv
```

#### Étape 2 : Modélisation Économétrique

```bash
# 3. Modèle OLS (régression linéaire)
cd ../ModelTraining
jupyter notebook LinearRegression.ipynb

# 4. Modèle Random Forest (apprentissage automatique)
jupyter notebook RandomForest.ipynb
```

### Reproduction des Résultats

Pour garantir la **reproductibilité** :

1. **Données** : Les données brutes DVF sont publiques et accessibles sur data.gouv.fr
2. **Seed aléatoire** : Le notebook Random Forest utilise `random_state=42` pour la reproductibilité des splits et de l'entraînement
3. **Environnement** : Utiliser les versions de packages spécifiées ci-dessus
4. **Ordre** : Respecter strictement l'ordre d'exécution (DataPreparation → LinearRegression / RandomForest)

## 📈 Méthodologie Économétrique

### 1. Modèle Linéaire Simple (OLS)

**Spécification** :
```
price_i = β₀ + β₁·surface_bati_i + β₂·nb_pieces_i + β₃·date_i + β₄·surface_terrain_i + Σ β_k·type_k + ε_i
```

**Hypothèses** :
- Linéarité de la relation prix-caractéristiques
- Homoscédasticité : Var(ε_i|X_i) = σ²
- Non-autocorrélation des erreurs (données cross-section)
- Exogénéité stricte : E(ε_i|X_i) = 0
- Normalité des résidus (pour tests d'hypothèse)

**Estimation** : Méthode des Moindres Carrés Ordinaires (OLS)
```
β̂ = (X'X)⁻¹X'y
```

**Tests réalisés** :
- Test de Student (t-test) : significativité individuelle des coefficients
- Test de Fisher (F-test) : significativité globale du modèle
- Test de Durbin-Watson : autocorrélation des résidus
- Test de Breusch-Pagan : hétéroscédasticité
- Test de White : forme générale d'hétéroscédasticité

### 2. Modèle Log-Linéaire (Semi-log)

**Spécification** :
```
log(price_i) = β₀ + β₁·X_i + ε_i
```

**Interprétation** : Les coefficients β représentent des **semi-élasticités**
- Une augmentation de 1 unité de X_j entraîne une variation de (e^βj - 1) × 100% du prix

**Avantages** :
- Réduit l'influence des valeurs extrêmes
- Stabilise la variance (corrige l'hétéroscédasticité)
- Interprétation en termes d'élasticité (plus économique)

### 3. Modèle Random Forest

**Principe** : Ensemble de **arbres de décision** (CART) construits par bootstrap aggregating (bagging)

**Paramètres** :
- Nombre d'arbres : 100
- Critère de split : MSE (Mean Squared Error)
- Profondeur maximale : illimitée (croissance complète)

**Avantages** :
- Non-paramétrique : pas d'hypothèse sur la forme fonctionnelle
- Capture les interactions et non-linéarités automatiquement
- Robuste aux outliers et valeurs manquantes
- Pas de multicolinéarité

**Évaluation** :
- Importance des variables : réduction moyenne de l'impureté (Gini importance)
- Validation croisée : split train/test (80/20)

## 📚 Concepts Économétriques Clés

### Prix Hédoniques

**Définition** : Le prix d'un bien hétérogène reflète la somme des valeurs implicites de ses caractéristiques.

**Formulation théorique** (Rosen, 1974) :
```
P(z₁, z₂, ..., z_k) = f(z₁, z₂, ..., z_k)
```
où z_j sont les attributs du bien (surface, localisation, qualité, etc.)

**Prix implicite** d'une caractéristique j :
```
∂P/∂z_j = prix hédonique marginal de z_j
```

Cela représente la disposition à payer (willingness to pay) pour une unité supplémentaire de la caractéristique j.

### Élasticité-Prix

**Définition** : Variation % du prix suite à une variation % d'une caractéristique

**Formule** :
```
ε_P,X = (∂P/∂X) × (X/P)
```

**Interprétation** :
- ε > 1 : demande élastique (sensible aux variations)
- ε < 1 : demande inélastique
- Dans un modèle log-log : le coefficient β est directement l'élasticité

### Multicolinéarité

**Problème** : Corrélation forte entre variables explicatives (ex: surface_bati et nb_pieces)

**Conséquences** :
- Variance des estimateurs élevée
- Coefficients instables
- Difficultés d'interprétation

**Détection** :
- Matrice de corrélation
- VIF (Variance Inflation Factor)

**Solutions** :
- Retrait de variables redondantes
- Régularisation (Ridge, Lasso)
- Composantes principales (PCA)

### Hétéroscédasticité

**Définition** : Variance des erreurs non constante : Var(ε_i|X_i) = σ_i² ≠ σ²

**Conséquences** :
- Estimateurs OLS non efficients (mais non biaisés)
- Tests statistiques invalides

**Détection** :
- Test de Breusch-Pagan
- Test de White
- Graphique des résidus

**Solutions** :
- Transformation logarithmique
- Erreurs robustes (White, HC3)
- WLS (Weighted Least Squares)

## 🔍 Résultats Attendus

### Déterminants Principaux du Prix (hypothèses)

1. **Surface bâtie** : effet positif et significatif (β₁ > 0, p < 0.01)
   - Élasticité élevée (~0.7-0.9)
2. **Nombre de pièces** : effet positif mais corrélé avec surface
3. **Type de bien** : Maisons > Appartements > Locaux commerciaux
4. **Surface terrain** : effet positif pour maisons, négligeable pour appartements
5. **Temporalité** : effet faible (données courte période)

### Performance des Modèles

| Modèle | R² attendu | RMSE attendu | Avantages |
|--------|-----------|--------------|-----------|
| OLS linéaire | 0.65-0.75 | 60 000€ | Interprétabilité, tests statistiques |
| OLS log-log | 0.70-0.80 | 50 000€ | Stabilité, élasticités |
| Random Forest | 0.75-0.85 | 45 000€ | Performance prédictive, non-linéarités |

## 🎓 Public Cible et Utilisation Pédagogique

Ce projet est conçu pour :

### Étudiants Débutants en Économétrie
- Apprentissage progressif des concepts (de OLS à ML)
- Code commenté ligne par ligne
- Visualisations pédagogiques à chaque étape

### Enseignants et Formateurs
- Matériel prêt à l'emploi pour cours de régression
- Cas d'application réel et concret
- Datasets préparés et documentés

### Chercheurs et Analystes
- Pipeline reproductible
- Tests statistiques complets
- Comparaison méthodologique (paramétrique vs non-paramétrique)

## 📖 Références Bibliographiques

### Ouvrages de Référence

1. **Wooldridge, J.M. (2020)**. *Introductory Econometrics: A Modern Approach*. 7th ed. Cengage Learning.
   - Chapitres 2-8 : Régression linéaire multiple, tests, hétéroscédasticité

2. **Greene, W.H. (2018)**. *Econometric Analysis*. 8th ed. Pearson.
   - Chapitres 4-9 : Théorie asymptotique, MCO, GLS, tests de spécification

3. **Hastie, T., Tibshirani, R., & Friedman, J. (2009)**. *The Elements of Statistical Learning*. 2nd ed. Springer.
   - Chapitre 15 : Random Forests et méthodes d'ensemble

### Articles Académiques

4. **Rosen, S. (1974)**. "Hedonic Prices and Implicit Markets: Product Differentiation in Pure Competition". *Journal of Political Economy*, 82(1), 34-55.
   - Fondements théoriques de la méthode hédonique

5. Voir `RevueDeLitterature/HedonicHousingPriceIndexes.pdf` pour une synthèse récente sur les indices hédoniques de prix immobiliers

### Ressources Techniques

6. **Documentation statsmodels** : https://www.statsmodels.org/
   - Régression OLS, diagnostics, tests statistiques

7. **Documentation scikit-learn** : https://scikit-learn.org/
   - Random Forest Regressor, preprocessing, métriques

8. **Documentation DVF** : Voir `DataPreprocessing/DataDocumentation/`
   - Notice descriptive officielle des fichiers DVF

## 🛠️ Diagnostic et Résolution de Problèmes

### Problèmes Courants

#### 1. Erreur de chargement des données
```
FileNotFoundError: valeursfoncieres-2025-s1.txt
```
**Solution** : Extraire le fichier .zip dans le dossier DataPreprocessing
```bash
cd DataPreprocessing
unzip valeursfoncieres-2025-s1.txt.zip
```

#### 2. Module statsmodels introuvable
```
ModuleNotFoundError: No module named 'statsmodels'
```
**Solution** :
```bash
pip install statsmodels
# OU dans le notebook :
%pip install statsmodels
```

#### 3. Avertissements DtypeWarning
```
DtypeWarning: Columns have mixed types
```
**Solution** : Normal pour les données DVF brutes. Le notebook gère automatiquement les conversions de types.

#### 4. Résultats différents (Random Forest)
**Cause** : Initialisation aléatoire différente
**Solution** : Vérifier que `random_state=42` est bien défini dans RandomForestRegressor

## 🔄 Évolutions Futures

### Extensions Possibles

1. **Données longitudinales** : Analyser plusieurs semestres pour capturer l'évolution temporelle des prix
2. **Données spatiales** : Intégrer des variables de localisation fine (quartier, distance CBD, aménités)
3. **Modèles avancés** :
   - Gradient Boosting (XGBoost, LightGBM)
   - Réseaux de neurones (Deep Learning)
   - Modèles spatiaux (SAR, SEM)
4. **Variables supplémentaires** :
   - État du bien (neuf, ancien)
   - Étage, présence ascenseur
   - Performance énergétique (DPE)
5. **Tests économétriques avancés** :
   - Variables instrumentales (2SLS) si endogénéité
   - Panel data (si données répétées)
   - Quantile regression (distribution complète)

## 📞 Contact et Contribution

### Auteur
Projet académique en économétrie appliquée

### Contribution
Ce projet est open source et à vocation pédagogique. Les contributions sont les bienvenues :
- Améliorations du code
- Ajout de tests statistiques
- Extension à d'autres communes
- Traduction en anglais

### Licence
Les données DVF sont publiques (licence ouverte). Le code est sous licence MIT.

---

## 📌 Checklist de Validation

Avant de considérer le projet complet, vérifier :

- [ ] Toutes les cellules des notebooks s'exécutent sans erreur
- [ ] Le fichier `df_grenoble_vente.csv` est généré avec 1288 observations
- [ ] Les modèles OLS affichent des coefficients significatifs (p < 0.05) pour surface_bati
- [ ] Le R² des modèles est > 0.60
- [ ] Les graphiques "Observé vs Prédit" montrent une corrélation visible
- [ ] Les tests d'hétéroscédasticité sont réalisés
- [ ] La matrice de corrélation est visualisée
- [ ] Les importances de variables (RF) sont cohérentes avec la théorie économique

---

**Version** : 1.0 (Novembre 2025)  
**Dernière mise à jour** : 13 novembre 2025
