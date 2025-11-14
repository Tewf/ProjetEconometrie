# DataPreprocessing — Préparation et Nettoyage des Données DVF

## Résumé Global

Ce dossier contient le **pipeline complet de prétraitement** des données DVF (Demandes de Valeurs Foncières) pour l'analyse du marché immobilier grenoblois. Il transforme un fichier brut national de 1,4 million de transactions en un dataset propre et exploitable de 1 288 observations prêtes pour la modélisation économétrique.

### Rôle dans le Projet

Le prétraitement constitue la **première étape critique** de toute analyse économétrique. Sans données nettoyées et structurées, les estimations seront biaisées et les conclusions invalides. Ce module :

1. **Filtre** les données pertinentes (Grenoble, ventes uniquement)
2. **Nettoie** les valeurs manquantes, aberrantes et incohérentes
3. **Transforme** les variables pour la modélisation (encodage, normalisation)
4. **Exporte** un dataset prêt à l'emploi pour les modèles

### Pourquoi C'est Utile ?

- **Gain de temps** : Les modèles consomment directement `df_grenoble_vente.csv` sans retraiter les données
- **Reproductibilité** : Toutes les transformations sont documentées et scriptées
- **Qualité** : Les outliers et erreurs de saisie sont éliminés
- **Transparence** : Chaque décision de nettoyage est justifiée et traçable

##  Contenu du Dossier

```
DataPreprocessing/
│
├── README.md                                   # Ce fichier 
│
├── DataPreparation.ipynb                       # Notebook principal de preprocessing
│                                               # Langage : Python
│                                               # Format : Jupyter Notebook (.ipynb)
│                                               # Cellules : 23 (markdown + code)
│
├── valeursfoncieres-2025-s1.txt.zip            # Données brutes DVF (compressées)
│                                               # Source : DGFiP / data.gouv.fr
│                                               # Période : 1er semestre 2025
│                                               # Observations : 1,387,077 mutations nationales
│                                               # Variables : 43 colonnes
│                                               # Séparateur : pipe (|)
│
├── PreprocessedData/                           # Dossier des données nettoyées
│   └── df_grenoble_vente.csv                   # Output final
│                                               # Observations : 1,288
│                                               # Variables : 7
│                                               # Séparateur : virgule (,)
│
└── DataDocumentation/                          # Documentation officielle DVF
    ├── README.md                               # Guide des documents
    ├── notice-descriptive-du-fichier-dvf-20221017.pdf
    │                                           # Dictionnaire des variables DVF
    │                                           # Définitions officielles
    │
    ├── faq-20221017.pdf                        # Questions fréquentes
    │                                           # Cas d'usage et limites
    │
    ├── conditions-generales-dutilisation-20201016.pdf
    │                                           # Licence d'utilisation
    │                                           # Restrictions et droits
    │
    └── information-des-personnes-concernees-par-le-traitement-informatique-20201016.pdf
                                                # RGPD et protection des données
```

## DataPreparation.ipynb 

### Vue d'Ensemble du Notebook

Le notebook `DataPreparation.ipynb` est le **cœur du preprocessing**. Il suit une méthodologie rigoureuse en 17 étapes pour transformer les données brutes en un dataset exploitable.

### Structure du Notebook

