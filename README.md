# Projet-1-Dashboard-analyse-de-ventes-SQL-Power-BI-
Projet d’analyse des ventes d’une boutique e-commerce fictive réalisé avec Python et Power BI. L’objectif : explorer les performances commerciales, identifier les meilleures ventes et proposer des recommandations stratégiques. J’ai nettoyé et analysé les données, créé des visualisations et un dashboard interactif pour aider à la prise de décision.

## 📌 Contexte
Dans le cadre de ce projet, nous avons été sollicités par une entreprise spécialisée dans la vente de modèles et de maquettes pour concevoir un tableau de bord dynamique. L'objectif : permettre au directeur de piloter son activité à partir d’indicateurs actualisés quotidiennement.
La base de données de l’entreprise contient des informations sur les employés, les produits, les commandes, et bien plus encore. Notre rôle est de structurer et analyser ces données pour proposer un tableau de bord fiable et efficace.

## 🎯 Objectifs
Construire un tableau de bord Power BI articulé autour de 4 thématiques :
📊 Ventes
💶 Finances
🚚 Logistique
🧑‍💼 Ressources humaines
En répondant aux indicateurs de performance (KPI) demandés et en proposant des indicateurs complémentaires pour enrichir l’analyse.

## 📐 Méthodologie

### 1️⃣ Partie 1 : Calcul des Métriques en SQL
Rédaction de requêtes SQL complexes pour extraire les KPI demandés.
Identification de nouveaux indicateurs pertinents.
Test de la performance et de la précision des requêtes.

📌 Exemples de KPI calculés :
Chiffre d’affaires par mois et région 📈
Produits les plus/moins vendus par catégorie 📦
Taux de recouvrement des créances par client 💰
Stock sous seuil critique 

### 2️⃣ Partie 2 : Modélisation en Schéma Étoile (OLAP)
Afin d’optimiser la performance sous Power BI :
Transformation de la base transactionnelle (OLTP) en modèle analytique (OLAP).
Création de vues SQL pour les tables de faits et de dimensions.
📌 Exemple de structure :🛒

### 3️⃣ Partie 3 : Création du Tableau de Bord Power BI
Importation des vues SQL optimisées.
Construction d’un modèle de données sous forme de schéma en étoile.
Conception de visualisations dynamiques et interactives :
- Graphiques par région
- KPI en tuiles
- Cartes géographiques 🌍
- Graphiques d’évolution 📊
- Mise en place de filtres multi-critères (dates, produits, bureaux, commerciaux…)
- Configuration de l’actualisation quotidienne des données

## 📊 Livrables
📦 Fichier Power BI (.pbix) avec tableau de bord interactif et actualisable.
📜 Script SQL complet (requêtes + création des vues).
📖 README explicatif du projet et de la méthodologie (ce document).

# 📌 Conclusion
Ce projet permet de valoriser les données de l’entreprise en les rendant accessibles et lisibles via un outil puissant et interactif. Il répond aux enjeux de pilotage quotidien, de détection des anomalies, et d’optimisation des performances commerciales, financières et logistiques.
