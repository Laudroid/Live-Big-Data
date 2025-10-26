# Architecture Lambda : définition, fonctionnement, forces et faiblesses

## 1. Définition de l’architecture Lambda

L'architecture Lambda est un modèle de conception pour le traitement de données massives qui combine deux voies de traitement : 
- une **voie batch** pour traiter de grands volumes de données de manière fiable et complète,
- une **voie temps réel** pour traiter rapidement les données récentes pour des analyses immédiates.

Cette architecture vise à profiter des avantages des traitements batch (exactitude, exhaustivité) et temps réel (faible latence), tout en compensant leurs limites respectives.

## 2. Fonctionnement détaillé

### Les trois couches principales

1. **Couche batch (batch layer)**
   - Stocke l’ensemble des données historiques dans un système robuste (comme un Data Lake).
   - Exécute régulièrement des traitements batch (MapReduce, Spark), produisant des vues pré-agrégées ou des modèles complets.
   - Donne une réponse « exacte » mais avec une latence élevée (minutes/heures).

2. **Couche vitesse (speed layer)**
   - Traite les flux de données en temps réel avec faible latence (ex : Apache Storm, Apache Flink).
   - Produit des résultats approximatifs et partiels permettant des décisions rapides.

3. **Couche service (serving layer)**
   - Agrège les résutats batch et speed pour fournir des vues unifiées accessibles aux applications.

### Illustration du flux de données

```mermaid
flowchart LR
    A[Sources de données] --> B[Batch Layer (stockage + traitement)]
    A --> C[Speed Layer (traitement temps réel)]

    B --> D[Serving Layer]
    C --> D

    D --> E[Applications / Utilisateurs]
```

---

## 3. Forces de l’architecture Lambda

- **Résilience et précision** : la couche batch garantit des données exactes malgré la complexité des traitements.
- **Faible latence** : la couche speed permet une prise de décision rapide avec des résultats quasi immédiats.
- **Tolérance aux erreurs** : si la couche speed est imparfaite, la batch rectifie les erreurs à posteriori.
- **Adaptabilité** : prise en charge de données historiques et en streaming dans un même modèle.

---

## 4. Faiblesses et limites

- **Complexité de mise en œuvre** : double pipeline (batch + speed) à maintenir, synchroniser et monitorer.
- **Redondance de la logique** : code souvent dupliqué entre les deux couches, ce qui augmente la maintenance.
- **Coût élevé** : infrastructure et ressources nécessaires pour gérer deux systèmes parallèles.
- **Latence batch** trop élevée pour certains cas nécessitant un traitement purement temps réel.

---

## 5. Exemple d’application

Par exemple, une plateforme de e-commerce peut analyser les historiques complets d’achat en batch pour affiner des modèles de recommandation, tout en traitant en temps réel les comportements récents (clics, ajouts au panier) pour adapter immédiatement les suggestions.

---

## 6. Sources utilisées

- Nathan Marz, *Big Data: Principles and best practices of scalable realtime data systems*, 2015.
- Cloudera, *Lambda Architecture Explained*, 2023. [source](https://www.cloudera.com/tutorials/lambda-architecture.html)
- AWS Big Data Blog, *Understanding Lambda Architecture*, 2022. [source](https://aws.amazon.com/blogs/big-data/)
- Confluent, *Lambda Architecture Overview*, 2023. [source](https://www.confluent.io/blog/lambda-architecture/)

---

L'architecture Lambda reste une conception puissante pour gérer des flux massifs en combinant traitement batch exact et analyse temps réel, mais exige des efforts importants pour la maintenir et garantir la cohérence entre ses différentes composantes.