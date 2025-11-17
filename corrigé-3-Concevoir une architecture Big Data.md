Bonjour à toutes et à tous ! Ce TP est une excellente occasion de vous confronter à la complexité de la conception d'architectures Big Data dans un contexte métier riche. Vous allez explorer comment des choix architecturaux fondamentaux peuvent impacter la capacité d'une entreprise comme OmniShop à innover et à optimiser ses opérations.

Voici deux propositions de solutions, chacune adoptant une approche légèrement différente pour répondre aux besoins d'OmniShop, tout en intégrant les paradigmes modernes du Big Data.

---

# Solution 1 : Architecture Lambda avec Data Lake Centralisé

Cette première solution propose une architecture Lambda, reconnue pour sa capacité à gérer à la fois les traitements en temps réel et les analyses historiques complètes, adossée à un Data Lake centralisé pour une gestion flexible et évolutive des données brutes.

## Chapitre 1 : Architecture Big Data pour OmniShop

### Introduction

OmniShop, une chaîne de magasins de détail omnicanal, fait face au défi d'exploiter un volume croissant de données hétérogènes pour améliorer l'expérience client, optimiser ses opérations et stimuler ses ventes. L'objectif de cette architecture est de fournir une plateforme robuste et évolutive, capable de transformer ces données brutes en informations actionnables pour la personnalisation, la détection de fraude, l'optimisation d'inventaire, l'analyse marketing et la prédiction du churn client.

### Description de l'Architecture Proposée

Nous proposons une **Architecture Lambda** s'appuyant sur un **Data Lake** centralisé. Ce choix permet de concilier la nécessité d'analyses en temps réel (pour la personnalisation et la fraude) avec la robustesse et la précision des traitements batch sur l'historique complet des données (pour l'optimisation d'inventaire, le marketing et le churn).

#### Schéma Architectural


```mermaid
graph TD
    subgraph Sources de Données
        A["Transactions Ventes (POS/E-commerce")] --> B
        C["Comportement E-commerce (Logs)"] --> B
        D[Données Inventaire] --> B
        E[Données Clients] --> B
        F[Réseaux Sociaux] --> B
        G[Capteurs Magasin] --> B
    end

    subgraph Couche d'Ingestion
        B -- Temps Réel --> H(Kafka / Kinesis)
        B -- Batch --> I(Apache NiFi / AWS DMS)
    end

    subgraph Couche de Stockage (Data Lake)
        H --> J[Raw Zone (S3 / ADLS Gen2)]
        I --> J
        J --> K[Curated Zone (S3 / ADLS Gen2)]
    end

    subgraph Couche de Traitement
        H -- Speed Layer --> L(Spark Streaming / Flink)
        K -- Batch Layer --> M(Spark Batch / Databricks)
    end

    subgraph Couche de Service
        L --> N(NoSQL DB - Cassandra / DynamoDB)
        M --> O(Data Warehouse - Snowflake / Redshift)
        M --> P(Search Engine - Elasticsearch)
    end

    subgraph Couche de Consommation
        N --> Q[APIs Temps Réel (Recommandations, Alertes Fraude)]
        O --> R[BI / Reporting (Tableau, Power BI)]
        O --> S[Modèles ML (Churn, Inventaire)]
        P --> T[Analyse Texte / Sentiment]
    end

    Q --- U(Applications Métier / E-commerce)
    R --- U
    S --- U
    T --- U
```


#### Description des Couches et Composants

