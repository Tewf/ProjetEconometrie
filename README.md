# Projet d'Économétrie : Modélisation Hédonique des Prix Immobiliers à Grenoble 2025
ANZID KELTOUM - EL KORAICHI Mohamed Yassine - Hamlil Tawfik

##  Résumé du Projet :

Ce projet académique en économétrie applique les méthodes de **modélisation hédonique des prix** (*hedonic pricing*) au marché immobilier grenoblois. Il analyse les données DVF (Demandes de Valeurs Foncières) du premier semestre 2025 pour identifier les déterminants du prix des biens immobiliers et construire des modèles prédictifs robustes.

### Contexte Économétrique

La **méthode des prix hédoniques** (Rosen, 1974) décompose le prix d'un bien composite (ici, un bien immobilier) en fonction de ses caractéristiques individuelles. Le prix observé P d'un bien i s'exprime comme :

```
P_i = f(X_i) + ε_i
```

où :
- `X_i` = vecteur des caractéristiques du bien (surface, nombre de pièces, localisation, etc.)
- `f()` = fonction de prix hédonique (linéaire ou non-linéaire)
- `ε_i` = terme d'erreur aléatoire

### Objectifs :

1. **Nettoyage et préparation** des données brutes DVF pour la commune de Grenoble
2. **Analyse exploratoire** des variables explicatives et de la variable dépendante (prix)
3. **Estimation économétrique** via régression linéaire et ses variantes (log-log, niv-log)
4. **Modélisation prédictive** par algorithmes d'apprentissage automatique (Random Forest: ensemble d’arbres capturant non-linéarités et interactions))
5. **Tests de robustesse** et diagnostics (hétéroscédasticité = variance non constante ; multicolinéarité = variables corrélées ; outliers = valeurs extrêmes)
6. **Interprétation économique** des résultats et implications pour le marché immobilier


##  Structure du Projet

```
ProjetEconometrie/
│
├── README.md                          # Ce fichier
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

##  Plan du Projet :

```
┌─────────────────────────────────────────────────────────────────┐
│                    Plan                      │
└─────────────────────────────────────────────────────────────────┘

    [1] Données Brutes DVF                                 [Fichier : valeursfoncieres-2025-s1.txt.zip]
           │                                               [Taille : 1,387,077 transactions nationales]
           │                                              
           ▼
    ┌──────────────────────────────┐
    │  PREPROCESSING               │                      [Notebook : DataPreparation.ipynb]
    │  ─────────────────           │
    │  • Filtrage géographique     │ ─────────────▶      Grenoble uniquement (INSEE: 38185)
    │    (commune de Grenoble)     │
    │  • Filtrage par nature       │ ─────────────▶      Ventes uniquement (pas de donations)
    │  • Gestion valeurs manquantes│ ─────────────▶      Suppression/imputation selon % missing
    │  • Détection outliers        │ ─────────────▶      Méthode IQR par type de bien (Méthode IQR : on repère les valeurs anormalement hautes ou basses en utilisant l’intervalle interquartile Q1–Q3, et on exclut les points situés au-delà de 1.5×IQR)
    │  • Ingénierie de features    │ ─────────────▶      Variables temporelles (mois, trimestre)
    │  • Encodage catégories       │ ─────────────▶      type_local → codes 1-4 ( on évite 0 , pour ne pas avoir des problème avec ln)
    └──────────────────────────────┘
           │
           │ [Output : df_grenoble_vente.csv]
           │ [Observations : 1,288]
           │ [Variables : 7]
           ▼
    ┌──────────────────────────────┐
    │  EXPLORATION & DIAGNOSTICS   │                      [Notebooks : Linear + Random Forest]
    │  ─────────────────────────   │
    │  • Statistiques descriptives │
    │  • Matrice de corrélation    │
    │  • Tests de normalité        │
    │  • Analyse graphique         │
    └──────────────────────────────┘
           │
           ├─────────────────────────────────┬─────────────────────────────────┐
           ▼                                 ▼                                 ▼
    ┌──────────────────┐            ┌──────────────────┐            ┌─────────────────────────┐
    │ MODÈLE 1 : OLS   │            │ MODÈLE 2 : Log   │            │ MODÈLE 3 : Random Forest│
    │                  │            │                  │            │                         │
    │ P = β₀ + βX + ε  │            │ log(P) = βX + ε  │            │ Ensemble de 100         │
    │                  │            │                  │            │ arbres CART             │
    │ Interprétation : │            │ Interprétation : │            │                         │
    │ β = effet        │            │ β = élasticité % │            │ Non-paramétrique        │
    │ marginal         │            │                  │            │                         │
    └──────────────────┘            └──────────────────┘            └─────────────────────────┘
           │                                 │                                 │
           └─────────────────────────────────┴─────────────────────────────────┘
                                             │
                                             ▼
                                    ┌──────────────────────────────┐
                                    │  ÉVALUATION & COMPARAISON    │
                                    │  ────────────────────────    │
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
                                    │  ────────────────────────    │
                                    │  • Élasticités-prix          │
                                    │  • Valeur marginale des      │
                                    │    caractéristiques          │
                                    │  • Prédiction de prix        │
                                    └──────────────────────────────┘
