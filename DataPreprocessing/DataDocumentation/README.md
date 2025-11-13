# DataDocumentation — Documentation Officielle DVF

## 📋 Résumé Global

Ce dossier contient la **documentation officielle** fournie par la Direction Générale des Finances Publiques (DGFiP) concernant la base de données DVF (Demandes de Valeurs Foncières). Ces documents sont essentiels pour comprendre la structure des données, leur signification juridique, et les conditions d'utilisation.

### Rôle dans le Projet

La documentation DVF joue un rôle **fondamental** :

1. **Dictionnaire de données** : Définitions précises des 43 variables DVF
2. **Contexte juridique** : Origines des données (actes notariés, publicité foncière)
3. **Limites** : Avertissements sur les données manquantes, erreurs possibles
4. **Conformité RGPD** : Informations sur l'anonymisation et la protection des données
5. **Conditions d'usage** : Licence ouverte, restrictions éventuelles

### Pourquoi C'est Utile

- **Éviter les erreurs d'interprétation** : Une variable mal comprise = analyse biaisée
- **Justifier les choix méthodologiques** : Ex: "Pourquoi imputer surface_terrain à 1 pour les appartements ?"
- **Citer les sources** : Transparence et reproductibilité scientifique
- **Conformité légale** : Respecter les CGU et le RGPD

## 📂 Contenu du Dossier

```
DataDocumentation/
│
├── README.md (ce fichier)
│
├── notice-descriptive-du-fichier-dvf-20221017.pdf
│   │ Titre : Notice descriptive du fichier DVF
│   │ Date : 17 octobre 2022
│   │ Pages : ~15
│   │ Contenu :
│   │   • Structure du fichier (format, séparateurs)
│   │   • Dictionnaire des variables (nom, type, description)
│   │   • Processus de collecte des données
│   │   • Avertissements et limites
│   │   • Exemples d'utilisation
│   │
│
├── faq-20221017.pdf
│   │ Titre : Foire Aux Questions (FAQ) DVF
│   │ Date : 17 octobre 2022
│   │ Pages : ~8
│   │ Contenu :
│   │   • Questions fréquentes des utilisateurs
│   │   • Cas particuliers (ventes multiples, divisions)
│   │   • Différences entre valeur foncière et prix de vente
│   │   • Absence de données pour certaines zones
│   │   • Contact pour support technique
│   │
│
├── conditions-generales-dutilisation-20201016.pdf
│   │ Titre : Conditions Générales d'Utilisation (CGU)
│   │ Date : 16 octobre 2020
│   │ Pages : ~5
│   │ Contenu :
│   │   • Licence ouverte Etalab 2.0
│   │   • Droits de réutilisation (libre, y compris commercial)
│   │   • Obligations (citation de la source)
│   │   • Exclusions de garantie
│   │   • Responsabilité limitée de l'État
│   │
│
└── information-des-personnes-concernees-par-le-traitement-informatique-20201016.pdf
    │ Titre : Information RGPD
    │ Date : 16 octobre 2020
    │ Pages : ~3
    │ Contenu :
    │   • Finalité du traitement (transparence du marché immobilier)
    │   • Base légale (article 6 RGPD)
    │   • Données anonymisées (pas de noms, adresses précises)
    │   • Durée de conservation
    │   • Droits des personnes (accès, rectification, effacement)
    │   • Contact du DPO (Délégué à la Protection des Données)
```

## 📖 Résumé des Documents

### 1. Notice Descriptive (notice-descriptive-du-fichier-dvf-20221017.pdf)

**Document principal** pour comprendre les données DVF.

#### Variables Clés Expliquées