1.  **Couche d'Ingestion :**
    *   **Temps Réel :** Utilisation de **Kafka** (ou AWS Kinesis) pour collecter les événements à haute vélocité (transactions, clics e-commerce, données capteurs). Ces systèmes de messagerie distribués garantissent une ingestion fiable et une faible latence.
    *   **Batch :** Utilisation d'**Apache NiFi** (ou AWS DMS) pour l'ingestion de données plus volumineuses ou moins fréquentes (données d'inventaire complètes, mises à jour de profils clients) depuis des bases de données ou des fichiers.

2.  **Couche de Stockage (Data Lake) :**
    *   **Raw Zone :** Les données brutes, telles qu'ingérées, sont stockées dans un format non modifié (ex: JSON, CSV, Parquet) sur **Amazon S3** ou **Azure Data Lake Storage Gen2**. Cette zone sert de source unique de vérité et permet de rejouer les traitements si nécessaire.
    *   **Curated Zone :** Les données sont nettoyées, transformées, enrichies et structurées (ex: format Parquet optimisé) pour faciliter les analyses ultérieures. Cette zone est la base des traitements batch.

3.  **Couche de Traitement :**
    *   **Speed Layer (Temps Réel) :** **Spark Streaming** (ou Apache Flink) traite les flux de données en temps réel depuis Kafka pour générer des vues agrégées ou des détections immédiates (ex: calcul de scores de fraude, personnalisation des recommandations). Les résultats sont stockés dans une base de données NoSQL.
    *   **Batch Layer (Historique) :** **Spark Batch** (ou Databricks) traite les données de la Curated Zone du Data Lake. Il effectue des analyses complexes, des entraînements de modèles de Machine Learning (prédiction du churn, optimisation d'inventaire) et des agrégations pour les rapports. Les résultats sont stockés dans un Data Warehouse ou un moteur de recherche.

4.  **Couche de Service :**
    *   **Base de Données NoSQL (Cassandra / DynamoDB) :** Pour les vues en temps réel générées par la Speed Layer, offrant une faible latence pour les requêtes.
    *   **Data Warehouse (Snowflake / Redshift) :** Pour les vues agrégées et les résultats des traitements batch, optimisé pour les requêtes analytiques complexes et les rapports.
    *   **Moteur de Recherche (Elasticsearch) :** Pour l'indexation et la recherche rapide sur les données textuelles (réseaux sociaux, avis clients) ou pour des analyses ad-hoc.

5.  **Couche de Consommation :**
    *   **APIs Temps Réel :** Exposer les recommandations personnalisées et les alertes de fraude aux applications e-commerce et aux systèmes en magasin.
    *   **Outils de BI / Reporting :** Tableau, Power BI pour les analyses de performance marketing et les tableaux de bord opérationnels.
    *   **Modèles ML :** Intégration des modèles de prédiction du churn et d'optimisation d'inventaire dans les systèmes métier.
    *   **Analyse Texte / Sentiment :** Outils spécifiques pour explorer les données des réseaux sociaux.

### Justification des Choix Architecturaux

*   **Paradigme Lambda :**
    *   **Pertinence pour OmniShop :** OmniShop a des besoins clairs en temps réel (personnalisation, fraude) qui nécessitent une faible latence, mais aussi des besoins d'analyse historique précise et complète (optimisation d'inventaire, churn, marketing) qui bénéficient de la re-calculabilité du batch layer. L'architecture Lambda est un excellent compromis, offrant la vitesse sans sacrifier la précision.
    *   **Avantages :** Haute disponibilité, tolérance aux pannes, capacité à rejouer les données brutes pour corriger les erreurs ou recalculer des vues, précision des résultats batch.
    *   **Inconvénients :** Complexité de maintenance due à la duplication de la logique de traitement entre les couches speed et batch.

*   **Data Lake :**
    *   **Pertinence pour OmniShop :** OmniShop gère une grande variété de données (structurées, semi-structurées, non structurées) provenant de nombreuses sources. Le Data Lake permet de stocker ces données brutes à faible coût, sans schéma prédéfini, offrant une flexibilité maximale pour les analyses futures et l'exploration.
    *   **Avantages :** Stockage économique et scalable, support de tous types de données, flexibilité pour l'évolution des besoins.
    *   **Inconvénients :** Nécessite une bonne gouvernance pour éviter de devenir un "data swamp".

*   **Technologies Clés Envisagées :**
    *   **Kafka / Kinesis :** Leaders du marché pour l'ingestion de flux de données à haute performance, garantissant la durabilité et la scalabilité.
    *   **S3 / ADLS Gen2 :** Solutions de stockage objet cloud hautement scalables, durables et économiques, idéales pour un Data Lake.
    *   **Spark (Streaming/Batch) :** Framework de traitement unifié, puissant et polyvalent, capable de gérer des volumes massifs de données en batch et en streaming, avec une large communauté et un écosystème riche.
    *   **Cassandra / DynamoDB :** Bases de données NoSQL distribuées, offrant une faible latence et une haute disponibilité pour les vues en temps réel.
    *   **Snowflake / Redshift :** Data Warehouses cloud-native, optimisés pour l'analyse et le reporting, avec une excellente scalabilité et performance.

*   **Réponse aux Besoins Métier :**
    *   **Personnalisation / Fraude :** La Speed Layer (Spark Streaming + NoSQL DB + APIs) permet des recommandations et des détections en temps réel.
    *   **Optimisation Inventaire / Churn / Marketing :** La Batch Layer (Spark Batch + Data Warehouse) permet des analyses complexes, des entraînements de modèles ML et des rapports précis sur l'historique complet.

### Gestion des Données

*   **Gouvernance :** Mise en place d'un Data Catalog (ex: Apache Atlas, AWS Glue Data Catalog) pour documenter les sources, les schémas et les transformations. Définition de propriétaires de données pour chaque domaine.
*   **Sécurité :** Chiffrement des données au repos et en transit. Gestion fine des accès (IAM sur AWS/Azure) basée sur le principe du moindre privilège. Audits de sécurité réguliers.
*   **Qualité :** Mise en place de contrôles de qualité des données à chaque étape du pipeline (ingestion, transformation). Alertes en cas de dégradation de la qualité.
*   **Conformité (RGPD) :** Anonymisation/pseudonymisation des données personnelles dès que possible. Gestion des consentements clients. Mise en place de politiques de rétention des données.

### Considérations Opérationnelles

*   **Scalabilité :** L'architecture est conçue pour être cloud-native, tirant parti de l'élasticité des services cloud (auto-scaling des clusters Spark, stockage S3/ADLS illimité) pour s'adapter à la croissance des données et des utilisateurs.
*   **Résilience :** Les composants sont distribués et tolérants aux pannes (Kafka, Spark). Des plans de reprise d'activité (PRA) et de continuité d'activité (PCA) seront définis.
*   **Monitoring :** Mise en place d'outils de monitoring (Prometheus, Grafana, CloudWatch, Azure Monitor) pour superviser la performance des composants, la latence des pipelines et la qualité des données.
*   **Coûts :** L'utilisation de services cloud permet une facturation à l'usage, optimisant les coûts. Cependant, la complexité de la Lambda Architecture peut entraîner des coûts de développement et de maintenance plus élevés. L'optimisation des ressources (taille des clusters, formats de stockage) sera cruciale.

### Conclusion

Cette architecture Lambda avec un Data Lake centralisé offre à OmniShop une solution robuste et performante pour exploiter ses données. Elle permet de répondre aux besoins métier critiques en temps réel et en batch, tout en posant les bases d'une plateforme évolutive et sécurisée. La clé de son succès résidera dans une implémentation rigoureuse et une gouvernance des données efficace.

---

# Solution 2 : Architecture Kappa avec Principes Data Mesh

Cette seconde solution propose une approche plus moderne et simplifiée avec une **Architecture Kappa**, qui unifie les traitements batch et temps réel sur un seul flux de données. Elle intègre également les principes du **Data Mesh** pour adresser la complexité organisationnelle et la diversité des domaines métier d'OmniShop.

## Chapitre 2 : Architecture Big Data pour OmniShop

### Introduction

OmniShop cherche à devenir une entreprise véritablement "data-driven". Pour y parvenir, il est essentiel de dépasser les silos de données et de permettre aux équipes métier de consommer et de produire des données de manière autonome. Cette architecture vise à simplifier la gestion des flux de données tout en favorisant la décentralisation et l'ownership des données par les domaines métier.

### Description de l'Architecture Proposée

Nous proposons une **Architecture Kappa** pour unifier le traitement des données, combinée aux principes du **Data Mesh** pour organiser la plateforme et les équipes autour de domaines de données.

#### Schéma Architectural


```mermaid
graph TD
    subgraph Sources de Données
        A["Transactions Ventes (POS/E-commerce)"] --> B
        C["Comportement E-commerce (Logs)"] --> B
        D[Données Inventaire] --> B
        E[Données Clients] --> B
        F[Réseaux Sociaux] --> B
        G[Capteurs Magasin] --> B
    end

    subgraph Couche d'Ingestion & Streaming Platform
        B --> H(Kafka / Confluent Platform)
    end

    subgraph Data Mesh - Domaines de Données (Data Products)
        subgraph Domaine 1: Client
            H -- Stream Client Events --> I(Data Product: Customer Profile)
            I --> J(Serving Layer: Graph DB / API)
        end
        subgraph Domaine 2: Produit & Inventaire
            H -- Stream Product & Inventory Events --> K(Data Product: Product Catalog & Stock)
            K --> L(Serving Layer: NoSQL DB / API)
        end
        subgraph Domaine 3: Ventes & Fraude
            H -- Stream Transaction Events --> M(Data Product: Sales & Fraud Events)
            M --> N(Serving Layer: Real-time Analytics / Alerts)
        end
        subgraph Domaine 4: Marketing
            H -- Stream Marketing Events --> O(Data Product: Campaign Performance)
            O --> P(Serving Layer: Data Warehouse / BI)
        end
    end

    J --> Q["Applications Métier / E-commerce (Personnalisation, Churn)"]
    L --> Q
    N --> Q["Applications Métier / E-commerce (Détection Fraude)"]
    P --> Q["Applications Métier / E-commerce (Analyse Marketing)"]
```


#### Description des Couches et Composants

1.  **Couche d'Ingestion & Streaming Platform :**
    *   **Kafka (ou Confluent Platform) :** Cœur de l'architecture. Toutes les données, qu'elles soient en temps réel ou batch, sont ingérées sous forme d'événements dans des topics Kafka. Cela inclut les transactions, les logs e-commerce, les mises à jour d'inventaire, les profils clients, etc. Kafka agit comme une source de vérité distribuée et unifiée.

2.  **Data Mesh - Domaines de Données (Data Products) :**
    *   Plutôt qu'un Data Lake centralisé, l'approche Data Mesh organise les données en **Data Products** (produits de données), chacun étant géré par une équipe métier ou un domaine spécifique (ex: Domaine Client, Domaine Produit, Domaine Ventes).
    *   Chaque Data Product est une unité autonome, responsable de la collecte, du traitement, du stockage et de l'exposition de ses données. Il consomme des événements depuis Kafka, les transforme et les expose via une **Serving Layer** (couche de service) dédiée.
    *   **Exemples de Data Products :**
        *   **Customer Profile :** Agrège l'historique d'achat, les données de fidélité, les interactions web pour fournir une vue 360° du client.
        *   **Product Catalog & Stock :** Gère les informations sur les produits et les niveaux de stock en temps réel.
        *   **Sales & Fraud Events :** Traite les transactions pour la détection de fraude et l'analyse des ventes.
        *   **Campaign Performance :** Consolide les données marketing et de vente pour évaluer l'efficacité des campagnes.

3.  **Couche de Traitement (Unifiée Kappa) :**
    *   **Apache Flink / Spark Streaming :** Ces frameworks de traitement de flux sont utilisés par chaque Data Product pour consommer les événements Kafka, appliquer des transformations, des agrégations, des enrichissements et des modèles de Machine Learning.
    *   Dans une architecture Kappa, les traitements "batch" sont réalisés en rejouant l'historique des événements depuis Kafka, ce qui simplifie la logique de code et la maintenance par rapport à la Lambda Architecture.

4.  **Couche de Service (par Data Product) :**
    *   Chaque Data Product expose ses données via une interface standardisée :
        *   **Graph Database (Neo4j / Amazon Neptune) :** Pour le Data Product "Customer Profile", idéal pour les relations complexes (clients, produits, interactions) et la personnalisation.
        *   **NoSQL Database (MongoDB / DynamoDB) :** Pour le Data Product "Product Catalog & Stock", offrant une faible latence pour les requêtes d'inventaire.
        *   **Real-time Analytics Engine (Druid / ClickHouse) ou APIs dédiées :** Pour le Data Product "Sales & Fraud Events", permettant des détections et des alertes rapides.
        *   **Data Warehouse (Snowflake / BigQuery) ou BI Tools :** Pour le Data Product "Campaign Performance", pour des analyses agrégées.

5.  **Couche de Consommation :**
    *   Les applications métier (e-commerce, outils marketing) consomment directement les Data Products via leurs Serving Layers (APIs, requêtes SQL/NoSQL).

### Justification des Choix Architecturaux

*   **Paradigme Kappa :**
    *   **Pertinence pour OmniShop :** OmniShop a des besoins qui évoluent rapidement. La Kappa Architecture simplifie le pipeline en n'ayant qu'une seule logique de traitement (stream processing). Cela réduit la complexité de développement et de maintenance, et permet une évolution plus agile des modèles et des analyses. Le "batch" est simulé par le rejeu du stream, garantissant la cohérence.
    *   **Avantages :** Simplicité de conception et de maintenance, cohérence des données (une seule source de vérité), agilité pour les évolutions.
    *   **Inconvénients :** Le rejeu de l'historique peut être coûteux en ressources pour de très grands volumes de données sur de très longues périodes.

*   **Data Mesh :**
    *   **Pertinence pour OmniShop :** Avec de multiples sources de données et des besoins métier variés, OmniShop risque de créer des silos ou un "Data Swamp" avec une approche centralisée. Le Data Mesh décentralise la propriété et la responsabilité des données aux équipes métier, qui sont les mieux placées pour comprendre et valoriser leurs données. Cela favorise l'innovation et la rapidité de mise sur le marché des nouveaux cas d'usage.
    *   **Avantages :** Scalabilité organisationnelle, ownership des données par les experts métier, promotion de la "data as a product", réduction des goulots d'étranglement.
    *   **Inconvénients :** Nécessite un changement culturel important, des investissements dans une plateforme de données self-service et une gouvernance fédérée.

*   **Technologies Clés Envisagées :**
    *   **Kafka / Confluent Platform :** La plateforme événementielle par excellence, essentielle pour une architecture Kappa et pour la communication entre Data Products.
    *   **Apache Flink / Spark Streaming :** Moteurs de traitement de flux de nouvelle génération, capables de gérer des états complexes et des fenêtres de temps, parfaits pour la logique unifiée de Kappa.
    *   **Bases de données spécialisées (Graph DB, NoSQL DB, Real-time Analytics) :** Chaque Data Product peut choisir la technologie de Serving Layer la plus adaptée à ses données et à ses cas d'usage, offrant une flexibilité maximale.

*   **Réponse aux Besoins Métier :**
    *   **Personnalisation / Churn :** Le Data Product "Customer Profile" (avec Graph DB) permet des analyses relationnelles complexes pour des recommandations et des prédictions de churn très fines.
    *   **Fraude :** Le Data Product "Sales & Fraud Events" (avec Real-time Analytics) permet une détection immédiate des comportements suspects.
    *   **Optimisation Inventaire :** Le Data Product "Product Catalog & Stock" fournit des données à jour pour les modèles de prévision de la demande.
    *   **Marketing :** Le Data Product "Campaign Performance" offre des vues consolidées pour l'analyse d'impact.

### Gestion des Données

*   **Gouvernance (Fédérée) :** Chaque Data Product est responsable de la gouvernance de ses propres données, mais une équipe centrale (Data Platform Team) fournit les outils et les standards (contrats de données, schémas).
*   **Sécurité :** La sécurité est intégrée à chaque Data Product. Les accès sont gérés au niveau des Serving Layers. Le chiffrement est appliqué sur Kafka et les stockages sous-jacents.
*   **Qualité :** Chaque équipe de Data Product est responsable de la qualité de ses données, avec des tests automatisés et des alertes.
*   **Conformité (RGPD) :** Les Data Products sont conçus avec le "Privacy by Design". La gestion des consentements est intégrée au Data Product "Customer Profile". Les données personnelles sont pseudonymisées ou anonymisées avant d'être exposées.

### Considérations Opérationnelles

*   **Scalabilité :** Le Data Mesh est intrinsèquement scalable car il décentralise la charge. Chaque Data Product peut scaler indépendamment. L'utilisation de Kafka et de moteurs de stream processing distribués garantit une haute performance.
*   **Résilience :** La nature distribuée de Kafka et des Data Products assure une haute résilience. La capacité de rejouer les événements depuis Kafka permet une récupération facile en cas d'erreur.
*   **Monitoring :** Un monitoring global de la plateforme Kafka et des outils de monitoring spécifiques à chaque Data Product seront mis en place.
*   **Coûts :** Les coûts peuvent être optimisés par l'utilisation de technologies open source (Kafka, Flink) et une gestion fine des ressources cloud. Cependant, l'investissement initial dans la mise en place de la plateforme self-service et le changement organisationnel peut être significatif.

### Conclusion

Cette architecture Kappa avec les principes du Data Mesh offre à OmniShop une voie vers une plateforme de données plus agile, résiliente et alignée sur l'organisation métier. Elle permet de simplifier les pipelines tout en favorisant l'autonomie et la responsabilité des équipes. C'est une approche moderne qui, bien que nécessitant un investissement culturel, promet une valorisation des données plus rapide et plus efficace à long terme.

---

J'espère que ces deux solutions vous éclairent sur les différentes manières d'aborder un projet d'architecture Big Data. Elles illustrent les compromis et les avantages de chaque paradigme. N'oubliez pas que le choix final dépendra toujours du contexte spécifique, de la culture d'entreprise et des ressources disponibles. Bon courage pour la suite de votre TP !