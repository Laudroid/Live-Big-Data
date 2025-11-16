Bonjour à toutes et à tous,

Ce TP a pour but de vous immerger dans la conception d'architectures Big Data, une compétence clé pour tout professionnel du domaine. Vous travaillerez en groupe pour relever un défi métier concret.

---

## TP : Conception d'une Architecture Big Data pour l'Analyse Retail Omnicanal

### Objectif du TP

Concevoir une architecture Big Data robuste et évolutive, capable de répondre à des besoins métier spécifiques, en intégrant et justifiant l'utilisation des paradigmes architecturaux modernes.

### Contexte et Énoncé

Une grande chaîne de magasins de détail, **"OmniShop"**, souhaite transformer son approche de la donnée pour améliorer l'expérience client, optimiser ses opérations et stimuler ses ventes. OmniShop opère à la fois via des magasins physiques et une plateforme e-commerce.

**Sources de données disponibles :**

*   **Transactions de vente :** En ligne et en magasin (produits achetés, prix, date, heure, magasin/canal, client ID).
*   **Comportement utilisateur e-commerce :** Clics, pages vues, produits ajoutés au panier, recherches (logs web, événements).
*   **Données d'inventaire :** Stock en temps réel par magasin et entrepôt.
*   **Données clients :** Profils, historique d'achats, adhésion au programme de fidélité.
*   **Interactions sur les réseaux sociaux :** Mentions de la marque, avis sur les produits (données non structurées).
*   **Données des capteurs en magasin :** Flux de clients, temps d'attente aux caisses (données de série temporelle).

**Besoins métier d'OmniShop :**

1.  **Personnalisation de l'expérience client :** Recommandations de produits en temps réel sur le site web et via des notifications push en magasin.
2.  **Détection de la fraude :** Identifier les comportements d'achat suspects (en ligne et en magasin) le plus rapidement possible.
3.  **Optimisation de l'inventaire :** Prévoir la demande pour chaque produit et optimiser les niveaux de stock pour minimiser les ruptures et les surstocks.
4.  **Analyse de la performance des campagnes marketing :** Mesurer l'impact des promotions et des publicités sur les ventes et le comportement client.
5.  **Prédiction du churn client :** Identifier les clients à risque de ne plus acheter et proposer des actions de rétention ciblées.

**Votre mission :**

Proposer une architecture Big Data complète et détaillée pour OmniShop. Votre proposition devra être capable de collecter, traiter, stocker et analyser ces données afin de répondre aux besoins métier énoncés.

Votre architecture devra intégrer et justifier le choix d'un ou plusieurs des paradigmes suivants :
*   **Architecture Lambda**
*   **Architecture Kappa**
*   **Data Lake**
*   **Data Mesh**

Vous devrez expliquer pourquoi le ou les paradigmes choisis sont les plus adaptés aux exigences d'OmniShop, en tenant compte des volumes, des vitesses, des variétés de données et des besoins d'analyse (batch et temps réel).

### Livrables

Chaque groupe devra soumettre :

1.  **Un Rapport de Conception (document écrit) :**
    *   **Introduction :** Présentation du besoin métier d'OmniShop et des objectifs de l'architecture.
    *   **Description de l'Architecture Proposée :**
        *   Un **schéma architectural clair et détaillé** (obligatoire), illustrant les différents composants et les flux de données.
        *   Une description textuelle de chaque couche ou composant (ingestion, stockage, traitement, consommation).
    *   **Justification des Choix Architecturaux :**
        *   Explication du ou des paradigmes (Lambda, Kappa, Data Lake, Data Mesh) retenus et pourquoi ils sont pertinents pour OmniShop.
        *   Justification des technologies clés envisagées pour chaque composant (ex: Kafka pour l'ingestion, Spark pour le traitement, S3/HDFS pour le stockage, etc. – *pas d'implémentation, juste des exemples de technologies pertinentes*).
        *   Comment l'architecture répond aux besoins métier spécifiques (personnalisation, fraude, etc.).
    *   **Gestion des Données :** Approche pour la gouvernance, la sécurité, la qualité et la conformité des données.
    *   **Considérations Opérationnelles :** Scalabilité, résilience, monitoring, et une estimation qualitative des coûts.
    *   **Conclusion :** Synthèse et perspectives.

2.  **Une Présentation Orale :**
    *   Une synthèse de votre rapport de conception (15-20 minutes par groupe), suivie d'une session de questions-réponses.

### Critères d'évaluation

*   **Compréhension du besoin métier :** L'architecture proposée répond-elle efficacement aux problèmes d'OmniShop ?
*   **Pertinence et justification des choix architecturaux :** La sélection des paradigmes (Lambda, Kappa, Data Lake, Data Mesh) et des technologies est-elle logique et bien argumentée ?
*   **Cohérence et complétude de l'architecture :** L'architecture couvre-t-elle l'ensemble du cycle de vie de la donnée (ingestion, stockage, traitement, consommation) de manière cohérente ?
*   **Qualité du rapport et de la présentation :** Clarté, structure, professionnalisme, qualité du schéma architectural.
*   **Originalité et innovation :** La proposition intègre-t-elle des idées novatrices ou des approches particulièrement astucieuses ?

### Ressources et Outils (Utilisation de l'IA)

Vous disposez de vos cours, de la documentation technique en ligne et de l'ensemble des ressources disponibles sur internet.

**L'utilisation d'outils d'IA générative (ChatGPT, Bard, Copilot, etc.) est non seulement autorisée mais encouragée.** Ces outils peuvent être de précieux assistants pour :
*   **Brainstorming initial :** Générer des idées pour les composants ou les approches architecturales.
*   **Clarification de concepts :** Obtenir des explications sur des technologies ou des paradigmes spécifiques.
*   **Comparaison de technologies :** Demander des avantages et inconvénients de différentes options.
*   **Aide à la rédaction :** Structurer des sections du rapport, reformuler des phrases.

**Cependant, l'IA est un assistant, pas un substitut à votre réflexion critique.** Vous êtes responsables de la validité, de la pertinence et de la cohérence de votre production finale. Toute information générée par l'IA doit être vérifiée, critiquée et intégrée de manière réfléchie dans votre travail. Indiquez clairement si et comment vous avez utilisé l'IA dans votre processus de travail (par exemple, dans une section "Méthodologie" ou "Remerciements" de votre rapport).

### Conseils pour la réussite

*   **Commencez par les besoins métier :** Chaque composant de votre architecture doit servir un objectif métier clair.
*   **Travaillez en équipe :** La conception d'architecture est un effort collaboratif.
*   **Dessinez tôt et souvent :** Un bon schéma vaut mille mots. N'hésitez pas à itérer sur votre diagramme.
*   **Pensez aux compromis :** Il n'y a pas d'architecture parfaite. Chaque choix implique des avantages et des inconvénients (coût, complexité, performance, latence). Justifiez vos arbitrages.
*   **Concentrez-vous sur le "pourquoi" :** Expliquez toujours la raison derrière vos choix, pas seulement le "quoi".

Bon courage à tous !