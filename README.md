# Projet-1-Dashboard-analyse-de-ventes-SQL-Power-BI-
Projet d’analyse des ventes d’une boutique e-commerce fictive réalisé avec Python et Power BI. L’objectif : explorer les performances commerciales, identifier les meilleures ventes et proposer des recommandations stratégiques. J’ai nettoyé et analysé les données, créé des visualisations et un dashboard interactif pour aider à la prise de décision.

## 📌 Contexte
Dans le cadre de ce projet, nous avons été sollicités par une entreprise spécialisée dans la vente de modèles et de maquettes pour concevoir un tableau de bord dynamique. L'objectif : permettre au directeur de piloter son activité à partir d’indicateurs actualisés quotidiennement.
La base de données de l’entreprise contient des informations sur les employés, les produits, les commandes, et bien plus encore. 
Nous sommes une équipe de 4 Data Analyst Junior, et notre rôle est de structurer et analyser ces données pour proposer un tableau de bord fiable et efficace.
Je suis personnellement en charge de la partie "Ventes" du projet.


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
**Chiffre d’affaires par mois et région 📈**
```
# Chiffre d’affaires par mois et par région + taux d’évolution mensuel :
    
WITH MonthlySales AS (
    SELECT YEAR(orders.orderDate) AS year, MONTH(orders.orderDate) AS month, offices.city, SUM(quantityOrdered * priceEach) AS totalSales
    FROM offices
    JOIN employees ON employees.officeCode = offices.officeCode
    JOIN customers ON customers.salesRepEmployeeNumber = employees.employeeNumber
    JOIN orders ON orders.customerNumber = customers.customerNumber
    JOIN orderdetails ON orderdetails.orderNumber = orders.orderNumber
    GROUP BY year, month, offices.city
)
SELECT year, month, city, totalSales, LAG(totalSales) OVER (PARTITION BY city ORDER BY year, month) AS previousMonthSales,
    COALESCE(((totalSales - LAG(totalSales) OVER (PARTITION BY city ORDER BY year, month)) / NULLIF(LAG(totalSales) OVER (PARTITION BY city ORDER BY year, month), 0)) * 100,0) AS growthRate
FROM MonthlySales
ORDER BY year, month, city DESC;
```

**Produits les plus/moins vendus par catégorie 📦**
```
# produits les plus vendus par catégorie :
SELECT products.productLine, SUM(orderdetails.quantityOrdered) AS "Quantité_vendue", ROUND(AVG(orderdetails.priceEach),2) AS "Prix_vente_moyen", 
ROUND(AVG(products.buyPrice),2) AS "Prix_achat_moyen", 
ROUND(AVG(orderdetails.priceEach),2) - ROUND(AVG(products.buyPrice),2) AS "Marge_brute_each",
ROUND(AVG(products.MSRP),2) AS "MSRP_moyen",
ROUND(AVG(orderdetails.priceEach),2) - ROUND(AVG(products.MSRP),2) AS "diff_msrp_px_vente"
FROM    products
JOIN    orderdetails ON orderdetails.productCode = products.productCode
JOIN    orders ON orders.orderNumber = orderdetails.orderNumber
GROUP BY     products.productLine
ORDER BY    Quantité_vendue DESC;
```
**Taux de recouvrement des créances par client 💰**
```
#  Taux de recouvrement des créances par client :

SELECT customers.customerName
, SUM(DISTINCT orderdetails.quantityOrdered * orderdetails.priceEach) AS "Montant des commandes"
, SUM(DISTINCT payments.amount) AS "Montant des paiements"
, SUM(DISTINCT orderdetails.quantityordered*orderdetails.priceeach) - SUM(DISTINCT payments.amount) AS "créance"
, ROUND((SUM(DISTINCT payments.amount) / SUM(DISTINCT orderdetails.quantityordered*orderdetails.priceeach))*100,2) AS "taux_recouvrement"
, customers.creditLimit
, ROUND((SUM(DISTINCT orderdetails.quantityordered*orderdetails.priceeach) - SUM(DISTINCT payments.amount)) / customers.creditLimit*100,2) AS "taux_crédit"
FROM orderdetails
JOIN orders on orderdetails.ordernumber = orders.ordernumber
JOIN customers on orders.customerNumber = customers.customerNumber
JOIN payments on payments.customerNumber = customers.customerNumber
GROUP BY customers.customerName, customers.creditLimit
ORDER BY créance desc;
```
**Stock sous seuil critique🚨**
```
# Stock des produits sous seuil critique : Identifier les produits dont le stock est faible pour éviter les ruptures.

WITH MonthlySales AS (
    SELECT YEAR(orders.orderDate) AS year,MONTH(orders.orderDate) AS month,products.productCode, products.productName,products.productLine,SUM(quantityOrdered) AS totalQuantity
    FROM products
    JOIN orderdetails ON orderdetails.productCode = products.productCode
    JOIN orders ON orders.orderNumber = orderdetails.orderNumber
    GROUP BY year, month, products.productCode,products.productName, products.productLine
),
AverageMonthlySales AS (
    SELECT productName,productLine, AVG(totalQuantity) AS avgMonthlyQuantity
    FROM MonthlySales
    GROUP BY productName, productLine
),
CriticalThreshold AS (
    SELECT productName,productLine, avgMonthlyQuantity * 0.50 AS criticalThreshold -- 10% des ventes mensuelles moyennes
    FROM AverageMonthlySales
),
CurrentStock AS (
    SELECT productName, productLine, quantityInStock
    FROM products 
)
SELECT
    c.productName, c.productLine, c.quantityInStock, ROUND(ct.criticalThreshold,0) AS SeuilCritique,
    CASE
        WHEN c.quantityInStock < ct.criticalThreshold THEN 'ALERTE ROUGE'
        ELSE 'Stock OK'
    END AS stockStatus
FROM CurrentStock c
JOIN CriticalThreshold ct ON c.productName = ct.productName AND c.productLine = ct.productLine
ORDER BY c.productName;
```

