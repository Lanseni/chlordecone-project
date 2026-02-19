# Analyse et Ingénierie des données de la chlordécone aux Antilles françaises

## 1. Contexte et objectifs

La chlordécone est un pesticide organochloré utilisé massivement dans les bananeraies des Antilles françaises jusqu’au début des années 1990. Sa persistance dans les sols constitue aujourd’hui un enjeu environnemental et sanitaire majeur.

L’objectif de ce projet est de :

structurer et nettoyer les données environnementales disponibles,

analyser les facteurs explicatifs de la contamination,

construire un modèle prédictif du risque,

produire des outils utiles à la décision publique.

## 2. Méthodologie
### 2.1 Ingénierie des données

Nettoyage des valeurs manquantes

Traitement des valeurs inférieures au seuil de détection

Transformation logarithmique des concentrations

Encodage des variables catégorielles

Reprojection des coordonnées UTM en WGS84

### 2.2 Analyse statistique

Analyse descriptive par commune

ANOVA pour tester l’effet de l’historique bananier

Test post-hoc de Tukey

Régression linéaire multiple

Le modèle linéaire explique environ 33 % de la variance.

### 2.3 Modélisation prédictive

Un modèle Random Forest a été entraîné afin de capturer les non-linéarités et interactions complexes.

Performance :

R² ≈ 0.70 (échantillon test)

Les variables les plus importantes sont :

l’historique bananier,

les indices topographiques,

certains types de sols.

## 3. Résultats principaux

Forte hétérogénéité territoriale

Confirmation statistique de l’effet historique

Influence significative des facteurs topographiques

Amélioration majeure via modèle non linéaire

Une carte de risque classée en trois niveaux (Faible / Modéré / Élevé) permet d’identifier les communes prioritaires.

## 4. Recommandations politiques

Priorisation des communes à forte proportion de parcelles à risque élevé

Intégration des facteurs environnementaux dans les politiques agricoles

Mise en place d’outils cartographiques dynamiques

Stratégie différenciée selon le niveau de risque

## 5. Limites

Données historiques simplifiées

Valeurs censurées

Corrélation ≠ causalité

Absence de validation externe indépendante

## 6. Conclusion

La contamination actuelle des sols par la chlordécone est fortement structurée par les pratiques agricoles historiques et modulée par les caractéristiques environnementales locales.

La combinaison d’analyses statistiques et de modèles prédictifs permet une compréhension robuste et opérationnelle du risque, contribuant ainsi à une meilleure aide à la décision publique.