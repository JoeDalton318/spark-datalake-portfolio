# Architecture Moderne de Data Lake avec Spark, Kafka & MinIO

![Apache Spark](https://img.shields.io/badge/Apache%20Spark-4.0.1-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)
![Kafka](https://img.shields.io/badge/Apache%20Kafka-8.0-231F20?style=for-the-badge&logo=apachekafka&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-18.1-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![MinIO](https://img.shields.io/badge/MinIO-Latest-C72E49?style=for-the-badge&logo=minio&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Lab-F37626?style=for-the-badge&logo=jupyter&logoColor=white)

## Vue d'ensemble

Infrastructure Data Lake prête pour la production implémentant l'**Architecture Medallion** (Bronze/Silver/Gold) pour des workflows modernes d'ingénierie de données. Ce projet démontre des patterns de traitement de données de niveau entreprise utilisant des outils standards de l'industrie.

### Fonctionnalités clés

- **Architecture Medallion** : Couches Bronze (Brut), Silver (Nettoyé), Gold (Agrégé)
- **Streaming temps réel** : Intégration Apache Kafka avec Spark Structured Streaming
- **Traitement batch** : PySpark pour des opérations ETL/ELT à grande échelle
- **Stockage objet** : MinIO compatible S3 pour la persistance du data lake
- **Conteneurisé** : Orchestration complète Docker Compose pour des environnements reproductibles
- **Prêt pour l'analytique** : Base de données Northwind pré-chargée pour des scénarios de Business Intelligence

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   ARCHITECTURE MODERNE DE DATA LAKE                         │
└─────────────────────────────────────────────────────────────────────────────┘

    ┌──────────────┐        ┌──────────────┐        ┌──────────────┐
    │  PostgreSQL  │        │ Kafka Broker │        │    MinIO     │
    │    (OLTP)    │        │  (Streaming) │        │ (Data Lake)  │
    │    :5433     │        │    :9092     │        │  :9000/:9001 │
    └──────┬───────┘        └──────┬───────┘        └──────┬───────┘
           │                       │                       │
           │    Ingestion          │   Temps réel          │
           │                       │                       │
    ┌──────▼───────────────────────▼───────────────────────▼───────┐
    │                   COUCHE DE TRAITEMENT SPARK                 │
    │                          (JupyterLab)                        │
    │  ┌────────────────────────────────────────────────────────┐  │
    │  │  Couche Bronze  →  Couche Silver  →  Couche Gold      │  │
    │  │  (Données brutes)   (Nettoyées)      (KPIs métier)    │  │
    │  └────────────────────────────────────────────────────────┘  │
    └──────────────────────────────────────────────────────────────┘
```

### Flux de données

1. **Couche Bronze** : Ingestion de données brutes depuis PostgreSQL (JDBC) et flux Kafka
2. **Couche Silver** : Nettoyage, déduplication et dénormalisation des données (schéma en étoile)
3. **Couche Gold** : Agrégations métier, KPIs et analyse RFM pour l'analytique

## Démarrage rapide

### Prérequis

- Docker Desktop 20.10+
- Docker Compose 2.0+
- 8GB RAM minimum (16GB recommandé)
- 20GB d'espace disque libre

### Lancer l'environnement

```bash
# Cloner le dépôt
git clone https://github.com/JoeDalton318/spark-datalake-portfolio.git
cd spark-datalake-portfolio

# Démarrer tous les services
docker compose up -d

# Vérifier que les services sont actifs
docker compose ps

# Accéder à JupyterLab
# Ouvrir http://localhost:8888 dans votre navigateur
```

### Exécuter le pipeline complet

Ouvrir le notebook `notebooks/TP_Groupe3_DataLake.ipynb` dans JupyterLab et exécuter toutes les cellules séquentiellement. Le pipeline va :

1. Ingérer les tables de la base Northwind vers la couche Bronze (format Parquet)
2. Appliquer les transformations et créer une vue maître dans la couche Silver
3. Produire des événements temps réel vers Kafka et les consommer avec Spark Streaming
4. Générer les KPIs métier et la segmentation client dans la couche Gold
5. Afficher un dashboard interactif avec des visualisations matplotlib/seaborn

## Services & Points d'accès

| Service | URL | Identifiants | Usage |
|---------|-----|--------------|-------|
| **JupyterLab** | http://localhost:8888 | Pas de token requis | Environnement de développement Spark |
| **Console MinIO** | http://localhost:9001 | `minioadmin` / `minioadmin123` | Gestion du stockage S3 |
| **Kafka UI** | http://localhost:7080 | - | Monitoring des flux et gestion des topics |
| **Adminer** | http://localhost:9080 | `postgres` / `postgres` | Client de base de données (DB: `app`) |
| **PostgreSQL** | localhost:5433 | `postgres` / `postgres` | Base de données source |
| **Kafka Broker** | localhost:9092 | - | Streaming d'événements |
| **MinIO API** | localhost:9000 | - | Stockage objet compatible S3 |

## Structure du projet

```
spark-datalake-portfolio/
├── docker-compose.yml              # Orchestration de l'infrastructure
├── notebooks/
│   ├── TP_Groupe3_DataLake.ipynb  # Implémentation complète du pipeline
│   ├── exemples/                   # Exemples de démarrage
│   └── exercices/                  # Exercices pratiques
├── vol/
│   ├── jupyter/                    # Configuration Jupyter
│   └── postgresql/
│       └── northwind.sql           # Schéma de la base de données exemple
└── README.md                       # Ce fichier
```

## Stack technique

### Technologies principales

- **Apache Spark 4.0.1** : Moteur de traitement de données distribué
- **PySpark** : API Python pour Spark
- **Apache Kafka 8.0** : Plateforme de streaming d'événements temps réel
- **MinIO** : Stockage objet haute performance compatible S3
- **PostgreSQL 18.1** : Base de données OLTP source avec dataset Northwind
- **JupyterLab** : Environnement de développement interactif

### Bibliothèques Python

- **kafka-python** : Clients producteur/consommateur Kafka
- **pyspark** : APIs Spark SQL, DataFrame et Streaming
- **matplotlib/seaborn** : Visualisation de données
- **pandas** : Manipulation et analyse de données

### Fonctionnalités de traitement

- Connectivité JDBC (PostgreSQL)
- Intégration système de fichiers S3A (Hadoop AWS)
- Spark Structured Streaming
- Intégration Kafka (spark-sql-kafka)
- Format de stockage colonnaire Parquet

## Cas d'usage démontrés

### 1. Ingestion de données batch
- Extraction complète de tables depuis base OLTP
- Suivi des métadonnées (timestamp, source)
- Écritures partitionnées vers stockage objet

### 2. Transformation & Qualité des données
- Gestion des valeurs nulles et conversions de types
- Dénormalisation pour requêtes analytiques
- Champs calculés et logique métier

### 3. Traitement de flux temps réel
- Simulation de producteur Kafka (Python)
- Consommateur Spark Structured Streaming
- Agrégations micro-batch
- Tables de requêtes en mémoire

### 4. Business Intelligence
- Segmentation client (analyse RFM)
- Performance des ventes par région
- Analyse par catégorie de produits
- Monitoring des revenus en temps réel

## Détails d'implémentation du pipeline

Le notebook principal (`TP_Groupe3_DataLake.ipynb`) implémente un workflow complet d'ingénierie de données :

### Phase 1 : Couche Bronze (Ingestion de données)
- Lecture de 6 tables de la base Northwind via JDBC
- Ajout de colonnes de métadonnées techniques
- Écriture de fichiers Parquet bruts vers MinIO (`s3a://bronze/`)

### Phase 2 : Couche Silver (Transformation de données)
- Nettoyage et standardisation des types de données
- Calcul de métriques dérivées (totaux de ligne, remises)
- Création d'une vue maître dénormalisée avec jointures de 5 tables
- Stockage des données nettoyées (`s3a://silver/`)

### Phase 3 : Intégration Streaming
- Producteur Kafka Python génère des événements de ventes synthétiques
- Spark Structured Streaming consomme depuis le topic
- Agrégations temps réel par pays
- Écriture vers table en mémoire pour requêtes dashboard

### Phase 4 : Couche Gold (Analytique métier)
- Calcul du KPI de revenu total
- Top 3 des pays par volume de ventes
- Segmentation client RFM (Récence, Fréquence, Montant)
- Logique de classification VIP
- Stockage des analytiques finales (`s3a://gold/`)

### Phase 5 : Dashboard de visualisation
- Graphiques Matplotlib/Seaborn pour insights
- Diagramme en barres top 10 pays
- Diagramme circulaire segmentation client
- Performance par catégorie de produit
- Comparaison streaming temps réel

## Configuration

### Configuration de la session Spark

```python
spark = SparkSession.builder \
    .appName("Pipeline Data Lake Moderne") \
    .config("spark.jars.packages", 
            "org.postgresql:postgresql:42.6.0,"
            "org.apache.hadoop:hadoop-aws:3.4.1,"
            "com.amazonaws:aws-java-sdk-bundle:1.12.262,"
            "org.apache.spark:spark-sql-kafka-0-10_2.13:3.5.0") \
    .config("spark.hadoop.fs.s3a.endpoint", "http://minio:9000") \
    .config("spark.hadoop.fs.s3a.access.key", "minioadmin") \
    .config("spark.hadoop.fs.s3a.secret.key", "minioadmin123") \
    .config("spark.hadoop.fs.s3a.path.style.access", "true") \
    .getOrCreate()
```

### Buckets MinIO

Le pipeline utilise automatiquement les buckets suivants :
- `bronze/` - Données brutes ingérées avec métadonnées techniques
- `silver/` - Données dimensionnelles nettoyées et transformées
- `gold/` - Métriques métier agrégées et KPIs

Créer les buckets via la Console MinIO (http://localhost:9001) ou utiliser S3 CLI.

## Arrêt de l'environnement

```bash
# Arrêter les services (conserve les données)
docker compose down

# Arrêter et supprimer toutes les données (ATTENTION)
docker compose down -v
```

## Résultats d'apprentissage

Ce projet démontre des compétences en :

- Patterns modernes d'architecture data lake
- Organisation des données Medallion (Bronze/Silver/Gold)
- Traitement distribué de données avec Apache Spark
- Streaming temps réel avec Kafka et Spark Structured Streaming
- Intégration de stockage objet compatible S3
- Développement de pipelines ETL/ELT
- Techniques de qualité et transformation de données
- Business Intelligence et analytique
- Conteneurisation de plateformes data avec Docker
- Écosystème d'ingénierie de données Python

## Crédits

### Infrastructure et matériel pédagogique
**François SALMON** - Développeur Web / Formateur
- Conception de l'environnement Docker complet
- Création des exercices pédagogiques (notebooks/exercices/)
- Setup de l'infrastructure (Spark, Kafka, MinIO, PostgreSQL)
- Base de données Northwind et exemples

### Travail réalisé sur ce repository

**Gills Daryl KETCHA** - Coordination du projet et implémentation
- Réalisation du TP complet (notebook principal)
- Coordination du groupe de travail
- Documentation technique (README, ARCHITECTURE)
- Mise en place du repository GitHub

**Groupe de travail TP Data Lake - Groupe 3** :
- Gills Daryl KETCHA (Coordination & Phase Gold)
- Narcisse Cabrel TSAFACK (Phase Silver - Transformations)
- Frédéric FERNANDES DA COSTA (Phase Streaming - Kafka)
- Jennifer HOUNGBEDJI (Phase Bronze - Ingestion)

## Licence

Ce projet est sous licence MIT - voir le fichier [LICENSE](LICENSE) pour plus de détails.

Usage éducatif et portfolios professionnels encouragés.

## Remerciements

Construit avec les meilleures pratiques issues de frameworks d'ingénierie de données standards de l'industrie et inspiré par des architectures data lake réelles. Merci à François SALMON pour l'environnement pédagogique complet et la qualité des exercices proposés.
