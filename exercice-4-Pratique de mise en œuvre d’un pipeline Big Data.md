Bonjour à toutes et à tous,

Ce TP a pour objectif de vous immerger dans la construction d'un pipeline Big Data simple mais fonctionnel. Vous allez mettre en pratique l'ingestion et le stockage de données en temps réel en utilisant deux composants clés de l'écosystème Big Data : Apache Kafka et Elasticsearch.

---

### **TP : Pipeline Big Data avec Kafka et Elasticsearch - Suivi de Capteurs Smart City**

**Objectif du TP :**
Mettre en œuvre un pipeline Big Data simple pour l'ingestion et le stockage de données en temps réel, en utilisant Apache Kafka et Elasticsearch.

**Contexte et Cas d'usage :**
Imaginez une initiative "Smart City" où des capteurs environnementaux sont déployés à travers la ville pour collecter des données en temps réel. Ces capteurs mesurent des paramètres comme la température, l'humidité, le niveau de CO2 et la luminosité. L'objectif est de collecter ces données de manière continue, de les stocker efficacement et de les rendre disponibles pour une analyse ultérieure ou une visualisation en temps réel.

Votre mission est de construire un prototype d'ingestion et de stockage pour ces données de capteurs.

**Architecture Cible :**

```
+---------------------+       +---------------------+       +---------------------+       +---------------------+
| Générateur de       |       | Apache Kafka        |       | Consommateur        |       | Elasticsearch       |
| Données (Python)    +-----> | (Topic: smart-city) | +---> | (Python)            +-----> | (Index: sensors_data)|
| (Simulateur Capteurs)|       |                     |       |                     |       |                     |
+---------------------+       +---------------------+       +---------------------+       +---------------------+
                                                                                                      |
                                                                                                      v
                                                                                             +---------------------+
                                                                                             | Kibana              |
                                                                                             | (Visualisation)     |
                                                                                             +---------------------+
```

**Prérequis :**

*   **Docker et Docker Compose :** Pour déployer facilement Kafka, Zookeeper, Elasticsearch et Kibana.
*   **Python 3 :** Pour les scripts de génération et de consommation de données.
*   **pip :** Le gestionnaire de paquets Python.
*   **Un éditeur de code :** VS Code, PyCharm, etc.
*   **Notions de base de JSON :** Le format de données utilisé.

---

**Étapes du TP :**

**Étape 1 : Préparation de l'environnement avec Docker Compose**

1.  Créez un fichier `docker-compose.yml` pour orchestrer les services suivants :
    *   **Zookeeper :** Nécessaire pour Kafka.
    *   **Kafka Broker :** Le cœur de votre système de messagerie.
    *   **Elasticsearch :** Le moteur de recherche et de stockage.
    *   **Kibana :** L'interface de visualisation pour Elasticsearch.
    *   *Conseil :* Vous pouvez trouver des exemples de `docker-compose.yml` pour ces services en ligne. Adaptez-les pour qu'ils fonctionnent ensemble (ports, réseaux, variables d'environnement).

2.  Lancez vos services : `docker-compose up -d`.
3.  Vérifiez que tous les conteneurs sont démarrés et accessibles (par exemple, accédez à Kibana via votre navigateur, généralement sur `http://localhost:5601`).

**Étape 2 : Génération de données de capteurs simulées (Python)**

1.  Créez un script Python (`sensor_generator.py`) qui simule des données de capteurs.
2.  Ce script doit générer des enregistrements JSON avec les champs suivants :
    *   `sensor_id` (chaîne de caractères unique, ex: "sensor_001", "sensor_002")
    *   `timestamp` (horodatage ISO 8601)
    *   `temperature` (nombre flottant, ex: 20.5, 21.3)
    *   `humidity` (nombre flottant, ex: 60.2, 61.5)
    *   `co2_level` (nombre entier, ex: 400, 450)
    *   `light_intensity` (nombre entier, ex: 500, 600)
3.  Le script doit générer une nouvelle donnée toutes les 1 à 3 secondes pour un ensemble de capteurs (par exemple, 3 à 5 capteurs différents).
4.  Pour l'instant, affichez simplement ces données dans la console pour vérifier le format.

**Étape 3 : Ingestion des données dans Kafka (Producteur)**

1.  Modifiez votre script `sensor_generator.py` pour qu'il agisse comme un producteur Kafka.
2.  Installez la bibliothèque Python `kafka-python` : `pip install kafka-python`.
3.  Configurez un `KafkaProducer` pour envoyer les messages JSON générés vers un topic Kafka que vous nommerez `smart-city-sensors`.
4.  Assurez-vous que les données JSON sont encodées en UTF-8 avant d'être envoyées à Kafka.
5.  Lancez le script et vérifiez que les messages sont bien envoyés (vous pouvez utiliser des outils comme `kafka-console-consumer` depuis le conteneur Kafka pour vérifier).

**Étape 4 : Stockage des données dans Elasticsearch (Consommateur)**

