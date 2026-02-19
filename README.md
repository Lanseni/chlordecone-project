# Analyse et Ingénierie des données de la chlordécone aux Antilles françaises
## Objectif

Ce projet vise à analyser la contamination des sols par la chlordécone aux Antilles françaises en combinant :

ingénierie des données,

analyses statistiques,

modélisation prédictive,

cartographie interactive.

L’objectif final est de produire des insights exploitables pour l’aide à la décision publique.

## Données utilisées

### Base de données : BaseCLD2026.csv

### Variables principales :

Taux_Chlordecone

Taux_5b_hydro

Historique bananier

Type de sol

Pluviométrie

Indices topographiques (pente, TRI, TPI, rugosité, exposition)

Coordonnées spatiales (UTM)

## Pipeline du projet

### Data cleaning

gestion des valeurs manquantes

traitement des valeurs censurées

transformation log

### Analyse exploratoire

statistiques descriptives

ANOVA + Tukey

visualisation spatiale

### Modélisation explicative

régression linéaire multiple (R² ≈ 0.33)

### Modélisation prédictive

Random Forest (R² ≈ 0.70)

importance des variables

### Cartographie de risque

classification Faible / Modéré / Élevé

hiérarchisation des communes prioritaires

## Résultats majeurs

L’historique bananier est un facteur explicatif central.

La topographie et le type de sol modulent fortement la contamination.

Les modèles non linéaires capturent mieux la complexité spatiale.

Plusieurs communes du Nord apparaissent prioritaires en termes de risque.

## Outils

Python (pandas, geopandas, statsmodels, scikit-learn, plotly)

VS Code

Git / GitHub

## Conclusion

La contamination actuelle est structurellement liée aux pratiques agricoles passées et modulée par des facteurs environnementaux locaux. La combinaison de modèles explicatifs et prédictifs permet une approche robuste et opérationnelle du risque.


## Online Version

The full analysis notebook is available on Kaggle:

 [Kaggle Notebook Link](https://www.kaggle.com/code/lascam/02-analysis-modeling)