```
┌────────────────────────────────────────────────────────────┐
│              PLAN DU NOTEBOOK (17 SECTIONS)                │
└────────────────────────────────────────────────────────────┘

[1] Importations et Configuration
     ├─ Chargement des bibliothèques Python
     ├─ Configuration de l'environnement
     └─ Définition des constantes

[2] Chargement des Données Brutes
     ├─ Lecture du fichier ValeursFoncieres-2025-S1.txt
     ├─ Détection automatique du séparateur (|)
     └─ Affichage des dimensions (1,387,077 × 43)

[3] Synthèse du Jeu de Données Initial
     ├─ Types des variables (numériques, catégorielles)
     ├─ Pourcentage de valeurs manquantes
     └─ Statistiques descriptives globales

[4] Filtrage Géographique (Grenoble)
     ├─ Sélection code_commune == '38185'
     ├─ Réduction à ~15,000 observations
     └─ Vérification de la couverture

[5] Synthèse des Données Filtrées
     ├─ Nouvelles dimensions
     └─ Distribution des types de mutation

[6] Construction du Sous-Ensemble Ventes
     ├─ Filtrage nature_mutation == 'Vente'
     ├─ Sélection des variables pertinentes
     └─ Création de df_grenoble_vente

[7] Diagnostic des Valeurs Manquantes
     ├─ Calcul du % missing par variable
     ├─ Visualisation (barplot)
     └─ Décisions de suppression ou imputation

[8] Normalisation des Placeholders
     ├─ Conversion '' → NaN
     ├─ Conversion '0.0' → NaN (si illogique)
     └─ Harmonisation des types

[9] Nettoyage Final et Filtrage des Aberrations
     ├─ Suppression des prix ≤ 0 ou extrêmes
     ├─ Suppression des surfaces ≤ 0
     ├─ Filtrage des transactions incomplètes
     └─ Réduction à ~1,800 observations

[10] Ingénierie de Variables Temporelles
     ├─ Extraction du mois depuis date_mutation
     ├─ Création de la variable 'date' (1-12)
     └─ Détection de saisonnalité

[11] Distribution des Types de Bien
     ├─ Comptage par type_local
     ├─ Visualisation (barplot)
     └─ Identification des types dominants

[12] Détection et Retrait des Valeurs Aberrantes
     ├─ Méthode IQR par type de bien
     ├─ Q1 - 1.5×IQR et Q3 + 1.5×IQR
     └─ Suppression de ~500 outliers

[13] Visualisation Avant/Après par Type
     ├─ Boxplots avant filtrage outliers
     ├─ Boxplots après filtrage
     └─ Comparaison de la distribution

[14] Encodage de type_local en Catégories 1-4
     ├─ Appartement → 1
     ├─ Maison → 2
     ├─ Local industriel/commercial → 3
     ├─ Dépendance → 4
     └─ Création de type_local_1234

[15] Matrice de Corrélation
     ├─ Calcul des corrélations bivariées
     ├─ Heatmap avec annotations
     └─ Détection de multicolinéarité

[16] Export des Jeux de Données
     ├─ Sauvegarde de df_grenoble_vente.csv
     ├─ 1,288 observations × 7 variables

[17] Conclusion
     └─ Récapitulatif et prochaines étapes
```
### Workflow (Entrée → Traitement → Sortie)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        WORKFLOW DÉTAILLÉ                                │
└─────────────────────────────────────────────────────────────────────────┘

INPUT
┌──────────────────────────────────────┐
│ ValeursFoncieres-2025-S1.txt         │
│ ─────────────────────────────────    │
│ Format : TXT délimité par |          │
│ Lignes : 1,387,077                   │
│ Colonnes : 43                        │
└──────────────────────────────────────┘
            │
            │ pd.read_csv(..., sep='|')
            ▼
┌──────────────────────────────────────┐
│ df (DataFrame initial)               │
│ ─────────────────────                │
│ Toutes transactions France           │
│ Mixte : ventes, donations, échanges  │
│ Toutes communes                      │
└──────────────────────────────────────┘
            │
            │ ÉTAPE 1 : Filtrage géographique
            │ df[df['code_commune'] == '38185']
            ▼
┌──────────────────────────────────────┐
│ df_grenoble (géo-filtré)             │
│ ─────────────────────                │
│ Lignes : ~15,000                     │
│ Zone : Grenoble uniquement           │
│ INSEE : 38185                        │
└──────────────────────────────────────┘
            │
            │ ÉTAPE 2 : Filtrage par nature de mutation
            │ df[df['nature_mutation'] == 'Vente']
            ▼
┌──────────────────────────────────────┐
│ df_grenoble_vente (mutations)        │
│ ─────────────────────                │
│ Lignes : ~2,500                      │
│ Nature : Ventes uniquement           │
│ Exclu : donations, échanges          │
└──────────────────────────────────────┘
            │
            │ ÉTAPE 3 : Sélection de variables
            │ Colonnes : price, type_local, surface_bati,
            │            surface_terrain, date_mutation, nb_pieces
            ▼
┌──────────────────────────────────────┐
│ df_grenoble_vente (colonnes réduites)│
│ ─────────────────────                │
│ Lignes : ~2,500                      │
│ Colonnes : 6 (+ dérivées)            │
└──────────────────────────────────────┘
            │
            │ ÉTAPE 4 : Gestion des valeurs manquantes
            │ • Suppression si price = NaN (non imputable)
            │ • Suppression si surface_bati = NaN (clé)
            │ • Imputation surface_terrain = 1 si NaN (appartements)
            │ • Suppression si type_local = NaN
            ▼
