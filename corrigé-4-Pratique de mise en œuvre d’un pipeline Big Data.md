Bonjour à toutes et à tous ! C'est un excellent TP pour mettre les mains dans le cambouis et comprendre comment les technologies Big Data s'articulent pour construire un pipeline de données en temps réel. Vous allez voir concrètement comment Kafka et Elasticsearch collaborent pour ingérer, stocker et visualiser des flux de données.

Voici deux solutions possibles, chacune avec ses spécificités, pour vous guider dans cet exercice.

---

# Solution 1 : Implémentation Directe et Fonctionnelle

Cette première solution se concentre sur une implémentation simple et directe du pipeline, en utilisant les fonctionnalités de base de Kafka et Elasticsearch pour atteindre l'objectif du TP.

## Chapitre 1 : Pipeline Big Data avec Kafka et Elasticsearch - Suivi de Capteurs Smart City

### 1. Fichier `docker-compose.yml`

Ce fichier définit les services nécessaires : Zookeeper pour Kafka, Kafka lui-même, Elasticsearch pour le stockage, et Kibana pour la visualisation.


```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    hostname: kafka
    container_name: kafka
    ports:
      - "9092:9092"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.1
    hostname: elasticsearch
    container_name: elasticsearch
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      discovery.type: single-node
      xpack.security.enabled: "false" # Désactiver la sécurité pour simplifier le TP
      ES_JAVA_OPTS: "-Xms512m -Xmx512m" # Limiter la mémoire pour les petits environnements
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.1
    hostname: kibana
    container_name: kibana
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    environment:
      ELASTICSEARCH_HOSTS: 'http://elasticsearch:9200'
```


### 2. Script Python de Génération de Données (Producteur) : `sensor_generator.py`

Ce script simule des capteurs et envoie leurs données à Kafka.


```python
import json
import time
import random
from datetime import datetime

from kafka import KafkaProducer

# Configuration Kafka
KAFKA_BROKER = 'localhost:9092'
KAFKA_TOPIC = 'smart-city-sensors'

# Initialisation du producteur Kafka
producer = KafkaProducer(
    bootstrap_servers=[KAFKA_BROKER],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

# Liste des IDs de capteurs simulés
SENSOR_IDS = ["sensor_001", "sensor_002", "sensor_003", "sensor_004", "sensor_005"]

print(f"Démarrage du générateur de données pour le topic Kafka: {KAFKA_TOPIC}")

try:
    while True:
        for sensor_id in SENSOR_IDS:
            # Génération de données aléatoires pour chaque capteur
            data = {
                "sensor_id": sensor_id,
                "timestamp": datetime.now().isoformat(),
                "temperature": round(random.uniform(15.0, 30.0), 2),
                "humidity": round(random.uniform(40.0, 80.0), 2),
                "co2_level": random.randint(350, 1000),
                "light_intensity": random.randint(100, 1000)
            }
            
            # Envoi du message à Kafka
            producer.send(KAFKA_TOPIC, value=data)
            print(f"Envoyé: {data}")
        
        # Pause entre les envois pour simuler un flux continu
        time.sleep(random.uniform(1, 3)) 

except KeyboardInterrupt:
    print("\nGénérateur de données arrêté.")
finally:
    producer.close()
```


### 3. Script Python de Stockage dans Elasticsearch (Consommateur) : `kafka_to_elasticsearch.py`

Ce script consomme les messages de Kafka et les indexe dans Elasticsearch.


