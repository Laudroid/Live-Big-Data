# Architecture Kappa : définition, fonctionnement, forces et faiblesses

## 1. Définition de l’architecture Kappa

L’architecture Kappa est un paradigme de traitement de données qui simplifie la gestion des big data en utilisant exclusivement un pipeline de traitement en temps réel (stream processing). Contrairement à l’architecture Lambda, elle ne distingue pas les traitements batch et streaming, mais fait reposer tout le système sur un flux de données unique ré-exécutable.

Ce modèle a été proposé pour réduire la complexité opérationnelle et éviter la duplication des couches de traitement.

## 2. Fonctionnement détaillé

### Principe clé

- Les données sont toutes ingérées dans un système de streaming distribué (ex : Apache Kafka).
- Le pipeline de traitement fonctionne exclusivement en mode streaming.
- Pour reprocesser des données historiques, le système relit les données à partir du journal de messages en conservant un historique complet.
- L’unification du traitement simplifie la maintenance et la cohérence des données.

### Illustration du flux

```mermaid
flowchart LR
    A[Sources données] --> B[Topic Kafka (log immuable)]

    B --> C[Pipeline de traitement en streaming (ex. Apache Flink, Kafka Streams)]
    
    C --> D[Stockage résultats]
    C --> E[Applications utilisateurs]
    
    style B fill:#f9f,stroke:#333,stroke-width:2px
```

---

## 3. Forces de l’architecture Kappa

- **Simplicité** : un seul pipeline de traitement à construire et maintenir, réduisant la duplication de code.
- **Rejouabilité des données** : en rejouant le flux depuis le log, on peut recalculer l’état exact à tout moment.
- **Faible latence** : traitement natif des données en streaming pour des résultats quasi instantanés.
- **Adaptation au volume grandissant** : scalabilité horizontale grâce aux technologies de streaming distribuées.

---

## 4. Faiblesses et limites

- **Risque de complexité dans le traitement en streaming** : certains algorithmes batch (ex. gros joins, agrégations complexes) peuvent être plus difficiles à implémenter en streaming.
- **Latence cumulative** : ré-exécution complète sur tout le flux peut représenter un coût important en ressources pour de très gros historiques.
- **Dépendance à la fiabilité du système de log** : le flux doit être durable et ordonné pour garantir des résultats corrects.
- **Adoption plus récente** : écosystème et outillage parfois moins mature que celui fondé sur Lambda.

---

## 5. Exemple d’application

Une plateforme de monitoring IoT avec un grand nombre de capteurs utilise un pipeline Kafka + Apache Flink pour analyser en continu les métriques. Si une dérive est détectée, la relecture des données historiques est possible via la rejouabilité du flux Kafka, ce qui permet un recalcul a posteriori sans système batch dédié.

---

## 6. Sources utilisées

- Jay Kreps, *Questioning the Lambda Architecture*, 2014. [source](https://www.confluent.io/blog/kappa-architecture-beyond-lambda-architecture/)
- Confluent, *Kappa Architecture Overview*, 2023. [source](https://www.confluent.io/blog/event-stream-processing-patterns-kafka/)
- Databricks, *Lambda vs Kappa Architecture*, 2023. [source](https://databricks.com/glossary/kappa-architecture)
- AWS Big Data Blog, *Streaming Analytics with Kappa Architecture*, 2023. [source](https://aws.amazon.com/blogs/big-data/streaming-analytics-kappa-architecture/)

---

Le modèle Kappa capitalise sur la puissance des technologies de streaming pour unifier le traitement des données en un flux continu, simplifiant ainsi la maintenance et augmentant la réactivité des systèmes Big Data. Il convient particulièrement aux environnements où la rapidité et la flexibilité du traitement en temps réel priment.