┌───────────────────────────────────────┐
│ df_grenoble_vente (sans NaN critiques)│
│ ─────────────────────                 │
│ Lignes : ~2,000                       │
│ Variables clés : 100% complètes       │
└───────────────────────────────────────┘
            │
            │ ÉTAPE 5 : Normalisation des placeholders
            │ • '' → NaN
            │ • '0' → NaN si contexte illogique
            ▼
┌──────────────────────────────────────┐
│ df_grenoble_vente (normalisé)        │
└──────────────────────────────────────┘
            │
            │ ÉTAPE 6 : Filtrage des aberrations
            │ • price ≤ 0 → suppression
            │ • price > 5M€ → suppression (vraisemblablement erreur)
            │ • surface_bati ≤ 0 → suppression
            │ • surface_bati > 500 m² → vérification manuelle
            ▼
┌──────────────────────────────────────┐
│ df_grenoble_vente (plausible)        │
│ ─────────────────────                │
│ Lignes : ~1,800                      │
│ Prix : [5k€ - 1M€]                   │
│ Surfaces : [10m² - 300m²]            │
└──────────────────────────────────────┘
            │
            │ ÉTAPE 7 : Ingénierie de features
            │ • Extraction mois depuis date_mutation
            │ • Création variable 'date' (1-12)
            ▼
┌───────────────────────────────────────┐
│ df_grenoble_vente (features enrichies)│
└───────────────────────────────────────┘
            │
            │ ÉTAPE 8 : Détection des outliers (IQR)
            │ Pour chaque type_local :
            │   Q1 = quantile(0.25)
            │   Q3 = quantile(0.75)
            │   IQR = Q3 - Q1
            │   outlier si price < Q1 - 1.5×IQR
            │           ou price > Q3 + 1.5×IQR
            ▼
┌──────────────────────────────────────┐
│ df_grenoble_vente (sans outliers)    │
│ ─────────────────────                │
│ Lignes : ~1,288                      │
│ Distribution : normale par type      │
└──────────────────────────────────────┘
            │
            │ ÉTAPE 9 : Encodage catégories
            │ • Appartement → type_local_1234 = 1
            │ • Maison → 2
            │ • Local industriel/commercial → 3
            │ • Dépendance → 4
            ▼
┌──────────────────────────────────────┐
│ df_grenoble_vente (encodé)           │
│ ─────────────────────                │
│ Colonnes finales : 7                 │
│   1. price (float)                   │
│   2. type_local (str)                │
│   3. surface_bati (float)            │
│   4. surface_terrain (float)         │
│   5. date (int 1-12)                 │
│   6. nb_pieces (int)                 │
│   7. type_local_1234 (int)           │
└──────────────────────────────────────┘
            │
            │ ÉTAPE 10 : Export
            │ df.to_csv('PreprocessedData/df_grenoble_vente.csv')
            ▼
OUTPUT
┌──────────────────────────────────────┐
│ df_grenoble_vente.csv                │
│ ─────────────────────                │
│ Observations : 1,288                 │
│ Variables : 7                        │
│ Format : CSV (séparateur virgule)    │
│ Prêt pour : Modélisation             │
└──────────────────────────────────────┘
```

##  Schémas Explicatifs

### Schéma 1 : Architecture des Données DVF

```
┌─────────────────────────────────────────────────────────────────┐
│              STRUCTURE DES DONNÉES DVF (BRUTES)                 │
└─────────────────────────────────────────────────────────────────┘

        ┌──────────────────────────────────────────────┐
        │ ValeursFoncieres-2025-S1.txt                 │
        │ ──────────────────────────────────           │
        │ Format : Texte délimité par |                │
        │ 1 ligne = 1 ligne de disposition DVF         │
        └──────────────────────────────────────────────┘
                           │
        ┌──────────────────┴───────────────────────────┐
        │       43 COLONNES (variables DVF)            │
        └──────────────────────────────────────────────┘
                           │
        ┌──────────────────┴────────────────────────────────────────┐
        │                         │                                 │
        ▼                         ▼                                 ▼
┌────────────────┐      ┌───────────────┐              ┌───────────────────┐
│ IDENTIFICATION │      │ GÉOGRAPHIQUES │              │  CARACTÉRISTIQUES │
│ ─────────────  │      │ ───────────── │              │ ────────────────  │
│ • id_mutation  │      │ • code_commune│              │ • nature_mutation │
│ • numero_dispo │      │ • nom_commune │              │   (Vente, Don...) │
│ • date_mutation│      │ • code_postal │              │ • valeur_fonciere │
│                │      │ • section     │              │   (= price)       │
│                │      │ • numero_plan │              │ • type_local      │
│                │      │               │              │ • surface_reelle_ │
│                │      │               │              │   bati (m²)       │
│                │      │               │              │ • nombre_pieces_  │
│                │      │               │              │   principales     │
│                │      │               │              │ • surface_terrain │
└────────────────┘      └───────────────┘              └───────────────────┘