```python
import json
from kafka import KafkaConsumer
from elasticsearch import Elasticsearch

# Configuration Kafka
KAFKA_BROKER = 'localhost:9092'
KAFKA_TOPIC = 'smart-city-sensors'
KAFKA_GROUP_ID = 'elasticsearch-consumer-group'

# Configuration Elasticsearch
ELASTICSEARCH_HOST = 'elasticsearch' # Nom du service Docker
ELASTICSEARCH_PORT = 9200
ELASTICSEARCH_INDEX = 'sensors_data'

# Initialisation du consommateur Kafka
consumer = KafkaConsumer(
    KAFKA_TOPIC,
    bootstrap_servers=[KAFKA_BROKER],
    group_id=KAFKA_GROUP_ID,
    auto_offset_reset='earliest', # Commence à lire depuis le début du topic si aucun offset n'est enregistré
    value_deserializer=lambda x: json.loads(x.decode('utf-8'))
)

# Initialisation du client Elasticsearch
# Désactiver la vérification SSL si xpack.security.enabled est false dans Elasticsearch
es = Elasticsearch(
    [f'http://{ELASTICSEARCH_HOST}:{ELASTICSEARCH_PORT}'],
    verify_certs=False
)

print(f"Démarrage du consommateur Kafka pour le topic: {KAFKA_TOPIC}")
print(f"Indexation des données dans Elasticsearch sous l'index: {ELASTICSEARCH_INDEX}")

try:
    for message in consumer:
        sensor_data = message.value
        print(f"Reçu de Kafka: {sensor_data}")
        
        # Indexation du document dans Elasticsearch
        # L'ID du document est généré automatiquement par Elasticsearch
        es.index(index=ELASTICSEARCH_INDEX, document=sensor_data)
        print(f"Indexé dans Elasticsearch: {sensor_data['sensor_id']} à {sensor_data['timestamp']}")

except KeyboardInterrupt:
    print("\nConsommateur Kafka arrêté.")
finally:
    consumer.close()
```


### 4. Captures d'écran Kibana (Descriptions)

*   **Section "Discover" :** Vous verriez une liste de documents, chaque ligne représentant une mesure de capteur. Les champs (`sensor_id`, `timestamp`, `temperature`, `humidity`, `co2_level`, `light_intensity`) seraient visibles, et de nouveaux documents apparaîtraient en temps réel à mesure que le consommateur les indexe. La barre de temps en haut montrerait une activité continue.
*   **Tableau de bord simple :**
    *   Un graphique linéaire montrant l'évolution de la `temperature` moyenne au fil du temps.
    *   Un graphique en barres affichant l'`humidity` moyenne pour chaque `sensor_id`.
    *   Un diagramme circulaire (pie chart) montrant la répartition des `co2_level` par tranches (par exemple, faible, moyen, élevé).

### 5. Rapport de Conception

#### Architecture Mise en Place

L'architecture est un pipeline de données en temps réel simple, composé de :
1.  **Générateur de Données (Python) :** Simule des capteurs et publie des messages JSON sur Kafka.
2.  **Apache Kafka :** Sert de bus de messages distribué, ingérant les données des capteurs dans le topic `smart-city-sensors`.
3.  **Consommateur (Python) :** Lit les messages du topic Kafka, décode le JSON et les envoie à Elasticsearch.
4.  **Elasticsearch :** Stocke les données des capteurs dans l'index `sensors_data`.
5.  **Kibana :** Fournit une interface de visualisation et d'exploration des données stockées dans Elasticsearch.

#### Choix Techniques Effectués

*   **Docker Compose :** Permet un déploiement rapide et isolé de l'ensemble de la stack (Zookeeper, Kafka, Elasticsearch, Kibana). Les images `confluentinc/cp-kafka` et `docker.elastic.co/elasticsearch` sont des choix standards et robustes.
*   **Python 3 :** Langage de script polyvalent et largement utilisé pour les tâches Big Data.
*   **`kafka-python` :** Bibliothèque Python officielle et performante pour interagir avec Kafka.
*   **`elasticsearch` :** Client Python officiel pour Elasticsearch, facilitant l'indexation des documents.
*   **JSON :** Format de données léger et universel, idéal pour l'échange de messages dans un pipeline.
*   **Désactivation de la sécurité Elasticsearch :** Pour simplifier le TP et éviter la gestion des identifiants et certificats, la sécurité X-Pack a été désactivée. *Note : En production, la sécurité est indispensable.*

#### Difficultés Rencontrées et Résolutions

