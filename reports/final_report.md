# Rapport Final — Modélisation de la contamination des sols au chlordécone en Martinique

**Auteur :** Projet Data Science — Chlordécone Martinique  
**Date :** Mars 2026  
**Données :** BaseCLD2026 — 31 126 observations, 2010–2019  
**Dépôt :** [github.com/chlordecone-project](https://github.com)

---

## Table des matières

1. [Introduction et enjeux](#1-introduction-et-enjeux)
2. [Données et préparation](#2-données-et-préparation)
3. [Analyse exploratoire](#3-analyse-exploratoire)
4. [Modélisation](#4-modélisation)
5. [Résultats et interprétation](#5-résultats-et-interprétation)
6. [Cartes interactives](#6-cartes-interactives)
7. [Limites et perspectives](#7-limites-et-perspectives)
8. [Conclusion](#8-conclusion)

---

## 1. Introduction et enjeux

### 1.1 Le scandale du chlordécone

Le chlordécone (C₁₀Cl₁₀O) est un pesticide organochloré utilisé en Martinique et en Guadeloupe pour lutter contre le charançon du bananier (*Cosmopolites sordidus*). Son usage a été autorisé aux Antilles françaises de **1972 à 1993**, soit dix ans après son interdiction aux États-Unis (1976) et alors que sa toxicité était documentée scientifiquement depuis la fin des années 1960.

Les conséquences sanitaires et environnementales de cette décision sont aujourd'hui bien établies :

- **Persistance extrême** dans les sols : demi-vie estimée entre 50 et plusieurs centaines d'années selon le type de sol et les conditions climatiques.
- **Contamination de la chaîne alimentaire** : fruits, légumes racines, produits animaux et ressources marines contaminés.
- **Risques sanitaires documentés** : le chlordécone est classé cancérogène possible (groupe 2B) par le Centre International de Recherche sur le Cancer (CIRC). Une étude majeure (Multigner et al., 2010) a établi un lien significatif avec le cancer de la prostate, dont la Martinique et la Guadeloupe affichent les taux d'incidence parmi les plus élevés au monde.
- **Impact socio-économique** : restrictions d'usage des terres agricoles, interdictions de pêche dans plusieurs zones côtières, perte de revenus pour les agriculteurs et pêcheurs.

L'État français a reconnu une part de responsabilité dans cette crise. Le **Plan Chlordécone IV (2021–2027)** prévoit 92 millions d'euros pour la surveillance, la recherche et l'accompagnement des populations. Néanmoins, un fossé persiste entre les mesures institutionnelles et le sentiment d'abandon des citoyens martiniquais et guadeloupéens.

### 1.2 Objectifs du projet

Face à ce contexte, ce projet applique des **méthodes quantitatives de science des données** aux données publiques de surveillance des sols pour répondre à trois questions :

**Q1 — Cartographie :** Quelles zones de la Martinique sont les plus contaminées, et selon quels facteurs géographiques et pédologiques ?

**Q2 — Explication :** Quelles variables expliquent le mieux la variabilité des concentrations ? L'historique agricole bananier laisse-t-il une empreinte statistiquement mesurable et significative ?

**Q3 — Prédiction :** Peut-on construire un modèle fiable pour prédire la concentration en chlordécone dans des parcelles non encore testées, afin de prioriser les contrôles terrain ?

---

## 2. Données et préparation

### 2.1 Source et description

Les données proviennent de la **Base nationale de surveillance des sols (BaseCLD2026)**, mise à disposition par les autorités sanitaires françaises dans le cadre du plan de surveillance de la contamination au chlordécone.

| Attribut | Valeur |
|---|---|
| Territoire | Martinique |
| Période | 2010 – 2019 |
| Observations brutes | 31 126 prélèvements |
| Variables | 26 colonnes |
| Seuil réglementaire utilisé | 0,1 mg/kg |

Le dataset couvre l'ensemble du territoire martiniquais avec une densité d'échantillonnage variable selon les années et les communes. Chaque observation représente un prélèvement de sol géolocalisé, caractérisé par son type de sol, sa zone pluviométrique, sa topographie et l'historique d'occupation bananière de la parcelle.

### 2.2 Traitement des valeurs censurées

**43 % des mesures de chlordécone sont censurées** : le laboratoire a détecté la présence de la molécule mais n'a pas pu la quantifier précisément (mesure inférieure à la limite de détection, notée `< LOD`). Ce taux élevé de censure est une contrainte analytique majeure.

**Approche retenue :** remplacement des valeurs censurées par `LOD / 2`, convention standard en épidémiologie environnementale. Cette approche introduit un biais conservateur (sous-estimation des niveaux bas) mais permet de conserver l'ensemble des observations dans les modèles.

**Conséquence :** les concentrations très faibles sont sous-estimées, ce qui penche en faveur d'une lecture optimiste de la contamination. Les niveaux réels pourraient être légèrement supérieurs à ceux modélisés dans la plage basse.

### 2.3 Valeurs manquantes

| Variable | NA (%) | Traitement |
|---|---|---|
| `HISTOBANANE_HISTO_BAN` | 57,8 % | Catégorie "Inconnu" distincte |
| `Date_analyse` | 8,7 % | Exclu des analyses temporelles |
| `type_sol` (détaillé) | 8,4 % | Non utilisé (remplacé par `SOL_SIMPLE`) |
| `COMMU_LAB` | 1,0 % | Suppression des lignes |
| Variables topographiques | < 0,1 % | Imputation par médiane communale |

Le fort taux de NA sur l'historique bananier (57,8 %) est lui-même informatif : il correspond à des parcelles pour lesquelles les archives agricoles n'existent pas — souvent des zones périurbaines ou des secteurs historiquement non cartographiés. Ces parcelles ont été codées dans une catégorie "Inconnu" distincte, qui a montré un comportement spécifique dans les modèles.

### 2.4 Transformation logarithmique

La distribution brute de `CHLORDECONE_RATE` est fortement asymétrique (skewness > 3) : la majorité des parcelles affichent des concentrations proches de zéro, mais quelques zones très contaminées produisent une queue très longue vers la droite. Tous les modèles ont été entraînés sur **`log_chl = log1p(CHLORDECONE_RATE)`**, qui normalise cette distribution et stabilise la variance.

Pour l'interprétation, les valeurs sont reconverties à l'échelle originale via `expm1()`.

### 2.5 Feature engineering

Les variables catégorielles ont été encodées pour la modélisation :
- `SOL_SIMPLE` → ordinal (basé sur la rétention historique de chlordécone)
- `RAIN` → ordinal (pluviométrie croissante)
- `histoBanane_cat` → numérique (`ban_num` : 0, 1, 2, 3 périodes)

La variable cible de classification `contamine` est définie comme `1` si `CHLORDECONE_RATE > 0.1 mg/kg`, `0` sinon. **34,6 % des observations sont classées contaminées** selon ce seuil.

---

## 3. Analyse exploratoire

### 3.1 Distribution de la contamination

La distribution de `CHLORDECONE_RATE` est bimodale sur l'échelle log : un premier pic autour des valeurs censurées (< LOD), un second pic autour de 0,1–1 mg/kg correspondant aux zones agricoles historiquement bananières. Cette bimodalité reflète la dualité du territoire martiniquais : des zones préservées et des zones fortement marquées par l'héritage pesticide.

**Statistiques clés :**
- Médiane brute : 0,003 mg/kg (très basse, tirée vers le bas par les valeurs censurées)
- Moyenne brute : 0,31 mg/kg
- Valeur maximale observée : > 10 mg/kg (100x le seuil réglementaire)
- 34,6 % des parcelles dépassent 0,1 mg/kg

### 3.2 Facteurs de contamination

#### Type de sol

Les **Andosols** (sols volcaniques jeunes, poreux, très présents dans le nord de la Martinique) montrent les taux de contamination les plus élevés. Leur structure chimique particulière — teneur élevée en allophane et matière organique — favorise la fixation du chlordécone, réduisant sa mobilité et prolongeant sa persistance.

À l'inverse, les **Ferralsols** et **Nitisols** (sols plus anciens, argileux, bien drainés) présentent des niveaux de contamination significativement inférieurs. Les **Vertisols** (sols à argiles gonflantes, plutôt côtiers) montrent également des valeurs plus faibles.

La différence entre types de sol est statistiquement significative (test de Kruskal-Wallis, p < 0.001).

#### Pluviométrie

Les zones à pluviométrie **intermédiaire (1 250–3 000 mm/an)** concentrent davantage de contamination. Ce résultat contre-intuitif s'explique par la géographie agricole : les zones de plantation bananière intense correspondent précisément à ces tranches pluviométriques (versants sous le vent, mi-altitude), pas aux zones les plus arrosées (hauteurs nuageuses, peu cultivées) ni aux zones les plus sèches (côtes sous le vent, peu propices à la banane).

#### Historique bananier

C'est **le facteur le plus discriminant** de l'ensemble de l'analyse. La variable `HISTOBANANE_HISTO_BAN` (nombre de périodes d'occupation bananière : 1970, 1980, 1992) montre une relation parfaitement monotone avec la contamination :

| Périodes bananières | % contaminé | Médiane log_chl |
|---|---|---|
| 0 (jamais cultivé) | ~15 % | faible |
| 1 période | ~35 % | modéré |
| 2 périodes | ~55 % | élevé |
| 3 périodes | ~70 % | très élevé |

Le test de Tukey confirme que chaque groupe est statistiquement différent des autres (p < 0.05). Cette **relation dose-réponse** est le résultat le plus politiquement significatif de l'EDA : elle établit quantitativement que chaque année d'épandage supplémentaire a laissé une empreinte chimique mesurable et persistante dans les sols martiniquais.

### 3.3 Corrélations (Spearman)

Les corrélations de Spearman (adaptées aux distributions asymétriques) confirment la hiérarchie des facteurs :

| Variable | ρ avec log_chl | ρ avec contamine |
|---|---|---|
| `ban_num` (historique bananier) | +0.45 | +0.40 |
| `sol_ord` (type de sol) | +0.28 | +0.24 |
| `rain_ord` (pluviométrie) | +0.22 | +0.19 |
| `mnt_tpi_mean` (position topographique) | -0.18 | -0.15 |
| `mnt_pente_mean` (pente) | -0.14 | -0.12 |

La corrélation **négative** du TPI s'explique physiquement : un TPI négatif indique une position en fond de vallée ou de versant, où l'eau et les sédiments — et donc le chlordécone — s'accumulent par ruissellement.

### 3.4 Analyse temporelle

L'analyse des données 2010–2019 révèle une irrégularité importante des campagnes de surveillance : le nombre d'échantillons varie de moins de 1 000 à plus de 5 000 selon les années. Les fluctuations apparentes du taux de contamination annuel doivent donc être interprétées avec prudence — elles peuvent refléter un biais d'échantillonnage (zones ciblées différemment selon les années) plutôt qu'une évolution réelle de la contamination.

Cette instabilité du dispositif de surveillance est elle-même un résultat : la surveillance des sols contaminés au chlordécone n'est pas systématique, exhaustive, ni homogène dans le temps.

### 3.5 Géographie de la contamination

La cartographie des points de prélèvement montre une **concentration spatiale nette** de la contamination dans le centre et le nord de la Martinique — précisément les zones historiques des grandes exploitations bananières. La contamination n'est pas homogène sur le territoire : certaines communes cumulent plusieurs facteurs aggravants (Andosol + pluviométrie intermédiaire + longue histoire bananière), tandis que les zones côtières et les secteurs urbanisés montrent des niveaux systématiquement plus bas.

La stabilité spatiale inter-annuelle (les mêmes zones sont contaminées en 2010 et en 2019) confirme le caractère **irréversible** de la pollution à l'échelle humaine.

---

## 4. Modélisation

### 4.1 Architecture générale

Deux tâches ont été menées en parallèle :

- **Régression** : prédire la concentration continue en chlordécone (`log_chl`)
- **Classification** : prédire si une parcelle dépasse le seuil réglementaire (`contamine`, seuil 0,1 mg/kg)

Pour chaque tâche, deux modèles ont été comparés : un modèle linéaire interprétable (OLS / Régression Logistique) et un modèle non-linéaire performant (Random Forest).

**Découpage train/test :** 70 % entraînement / 30 % test, stratifié sur la variable `contamine`.

**Features utilisées :** `rain_ord`, `sol_ord`, `ban_num`, `mnt_pente_mean`, `mnt_tpi_mean`, `mnt_tri_mean`, `mnt_rugosite_mean`, `mnt_ombrage_mean`, `mnt_exposition_mean`, `ANNEE`, `log_5b`.

### 4.2 Régression linéaire (OLS)

Le modèle OLS utilise la formule statsmodels avec variables catégorielles encodées automatiquement (`C()`). Il sert de **baseline interprétable** et de référence pour évaluer le gain apporté par les méthodes non-linéaires.

**Résultats :**

| Métrique | Valeur |
|---|---|
| R² | 0.333 |
| RMSE | 0.444 |
| MAE | 0.317 |

Le modèle explique **33,3 % de la variance** — correct pour un modèle interprétable sur données environnementales complexes, mais insuffisant pour des prédictions opérationnelles précises. L'examen des résidus révèle une asymétrie persistante (skewness positive) et une hétéroscédasticité : le modèle sous-estime systématiquement les concentrations élevées, précisément celles qui sont les plus critiques pour la surveillance sanitaire.

**Coefficients significatifs (p < 0.05) :** l'historique bananier catégorie 3 (+0.34), les Andosols (+0.19), la pluviométrie 3 000–5 000 mm (+0.28) contribuent positivement. Les Ferralsols (-0.10), Nitisols (-0.11) et Vertisols (-0.12) contribuent négativement. Le TPI montre un effet négatif significatif (-0.012 par unité).

### 4.3 Random Forest — Régression

Le Random Forest Régresseur (`n_estimators=200, max_depth=None, random_state=42`) capture les **interactions non-linéaires** et les **effets de seuil** entre variables — par exemple, l'effet d'un Andosol se combine différemment avec l'historique bananier selon la position topographique.

**Résultats :**

| Métrique | Entraînement | Test |
|---|---|---|
| R² | 0.994 | **0.955** |
| RMSE | — | **0.116** |
| MAE | — | **0.050** |

Le modèle explique **95,5 % de la variance en test**, soit un gain de +62 points de R² par rapport à l'OLS. Le RMSE est réduit de 74 % (0.444 → 0.116). L'écart train/test (0.994 vs 0.955) indique un léger surapprentissage, inhérent aux Random Forests non élagués, mais sans compromettre la généralisation.

### 4.4 Régression Logistique (classification baseline)

La régression logistique est entraînée avec les mêmes features que l'OLS, après standardisation (`StandardScaler`). Elle constitue la baseline interprétable pour la tâche de classification.

**Résultats :**

| Métrique | Valeur |
|---|---|
| AUC-ROC | 0.894 |
| Accuracy | 83 % |
| F1 (contaminé) | 0.74 |
| Recall (contaminé) | 68 % |

L'AUC de 0.894 est déjà très correct — la régression logistique capture bien les grandes tendances. Mais son recall de 68 % sur la classe "contaminé" signifie que **32 % des zones contaminées seraient classées "saines"** — un niveau de faux négatifs inacceptable dans un contexte de surveillance sanitaire.

### 4.5 Random Forest — Classification

Le Random Forest Classifieur (`n_estimators=200, class_weight='balanced'`) gère le déséquilibre des classes (65 % non-contaminé / 35 % contaminé) grâce au rééquilibrage automatique des poids.

**Résultats :**

| Métrique | Non contaminé | Contaminé |
|---|---|---|
| Précision | 0.95 | 0.96 |
| Recall | 0.98 | 0.90 |
| F1-score | 0.96 | 0.93 |
| **Accuracy globale** | | **95 %** |
| **AUC-ROC** | | **0.986** |

Le Random Forest réduit le taux de faux négatifs (zones contaminées classées saines) de 32 % à **10 %** par rapport à la régression logistique. Avec AUC = 0.986, le modèle distingue quasi-parfaitement les zones contaminées.

### 4.6 Importance des variables

Les deux approches (coefficients OLS et importance RF) convergent vers le même classement :

| Rang | Variable | Contribution |
|---|---|---|
| 1 | `ban_num` — historique bananier | Dominante |
| 2 | `sol_ord` — type de sol | Forte |
| 3 | `mnt_tpi_mean` — position topographique | Modérée |
| 4 | `mnt_pente_mean` — pente | Modérée |
| 5 | `rain_ord` — pluviométrie | Modérée |
| 6 | `log_5b` — métabolite 5b | Modérée |
| 7–11 | Autres variables topographiques (`TRI`, rugosité, ombrage, exposition) | Faible à marginale |

**La convergence des deux méthodes sur le même classement est un résultat robuste** : elle indique que ce classement n'est pas un artefact d'une méthode particulière, mais reflète la structure réelle des données.

---

## 5. Résultats et interprétation

### 5.1 Synthèse des performances

| Modèle | Tâche | Métrique | Score |
|---|---|---|---|
| OLS | Régression | R² | 0.333 |
| **Random Forest** | **Régression** | **R²** | **0.955** |
| Régression Logistique | Classification | AUC | 0.894 |
| **Random Forest** | **Classification** | **AUC** | **0.986** |

### 5.2 Analyse des erreurs

L'analyse des erreurs du modèle RF par commune et par type de sol révèle des **patterns géographiques dans les résidus** — certaines communes concentrent des erreurs systématiquement plus élevées, indépendamment du type de sol ou de l'historique bananier. Ces clusters d'erreurs signalent l'existence de facteurs locaux non capturés par les données disponibles : réseau hydrographique local, pratiques d'irrigation historiques, densité de stockage d'intrants agricoles.

La **carte des résidus** (rouge = sous-estimation, bleu = surestimation) traduit visuellement cette autocorrélation spatiale résiduelle. Les zones rouges — où la réalité est pire que prévu — sont des zones d'alerte prioritaires pour des investigations complémentaires.

### 5.3 Interprétation environnementale et sanitaire

Les résultats quantitatifs permettent de formuler des conclusions environnementales solides :

**1. Le lien causal historique est statistiquement établi.** La relation dose-réponse entre nombre de périodes bananières et niveau de contamination actuel est significative, monotone et robuste à la méthode. Chaque période d'occupation bananière entre 1970 et 1992 a laissé une empreinte chimique mesurable en 2010–2019.

**2. La géologie amplifie ou atténue la contamination mais ne la crée pas.** Les Andosols retiennent davantage le chlordécone, mais une parcelle sur Andosol sans historique bananier reste significativement moins contaminée qu'une parcelle sur Ferralsol avec trois périodes bananières. Le sol est un amplificateur, pas un facteur primaire.

**3. La topographie désigne les zones d'accumulation.** Les fonds de vallée concentrent le chlordécone transporté par ruissellement — ce sont souvent les zones où se trouvent les cultures maraîchères, les élevages et les captages d'eau. La topographie comme variable prédictive a donc une valeur opérationnelle directe pour orienter la surveillance.

**4. La contamination est irréversible à l'échelle humaine.** La stabilité spatiale inter-annuelle des zones contaminées, combinée aux propriétés physico-chimiques du chlordécone, confirme que la pollution persistera au-delà de toute génération vivante.

### 5.4 Communes prioritaires

L'agrégation des prédictions RF à l'échelle communale identifie les **10 communes à risque moyen prédit le plus élevé**. Ces communes se caractérisent par la combinaison de trois facteurs : forte proportion d'Andosols, longue histoire bananière (2–3 périodes), et position topographique en fond de bassins versants. Elles constituent les cibles prioritaires pour :
- l'intensification des contrôles de sols,
- les mesures de restriction d'usage,
- l'accompagnement des agriculteurs et populations concernés.

---

## 6. Cartes interactives

Deux livrables cartographiques interactifs ont été produits avec Plotly Express sur fond OpenStreetMap :

### Carte de risque (3 niveaux)
**Fichier :** `outputs/carte_risque_rf_interactive.html`

Classification des points de prélèvement en trois niveaux de risque (faible / moyen / élevé) définis par les quantiles de la prédiction RF. Ce format binaire simplifié est conçu pour la **communication avec les décideurs et le grand public** : il permet une lecture immédiate, sans formation statistique, de la géographie du risque. Au survol : commune, taux mesuré, niveau de risque.

### Carte prédictive continue
**Fichier :** `outputs/carte_predictions_rf_interactive.html`

Valeurs continues de concentration prédite (log_chl) sur dégradé YlOrRd. Ce format est conçu pour les **experts et gestionnaires** : il révèle les gradients fins de contamination à l'intérieur de chaque zone de risque et permet la comparaison directe entre valeur mesurée et valeur prédite sur chaque point. Il facilite la détection d'anomalies (points où la mesure terrain diverge fortement de la prédiction) qui méritent une vérification.

---

## 7. Limites et perspectives

### 7.1 Limites identifiées

**Données :**
- 43 % de valeurs censurées introduisent un biais conservateur dans les estimations de contamination basse.
- L'historique bananier présente 57,8 % de valeurs manquantes — pour ces parcelles, le facteur le plus prédictif est absent.
- Le dispositif de surveillance n'est pas aléatoire spatialement : les zones déjà suspectées sont sur-représentées, ce qui peut biaiser les performances apparentes du modèle.

**Modèles :**
- Le Random Forest ne peut pas extrapoler hors de sa plage d'entraînement (pas de prédictions fiables pour des configurations géographiques ou pédologiques non représentées dans les données).
- L'autocorrélation spatiale résiduelle indique que des variables non disponibles (réseau hydrographique, données de pratiques agricoles détaillées) amélioreraient significativement les prédictions.
- La validation croisée spatiale (par exemple, leave-one-region-out) serait plus appropriée que la validation aléatoire pour évaluer la généralisation géographique.

### 7.2 Perspectives de développement

**Court terme :**
- Intégrer des **variables hydrologiques** : réseau de drainage (TWI — Topographic Wetness Index), distance aux cours d'eau, zones d'accumulation d'eau.
- Optimiser les hyperparamètres du Random Forest via `GridSearchCV` ou `RandomizedSearchCV`.
- Appliquer une **validation croisée spatiale** (blocs géographiques) pour une estimation honnête de la généralisation.

**Moyen terme :**
- Tester des **modèles géostatistiques** (krigeage, GWR) pour modéliser explicitement l'autocorrélation spatiale résiduelle.
- Étendre l'analyse à la **Guadeloupe** avec le même protocole.
- Coupler les prédictions de contamination des sols avec des **données de santé publique** (incidence du cancer de la prostate par commune) pour une analyse épidémiologique spatiale.

**Long terme :**
- Développer un **tableau de bord de surveillance** mis à jour annuellement avec les nouvelles campagnes de prélèvement.
- Intégrer des données de **télédétection** (NDVI, indices de végétation) pour améliorer la caractérisation de l'occupation des sols.

---

## 8. Conclusion

Ce projet démontre qu'il est possible de **prédire avec une précision élevée (R² = 0.955 en régression, AUC = 0.986 en classification)** la contamination des sols au chlordécone en Martinique à partir de variables géographiques, pédologiques et d'historique agricole.

Le résultat le plus important n'est pas la performance des modèles — c'est ce qu'ils révèlent sur la **structure de la contamination** :

> L'historique d'occupation bananière entre 1970 et 1992 est, de loin, le prédicteur dominant de la contamination mesurée en 2010–2019. Cette relation est monotone, robuste à la méthode, et statistiquement certaine. Elle établit quantitativement le lien entre les décisions agricoles et industrielles du passé et la réalité chimique des sols martiniquais aujourd'hui.

Les modèles développés ici ont une valeur **opérationnelle directe** : ils permettent de prioriser les contrôles terrain dans les zones non encore testées, d'identifier les communes à risque élevé, et de cibler les ressources de surveillance là où elles sont le plus nécessaires.

Ce projet illustre comment la science des données peut contribuer à la **transparence et à la justice environnementale** dans des crises sanitaires complexes — en transformant des données publiques dispersées en outils cartographiques accessibles et en résultats quantifiés qui complètent et objectivent le témoignage des populations touchées.

---

## Annexes

### A. Tableau de synthèse des modèles

| Modèle | Tâche | R² / AUC | RMSE | Accuracy | Interprétabilité |
|---|---|---|---|---|---|
| OLS | Régression | 0.333 | 0.444 | — | ⭐⭐⭐⭐ |
| Random Forest | Régression | 0.955 | 0.116 | — | ⭐⭐ |
| Régression Logistique | Classification | AUC 0.894 | — | 83 % | ⭐⭐⭐⭐ |
| Random Forest | Classification | AUC 0.986 | — | 95 % | ⭐⭐ |

### B. Dictionnaire des variables principales

| Variable | Type | Description |
|---|---|---|
| `CHLORDECONE_RATE` | Numérique | Concentration de chlordécone (mg/kg) |
| `log_chl` | Numérique | log1p(CHLORDECONE_RATE) — variable cible régression |
| `contamine` | Binaire | 1 si CHLORDECONE_RATE > 0.1 mg/kg |
| `SOL_SIMPLE` | Catégorielle | Type de sol (Andosol, Ferralsol, Vertisol, Nitisol…) |
| `RAIN` | Catégorielle | Tranche pluviométrique (mm/an) |
| `HISTOBANANE_HISTO_BAN` | Numérique | Nombre de périodes bananières (0–3) |
| `MNT_TPI_MEAN` | Numérique | Position topographique (crête = + / fond = -) |
| `MNT_SLOPE_MEAN` | Numérique | Pente moyenne (degrés) |
| `RATE_5B_HYDRO` | Numérique | Concentration métabolite 5b-hydroxychlordécone |
| `X`, `Y` | Numérique | Coordonnées GPS (Lambert 93) |
| `COMMU_LAB` | Catégorielle | Commune de prélèvement |
| `ANNEE` | Numérique | Année du prélèvement (2010–2019) |

### C. Références

- Multigner L. et al. (2010). *Chlordecone exposure and risk of prostate cancer*. Journal of Clinical Oncology, 28(21), 3457–3462.
- IARC / CIRC (2001). *Chlordecone*. IARC Monographs on the Evaluation of Carcinogenic Risks to Humans, Vol. 79.
- ANSES (2021). *Évaluation des risques liés au chlordécone pour les populations des Antilles françaises*.
- Coat S. et al. (2006). *Polychlorobiphenyls and chlordecone in surface waters and sediments in Martinique*. Chemosphere, 65(11), 2458–2467.
- Plan Chlordécone IV (2021–2027). Ministère des Solidarités et de la Santé / Ministère des Outre-mer.
- Cabidoche Y.M. et al. (2009). *Long-term pollution by chlordecone of tropical volcanic soils in the French West Indies: a simple leaching model accounts for current residue*. Environmental Pollution, 157(5), 1697–1705.

---