Note : Chaque mutation peut concerner plusieurs lots (appartements dans 
       un même immeuble). D'où la présence de plusieurs lignes avec même 
       id_mutation mais dispositions différentes.
```

### Schéma 2 : Pipeline de Nettoyage (Vue Systémique)

```
┌────────────────────────────────────────────────────────────────────────┐
│                    PIPELINE DE NETTOYAGE (Vue Systémique)              │
└────────────────────────────────────────────────────────────────────────┘

[INPUT]  Fichier brut
         1,387,077 lignes × 43 colonnes
                │
                │  PHASE 1 : FILTRAGE
                │  ═══════════════════
                ▼
         ┌──────────────────┐
         │ Filtre Géographique│──────────► Garde : code_commune = '38185'
         └──────────────────┘            Rejette : 1,372,077 lignes (98.9%)
                │
                ▼
         15,000 lignes (Grenoble)
                │
                │
         ┌──────────────────┐
         │ Filtre par Nature│────────────► Garde : nature_mutation = 'Vente'
         └──────────────────┘             Rejette : Donations, Échanges
                │
                ▼
         2,500 lignes (Ventes)
                │
                │  PHASE 2 : SÉLECTION DE VARIABLES
                │  ═══════════════════════════════════
                ▼
         ┌──────────────────┐
         │ Projection       │────────────► Garde : 6 colonnes pertinentes
         │ (colonnes)       │              Rejette : 37 colonnes administratives
         └──────────────────┘
                │
                ▼
         2,500 lignes × 6 colonnes
                │
                │  PHASE 3 : GESTION DES MANQUANTS
                │  ═══════════════════════════════════
                ▼
         ┌──────────────────┐
         │ Imputation       │────────────► surface_terrain NaN → 1.0
         │                  │              (appartements sans terrain)
         └──────────────────┘
                │
         ┌──────────────────┐
         │ Suppression      │────────────► Lignes avec NaN sur price,
         │ (listwise)       │              type_local, surface_bati
         └──────────────────┘
                │
                ▼
         2,000 lignes
                │
                │  PHASE 4 : VALIDATION & CONTRAINTES
                │  ═══════════════════════════════════════
                ▼
         ┌──────────────────┐
         │ Contraintes      │────────────► Rejette :
         │ logiques         │              • price ≤ 0 ou > 5M€
         └──────────────────┘              • surface_bati ≤ 0 ou > 500m²
                │
                ▼
         1,800 lignes
                │
                │  PHASE 5 : DÉTECTION DES OUTLIERS
                │  ═══════════════════════════════════
                ▼
         ┌──────────────────┐
         │ Méthode IQR      │────────────► Par groupe (type_local)
         │ (par type)       │              Rejette extrêmes (±1.5 IQR)
         └──────────────────┘
                │
                ▼
         1,288 lignes
                │
                │  PHASE 6 : ENRICHISSEMENT
                │  ═══════════════════════════
                ▼
         ┌──────────────────┐
         │ Feature          │────────────► Ajout :
         │ Engineering      │              • date (mois extrait)
         └──────────────────┘              • type_local_1234 (encodage)
                │
                ▼
         1,288 lignes × 7 colonnes
                │
                │
[OUTPUT] df_grenoble_vente.csv
```

### Schéma 3 : Détection des Outliers (Boxplot Conceptuel)

```
┌────────────────────────────────────────────────────────────────────┐
│          MÉTHODE IQR : DÉTECTION DES VALEURS ABERRANTES             │
└────────────────────────────────────────────────────────────────────┘

   Distribution des prix (exemple : Appartements)

   OUTLIERS
     (sup)
       │
       •  ← 650k€ (transaction exceptionnelle)
       •  ← 580k€
       │
       │
    ───┼─────────────────────────────────────  ← Upper Fence
       │                                         (Q3 + 1.5×IQR)
       │                                         = 520k€
       │
       │  ┌─────────────────────────────────┐
       │  │█████████████████████████████████│ ← Whisker (max dans limites)
       │  └─────────────────────────────────┘   = 480k€
       │
       │  ┌─────────────────────────────────┐
       │  │█████████████████████████████████│
   Q3 ─┼──┤█████████████████████████████████├── 75% des données
       │  │█████████████████████████████████│
       │  │█████████████████████████████████│
       │  │                                 │
       │  │         BOÎTE (IQR)             │
       │  │                                 │
  MED ─┼──┼─────────────────────────────────┤── 50% (médiane)
       │  │                                 │   = 200k€
       │  │█████████████████████████████████│
       │  │█████████████████████████████████│
   Q1 ─┼──┤█████████████████████████████████├── 25% des données
       │  │█████████████████████████████████│   = 120k€
       │  └─────────────────────────────────┘
       │
       │  ┌─────────────────────────────────┐
       │  │█████████████████████████████████│ ← Whisker (min dans limites)
       │  └─────────────────────────────────┘   = 60k€
       │
    ───┼─────────────────────────────────────  ← Lower Fence
       │                                         (Q1 - 1.5×IQR)
       │                                         = -120k€ → 0€ (borne physique)
       │
   OUTLIERS
     (inf)
       •  ← 15k€ (garage vendu seul, ou erreur)
       │