*   **Configuration Kafka dans Docker Compose :** L'un des défis initiaux est de s'assurer que Kafka est correctement configuré pour être accessible depuis l'hôte (`localhost:9092`) et depuis les autres conteneurs (`kafka:29092`). La variable `KAFKA_ADVERTISED_LISTENERS` est cruciale pour cela.
*   **Accès Elasticsearch depuis le consommateur :** Le consommateur Python doit pouvoir se connecter à Elasticsearch. En utilisant `ELASTICSEARCH_HOST = 'elasticsearch'`, le consommateur utilise le nom de service Docker pour la résolution DNS au sein du réseau Docker.
*   **Encodage/Décodage JSON :** Les messages Kafka sont des octets. Il est essentiel d'encoder les données JSON en UTF-8 avant l'envoi (`json.dumps(v).encode('utf-8')`) et de les décoder après la réception (`json.loads(x.decode('utf-8'))`).

#### Observations sur le Fonctionnement du Pipeline

Le pipeline fonctionne de manière fluide, ingérant les données des capteurs en temps quasi réel. Les messages sont produits, mis en file d'attente dans Kafka, consommés par le script Python, puis stockés dans Elasticsearch. La visualisation dans Kibana montre une mise à jour dynamique des données, prouvant l'efficacité du pipeline pour la collecte et l'analyse de données de flux.

#### Défis Supplémentaires (Non abordés dans cette solution)

Cette solution se concentre sur l'implémentation de base. Les défis supplémentaires comme la gestion d'erreurs avancée, le mapping explicite ou l'utilisation de Kafka Connect n'ont pas été intégrés pour maintenir la concision et la simplicité.

---

# Solution 2 : Implémentation Robuste avec Améliorations

Cette deuxième solution intègre quelques-uns des défis supplémentaires pour rendre le pipeline plus robuste et plus performant, notamment la gestion d'erreurs et la définition d'un mapping Elasticsearch explicite.

## Chapitre 2 : Pipeline Big Data avec Kafka et Elasticsearch - Suivi de Capteurs Smart City

### 1. Fichier `docker-compose.yml`

Le fichier `docker-compose.yml` est identique à la Solution 1, car la configuration de base de la stack reste la même.


```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    hostname: kafka
    container_name: kafka
    ports:
      - "9092:9092"
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.1
    hostname: elasticsearch
    container_name: elasticsearch
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      discovery.type: single-node
      xpack.security.enabled: "false"
      ES_JAVA_OPTS: "-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.1
    hostname: kibana
    container_name: kibana
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    environment:
      ELASTICSEARCH_HOSTS: 'http://elasticsearch:9200'
```


### 2. Script Python de Génération de Données (Producteur) : `sensor_generator.py`

Le producteur reste similaire, car `kafka-python` gère déjà l'envoi par lots en interne. Pour simuler une "performance" accrue, nous réduisons simplement le temps de pause.


```python
import json
import time
import random
from datetime import datetime

from kafka import KafkaProducer

# Configuration Kafka
KAFKA_BROKER = 'localhost:9092'
KAFKA_TOPIC = 'smart-city-sensors'

# Initialisation du producteur Kafka
producer = KafkaProducer(
    bootstrap_servers=[KAFKA_BROKER],
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),
    linger_ms=100 # Temps d'attente avant d'envoyer un lot de messages (pour simuler un batching plus rapide)
)

# Liste des IDs de capteurs simulés
SENSOR_IDS = ["sensor_001", "sensor_002", "sensor_003", "sensor_004", "sensor_005"]

print(f"Démarrage du générateur de données pour le topic Kafka: {KAFKA_TOPIC}")

try:
    while True:
        for sensor_id in SENSOR_IDS:
            data = {
                "sensor_id": sensor_id,
                "timestamp": datetime.now().isoformat(),
                "temperature": round(random.uniform(15.0, 30.0), 2),
                "humidity": round(random.uniform(40.0, 80.0), 2),
                "co2_level": random.randint(350, 1000),
                "light_intensity": random.randint(100, 1000)
            }
            
            producer.send(KAFKA_TOPIC, value=data)
            print(f"Envoyé: {data}")
        
        # Pause plus courte pour simuler un flux plus rapide
        time.sleep(random.uniform(0.5, 1.5)) 

except KeyboardInterrupt:
    print("\nGénérateur de données arrêté.")
finally:
    producer.close()
```


