# Documentation d'Architecture

## Vue d'ensemble du système

Ce projet implémente une architecture moderne de Data Lake basée sur le pattern Medallion, intégrant des capacités de traitement batch et streaming à travers Apache Spark et Kafka.

## Principes de conception

### 1. Architecture Medallion (Bronze-Silver-Gold)

**Couche Bronze - Zone de données brutes**
- Objectif : Zone d'atterrissage immuable pour toutes les données sources
- Format : Parquet (stockage colonnaire pour requêtage efficace)
- Schéma : Schéma source + métadonnées techniques (_ingestion_timestamp, _source)
- Emplacement : `s3a://bronze/{nom_table}/{date}`
- Qualité : Aucune transformation, copie exacte des données sources

**Couche Silver - Zone de données nettoyées**
- Objectif : Données de haute qualité, conformes, prêtes pour l'analytique
- Transformations :
  - Standardisation des types de données
  - Gestion des valeurs nulles et validation
  - Application des règles métier
  - Dénormalisation pour la performance
- Format : Parquet avec schéma optimisé
- Emplacement : `s3a://silver/{entité}/{date}`
- Qualité : Validées, nettoyées et enrichies

**Couche Gold - Agrégations métier**
- Objectif : Métriques pré-calculées et KPIs
- Transformations :
  - Agrégations et consolidations
  - Segmentation client (RFM)
  - Indicateurs de performance
- Format : Parquet optimisé pour outils BI
- Emplacement : `s3a://gold/{métrique}/{date}`
- Qualité : Prêtes pour l'analytique, définies par le métier

### 2. Composants de l'Architecture Lambda

**Traitement Batch (Spark SQL)**
- Analyse de données historiques
- Transformations et jointures complexes
- Workflows ETL planifiés
- Scans complets de tables et agrégations

**Traitement Streaming (Spark Structured Streaming)**
- Ingestion d'événements temps réel
- Traitement micro-batch
- Calculs avec état
- Dashboards à faible latence

**Couche de service**
- Tables en mémoire pour requêtes temps réel
- Fichiers Parquet pour analyse historique
- Vues combinées pour analytique unifiée

## Diagramme de flux de données

```
┌────────────────────────────────────────────────────────────────────────┐
│                         SOURCES DE DONNÉES                             │
├─────────────────────────────┬──────────────────────────────────────────┤
│   BATCH (PostgreSQL)        │    STREAMING (Kafka)                     │
│   - Base OLTP Northwind     │    - Événements ventes temps réel        │
│   - 6 tables (normalisées)  │    - Payloads JSON                       │
└─────────────┬───────────────┴──────────────┬───────────────────────────┘
              │                               │
              │  Lecture JDBC                 │  Consommateur Kafka
              │                               │
┌─────────────▼───────────────────────────────▼───────────────────────────┐
│                      COUCHE BRONZE (BRUT)                               │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Fichiers Parquet : orders, order_details, products,          │     │
│  │                     customers, employees, categories           │     │
│  │  + Métadonnées : _ingestion_timestamp, _source                │     │
│  └────────────────────────────────────────────────────────────────┘     │
└─────────────────────────────┬───────────────────────────────────────────┘
                              │
                              │  Transformations PySpark
                              │
┌─────────────────────────────▼───────────────────────────────────────────┐
│                     COUCHE SILVER (NETTOYÉE)                            │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Vue Maître Ventes (Dénormalisée)                             │     │
│  │  - Jointure 5 tables (orders → details → products →           │     │
│  │                       categories → customers)                  │     │
│  │  - Champs calculés : line_total, pays standardisé             │     │
│  │  - Contrôles qualité : gestion nulles, casting types          │     │
│  └────────────────────────────────────────────────────────────────┘     │
└─────────────────────────────┬───────────────────────────────────────────┘
                              │
                              │  Agrégations & Analytique
                              │
┌─────────────────────────────▼───────────────────────────────────────────┐
│                       COUCHE GOLD (ANALYTIQUE)                          │
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────┐ │
│  │  Dashboard KPI      │  │   Analyse RFM       │  │  Vue Streaming  │ │
│  │                     │  │                     │  │                 │ │
│  │ - Revenu total      │  │ - Récence           │  │ - Ventes live   │ │
│  │ - Top 3 pays        │  │ - Fréquence         │  │ - Par pays      │ │
│  │ - Ventes catégorie  │  │ - Montant           │  │ - Temps réel    │ │
│  │                     │  │ - Segmentation      │  │                 │ │
│  └─────────────────────┘  └─────────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

## Détails du stack technique

### Apache Spark 4.0.1
- **Spark SQL** : Moteur de requête déclaratif pour traitement batch
- **Spark Structured Streaming** : Traitement de flux scalable avec APIs SQL
- **API DataFrame** : Abstraction haut niveau pour manipulation de données distribuées
- **Optimiseur Catalyst** : Optimisation de requêtes pour la performance
- **Moteur Tungsten** : Améliorations efficacité mémoire et CPU

### Apache Kafka 8.0 (Plateforme Confluent)
- **Topic** : `ventes-temps-reel` (ventes temps réel)
- **Partitions** : 1 (configuration mono-nœud)
- **Réplication** : 1 (mode développement)
- **Sérialisation** : Format JSON pour flexibilité
- **Groupe consommateur** : Spark Structured Streaming

### MinIO (Stockage compatible S3)
- **API** : Compatible AWS S3 (protocole s3a://)
- **Buckets** : bronze, silver, gold (séparation logique)
- **Format** : Parquet (colonnaire, compressé)
- **Accès** : Adressage path-style pour réseau Docker
- **Cohérence** : Cohérence forte pour intégrité des données

### PostgreSQL 18.1 (Base Northwind)
- **Schéma** : Design OLTP normalisé (3NF)
- **Tables** : 6 tables principales pour scénario e-commerce
- **Accès** : Driver JDBC pour connectivité Spark
- **Port** : 5433 (externe), 5432 (interne Docker)

### Orchestration Docker Compose
- **Réseaux** : Réseau bridge personnalisé pour découverte de services
- **Volumes** : Volumes nommés pour persistance des données
- **Health Checks** : Gestion des dépendances de services
- **Limites ressources** : Configurées pour développement local

## Patterns d'ingénierie de données

### 1. Chargement incrémental
```python
# Partitionnement par date pour mises à jour incrémentales
chemin_sortie = f"s3a://bronze/{nom_table}/{DATE_JOUR}"
df_meta.write.mode("overwrite").parquet(chemin_sortie)
```

### 2. Écritures idempotentes
- Mode overwrite garantit des résultats cohérents lors des ré-exécutions
- Partitionnement par date permet le retraitement historique

### 3. Évolution de schéma
- Format Parquet supporte les changements de schéma
- Champs de métadonnées techniques pour le suivi

### 4. Stratégie de dénormalisation
```python
# Couche silver crée une table large pour analytique
df_silver_master = df_details \
    .join(df_orders, on="order_id") \
    .join(df_products, on="product_id") \
    .join(df_categories, on="category_id") \
    .join(df_customers, on="customer_id")