Légende :
  Q1 = Premier quartile (25e percentile)
  Q3 = Troisième quartile (75e percentile)
  IQR = Q3 - Q1 (Interquartile Range)
  Whiskers = Extensions jusqu'aux valeurs extrêmes NON-outliers
  • = Outliers détectés (en dehors des fences)

Action : Les outliers sont retirés du dataset final
```

### Schéma 4 : Workflow Décisionnel (Valeurs Manquantes)

```
┌────────────────────────────────────────────────────────────────┐
│      ARBRE DE DÉCISION : GESTION DES VALEURS MANQUANTES        │
└────────────────────────────────────────────────────────────────┘

                        Variable avec NaN détectée
                                    │
                  ┌─────────────────┴─────────────────┐
                  ▼                                   ▼
       Variable CLEF ?                       Variable SECONDAIRE ?
       (price, surface_bati,                 (surface_terrain,
        type_local)                          date_mutation)
                  │                                   │
         ┌────────┴────────┐                ┌────────┴────────┐
         ▼                 ▼                ▼                 ▼
   % NaN < 5% ?     % NaN ≥ 5% ?     % NaN < 30% ?    % NaN ≥ 30% ?
         │                 │                │                 │
         │                 │                │                 │
         ▼                 ▼                ▼                 ▼
   ┌──────────┐     ┌──────────┐     ┌───────────┐     ┌──────────┐
   │ Supprimer│     │ Supprimer│     │  Imputer  │     │ Supprimer│
   │ la ligne │     │ la ligne │     │  (médiane │     │la colonne│
   │ (listwise│     │ (listwise│     │  ou valeur│     │          │
   │ deletion)│     │ deletion)│     │par défaut)│     │          │
   └──────────┘     └──────────┘     └───────────┘     └──────────┘
         │                 │                │                 │
         └─────────────────┴────────────────┴─────────────────┘
                                    │
                                    ▼
                      Dataset nettoyé (sans NaN critiques)

Exemples appliqués à notre dataset :
────────────────────────────────────────────────────────────────
1. price NaN → SUPPRIMER (variable clef, non imputable)
2. surface_bati NaN → SUPPRIMER (clef, corrélée avec prix)
3. surface_terrain NaN (appartements) → IMPUTER à 1 m²
   (valeur symbolique, non-clef, 60% NaN)