```

##  Données : DVF (Demandes de Valeurs Foncières)

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





### Reproduction des Résultats

Pour garantir la **reproductibilité** :

1. **Données** : Les données brutes DVF sont publiques et accessibles sur data.gouv.fr
2. **Seed aléatoire** : Le notebook Random Forest utilise `random_state=42` pour la reproductibilité des splits et de l'entraînement
3. **Environnement** : Utiliser les versions de packages spécifiées ci-dessus
4. **Ordre** : Respecter strictement l'ordre d'exécution (DataPreparation → LinearRegression / RandomForest)

##  Méthodologie Économétrique

### 1. Modèle Linéaire Simple (OLS)

**Spécification** :
```
price_i = β₀ + β₁·surface_bati_i + β₂·nb_pieces_i + β₃·date_i + β₄·surface_terrain_i + Σ β_k·type_k + ε_i
```

**Hypothèses** :
- Linéarité de la relation prix-caractéristiques
- Exogénéité stricte : E(ε_i|X_i) = 0
- Homoscédasticité : Var(ε_i|X_i) = σ²
- Non-autocorrélation des erreurs (données cross-section)
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

### 2. Modèle Log-Linéaire (log-niv)

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

**Principe** : Ensemble de **arbres de décision** (CART) construits par bootstrap aggregating (bagging).

La Random Forest est une méthode de prédiction qui utilise beaucoup d’arbres de décision.
Un arbre de décision, c’est comme une série de questions du type :

« La surface est-elle > 40 m² ? »

« Le type du bien est-il Appartement ? »

« Le prix est-il supérieur à X ? »

Chaque arbre fait sa prédiction individuellement, puis la Random Forest moyenne les prédictions de tous les arbres pour obtenir un résultat plus fiable.

C’est un peu comme demander leur avis à 100 experts différents au lieu d’un seul.

**Paramètres** :
- Nombre d'arbres : 100  ➝ Plus il y a d’arbres, plus la prédiction est fiable. 100 arbres = un bon compromis entre précision et rapidité.
- Critère de split : MSE (Mean Squared Error)  ➝ À chaque étape, l’arbre choisit la meilleure question (split) qui minimise l’erreur quadratique moyenne.
- Profondeur maximale : illimitée (croissance complète) ➝ Les arbres peuvent grandir tant qu’ils trouvent de la différence dans les données.
Cela leur permet de capturer des relations complexes.

**Avantages** :
- Non-paramétrique : pas d'hypothèse sur la forme fonctionnelle.  Contrairement à la régression linéaire, on n’impose pas une relation linéaire entre prix et variables.
Le modèle découvre lui-même la forme de la relation.
- Capture les interactions et non-linéarités automatiquement.     Exemple :
l’effet de la surface peut dépendre du nombre de pièces, du type de bien, du quartier…
La Random Forest apprend tout ça toute seule.
- Robuste aux outliers et valeurs manquantes.     Quelques ventes aberrantes (trop chères ou trop basses) ne cassent pas la prédiction.
- Pas de multicolinéarité.    Même si deux variables sont très corrélées (surface ↔ pièces), la Random Forest le gère sans problème.

**Évaluation** :
- Importance des variables : réduction moyenne de l'impureté (Gini importance) ➝ Le modèle mesure quelles variables aident le plus à réduire l’erreur.
C’est un indicateur de “quelles caractéristiques expliquent le mieux le prix ?”
- Validation croisée : split train/test (80/20) ➝ 80% des données pour apprendre, 20% pour vérifier si le modèle prédit bien sur des données jamais vues.
C’est essentiel pour tester la fiabilité du modèle.

##  Concepts Économétriques Clés

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
- Matrice de corrélation ➝ Permet de visualiser rapidement quelles variables explicatives sont fortement corrélées entre elles.
Si deux variables ont une corrélation proche de 0.8–0.9, cela indique une possible multicolinéarité.
- VIF (Variance Inflation Factor) ➝ Indicateur qui mesure combien la variance d’un coefficient est “gonflée” à cause de la corrélation avec d’autres variables.
Un VIF > 10 est généralement considéré comme un signe sérieux de multicolinéarité.

**Solutions** :
- Retrait de variables redondantes ➝ Si deux variables sont presque identiques (ex : surface et nombre de pièces), on peut en supprimer une pour stabiliser le modèle et éviter la multicolinéarité.
- Régularisation (Ridge, Lasso) ➝ Ce sont des modèles de régression qui ajoutent une pénalité lorsque les coefficients deviennent trop grands.
Ridge réduit la variance des coefficients (multicolinéarité), Lasso peut même éliminer des variables inutiles.
- Composantes principales (PCA) ➝ Méthode qui transforme les variables corrélées en nouvelles variables indépendantes appelées “composantes principales”.
Cela réduit la dimension du problème tout en conservant l’information essentielle.

### Hétéroscédasticité

**Définition** : Variance des erreurs non constante : Var(ε_i|X_i) = σ_i² ≠ σ²

**Conséquences** :
- Estimateurs OLS non efficients (mais non biaisés)
- Tests statistiques invalides

**Détection** :
- Test de Breusch-Pagan ➝ Test statistique qui vérifie si la variance des erreurs dépend des variables explicatives. S’il est significatif, il y a hétéroscédasticité.
- Test de White ➝ Version plus générale du test, capable de détecter toutes formes possibles d’hétéroscédasticité (même non linéaires).
- Graphique des résidus ➝ On trace les résidus en fonction des valeurs prédites : si on observe un “cône” ou un motif, cela indique une variance non constante.

**Solutions** :
- Transformation logarithmique ➝ Prendre log(price) stabilise souvent la variance, surtout quand les prix élevés ont des écarts plus grands.
- Erreurs robustes (White, HC3) ➝ Permet de corriger les tests t et F même si l’hétéroscédasticité est présente, sans modifier le modèle.
- WLS (Weighted Least Squares) ➝ Méthode qui “pondère” les observations selon leur variance : les points avec forte variance comptent moins, ce qui résout l’hétéroscédasticité à la source.




##  Résultats Attendus

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



##  Références Bibliographiques

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


##  Installation et Exécution

### Prérequis

- **Python** ≥ 3.8 (testé avec Python 3.12.11)
- **Jupyter Notebook** ou **JupyterLab**
- **Gestionnaire de paquets** : pip ou conda

### Packages Requis

```bash
# Installation des dépendances
pip install pandas numpy scipy matplotlib seaborn statsmodels scikit-learn

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


##  Diagnostic et Résolution de Problèmes

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