| Variable DVF | Description Officielle | Utilisation dans notre projet |
|--------------|------------------------|-------------------------------|
| **id_mutation** | Identifiant unique de la mutation | Pas utilisé (on agrège au niveau transaction) |
| **date_mutation** | Date de la vente (format AAAA-MM-JJ) | Transformé en variable `date` (mois 1-12) |
| **valeur_fonciere** | Montant de la transaction en euros | Variable cible `price` |
| **type_local** | Nature du bien (Appartement, Maison, etc.) | Gardé tel quel + encodé en `type_local_1234` |
| **surface_reelle_bati** | Surface bâtie (loi Carrez) en m² | Renommé `surface_bati` |
| **nombre_pieces_principales** | Nombre de pièces (hors cuisine, SdB) | Renommé `nb_pieces` |
| **surface_terrain** | Surface du terrain en m² | Gardé tel quel, imputé à 1 si NaN (appartements) |
| **code_commune** | Code INSEE de la commune | Utilisé pour filtrer Grenoble (38185) |
| **nature_mutation** | Type d'acte (Vente, Donation, Échange, etc.) | Filtré sur 'Vente' uniquement |

#### Avertissements Importants (extrait de la notice)

1. **Transactions complexes** :
   > "Une mutation peut concerner plusieurs biens (lots). Dans ce cas, la valeur foncière est la somme totale, non ventilée par lot."
   
   **Impact** : On peut avoir plusieurs lignes avec le même `id_mutation` mais des surfaces différentes (ex: vente d'un immeuble avec plusieurs appartements).
   
   **Solution dans notre projet** : On garde les lignes individuelles (niveau disposition) car chacune représente un bien physique.

2. **Valeurs manquantes** :
   > "Les informations relatives aux locaux (surface, nombre de pièces) ne sont pas toujours renseignées, notamment pour les ventes anciennes ou atypiques."
   
   **Impact** : 30-60% de NaN sur certaines variables.
   
   **Solution** : Suppression listwise si variable clef (price, surface_bati), imputation sinon.

3. **Erreurs de saisie** :
   > "La base DVF est construite à partir de déclarations notariales. Des erreurs de saisie sont possibles (inversions de chiffres, unités incorrectes)."
   
   **Impact** : Prix aberrants (ex: 50M€ au lieu de 50k€), surfaces incohérentes.
   
   **Solution** : Filtrage des outliers par méthode IQR, bornes de plausibilité.

4. **Exclusions géographiques** :
   > "Les départements d'Alsace-Moselle (67, 68, 57) ne sont pas couverts en raison d'un régime juridique différent."
   
   **Impact** : Pas de données pour Strasbourg, Mulhouse, Metz.
   
   **Impact sur notre projet** : Aucun (Grenoble = 38).

#### Processus de Collecte

```
┌─────────────────────────────────────────────────────────────────┐
│          CHAÎNE DE PRODUCTION DES DONNÉES DVF                    │
└─────────────────────────────────────────────────────────────────┘

[1] Transaction Immobilière
      │
      │ Acte notarié signé chez le notaire
      │ (contient : prix, adresse, surfaces, parties, etc.)
      ▼
[2] Enregistrement aux Services de Publicité Foncière
      │
      │ Le notaire transmet l'acte à l'administration fiscale
      │ Paiement des droits de mutation (taxe)
      ▼
[3] Saisie dans le Système MAJIC (DGFiP)
      │
      │ Extraction des informations pertinentes
      │ Codification (type de bien, nature de mutation)
      ▼
[4] Consolidation Nationale
      │
      │ Agrégation des données de tous les départements
      │ Contrôles de cohérence basiques
      ▼
[5] Anonymisation (RGPD)
      │
      │ Suppression : noms des parties, adresses précises
      │ Conservation : surfaces, prix, commune, date
      ▼
[6] Publication en Open Data (data.gouv.fr)
      │
      │ Format TXT délimité par |
      │ Mise à jour semestrielle (S1, S2)
      ▼
[7] Téléchargement par les Utilisateurs
      │
      │ Chercheurs, collectivités, entreprises, particuliers
      │ Utilisation : observatoires fonciers, modèles, études
      ▼
[8] Notre Projet : Analyse Économétrique
```

**Délai** : Environ 6 mois entre la transaction réelle et la disponibilité en open data.

### 2. FAQ (faq-20221017.pdf)

**Questions Fréquentes** (sélection pertinente) :

#### Q1 : Quelle est la différence entre "valeur foncière" et "prix de vente" ?

**Réponse** : 
- **Valeur foncière** = Montant déclaré dans l'acte notarié (base DVF)
- **Prix de vente** = Somme effectivement payée par l'acheteur

Généralement identiques, mais peuvent différer si :
- Vente incluant des meubles (non foncier)
- Arrangements particuliers (viager, etc.)
- Erreurs de déclaration

**Impact** : Considérer `valeur_fonciere` comme proxy du prix de vente (approximation acceptable).

#### Q2 : Pourquoi certaines communes n'ont pas de données ?

**Réponse** :
- Volume de transactions trop faible (< 10/an)
- Données confidentielles pour préserver l'anonymat dans les petites communes
- Départements exclus (Alsace-Moselle)

**Impact** : Pas de problème pour Grenoble (grande ville, nombreuses transactions).

#### Q3 : Comment interpréter une mutation avec plusieurs lignes ?

**Réponse** :
Une ligne = une disposition = un local spécifique
Plusieurs lignes avec même `id_mutation` = vente groupée (ex: immeuble)

**Exemple** :
```
id_mutation | valeur_fonciere | type_local    | surface_bati
─────────────────────────────────────────────────────────────
12345ABC    | 500,000         | Appartement   | 60
12345ABC    | 500,000         | Appartement   | 45
12345ABC    | 500,000         | Dépendance    | 10
```
→ Vente d'un ensemble immobilier pour 500k€ (2 appartements + 1 cave)

**Traitement dans notre projet** :
- On garde chaque ligne (représente un bien physique)
- La valeur foncière est répétée (non ventilée par bien)
- Peut créer un biais : on "compte plusieurs fois" certaines transactions

**Solution alternative** : Agréger au niveau `id_mutation` (une ligne par transaction), mais perte d'information sur les biens individuels.

### 3. Conditions Générales d'Utilisation (CGU)

**Licence** : **Licence Ouverte / Open Licence Etalab 2.0**

#### Droits Accordés

✅ **Reproduction** : Libre  
✅ **Modification** : Libre (agrégations, calculs, visualisations)  
✅ **Diffusion** : Libre (publications, applications web, etc.)  
✅ **Usage commercial** : Autorisé (startups PropTech, agences immobilières)

#### Obligations

📌 **Citation de la source** : Obligatoire
```
Source : Direction Générale des Finances Publiques (DGFiP)
Données DVF (Demandes de Valeurs Foncières)
Disponibles sur data.gouv.fr
```

📌 **Mention de la licence** : Recommandée
```
Licence Ouverte / Open Licence Etalab 2.0
```

📌 **Date d'extraction** : Bonne pratique
```
Données extraites le : 13 novembre 2025
Période couverte : 1er semestre 2025
```

#### Exclusions de Garantie

⚠️ L'État ne garantit pas :
- L'exactitude des données (erreurs de saisie possibles)
- L'exhaustivité (transactions manquantes)
- La disponibilité permanente du service

→ L'utilisateur assume la responsabilité de la validation des données.

### 4. Information RGPD

**Conformité** : Règlement Général sur la Protection des Données (RGPD)

#### Finalité du Traitement

> "Assurer la transparence du marché immobilier et fournir des données statistiques pour l'analyse économique."

#### Base Légale

Article 6.1.e du RGPD : Mission d'intérêt public

#### Données Personnelles

**Collectées initialement** :
- Noms et prénoms des acheteurs/vendeurs
- Adresses complètes des biens

**Publiées (anonymisées)** :
- ✅ Prix, surfaces, types de biens
- ✅ Commune (niveau agrégé)
- ✅ Date de transaction (mois/année)
- ❌ Noms des parties (supprimés)
- ❌ Adresses précises (arrondies au code postal)

#### Droits des Personnes

Les personnes concernées (acheteurs/vendeurs) peuvent exercer :
- **Droit d'accès** : Consulter leurs propres données
- **Droit de rectification** : Corriger les erreurs
- **Droit d'effacement** : Demander la suppression (sous conditions)

**Contact** : Délégué à la Protection des Données (DPO) de la DGFiP

#### Impact pour Notre Projet

✅ **Pas de données personnelles** : Les données publiées sont anonymes  
✅ **Usage légal** : Recherche académique couverte par la licence ouverte  
✅ **Pas de ré-identification** : Ne pas croiser avec d'autres sources pour retrouver les personnes

## 🔍 Utilisation des Documents dans l'Analyse

### Lors du Preprocessing

1. **Consulter la notice** pour comprendre chaque variable :
   - Type (numérique, catégorielle)
   - Unité (euros, m², nombre)
   - Valeurs possibles (ex: nature_mutation ∈ {Vente, Donation, Échange, ...})

2. **Vérifier la FAQ** en cas d'anomalie :
   - Ex: Pourquoi `valeur_fonciere` = 0 pour certaines lignes ?
     → Réponse : Donations (prix = 0)

3. **Respecter les CGU** :
   - Citer la source dans les publications
   - Mentionner les limites des données

### Lors de la Rédaction du Rapport

**Section "Données"** :
```markdown
### Source des Données

Les données proviennent de la base DVF (Demandes de Valeurs Foncières) 
mise à disposition par la Direction Générale des Finances Publiques (DGFiP) 
via data.gouv.fr. Cette base recense l'ensemble des mutations immobilières 
enregistrées par les services de publicité foncière.

**Fichier utilisé** : ValeursFoncieres-2025-S1.txt
**Période** : 1er janvier - 30 juin 2025
**Zone** : Commune de Grenoble (INSEE 38185)

**Licence** : Licence Ouverte / Open Licence Etalab 2.0

**Documentation** : 
- Notice descriptive : DGFiP (2022)
- FAQ : DGFiP (2022)

**Limites reconnues** :
- Erreurs de saisie possibles (source notariale)
- Valeurs manquantes (~40% pour surface_terrain)
- Transactions atypiques (viager, échanges) exclues de l'analyse
```

## 🎓 Points Pédagogiques

### Pour un Étudiant

**Exercice** : Lire la notice descriptive et répondre :
1. Quelle est la différence entre `surface_reelle_bati` et `surface_terrain` ?
2. Pourquoi `nombre_pieces_principales` exclut-il la cuisine et la salle de bain ?
3. Que signifie `nature_culture` pour un terrain ?

**Discussion** : 
- Pourquoi l'anonymisation est-elle importante ?
- Quels biais peuvent résulter des données manquantes ?
- Comment vérifier la fiabilité d'une base administrative ?

### Pour un Enseignant

**Points à Souligner** :

1. **Importance de la documentation primaire** :
   - Toujours consulter la source officielle
   - Ne pas se fier uniquement aux descriptions secondaires (blogs, forums)

2. **Données administratives vs enquêtes** :
   - Avantages DVF : exhaustivité, fraîcheur, gratuité
   - Inconvénients : erreurs de saisie, variables limitées, pas de contrôle du chercheur

3. **Éthique et RGPD** :
   - Responsabilité du chercheur dans l'usage des données
   - Respect de l'anonymat (ne pas ré-identifier)

### Pour un Évaluateur

**Critères de Qualité** :

✅ **Documentation des sources** :
- Citation complète de la DGFiP
- Mention de la notice descriptive
- Date d'extraction précisée

✅ **Compréhension des limites** :
- Avertissements de la FAQ pris en compte
- Biais potentiels identifiés
- Solutions de nettoyage justifiées

✅ **Conformité légale** :
- Licence ouverte respectée
- RGPD mentionné
- Pas de diffusion de données personnelles

## 📎 Références et Liens

### Documents Officiels

- **Portail Open Data** : https://www.data.gouv.fr/fr/datasets/demandes-de-valeurs-foncieres/
- **Notice descriptive** : Disponible en téléchargement sur le portail
- **Etalab (licence)** : https://www.etalab.gouv.fr/licence-ouverte-open-licence/

### Autres Ressources DVF

- **DVF+** : Version enrichie avec géocodage (Cerema)
- **Géo-DVF** : Visualisation cartographique des transactions
- **API DVF** : Accès programmatique aux données

### Contact

**Support technique DGFiP** :  
Email : bureau.communication-dif@dgfip.finances.gouv.fr

**DPO (Protection des données)** :  
Adresse : DGFiP, 139 rue de Bercy, 75012 Paris

---

**Version** : 1.0  
**Dernière mise à jour** : 13 novembre 2025  
**Auteur** : Projet Économétrie Appliquée