```

### 5. Agrégation temps réel
```python
# Traitement micro-batch avec watermarking
df_stream_agg = df_stream_parsed.groupBy("pays").agg(
    F.sum("montant").alias("total_ventes_live"),
    F.count("*").alias("nb_transactions")
)
```

## Optimisations de performance

### Couche stockage
- **Compression Parquet** : Codec Snappy (par défaut) pour équilibre vitesse/taille
- **Format colonnaire** : Pushdown de prédicats efficace pour requêtes analytiques
- **Partitionnement** : Partitions basées sur dates pour élagage efficace des données

### Couche traitement
- **Broadcast Joins** : Petites tables de dimension diffusées aux exécuteurs
- **Pushdown de prédicats** : Opérations de filtre poussées à la couche stockage
- **Élagage de colonnes** : Seules les colonnes requises lues depuis Parquet

### Couche streaming
- **Trigger** : `availableNow=True` pour traitement micro-batch
- **Mode sortie** : `complete` pour mises à jour complètes d'agrégation
- **Sink mémoire** : Table en mémoire rapide pour requêtes dashboard

## Considérations de scalabilité

### Scaling horizontal
- Ajouter des workers Spark pour parallélisme accru
- Partitions Kafka pour consommation parallèle
- Mode distribué MinIO pour datasets plus larges

### Scaling vertical
- Augmenter mémoire exécuteur pour agrégations plus grandes
- Ajuster mémoire driver pour transformations complexes
- Adapter partitions de shuffle pour performance optimale

## Bonnes pratiques de sécurité (Production)

1. **Gestion des identifiants**
   - Utiliser variables d'environnement pour secrets
   - Implémenter rôles IAM AWS pour accès S3
   - Authentification SASL Kafka

2. **Segmentation réseau**
   - VLANs séparés pour tiers de données
   - Règles firewall pour isolation de services
   - TLS/SSL pour données en transit

3. **Gouvernance des données**
   - Chiffrement niveau colonne pour PII
   - Logging d'audit pour suivi d'accès
   - Masquage de données en non-production

## Monitoring & Observabilité

### Spark UI (Port 4040)
- Métriques d'exécution de jobs
- Performance niveau stage
- Utilisation des exécuteurs

### Kafka UI (Port 7080)
- Monitoring du lag des topics
- Débit des messages
- Santé des groupes consommateurs

### Console MinIO (Port 9001)
- Statistiques des buckets
- Cycle de vie des objets
- Patterns d'accès

## Améliorations futures

1. **Framework de qualité des données**
   - Intégration Great Expectations
   - Règles de validation automatisées
   - Rapports de profilage de données

2. **Orchestration**
   - Apache Airflow pour planification de workflows
   - Dépendances de pipeline basées sur DAG
   - Mécanismes de retry et alertes

3. **Analytique avancée**
   - MLlib pour modèles de machine learning
   - Prédiction de churn client
   - Moteurs de recommandation

4. **Catalogue de données**
   - Apache Atlas pour gestion de métadonnées
   - Suivi de lignage de données
   - Intégration glossaire métier

## Références

- [Architecture Medallion (Databricks)](https://www.databricks.com/glossary/medallion-architecture)
- [Guide Spark Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)
- [Documentation S3A Hadoop](https://hadoop.apache.org/docs/stable/hadoop-aws/tools/hadoop-aws/index.html)
- [Documentation Kafka](https://kafka.apache.org/documentation/)