[>> fichier répertoriant toutes les requêtes SQL réalisées dans le cadre de ce projet <<](https://github.com/AntoineCorrignan/Projet-1-Dashboard-analyse-de-ventes-SQL-Power-BI-/blob/main/requ%C3%AAtes_SQL)


### 2️⃣ Partie 2 : Modélisation en Schéma Étoile (OLAP)
Afin d’optimiser la performance sous Power BI :
Transformation de la base transactionnelle (OLTP) en modèle analytique (OLAP).
Création de vues SQL pour les tables de faits et de dimensions.

**Structure de BDD de départ :**
![image](https://github.com/user-attachments/assets/900d6747-b50d-4bea-9eb1-67acb70b7e94)

**Structure de BDD "en étoile" après l'avoir retravaillée, avec deux tables de faits (la principale FACT_order, et FACT_payments), cinq tables de dimensions, et une table "mesures" :**
![image](https://github.com/user-attachments/assets/df711d70-e446-4bb5-af61-f2e6c6a2317e)


### 3️⃣ Partie 3 : Création du Tableau de Bord Power BI
Importation des vues SQL optimisées.
Construction d’un modèle de données sous forme de schéma en étoile.
Conception de visualisations dynamiques et interactives :
- Graphiques par région
- KPI en tuiles
- Cartes géographiques 🌍
- Graphiques d’évolution 📊
- Mise en place de filtres multi-critères (dates, produits, bureaux, commerciaux…)

**Page principale du dashboard "Ventes" :**
![image](https://github.com/user-attachments/assets/93a214b3-6841-4193-a106-056e6654fb26)

**Focus "Clients" :**
![image](https://github.com/user-attachments/assets/cdb814db-5e59-44d3-b76f-c11c07718190)

**Focus "Produits" :**
![image](https://github.com/user-attachments/assets/786cc953-f3ba-4405-bd0f-34d0ba33dfe5)

### 4️⃣ Partie 4 : Challenges et observations 
#### 🔧 Challenges techniques : 
- Travaillant sur Mac, j’ai dû recourir à l’installation d’une machine virtuelle (UTM) afin d’exécuter Power BI Desktop, non disponible nativement sur macOS.  
- La structure initiale de la base de données n’était pas optimisée pour la création d’un tableau de bord performant. Les relations entre les tables limitaient la possibilité de réaliser certains calculs et visuels pertinents.  
➡️ J’ai donc restructuré le modèle en distinguant des tables de faits et des tables de dimensions, en m’inspirant des principes de la méthode MERISE afin de garantir cohérence et efficacité dans l’analyse.  
#### 📈 Observations – Analyse des ventes : 
- Forte saisonnalité des ventes constatée en fin d’année, avec un pic marqué avant Noël — un phénomène attendu pour une entreprise spécialisée dans les jouets.  
- L’entreprise opère sur trois zones géographiques : Amérique du Nord, Europe et Océanie.  
➡️ Le marché américain est de loin le plus important en termes de chiffre d’affaires, suivi par la France et l’Espagne.  
- 100 % des clients ont effectué des achats répétés, témoignant d’une clientèle exclusivement récurrente, ce qui est notable dans le secteur BtoB.  
- Analyse des prix : les produits sont vendus en dessous du prix public conseillé (MSRP), ce qui s’explique probablement par la pratique de tarifs dégressifs liés aux ventes en gros.  
➡️ Malgré cela, l’entreprise dégage une marge brute moyenne de près de 40 %, confirmant une stratégie tarifaire rentable.  
- Tous les produits restent profitables, bien que le taux de marge varie selon les références. Aucun article ne présente de performance déficitaire.  
- Deux clients se démarquent par leur poids stratégique dans le chiffre d’affaires et leur régularité de commandes.  
➡️ Il serait pertinent de mettre en place des actions spécifiques pour valoriser leur fidélité et sécuriser leur relation commerciale (suivi dédié, avantages exclusifs, conditions préférentielles…).  


## 📊 Livrables
📦 Fichier Power BI (.pbix) avec tableau de bord interactif et actualisable.
📜 Script SQL complet (requêtes + création des vues).
📖 README explicatif du projet et de la méthodologie (ce document).

# 📌 Conclusion
Ce projet permet de valoriser les données de l’entreprise en les rendant accessibles et lisibles via un outil puissant et interactif. Il répond aux enjeux de pilotage quotidien, de détection des anomalies, et d’optimisation des performances commerciales, financières et logistiques.