1.  Créez un nouveau script Python (`kafka_to_elasticsearch.py`) qui agira comme un consommateur Kafka.
2.  Installez la bibliothèque Python elasticsearch : `pip install "elasticsearch<9,>=8"`
`.
3.  Configurez un `KafkaConsumer` pour lire les messages du topic `smart-city-sensors`.
4.  Pour chaque message reçu :
    *   Décodez le message JSON.
    *   Connectez-vous à votre instance Elasticsearch.
    *   Indexez le document dans un index Elasticsearch que vous nommerez `sensors_data`. Laissez Elasticsearch gérer le mapping des champs automatiquement pour ce TP.
    *   *Conseil :* L'ID du document Elasticsearch peut être généré automatiquement par Elasticsearch, ou vous pouvez utiliser une combinaison de `sensor_id` et `timestamp` pour créer un ID unique si vous le souhaitez.
5.  Lancez ce script *après* avoir démarré votre producteur.

**Étape 5 : Visualisation avec Kibana**

1.  Accédez à l'interface de Kibana (généralement `http://localhost:5601`).
2.  Dans la section "Stack Management" (ou "Management"), créez un "Index Pattern" pour votre index `sensors_data`. Utilisez `sensors_data*` comme motif d'index.
3.  Sélectionnez le champ `timestamp` comme champ de temps pour votre index pattern.
4.  Allez dans la section "Discover" et vérifiez que les données de vos capteurs apparaissent et sont mises à jour en temps réel.
5.  Amusez-vous à créer un tableau de bord simple ("Dashboard") :
    *   Visualisez la température moyenne au fil du temps.
    *   Affichez l'humidité actuelle pour chaque capteur.
    *   Créez une visualisation de la répartition des niveaux de CO2.

---

**Défis supplémentaires (Optionnel - pour les plus rapides ou curieux) :**

*   **Gestion des erreurs :** Que se passe-t-il si un message Kafka est mal formé ? Implémentez une gestion d'erreur basique dans le consommateur.
*   **Mapping Elasticsearch :** Définissez un mapping explicite pour votre index `sensors_data` au lieu de laisser Elasticsearch le deviner.
*   **Performance :** Modifiez le producteur pour envoyer des lots de messages (batching) au lieu d'un message à la fois.
*   **Kafka Connect :** Recherchez comment Kafka Connect (avec un connecteur Elasticsearch) pourrait simplifier l'Étape 4 sans écrire de code Python.

---

**Rendu du TP :**

Vous devrez fournir les éléments suivants :

1.  Le fichier `docker-compose.yml` utilisé.
2.  Les scripts Python (`sensor_generator.py`, `kafka_to_elasticsearch.py`).
3.  Une ou plusieurs captures d'écran de Kibana montrant :
    *   Les données dans la section "Discover".
    *   Un tableau de bord simple que vous avez créé.
4.  Un court rapport (format PDF ou Markdown) décrivant :
    *   L'architecture mise en place.
    *   Les choix techniques effectués (bibliothèques Python, configuration Docker, etc.).
    *   Les difficultés rencontrées et comment vous les avez résolues.
    *   Vos observations sur le fonctionnement du pipeline.
    *   Si vous avez abordé les défis supplémentaires, décrivez votre approche.

---

**Conseils pour l'utilisation de l'Intelligence Artificielle (IA) :**

L'IA est un outil puissant qui peut vous accompagner dans ce TP. N'hésitez pas à l'utiliser de manière intelligente :

*   **Génération de boilerplate :** Demandez-lui de vous aider à démarrer avec le `docker-compose.yml` ou la structure de base des scripts Python.
*   **Recherche de syntaxe :** Si vous bloquez sur la syntaxe d'une fonction `kafka-python` ou `elasticsearch-py`, l'IA peut vous fournir des exemples.
*   **Débogage :** Si vous rencontrez une erreur, copiez le message d'erreur et demandez à l'IA de vous aider à comprendre la cause et à suggérer des solutions.
*   **Explication de concepts :** Si un concept comme les "groupes de consommateurs Kafka" ou le "mapping Elasticsearch" vous semble flou, l'IA peut vous donner des explications claires.

**Cependant, gardez à l'esprit :**

*   **Compréhension avant tout :** Ne vous contentez pas de copier-coller. Assurez-vous de comprendre *pourquoi* le code fonctionne et *comment* il s'intègre dans l'architecture.
*   **Adaptation :** Les solutions générées par l'IA sont souvent génériques. Vous devrez les adapter précisément à votre contexte (noms de topics, ports, noms d'index, etc.).
*   **Vérification :** L'IA peut parfois générer du code erroné ou obsolète. Testez toujours ce qu'elle vous propose et vérifiez la documentation officielle si vous avez un doute.
*   **Votre rapport :** Le rapport est l'occasion de montrer votre compréhension et votre démarche, même si l'IA vous a aidé. Expliquez vos choix et votre raisonnement.

Amusez-vous bien à construire ce pipeline ! C'est une excellente occasion de voir comment ces technologies s'articulent pour gérer des flux de données.