4. type_local NaN → SUPPRIMER (clef catégorielle, pas de mode évident)
5. date_mutation NaN → SUPPRIMER (peu de cas, clef temporelle)
```

##  Données en Sortie : df_grenoble_vente.csv

### Dictionnaire des Variables

| Variable | Type | Description | Unité | Valeurs | Statistiques |
|----------|------|-------------|-------|---------|--------------|
| **price** | `float` | Prix de vente (valeur foncière) | Euros (€) | [5,000 - 950,000] | μ = 245k, σ = 125k |
| **type_local** | `object` (string) | Type de bien immobilier | Catégoriel | {Appartement, Maison, Local industriel..., Dépendance} | 85% Appartements |
| **surface_bati** | `float` | Surface bâtie (loi Carrez) | m² | [10 - 300] | μ = 62, σ = 35 |
| **surface_terrain** | `float` | Surface de terrain privé | m² | [1 - 5000] | μ = 150, médiane = 1 |
| **date** | `int` | Mois de la transaction | 1-12 | [1=Jan, 12=Déc] | Uniform (données S1) |
| **nb_pieces** | `int` | Nombre de pièces principales | Entier ≥ 0 | [0 - 8] | μ = 2.5, médiane = 2 |
| **type_local_1234** | `int` | Encodage numérique de type_local | 1-4 | 1=Appart, 2=Maison, 3=Local, 4=Dép | - |

### Type de Données

**Classification** : **Cross-section** (coupe transversale)

- **Définition** : Observations de multiples unités (ici, transactions immobilières) à un instant donné (ou période courte)
- **Caractéristiques** :
  - Pas de dimension temporelle exploitable (6 mois seulement)
  - Indépendance des observations (chaque transaction est unique)
  - Pas d'autocorrélation temporelle (contrairement aux séries temporelles)


### Unités et Périodes

- **Unité d'observation** : Une transaction immobilière (mutation)
- **Unité géographique** : Commune de Grenoble (38185)
- **Période** : 1er semestre 2025 (janvier-juin)



### Nettoyage Appliqué

**Résumé des opérations** :
1. Filtrage géographique : 1,387,077 → 15,000 obs
2. Filtrage par nature : 15,000 → 2,500 obs (ventes uniquement)
3. Suppression des NaN critiques : 2,500 → 2,000 obs
4. Filtrage des aberrations (prix/surfaces illogiques) : 2,000 → 1,800 obs
5. Retrait des outliers (IQR par type) : 1,800 → 1,288 obs

**Taux de rétention** : 1,288 / 15,000 = **8.6%**
- Peut sembler faible, mais normal :
  - Beaucoup de donations/échanges (non pertinents)
  - Valeurs manquantes sur variables clés
  - Erreurs de saisie fréquentes dans bases administratives
    
### Sources et Traçabilité

**Source primaire** : 
- Base DVF (Demandes de Valeurs Foncières)
- Producteur : Direction Générale des Finances Publiques (DGFiP)
- Mise à disposition : Etalab / data.gouv.fr
- Licence : Licence Ouverte / Open Licence
- URL : https://www.data.gouv.fr/fr/datasets/demandes-de-valeurs-foncieres/

**Processus de collecte** :
1. Actes notariés de mutation immobilière
2. Enregistrement aux services de publicité foncière
3. Extraction par la DGFiP (semi-annuelle)
4. Anonymisation (suppression des noms, adresses précises)
5. Publication en open data



##  Explication Ligne par Ligne du code avec justification (Extraits Clés)

### Section 2 : Chargement des Données

```python
# Ligne 77
df = pd.read_csv("ValeursFoncieres-2025-S1.txt", sep="|")
```

**Explication** :
- `pd.read_csv()` : Fonction pandas pour lire un fichier délimité
- `sep="|"` : Le fichier DVF utilise le pipe comme séparateur (non standard mais fréquent dans les bases administratives)

**Avertissement** :
```
DtypeWarning: Columns (18,23,24,26,28,29,31,33) have mixed types.
```
- **Cause** : Certaines colonnes contiennent à la fois des nombres et du texte (ex: code postal comme '38000' ou 'Non renseigné')
- **Conséquence** : pandas infère le type `object` (string) par sécurité
- **Solution** : Conversion manuelle des types après filtrage (cf. section 9)

### Section 4 : Filtrage Géographique

```python
# Filtrer Grenoble (code INSEE 38185)
df_grenoble = df[df['code_commune'] == '38185'].copy()
```

**Explication** :
- **Masque booléen** : `df['code_commune'] == '38185'` retourne un vecteur de True/False
- `.copy()` : Crée une copie indépendante (évite les SettingWithCopyWarning de pandas)
- **38185** : Code INSEE officiel de Grenoble
  - Format : 38 (Isère) + 185 (numéro commune)
  - Invariant dans le temps (contrairement au nom qui peut changer)


### Section 7 : Diagnostic des Valeurs Manquantes

```python
# Calcul du pourcentage de NaN par colonne
missing_pct = df_grenoble_vente.isna().sum() / len(df_grenoble_vente) * 100
```

**Explication ligne par ligne** :
1. `df.isna()` : Retourne un DataFrame de booléens (True si NaN)
2. `.sum()` : Compte les True (par défaut sur l'axe 0, c.-à-d. par colonne)
3. `/ len(df)` : Division par le nombre total de lignes
4. `* 100` : Conversion en pourcentage

**Décision de suppression/imputation** :
- **> 80% missing** : Supprimer la colonne (information insuffisante)
- **30-80%** : Imputation si possible (médiane, mode, régression)
- **< 30%** : Suppression des lignes ou imputation selon contexte

**Exemple pour surface_terrain** :
- ~60% de NaN pour les appartements (normal : pas de terrain privé)
- Solution : Imputer à 1 m² (valeur symbolique pour ne pas perdre les observations)

### Section 9 : Nettoyage des Aberrations

```python
# Filtrer les prix plausibles
df_clean = df[(df['price'] > 0) & (df['price'] < 5_000_000)]
```

**Justifications** :
- `price > 0` : Évident (prix négatifs ou nuls = erreur de saisie)
- `price < 5M€` : 
  - Seuil arbitraire mais raisonnable pour Grenoble
  - Évite les erreurs de saisie (ex: 50M€ au lieu de 50k€)
  - Les biens > 5M€ à Grenoble sont exceptionnels (châteaux, immeubles entiers)

**Analyse des valeurs extrêmes retirées** :
```python
outliers = df[df['price'] > 5_000_000]
print(outliers[['price', 'type_local', 'surface_bati']])
```
→ Permet de vérifier si on retire des observations légitimes ou des erreurs

### Section 12 : Détection des Outliers (IQR)

```python
def remove_outliers_by_type(df, column='price', types_col='type_local'):
    """Supprime les outliers via méthode IQR, par groupe."""
    clean_df = pd.DataFrame()
    
    for type_local in df[types_col].unique():
        subset = df[df[types_col] == type_local]
        
        # Calcul des quartiles
        Q1 = subset[column].quantile(0.25)
        Q3 = subset[column].quantile(0.75)
        IQR = Q3 - Q1
        
        # Bornes de Tukey
        lower_bound = Q1 - 1.5 * IQR
        upper_bound = Q3 + 1.5 * IQR
        
        # Filtrage
        filtered = subset[
            (subset[column] >= lower_bound) & 
            (subset[column] <= upper_bound)
        ]
        
        clean_df = pd.concat([clean_df, filtered])
    
    return clean_df