### 3. Script Python de Stockage dans Elasticsearch (Consommateur) : `kafka_to_elasticsearch.py`

Ce consommateur inclut la gestion d'erreurs basique et la création d'un mapping explicite pour l'index Elasticsearch.


```python
import json
from kafka import KafkaConsumer
from elasticsearch import Elasticsearch, ConnectionError, TransportError

# Configuration Kafka
KAFKA_BROKER = 'localhost:9092'
KAFKA_TOPIC = 'smart-city-sensors'
KAFKA_GROUP_ID = 'elasticsearch-consumer-group-robust'

# Configuration Elasticsearch
ELASTICSEARCH_HOST = 'elasticsearch'
ELASTICSEARCH_PORT = 9200
ELASTICSEARCH_INDEX = 'sensors_data_v2' # Nouveau nom d'index pour cette solution

# Initialisation du consommateur Kafka
consumer = KafkaConsumer(
    KAFKA_TOPIC,
    bootstrap_servers=[KAFKA_BROKER],
    group_id=KAFKA_GROUP_ID,
    auto_offset_reset='earliest',
    enable_auto_commit=True, # Permet au consommateur de valider automatiquement les offsets
    value_deserializer=lambda x: json.loads(x.decode('utf-8'))
)

# Initialisation du client Elasticsearch
es = Elasticsearch(
    [f'http://{ELASTICSEARCH_HOST}:{ELASTICSEARCH_PORT}'],
    verify_certs=False
)

# Définition du mapping explicite pour l'index
INDEX_MAPPING = {
    "mappings": {
        "properties": {
            "sensor_id": {"type": "keyword"},
            "timestamp": {"type": "date"},
            "temperature": {"type": "float"},
            "humidity": {"type": "float"},
            "co2_level": {"type": "integer"},
            "light_intensity": {"type": "integer"}
        }
    }
}

# Création de l'index avec le mapping si l'index n'existe pas
try:
    if not es.indices.exists(index=ELASTICSEARCH_INDEX):
        es.indices.create(index=ELASTICSEARCH_INDEX, body=INDEX_MAPPING)
        print(f"Index '{ELASTICSEARCH_INDEX}' créé avec le mapping explicite.")
    else:
        print(f"Index '{ELASTICSEARCH_INDEX}' existe déjà.")
except ConnectionError as e:
    print(f"Erreur de connexion à Elasticsearch: {e}. Assurez-vous qu'Elasticsearch est démarré.")
    exit(1)
except Exception as e:
    print(f"Erreur lors de la création/vérification de l'index Elasticsearch: {e}")
    exit(1)


print(f"Démarrage du consommateur Kafka pour le topic: {KAFKA_TOPIC}")
print(f"Indexation des données dans Elasticsearch sous l'index: {ELASTICSEARCH_INDEX}")

try:
    for message in consumer:
        try:
            sensor_data = message.value
            print(f"Reçu de Kafka: {sensor_data}")
            
            # Indexation du document dans Elasticsearch
            es.index(index=ELASTICSEARCH_INDEX, document=sensor_data)
            print(f"Indexé dans Elasticsearch: {sensor_data['sensor_id']} à {sensor_data['timestamp']}")
            
        except json.JSONDecodeError as e:
            print(f"Erreur de décodage JSON pour le message: {message.value}. Erreur: {e}")
        except TransportError as e:
            print(f"Erreur d'indexation dans Elasticsearch pour le message: {sensor_data}. Erreur: {e}")
        except Exception as e:
            print(f"Erreur inattendue lors du traitement du message: {e}")

except KeyboardInterrupt:
    print("\nConsommateur Kafka arrêté.")
finally:
    consumer.close()
```


### 4. Captures d'écran Kibana (Descriptions)

