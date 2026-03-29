# 🌿 Chlordécone — Modélisation de la contamination des sols en Martinique

> *"Le chlordécone est présent dans les sols martiniquais pour plusieurs siècles. Ce projet ne résout pas ce problème — il le mesure, le cartographie, et tente de le prédire pour aider à agir."*

---

## 🧪 Contexte

Le **chlordécone** est un pesticide organochloré utilisé massivement dans les plantations bananières des Antilles françaises entre **1972 et 1993**, malgré des alertes scientifiques sur sa toxicité dès les années 1960. Classé cancérogène possible (groupe 2B, CIRC), il est aujourd'hui interdit dans la quasi-totalité des pays.

En Martinique, son usage prolongé a conduit à une **pollution durable et irréversible** des sols et des ressources alimentaires. Sa demi-vie dans les sols tropicaux est estimée à plusieurs décennies, voire plusieurs siècles selon les conditions pédologiques. Les populations locales sont exposées à des risques sanitaires graves (cancers de la prostate, perturbations endocriniennes, neurotoxicité) et dénoncent un empoisonnement systémique aggravé par des décennies d'inaction institutionnelle.

Ce projet applique des **méthodes de science des données** aux données de surveillance des sols publiées par l'État pour :
- quantifier l'étendue spatiale et l'intensité de la contamination,
- identifier les facteurs géographiques, pédologiques et agricoles qui l'expliquent,
- construire des modèles prédictifs opérationnels pour prioriser les contrôles.

---

## 📁 Structure du projet

```
chlordecone-project/
│
├── data/
│   ├── raw/                        # Données brutes (BaseCLD2026.csv)
│   └── processed/
│       ├── BaseCLD2026_clean.csv   # Après nettoyage (notebook 01)
│       └── BaseCLD2026_features.csv# Après feature engineering (notebook 02)
│
├── notebooks/
│   ├── 01_data_engineering_vf.ipynb       # Nettoyage & préparation des données
│   ├── 02_exploratory_analysis_vf.ipynb   # Analyse exploratoire (EDA)
│   └── 03_modeling_vf.ipynb               # Modélisation & cartes interactives
│
├── outputs/
│   ├── carte_risque_rf_interactive.html       # Carte de risque (3 niveaux)
│   └── carte_predictions_rf_interactive.html  # Carte prédictive (valeurs continues)
│
└── README.md
```

---

## 📊 Données

| Attribut | Valeur |
|---|---|
| **Source** | Base de données nationale des sols (BaseCLD2026) |
| **Territoire** | Martinique |
| **Période** | 2010 – 2019 |
| **Observations** | 31 126 prélèvements de sol |
| **Variables** | 26 (concentrations, coordonnées GPS, type de sol, pluviométrie, topographie, historique agricole) |
| **Seuil réglementaire** | 0,1 mg/kg (chlordécone dans les sols) |
| **Taux de contamination** | 34,6 % des observations dépassent le seuil |
| **Valeurs censurées** | ~43 % (sous la limite de détection du laboratoire) |

### Variables clés

| Variable | Type | Description |
|---|---|---|
| `CHLORDECONE_RATE` | Numérique | Concentration de chlordécone mesurée (mg/kg) |
| `RATE_5B_HYDRO` | Numérique | Concentration du métabolite 5b-hydroxychlordécone |
| `SOL_SIMPLE` | Catégorielle | Type de sol (Andosol, Ferralsol, Vertisol, Nitisol…) |
| `RAIN` | Catégorielle | Tranche pluviométrique annuelle (mm/an) |
| `HISTOBANANE_HISTO_BAN` | Numérique | Nombre de périodes bananières (1970, 1980, 1992) |
| `MNT_TPI_MEAN` | Numérique | Position topographique relative (crête vs fond de vallée) |
| `MNT_SLOPE_MEAN` | Numérique | Inclinaison moyenne de la pente (degrés) |
| `COMMU_LAB` | Catégorielle | Commune de prélèvement |
| `X`, `Y` | Numérique | Coordonnées GPS du point de prélèvement |

---

## 🔬 Méthodologie

Le projet est structuré en **3 notebooks séquentiels** :