```

**Explication de la Méthode IQR** :

La méthode IQR (InterQuartile Range) est une technique statistique robuste pour détecter les valeurs aberrantes.

**Formules** :
```
Q1 = 25e percentile = quantile(0.25)
Q3 = 75e percentile = quantile(0.75)
IQR = Q3 - Q1  (étendue interquartile)

Bornes de détection :
  Lower bound = Q1 - 1.5 × IQR
  Upper bound = Q3 + 1.5 × IQR

Outlier si :
  x < Lower bound  OU  x > Upper bound
```

**Pourquoi 1.5 ?** :
- Règle empirique de Tukey (statisticien John Tukey, 1977)
- Compromis : pas trop strict (ne retire pas trop de données), pas trop laxiste
- Pour une distribution normale : ~0.7% de la population est considérée outlier

**Pourquoi par type de bien ?** :
- Un appartement à 800k€ peut être normal (grand, centre-ville)
- Une dépendance à 800k€ est aberrant
- Chaque type a sa propre distribution de prix

**Exemple numérique** :
```
Appartements à Grenoble :
  Q1 = 120,000 €
  Q3 = 280,000 €
  IQR = 160,000 €
  
  Lower = 120,000 - 1.5×160,000 = -120,000 € → 0 € (borne min physique)
  Upper = 280,000 + 1.5×160,000 = 520,000 €
  
  → Tout appartement > 520k€ est considéré outlier
```

**Visualisation** :
```
    Boxplot de la distribution des prix (Appartements)

     outlier →  •                                    • ← outlier
                │                                    │
                │                                    │
        max →   ├────────────────────────────────────┤  ← upper bound (520k€)
                │xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx│
                │xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx│
         Q3 →   ├────────────────────────────────────┤  (280k€)
                │xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx│
    médiane →   ├────────────────────────────────────┤  (200k€)
                │xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx│
         Q1 →   ├────────────────────────────────────┤  (120k€)
                │xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx│
        min →   ├────────────────────────────────────┤  ← lower bound (0€)
                │
                │
     outlier →  •

                      Distribution des prix
```

### Section 14 : Encodage de type_local

```python
# Mapping des catégories vers codes numériques
type_mapping = {
    'Appartement': 1,
    'Maison': 2,
    'Local industriel. commercial ou assimilé': 3,
    'Dépendance': 4
}