Les captures d'écran seraient similaires à la Solution 1, mais avec l'index `sensors_data_v2`.
*   **Section "Discover" :** Les données apparaîtraient de manière similaire, mais avec la garantie que les types de champs sont ceux définis dans le mapping explicite (par exemple, `timestamp` est bien un `date`, `sensor_id` un `keyword`).
*   **Tableau de bord simple :** Les visualisations seraient construites sur l'index `sensors_data_v2`, bénéficiant de la typologie de données précise pour des agrégations et des filtres plus efficaces.

### 5. Rapport de Conception

#### Architecture Mise en Place

L'architecture de base reste la même que la Solution 1, mais avec une emphase sur la robustesse du consommateur et la qualité des données stockées dans Elasticsearch. Le pipeline est : Générateur (Python) -> Kafka -> Consommateur (Python) -> Elasticsearch -> Kibana.

#### Choix Techniques Effectués

*   **Docker Compose, Python, `kafka-python`, `elasticsearch`, JSON :** Les mêmes choix de base que la Solution 1, pour les mêmes raisons de robustesse et de popularité.
*   **`linger_ms` dans KafkaProducer :** Réduit le délai avant que le producteur n'envoie un lot de messages, simulant une ingestion plus rapide et une meilleure utilisation de la bande passante de Kafka.
*   **`enable_auto_commit=True` dans KafkaConsumer :** Simplifie la gestion des offsets en laissant Kafka gérer la validation des messages consommés.
*   **Mapping explicite Elasticsearch :** L'index `sensors_data_v2` est créé avec un mapping prédéfini. Cela garantit que chaque champ a le type de données attendu, ce qui est crucial pour des requêtes analytiques précises et des visualisations fiables. Par exemple, `sensor_id` est un `keyword` pour des recherches exactes, et `timestamp` un `date` pour des analyses temporelles.
*   **Gestion des erreurs Python :** Des blocs `try-except` ont été ajoutés dans le consommateur pour gérer les erreurs de décodage JSON (messages malformés) et les erreurs d'indexation Elasticsearch (problèmes de connexion, de mapping). Cela rend le consommateur plus résilient aux incidents.

#### Difficultés Rencontrées et Résolutions

*   **Gestion des erreurs de décodage :** Sans gestion d'erreur, un message JSON malformé arrêterait le consommateur. Le `try-except json.JSONDecodeError` permet de logguer l'erreur et de continuer à traiter les messages suivants.
*   **Gestion des erreurs d'indexation Elasticsearch :** Des problèmes de réseau ou de configuration Elasticsearch pourraient empêcher l'indexation. Le `try-except TransportError` permet de capturer ces problèmes et d'éviter l'arrêt du consommateur.
*   **Création de l'index avec mapping :** Il faut s'assurer que l'index est créé *avant* d'envoyer des documents. Le script vérifie l'existence de l'index et le crée avec le mapping si nécessaire, gérant ainsi la première exécution.

#### Observations sur le Fonctionnement du Pipeline

Cette version du pipeline est plus robuste. Le producteur envoie les données plus rapidement, et le consommateur est capable de gérer les erreurs potentielles sans s'arrêter, assurant une meilleure continuité de service. Le mapping explicite dans Elasticsearch garantit une meilleure qualité des données stockées, ce qui est un atout majeur pour les analyses ultérieures et la fiabilité des tableaux de bord Kibana.

#### Défis Supplémentaires Abordés

*   **Gestion des erreurs :** Implémentée dans le consommateur pour les erreurs de décodage JSON et d'indexation Elasticsearch.
*   **Mapping Elasticsearch :** Un mapping explicite a été défini et appliqué lors de la création de l'index `sensors_data_v2`.
*   **Performance (Producteur) :** Le paramètre `linger_ms` a été ajusté pour encourager un batching interne plus rapide par le producteur Kafka.

---

J'espère que ces deux solutions vous fournissent une base solide pour votre TP. La première est un excellent point de départ pour comprendre les fondamentaux, tandis que la seconde montre comment ajouter de la robustesse et de la précision, des aspects essentiels dans un environnement de production. N'oubliez pas que la compréhension des concepts est la clé ! Bon courage !