### Notebook 01 — Data Engineering
- Typage et correction des colonnes (virgules décimales, formats de dates)
- Traitement des valeurs censurées (`< LOD`) : remplacement par `LOD / 2`
- Analyse et imputation des valeurs manquantes (57,8 % de NA sur l'historique bananier)
- Détection des outliers (méthode IQR)
- Transformation logarithmique (`log1p`) de la variable cible
- Export du dataset nettoyé

### Notebook 02 — Analyse Exploratoire (EDA)
- Distribution des concentrations (brute vs log1p)
- Analyse de la contamination par type de sol, pluviométrie et historique bananier
- Corrélations de Spearman (variables asymétriques)
- Analyse temporelle 2010–2019
- Cartographie spatiale (scatter GPS)
- Feature engineering et encodage pour la modélisation

### Notebook 03 — Modélisation
- **Régression OLS** (baseline interprétable)
- **Random Forest Régresseur** (modèle principal)
- **Régression Logistique** (classification baseline)
- **Random Forest Classifieur** (modèle de détection)
- Analyse des erreurs (MAE par commune et par sol)
- Carte des résidus (autocorrélation spatiale résiduelle)
- Cartes interactives Plotly (risque + prédictions continues)

---

## 📈 Résultats

### Performance des modèles

| Modèle | Tâche | Métrique principale | RMSE / — | Accuracy |
|---|---|---|---|---|
| OLS | Régression | R² = 0.333 | RMSE = 0.444 | — |
| **Random Forest** | **Régression** | **R² = 0.955** | **RMSE = 0.116** | — |
| Régression Logistique | Classification | AUC = 0.894 | — | 83 % |
| **Random Forest** | **Classification** | **AUC = 0.986** | — | **95 %** |

> Le Random Forest atteint **+62 points de R²** vs OLS en régression, et **95 % de précision** en classification (seuil 0,1 mg/kg).

### Variables les plus influentes (convergence OLS + RF)

1. **`HISTOBANANE_HISTO_BAN`** — l'historique bananier est de loin le prédicteur dominant. Chaque période d'occupation supplémentaire multiplie significativement le risque de contamination.
2. **`SOL_SIMPLE` — Andosol** — les sols volcaniques poreux fixent et retiennent davantage le chlordécone.
3. **`MNT_TPI_MEAN`** (négatif) — les fonds de vallée accumulent la molécule par ruissellement.
4. **`MNT_SLOPE_MEAN`** — la pente conditionne le transport et l'accumulation.
5. **`RAIN`** — les zones à pluviométrie intermédiaire (1 250–3 000 mm/an) concentrent la contamination.

### Principaux constats EDA

- La contamination est **géographiquement non-aléatoire** — elle suit les zones historiques de plantation bananière intensive, principalement le centre et le nord de la Martinique.
- La relation entre historique bananier et contamination est **monotone, régulière et statistiquement certaine** (test de Tukey, p < 0.05 pour chaque niveau).
- Le chlordécone est **immobile dans le temps** — les zones contaminées en 2010 le restent en 2019, confirmant sa persistance décennale.

---

## 🗺️ Cartes interactives

Deux cartes Plotly interactives (fichiers `.html`, ouvrir dans un navigateur) :

| Fichier | Description |
|---|---|
| `carte_risque_rf_interactive.html` | Zones classées en **3 niveaux de risque** (faible / moyen / élevé) par quantiles RF |
| `carte_predictions_rf_interactive.html` | **Valeurs continues** de concentration prédite (échelle log, dégradé YlOrRd) |

Les deux cartes sont interactives : zoom, survol (commune + valeurs mesurées et prédites), fond OpenStreetMap.

---

## ⚙️ Installation & utilisation

### Prérequis

```bash
Python >= 3.9
```

### Dépendances

```bash
pip install pandas numpy matplotlib seaborn scipy statsmodels scikit-learn geopandas plotly
```

### Exécution

Les notebooks doivent être exécutés **dans l'ordre** :

```bash
jupyter notebook notebooks/01_data_engineering_vf.ipynb
jupyter notebook notebooks/02_exploratory_analysis_vf.ipynb
jupyter notebook notebooks/03_modeling_vf.ipynb
```

> ⚠️ Adapter les chemins de fichiers dans les cellules de chargement/export si nécessaire (actuellement configurés pour un chemin local Windows).

---

## ⚠️ Limites

- **43 % de valeurs censurées** : les concentrations sous la limite de détection sont imputées à `LOD / 2`, introduisant un biais vers le bas sur les niveaux faibles.
- **Absence de variables hydrologiques** : le réseau de drainage, les cours d'eau et les zones d'accumulation d'eau ne sont pas intégrés — c'est la principale source de résidus spatiaux.
- **Le Random Forest n'extrapole pas** : ses prédictions ne sont fiables que dans la plage des données d'entraînement (territoire et période couverts).
- **Autocorrélation spatiale résiduelle** : des clusters d'erreurs géographiquement localisés suggèrent des facteurs locaux non capturés.

---

## 🔭 Perspectives

- Intégrer des **variables hydrologiques** (réseau hydrographique, indice d'accumulation d'eau topographique — TWI)
- Tester des **modèles spatiaux** (krigeage, GWR — Geographically Weighted Regression) pour capturer l'autocorrélation spatiale
- Optimiser les hyperparamètres RF avec `GridSearchCV`
- Étendre l'analyse à la **Guadeloupe** (données similaires disponibles)
- Coupler avec des données de **santé publique** (incidence du cancer de la prostate par commune) pour une analyse épidémiologique

---

## 📚 Références

- CIRC / IARC — Chlordécone : Monographies, Vol. 79 (2001)
- Plan Chlordécone IV (2021–2027) — Ministère des Solidarités et de la Santé
- ANSES — Évaluation des risques liés au chlordécone (2021)
- Multigner et al. (2010) — *Chlordecone exposure and risk of prostate cancer*, Journal of Clinical Oncology
- Coat et al. (2006) — *Polychlorobiphenyls and chlordecone in surface waters and sediments in Martinique*

---

## 📄 Licence

Ce projet est open-source. Les données utilisées sont issues de bases publiques mises à disposition par les autorités sanitaires françaises.

---
## Online Version
The full analysis notebook is available on Kaggle:
https://www.kaggle.com/work/collections/17744539