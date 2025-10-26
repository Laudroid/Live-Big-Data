# Revue détaillée des architectures Lambda et Kappa : avantages et limites

## 1. Introduction

Les architectures Lambda et Kappa sont des paradigmes clé dans la conception des systèmes Big Data modernes, chacune adressant la gestion du traitement des données massives à sa manière. Comprendre leurs mécanismes, avantages et inconvénients permet de choisir l’approche la plus adaptée à un cas d’usage donné.

---

## 2. Architecture Lambda : dualité batch et temps réel

### Fonctionnement

L’architecture Lambda combine deux voies de traitement distinctes :

- **Couche batch**: stockage massif des données brutes et exécution de traitements lourds (MapReduce, Spark) pour obtenir des résultats précis mais à latence élevée.
- **Couche speed (temps réel)**: traitement rapide des données en streaming (Apache Storm, Flink) pour fournir des résultats approximatifs mais quasi instantanés.
- **Couche serving**: fusionne les résultats des deux couches pour une vue unifiée.

### Avantages

- **Précision** : la couche batch rectifie les erreurs du traitement réel.
- **Faible latence** : la couche speed fournit des insights rapides.
- **Résilience** : double pipeline limitant les impacts d’une défaillance d’une couche.
- **Cas d’usage mixte** : compatible avec données historiques et flux récents.

### Limites

- **Complexité opérationnelle élevée** : maintenir deux pipelines distincts demande des ressources et compétences importantes.
- **Duplication de logique métier** : code souvent répliqué dans les deux couches.
- **Coûts augmentés** : infrastructure et monitoring pour deux systèmes parallèles.
- **Latence batch pouvant être trop longue** selon certains besoins.

### Exemple

Une entreprise de commerce en ligne analyse ses historiques de ventes en batch pour affiner les recommandations produits tout en traitant en temps réel les clics et achats récents pour ajuster immédiatement les suggestions.

---

## 3. Architecture Kappa : pipeline streaming unique

### Fonctionnement

L’architecture Kappa repose sur un flux de données unique ingéré dans un système de log immuable (ex : Apache Kafka). Le traitement s’effectue exclusivement en streaming, en rejouant le flux complet si besoin pour recalculer les états.

### Avantages

- **Simplicité d’architecture** : un seul pipeline à maintenir.
- **Rejouabilité native** : rejouer le log permet de recalculer ou corriger les données.
- **Latence minimale** : traitement natif en streaming.
- **Scalabilité horizontale** : adapté pour les systèmes à très forte volumétrie.

### Limites

- **Complexité algorithmique accrue** pour certains traitements batch complexes (agrégations longues, jointures lourdes).
- **Ressources potentiellement plus élevées** lors du recalcul sur de gros historiques.
- **Dépendance à la durabilité et ordonnancement du log** pour garantir la cohérence.
- **Moins mature dans certains écosystèmes** comparé à Lambda.

### Exemple

Une infrastructure IoT collecte et analyse en continu des millions de données de capteurs via Kafka et Apache Flink, permettant la détection instantanée d’anomalies avec possibilité de relecture historique via le log Kafka.

---

## 4. Comparaison synthétique

| Critère            | Architecture Lambda                      | Architecture Kappa                      |
|--------------------|----------------------------------------|---------------------------------------|
| Pipeline           | Double (batch + streaming)              | Unique streaming                      |
| Complexité         | Élevée (maintien de deux pipelines)    | Plus faible                          |
| Latence de traitement | Temps réel + batch (latence variable) | Strictement temps réel               |
| Rejouabilité       | Batch uniquement                       | Flux streaming                        |
| Cas d’usage        | Scénarios mixtes, analyses précises   | Streaming temps réel, réactivité     |
| Maintenance        | Coûts et efforts importants            | Simplification mais challenges algos |

---

## 5. Diagramme Mermaid : architectures Lambda et Kappa

```mermaid
flowchart LR
    subgraph Lambda
        A1[Sources de données] --> B1[Batch Layer]
        A1 --> C1[Speed Layer]
        B1 --> D1[Serving Layer]
        C1 --> D1
        D1 --> E1[Applications]
    end

    subgraph Kappa
        A2[Sources de données] --> B2[Log immuable (Kafka)]
        B2 --> C2[Streaming Pipeline]
        C2 --> D2[Stockage & Applications]
    end
```

---

## 6. Sources utilisées

- Nathan Marz, *Big Data: Principles and best practices of scalable realtime data systems*, 2015.
- Jay Kreps, *Questioning the Lambda Architecture*, 2014. [source](https://www.confluent.io/blog/kappa-architecture-beyond-lambda-architecture/)
- Cloudera, *Lambda Architecture Explained*, 2023. [source](https://www.cloudera.com/tutorials/lambda-architecture.html)
- Databricks, *Lambda vs Kappa Architecture*, 2023. [source](https://databricks.com/glossary/kappa-architecture)
- AWS Big Data Blog, *Streaming Analytics with Kappa Architecture*, 2023. [source](https://aws.amazon.com/blogs/big-data/streaming-analytics-kappa-architecture/)

---

Les architectures Lambda et Kappa répondent à des besoins complémentaires. Lambda s'impose dans les scénarios combinant analyses précises et réactivité, tandis que Kappa privilégie la simplicité et l'efficacité du streaming pur. Le choix dépendra du contexte métier, de la volumétrie et des exigences opérationnelles.