df_grenoble_vente['type_local_1234'] = df_grenoble_vente['type_local'].map(type_mapping)
```

**Pourquoi encoder ?** :
- Les algorithmes ML (Random Forest, régression) nécessitent des variables numériques
- Alternative : One-Hot Encoding (variable binaire par catégorie)
  - Exemple : type_appart=1, type_maison=0, type_local=0, type_dep=0
  - Avantage : pas d'ordre implicite

**Ordre choisi** :
- 1 = Appartement (le plus fréquent, référence)
- 2 = Maison (valorisation généralement supérieure)
- 3 = Local commercial (logique économique différente)
- 4 = Dépendance (valorisation faible)


### Section 15 : Matrice de Corrélation

```python
corr = df_grenoble_vente.corr(numeric_only=True)
sns.heatmap(corr, annot=True, cmap="YlGnBu")
plt.title("Correlation Heatmap of Grenoble_Vente Dataset")
plt.show()
```

**Interprétation des Corrélations** :

Une corrélation mesure la relation linéaire entre deux variables :
```
corr(X, Y) = Cov(X,Y) / (σ_X × σ_Y)

où :
  Cov(X,Y) = E[(X - μ_X)(Y - μ_Y)]  (covariance)
  σ_X, σ_Y = écarts-types
```

**Valeurs** :
- `+1` : Corrélation positive parfaite (si X↑ alors Y↑ proportionnellement)
- `0` : Pas de relation linéaire
- `-1` : Corrélation négative parfaite (si X↑ alors Y↓)

**Seuils d'interprétation** (règles empiriques) :
- `|r| < 0.3` : Corrélation faible
- `0.3 ≤ |r| < 0.7` : Corrélation modérée
- `|r| ≥ 0.7` : Corrélation forte

**Attendu pour notre dataset** :
- `corr(price, surface_bati)` ≈ 0.75 (forte et positive)
- `corr(surface_bati, nb_pieces)` ≈ 0.80 (multicolinéarité !)
- `corr(price, date)` ≈ 0.05 (faible car courte période)

**Problème de Multicolinéarité** :
Si deux variables X1 et X2 sont fortement corrélées (|r| > 0.9) :
- Instabilité des coefficients β̂
- Variance élevée des estimateurs
- Difficultés d'interprétation (effet propre de X1 vs X2 indissociable)

**Solutions** :
1. Retirer une des deux variables (ex: garder surface_bati, retirer nb_pieces)
2. Créer une variable composite (ex: surface_par_piece = surface_bati / nb_pieces)


### PArtie technique et informatique (Packages Utilisés et Justification)

#### Packages Principaux

1. **pandas** (`import pandas as pd`)
   - **Rôle** : Manipulation de données tabulaires
   - **Fonctions clés** :
     - `pd.read_csv()` : Lecture des fichiers DVF
     - `df.groupby()` : Agrégations par groupe
     - `df.isna()` : Détection des valeurs manquantes
     - `df.to_csv()` : Export des résultats
   - **Pourquoi** : Standard de facto pour l'analyse de données en Python

2. **numpy** (`import numpy as np`)
   - **Rôle** : Calculs numériques et algèbre linéaire
   - **Fonctions clés** :
     - `np.log()` : Transformations logarithmiques
     - `np.std()` : Écart-type
     - `np.percentile()` : Calcul des quantiles
   - **Pourquoi** : Opérations vectorisées (100× plus rapides que boucles Python)

3. **matplotlib.pyplot** (`import matplotlib.pyplot as plt`)
   - **Rôle** : Visualisations statiques
   - **Fonctions clés** :
     - `plt.figure()` : Création de graphiques
     - `plt.hist()` : Histogrammes
     - `plt.show()` : Affichage
   - **Pourquoi** : Contrôle fin des graphiques

4. **seaborn** (`import seaborn as sns`)
   - **Rôle** : Visualisations statistiques de haut niveau
   - **Fonctions clés** :
     - `sns.heatmap()` : Matrice de corrélation
     - `sns.boxplot()` : Détection des outliers
     - `sns.set_style()` : Thème graphique
   - **Pourquoi** : Graphiques prêts à publier, intégration avec pandas

#### Packages Utilitaires

5. **re** (expressions régulières)
   - **Rôle** : Nettoyage de texte (normalisation des strings)
   
6. **typing** (annotations de type)
   - **Rôle** : Documentation du code, auto-complétion IDE

7. **unicodedata**
   - **Rôle** : Normalisation des caractères accentués
   - **Exemple** : "Appartement" et "Appartement" (Unicode différent) → harmonisation

8. **textwrap**
   - **Rôle** : Formatage des légendes de graphiques

9. **pathlib.Path**
   - **Rôle** : Gestion des chemins de fichiers (cross-platform)




---

**Version** : 1.0  
**Dernière mise à jour** : 13 novembre 2025   
