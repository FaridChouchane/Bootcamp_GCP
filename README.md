# ☁️ Bootcamp Cloud GCP — 28 jours

> Formation intensive **Google Cloud Platform** orientée Data Engineering.
> De l'ingestion batch au streaming temps réel, du serverless au Machine Learning dans le cloud.

![GCP](https://img.shields.io/badge/Google_Cloud-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)
![BigQuery](https://img.shields.io/badge/BigQuery-669DF6?style=for-the-badge&logo=googlebigquery&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-336791?style=for-the-badge&logo=postgresql&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Status](https://img.shields.io/badge/Bootcamp-Complété_✓-success?style=for-the-badge)

---

## 📋 Table des matières

- [Vue d'ensemble](#-vue-densemble)
- [Semaine 1 — Fondations GCP & BigQuery](#-semaine-1--fondations-gcp--bigquery)
- [Semaine 2 — PostgreSQL, Flask, Cloud SQL & Looker Studio](#-semaine-2--postgresql-flask-cloud-sql--looker-studio)
- [Semaine 3 — Streaming temps réel avec Pub/Sub & Cloud Functions](#-semaine-3--streaming-temps-réel-avec-pubsub--cloud-functions)
- [Semaine 4 — CLI, Secret Manager & ML](#-semaine-4--cli-secret-manager--ml)
- [Projets personnels](#-projets-personnels)
- [Stack & Services GCP](#️-stack--services-gcp)
- [Ce que j'ai appris](#-ce-que-jai-appris)

---

## 🗺️ Vue d'ensemble

Ce repo documente 4 semaines de formation intensive sur GCP, avec des projets bout-en-bout à chaque fin de semaine.

```
Semaine 1   →   GCS + BigQuery + Partitionnement + Batch Pipeline
Semaine 2   →   PostgreSQL VM + Flask + Cloud SQL + Federated Queries + Looker Studio
Semaine 3   →   Pub/Sub + Cloud Functions + Streaming temps réel
Semaine 4   →   gcloud CLI + Secret Manager + BigQuery ML
```

| | Semaine 1 | Semaine 2 | Semaine 3 | Semaine 4 |
|:--|:--|:--|:--|:--|
| **Focus** | Fondations GCP | SQL avancé & Viz | Streaming | Serverless & ML |
| **Services** | GCS · BigQuery · IAM | Compute Engine · Cloud SQL · Looker | Pub/Sub · Cloud Functions | CLI · Secret Manager · BQML |
| **Projet** | Batch Pipeline E2E | App web + BDD cloud | Pipeline streaming | Pipeline event-driven |
| **Difficulté** | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

## 📦 Semaine 1 — Fondations GCP & BigQuery

### Jour 1–2 · IAM & Google Cloud Storage

**IAM (Identity & Access Management)** : qui peut faire quoi sur quelle ressource.

> Les projets GCP permettent de séparer les environnements (dev/staging/prod), contrôler les accès par équipe et suivre la facturation indépendamment.

**Manipulation des buckets GCS :**

```bash
# Créer un bucket
gcloud storage buckets create gs://mon-bucket --location=europe-west9

# Uploader un fichier
gcloud storage cp mon_fichier.csv gs://mon-bucket/

# Lister le contenu
gcloud storage ls gs://mon-bucket/

# Rendre un objet public
gcloud storage objects update gs://mon-bucket/fichier.csv --add-acl-grant=READER:allUsers

# Supprimer un bucket et tout son contenu
gcloud storage rm -r gs://mon-bucket/
```

---

### Jour 3–4 · BigQuery — Tables, Partitionnement

#### Tables externes vs tables natives

Une **table externe** (External Table) pointe vers un fichier GCS sans copier la donnée dans BigQuery. Utile pour explorer sans ingérer.

```sql
-- Créer un schema/dataset
CREATE SCHEMA IF NOT EXISTS raw
OPTIONS (location = "europe-west9");

-- Table EXTERNE : la donnée reste sur GCS
CREATE OR REPLACE EXTERNAL TABLE raw.orders_ext
OPTIONS (
  format = 'CSV',
  field_delimiter = ',',
  skip_leading_rows = 1,
  uris = ['gs://sales_data_1337/orders_1m_dirty_simple.csv']
);

-- Table NATIVE : la donnée est copiée dans BigQuery
LOAD DATA INTO raw.orders (
  order_id    STRING,
  order_date  STRING,
  customer_id STRING,
  product_id  STRING,
  category    STRING,
  amount      STRING,
  status      STRING
)
FROM FILES (
  format = 'CSV',
  uris = ['gs://sales_data_1337/orders_1m_dirty_simple.csv'],
  skip_leading_rows = 1
);
```

> **Syntaxe BigQuery :** les noms de tables s'écrivent avec des backticks `` ` `` :
> ```sql
> FROM `project_id.dataset.table`
> -- ex: FROM `data-projects-1337.decouverte_bq.sales_events_interne`
> ```

#### Partitionnement

BigQuery (via Colossus) crée un fichier physique par valeur de partition — 365 fichiers pour 365 jours. Une requête filtrée sur une date ne scanne que les partitions concernées → **moins de data scannée = moins cher**.

> 💡 Rule of thumb : viser des partitions d'au moins **1 GB**. Des partitions trop petites génèrent de la metadata excessive.

```sql
-- Inspecter les partitions d'une table
SELECT *
FROM `mon_dataset.INFORMATION_SCHEMA.PARTITIONS`
WHERE partition_id IS NOT NULL;
```

---

### Jour 5 · Transformation de données avec BigQuery SQL

```sql
-- Créer une table externe directement depuis GCS via SQL
CREATE SCHEMA IF NOT EXISTS raw OPTIONS (location = "europe-west9");

CREATE OR REPLACE EXTERNAL TABLE raw.simple_orders_ext
OPTIONS (
  format = 'CSV',
  field_delimiter = ',',
  skip_leading_rows = 1,
  uris = ['gs://hello_world_bucket_1337/simple_orders_10k.csv']
);
```

---

### Jour 6 · 📦 Mini-Projet #1 — Batch Pipeline E2E

**Objectif :** bâtir une pipeline batch complète qui ingère, nettoie, optimise et benchmark.

```
GCS (CSV brut, 1M lignes)
    ↓
BigQuery External Table  ← check de la donnée brute
    ↓
BigQuery Native Table    ← ingestion
    ↓
Cleaning SQL             ← nettoyage, typage
    ↓
Table Partitionnée       ← optimisation
    ↓
Benchmark SQL            ← mesure de la performance
```

**Étape 1 — Ingestion :**

```sql
LOAD DATA INTO raw.orders (
  order_id    STRING,
  order_date  STRING,
  customer_id STRING,
  product_id  STRING,
  category    STRING,
  amount      STRING,
  status      STRING
)
FROM FILES (
  format = 'CSV',
  uris = ['gs://sales_data_1337/orders_1m_dirty_simple.csv'],
  skip_leading_rows = 1
);
```

**Étape 2 — Nettoyage :**

```sql
-- Typer, nettoyer et filtrer les données
CREATE OR REPLACE TABLE retail.orders_clean AS
SELECT
  CAST(order_id AS INT64)       AS order_id,
  DATE(order_date)              AS order_date,
  customer_id,
  product_id,
  TRIM(category)                AS category,    -- supprimer les espaces parasites
  CAST(amount AS NUMERIC)       AS amount,
  UPPER(TRIM(status))           AS status       -- normaliser les statuts
FROM retail.orders_raw
WHERE order_id IS NOT NULL
  AND amount  IS NOT NULL;
```

**Étape 3 — Partitionnement :**

```sql
CREATE SCHEMA IF NOT EXISTS cleaned OPTIONS (location = "europe-west9");

-- Créer la table optimisée, partitionnée par jour
CREATE OR REPLACE TABLE cleaned.orders_partitioned
PARTITION BY order_date AS
SELECT * FROM raw.orders_clean;
```

**Étape 4 — Benchmark :**

```sql
-- Requête sur table NON partitionnée (scanne TOUT)
SELECT category, SUM(amount) AS total_sales
FROM raw.orders_clean
WHERE order_date BETWEEN '2024-07-01' AND '2024-07-15'
GROUP BY category;

-- Même requête sur table PARTITIONNÉE (scanne uniquement 15 partitions)
SELECT category, SUM(amount) AS total_sales
FROM cleaned.orders_partitioned
WHERE order_date BETWEEN '2024-07-01' AND '2024-07-15'
GROUP BY category;

-- → Comparer les "bytes processed" dans les query stats BigQuery
```

> ✅ **Résultat :** réduction significative des octets scannés grâce au partitionnement par date.

---

## 🌐 Semaine 2 — PostgreSQL, Flask, Cloud SQL & Looker Studio

### Jour 8 · PostgreSQL sur une VM Compute Engine

**Compute Engine** = machines virtuelles dans le cloud GCP.

**Setup de la VM :**

```bash
# Mise à jour + installation PostgreSQL & outils Python
sudo apt-get update
sudo apt-get install -y postgresql postgresql-contrib python3-pip python3-venv

# Démarrer PostgreSQL et l'activer au boot
sudo systemctl enable --now postgresql

# Vérifications
sudo systemctl status postgresql
psql --version
```

**Création de la base de données :**

```bash
# Créer un rôle utilisateur avec mot de passe
sudo -u postgres psql -c "CREATE ROLE appuser WITH LOGIN PASSWORD 'demo123';"

# Créer la base de données
sudo -u postgres psql -c "CREATE DATABASE appdb OWNER appuser;"

# Créer la table visits
sudo -u postgres psql -d appdb -c "
  CREATE TABLE visits (
    id SERIAL PRIMARY KEY,
    ts TIMESTAMPTZ DEFAULT now()
  );"

# Transférer la propriété (nécessaire pour les INSERTs sans erreur de droits)
sudo -u postgres psql -d appdb -c "ALTER TABLE public.visits OWNER TO appuser;"
sudo -u postgres psql -d appdb -c "ALTER SEQUENCE public.visits_id_seq OWNER TO appuser;"

# Se connecter (-h localhost force TCP au lieu de socket Unix → évite "peer auth failed")
psql -U appuser -d appdb -h localhost -W
# → Saisir le mot de passe : demo123
# → Pour quitter : \q
```

---

### Jour 9 · Application Flask + PostgreSQL sur VM

**Installation de l'environnement :**

```bash
# Créer le dossier de l'app
sudo mkdir -p /srv/flaskapp && cd /srv/flaskapp
sudo chown -R "$USER":"$USER" /srv/flaskapp

# Environnement virtuel Python isolé
python3 -m venv venv
source venv/bin/activate

# Installer les dépendances
pip install flask SQLAlchemy psycopg2-binary
```

**Code de l'application (`app.py`) :**

```python
# /srv/flaskapp/app.py
from flask import Flask
from sqlalchemy import create_engine, text

# Connexion PostgreSQL locale via SQLAlchemy
PG_URL = "postgresql+psycopg2://appuser:demo123@localhost/appdb"
engine = create_engine(PG_URL, pool_pre_ping=True)

app = Flask(__name__)

@app.get("/")
def root():
    return (
        "<h3>Postgres Demo</h3>"
        "<ul>"
        "<li><a href='/pg/insert'>/pg/insert</a> – insérer une ligne</li>"
        "<li><a href='/pg/count'>/pg/count</a>  – compter les lignes</li>"
        "</ul>"
    )

@app.get("/pg/insert")
def pg_insert():
    with engine.begin() as conn:
        conn.execute(text("INSERT INTO visits DEFAULT VALUES"))
    return "Inserted 1 row into Postgres."

@app.get("/pg/count")
def pg_count():
    with engine.begin() as conn:
        n = conn.execute(text("SELECT COUNT(*) FROM visits")).scalar()
    return f"Postgres visits count = {n}"
```

**Lancement :**

```bash
# Créer le fichier avec vim
touch app.py && vim app.py
# → coller le code, puis Echap + :wq + Entrée
# → vérifier : cat app.py

# Lancer Flask sur le port 80 (accessible depuis l'extérieur)
export FLASK_APP=/srv/flaskapp/app.py
sudo /srv/flaskapp/venv/bin/flask run --host=0.0.0.0 --port=80
```

> Accessible via l'**External IP** de la VM dans la console Compute Engine.
> Routes : `/` · `/pg/insert` · `/pg/count`

> ⚠️ `flask run` est un serveur de dev. En production : **Gunicorn + Nginx + systemd service**.

---

### Jour 9 · Cloud SQL — PostgreSQL managé

**Cloud SQL** = PostgreSQL géré par GCP (backups auto, haute dispo, pas de maintenance serveur).

```bash
PROJECT_ID=mon-projet
REGION=europe-west9
INSTANCE=bootcamp-pg
PASSWORD='StrongPassw0rd!'

# Créer l'instance PostgreSQL 17
gcloud sql instances create $INSTANCE \
  --database-version=POSTGRES_17 \
  --cpu=2 --memory=4GB \
  --region=$REGION \
  --root-password=$PASSWORD \
  --edition=ENTERPRISE

# Créer la base, l'utilisateur, se connecter
gcloud sql databases create retail --instance=$INSTANCE
gcloud sql users create bootuser --instance=$INSTANCE --password='AnotherStrongPassw0rd!'
gcloud sql connect $INSTANCE --user=postgres --quiet
```

> ⚠️ **Attention facturation :** une instance Cloud SQL tourne et facture même à l'arrêt. Penser à la supprimer après les exercices.

**Génération de 200 000 lignes synthétiques :**

```sql
-- Créer la table
CREATE TABLE public.orders (
  order_id    BIGINT PRIMARY KEY,
  order_date  DATE          NOT NULL,
  country     TEXT,
  category    TEXT,
  quantity    INT,
  price       NUMERIC(10,2),
  status      TEXT,
  email       TEXT
);

-- Injecter 200k lignes sans fichier externe via generate_series
-- generate_series(1, 200000) crée 200k entiers g=1..200000 utilisés comme graine
INSERT INTO public.orders
SELECT
  g                                                       AS order_id,
  DATE '2024-01-01' + (g % 180)                           AS order_date,   -- 6 mois cycliques
  (ARRAY['US','FR','DE','GB','ES'])[1 + (g % 5)]          AS country,
  (ARRAY['electronics','fashion','home','toys','books'])
    [1 + (g % 5)]                                         AS category,
  1 + (g % 5)                                             AS quantity,      -- 1 à 5
  (10 + (g % 190))::numeric(10,2)                         AS price,        -- 10 à 199 €
  (ARRAY['paid','pending','refunded','cancelled'])
    [1 + (g % 4)]                                         AS status,
  'user' || g || '@example.com'                           AS email         -- unique par ligne
FROM generate_series(1, 200000) AS g;

-- Vérifications post-insert
SELECT COUNT(*) FROM public.orders;                            -- → 200000
SELECT country, COUNT(*) FROM public.orders GROUP BY country;  -- distribution uniforme
SELECT status, SUM(quantity * price) FROM public.orders GROUP BY status;
```

> **Pourquoi cette approche ?** Pas de CSV, ~quelques secondes pour 200k lignes, reproductible. Technique typiquement utilisée pour benchmarks SQL, tests de window functions, simulations BigQuery.

---

### Jour 10 · Federated Queries — Cloud SQL → BigQuery

```sql
-- Requête fédérée : BigQuery interroge Cloud SQL à la volée (sans copier)
SELECT
  country,
  COUNT(*)               AS num_orders,
  SUM(quantity * price)  AS revenue
FROM EXTERNAL_QUERY(
  "project-id.europe-west9.bootcamp_pg_federated_conn",
  "SELECT country, quantity, price FROM orders"
)
GROUP BY country;

-- Copier la donnée dans BigQuery pour des analyses plus rapides (snapshot)
CREATE OR REPLACE TABLE bootcamp.orders_bq AS
SELECT *
FROM EXTERNAL_QUERY(
  "project-id.europe-west9.pg_federated_conn",
  "SELECT order_id, order_date, country, category, quantity, price, status, email FROM orders"
);

-- Scheduled Query : mise à jour journalière depuis Cloud SQL
INSERT INTO bootcamp.orders_bq
SELECT *
FROM EXTERNAL_QUERY(
  "project-id.europe-west9.pg_federated_conn",
  "SELECT * FROM orders WHERE order_date = CURRENT_DATE"
);
```

> **Prérequis IAM :** donner au service agent BigQuery le rôle `Cloud SQL Client` sur le projet.

---

### Jour 11 · OLTP vs OLAP

| | OLTP (PostgreSQL) | OLAP (BigQuery) |
|:--|:--|:--|
| **Usage** | Servir le produit (app, site web) | Analyser les données (BI, Data) |
| **Optimisé pour** | Lectures/écritures rapides par ligne | Scans de colonnes sur des milliards de lignes |
| **Exemple** | Enregistrer une commande client | Calculer le CA du mois par catégorie |
| **Stockage** | Row-oriented | Column-oriented |

```
Site web / App  →  PostgreSQL (OLTP, prod)  →  BigQuery (OLAP, analyse)
```

---

### Jour 12–13 · Looker Studio & Mini-Projet #2

**Connexion :** `lookerstudio.google.com` → New Report → BigQuery → sélectionner projet/dataset/table.

**Mini-Projet #2 — Data Analytics E2E (Logistique) :**

Dataset multi-sources : livraisons, véhicules, conducteurs, météo, prix carburant.

```sql
-- A) Performance des livraisons par climat de destination
WITH s AS (
  SELECT
    ship.shipment_id, ship.actual_duration_hr,
    r.avg_expected_duration_hr, r.destination_city, cw.climate_zone
  FROM `bq_demo.ops_shipments` ship
  JOIN `bq_demo.static_route_reference`    r  USING (route_id)
  JOIN `bq_demo.static_city_weather_zones` cw ON cw.city_name = r.destination_city
)
SELECT
  destination_city, climate_zone,
  COUNT(*) AS trips,
  ROUND(AVG(actual_duration_hr - avg_expected_duration_hr), 2) AS avg_delay_hr,
  ROUND(100 * AVG(CASE WHEN actual_duration_hr <= avg_expected_duration_hr
                       THEN 1 ELSE 0 END), 1)                 AS pct_on_time
FROM s
GROUP BY destination_city, climate_zone
ORDER BY avg_delay_hr DESC;

-- B) Efficacité énergétique & coût par type de véhicule
WITH f AS (
  SELECT DATE_TRUNC(DATE(departure_time), MONTH) AS month,
    vehicle_id, distance_km, fuel_consumed_litres, energy_used_kwh, hydrogen_used_kg
  FROM `bq_demo.ops_shipments`
),
prices AS (
  SELECT DATE(month) AS month, diesel_eur_per_l, electric_eur_per_kwh, hydrogen_eur_per_kg
  FROM `bq_demo.static_fuel_prices_monthly`
),
veh AS (SELECT vehicle_id, vehicle_type, fuel_type FROM `bq_demo.static_vehicle_specs`)
SELECT
  v.vehicle_type, v.fuel_type, COUNT(*) AS trips,
  ROUND(SUM(distance_km) / NULLIF(SUM(fuel_consumed_litres), 0), 2) AS km_per_litre,
  ROUND(SUM(f.fuel_consumed_litres * p.diesel_eur_per_l
          + f.energy_used_kwh      * p.electric_eur_per_kwh
          + f.hydrogen_used_kg     * p.hydrogen_eur_per_kg), 2)      AS total_energy_cost_eur
FROM f JOIN prices p USING (month) JOIN veh v USING (vehicle_id)
GROUP BY v.vehicle_type, v.fuel_type
ORDER BY total_energy_cost_eur DESC;

-- C) Coût de maintenance par véhicule (per 10k km)
WITH km AS (SELECT vehicle_id, SUM(distance_km)  AS km_total   FROM `bq_demo.ops_shipments`           GROUP BY vehicle_id),
     mc AS (SELECT vehicle_id, SUM(cost_usd)      AS maint_cost FROM `bq_demo.ops_vehicle_maintenance` GROUP BY vehicle_id)
SELECT
  vs.vehicle_id, vs.assigned_depot_city,
  COALESCE(km.km_total, 0) AS km_total,
  COALESCE(mc.maint_cost, 0) AS maint_cost_usd,
  ROUND(COALESCE(mc.maint_cost,0) / NULLIF(COALESCE(km.km_total,0),0) * 10000, 2)
    AS maint_cost_per_10k_km
FROM `bq_demo.ops_vehicles_status` vs
JOIN  `bq_demo.static_vehicle_specs` v USING (vehicle_id)
LEFT JOIN km USING (vehicle_id)
LEFT JOIN mc USING (vehicle_id)
ORDER BY maint_cost_per_10k_km DESC NULLS LAST;

-- D) Performance conducteurs : sécurité & efficacité
WITH hrs  AS (SELECT driver_id, SUM(hours_driven)        AS hours_total FROM `bq_demo.ops_driver_logs` GROUP BY driver_id),
     inc  AS (SELECT driver_id, SUM(incidents_reported)  AS incidents   FROM `bq_demo.ops_driver_logs` GROUP BY driver_id),
     fuel AS (SELECT driver_id, SUM(distance_km)         AS km_total,
                                SUM(fuel_consumed_litres) AS fuel_l      FROM `bq_demo.ops_shipments`  GROUP BY driver_id)
SELECT
  d.driver_id, d.driver_name, d.home_depot_city,
  ROUND(COALESCE(i.incidents,0) / NULLIF(h.hours_total,0) * 100, 2)              AS incidents_per_100hrs,
  ROUND(COALESCE(f.fuel_l,0)   / NULLIF(COALESCE(f.km_total,0),0) * 100, 2)     AS litres_per_100km
FROM `bq_demo.ops_drivers` d
LEFT JOIN hrs  h USING (driver_id)
LEFT JOIN inc  i USING (driver_id)
LEFT JOIN fuel f USING (driver_id)
ORDER BY incidents_per_100hrs DESC NULLS LAST;

-- E) Empreinte CO₂ par route (facteurs illustratifs)
WITH co2 AS (
  SELECT route_id,
    SUM(fuel_consumed_litres * 2.68    -- Diesel     : 2.68 kg CO2/l
      + energy_used_kwh      * 0.25   -- Électrique : 0.25 kg CO2/kWh
      + hydrogen_used_kg     * 0.5 )  -- Hydrogène  : 0.5  kg CO2/kg (mix)
    AS kg_co2
  FROM `bq_demo.ops_shipments` GROUP BY route_id
)
SELECT r.origin_city, r.destination_city, ROUND(c.kg_co2, 1) AS kg_co2
FROM co2 c JOIN `bq_demo.static_route_reference` r USING (route_id)
ORDER BY kg_co2 DESC LIMIT 50;
```

---

## 📡 Semaine 3 — Streaming temps réel avec Pub/Sub & Cloud Functions

### Jour 15 · Pub/Sub — Architecture événementielle

**Problème sans Pub/Sub :** si 3 services ont besoin du même message, l'émetteur gère 3 destinataires, les erreurs, les retransmissions… C'est du couplage fort et du code if/elif/else sans fin.

**Solution Pub/Sub :** l'émetteur publie une seule fois sur un **topic**. Chaque service souscrit au topic, Pub/Sub gère la livraison.

```
Application  →  Pub/Sub Topic  →  Subscription A  →  BigQuery
                              →  Subscription B  →  Microservice
                              →  Subscription C  →  Alerting
```

| Concept | Définition |
|:--|:--|
| **Topic** | Canal où les éditeurs envoient des messages |
| **Subscription** | Abonnement d'un service à un topic (chaque abonnement reçoit sa copie) |
| **Pull** | Le consommateur va chercher les messages lui-même |
| **Push** | Pub/Sub envoie un POST HTTP à un endpoint |
| **ACK** | Accusé de réception — un message non ACKé est renvoyé |

---

### Jour 16 · Pub/Sub → BigQuery (subscription directe)

**Créer la table de destination :**

```sql
CREATE SCHEMA IF NOT EXISTS `your-project-id.ecommerce`;

CREATE TABLE IF NOT EXISTS `your-project-id.ecommerce.orders_streaming` (
  event_id        STRING,
  event_type      STRING,
  event_timestamp TIMESTAMP,
  order_id        STRING,
  customer_id     STRING,
  item_id         STRING,
  item_category   STRING,
  amount          NUMERIC,
  currency        STRING,
  store_id        STRING,
  country_code    STRING,
  source          STRING
)
PARTITION BY DATE(event_timestamp);
```

**IAM — service agent Pub/Sub :**

Pub/Sub écrit dans BigQuery via son service agent. Sans le bon rôle IAM → `403 Permission denied`.

```
Email du service agent :
service-<PROJECT_NUMBER>@gcp-sa-pubsub.iam.gserviceaccount.com

⚠️  PROJECT_NUMBER ≠ PROJECT_ID
    (visible dans GCP Console → "..." → Project Settings)

Exemple concret :
service-323756541709@gcp-sa-pubsub.iam.gserviceaccount.com
```

Rôle à attribuer : `roles/bigquery.dataEditor` sur le dataset uniquement (principe du moindre privilège).

**Publier des événements depuis la CLI :**

```bash
# Événement order_created
gcloud pubsub topics publish orders-topic \
  --message='{
    "event_id":        "evt-3001",
    "event_type":      "order_created",
    "event_timestamp": "2025-08-22T08:10:00Z",
    "order_id":        "O-100047",
    "customer_id":     "C-222",
    "item_id":         "SKU-PS5",
    "item_category":   "gaming",
    "amount":          549.00,
    "currency":        "EUR",
    "store_id":        "FR-PARIS-01",
    "country_code":    "FR",
    "source":          "web"
  }' \
  --attribute=event_type=order_created,source=web,schema_version=v1

# Événement payment_authorized (même order, 6s plus tard)
gcloud pubsub topics publish orders-topic \
  --message='{
    "event_id":        "evt-3002",
    "event_type":      "payment_authorized",
    "event_timestamp": "2025-08-22T08:10:06Z",
    "order_id":        "O-100047",
    "customer_id":     "C-222",
    "item_id":         "SKU-PS5",
    "item_category":   "gaming",
    "amount":          549.00,
    "currency":        "EUR",
    "store_id":        "FR-PARIS-01",
    "country_code":    "FR",
    "source":          "payments"
  }' \
  --attribute=event_type=payment_authorized,source=payments,schema_version=v1
```

| | Message (`--message`) | Attributes (`--attribute`) |
|:--|:--|:--|
| **Contient** | JSON métier | Paires clé/valeur techniques |
| **Lisible par BigQuery** | ✅ | ⚠️ optionnel |
| **Filtrable côté Pub/Sub** | ❌ | ✅ |
| **Usage** | Données de la commande | Routage, filtrage, versioning |

---

### Jour 17 · Cloud Functions — Trigger GCS → BigQuery

**Cloud Run Functions Gen2** : code serverless déclenché par un événement. Ici, un upload sur GCS.

```
Fichier uploadé sur GCS
    ↓  (trigger automatique Cloud Storage)
Cloud Function (Python 3.11)
    ↓
Parse + transformation
    ↓
BigQuery (table retail.orders_raw)
```

**Table de destination :**

```sql
CREATE SCHEMA IF NOT EXISTS `@PROJECT_ID`.retail;

CREATE OR REPLACE TABLE `@PROJECT_ID.retail.orders_raw` (
  order_id    INT64,
  order_date  DATE,
  country     STRING,
  category    STRING,
  quantity    INT64,
  price       NUMERIC,
  status      STRING,
  email       STRING
);
```

**`requirements.txt` :**

```
functions-framework==3.*
google-cloud-storage>=2.14.0
google-cloud-bigquery>=3.36.0
pandas>=2.3.24
```

---

### Jour 18 · Mini-Projet #3 — Pub/Sub → Cloud Function → BigQuery (avec masquage PII)

Pipeline streaming avec validation des données et **masquage des emails** (données personnelles).

**Table de destination :**

```sql
CREATE SCHEMA IF NOT EXISTS `test-semaine-3.ecom_dataset`;

CREATE OR REPLACE TABLE `test-semaine-3.ecom_dataset.orders_streaming_safe` (
  order_id    INT64,
  event_time  TIMESTAMP,
  country     STRING,
  category    STRING,
  quantity    INT64,
  price       NUMERIC,
  status      STRING,
  email       STRING,       -- email masqué côté Cloud Function
  ingested_at TIMESTAMP
);
```

**Configuration Cloud Function :**

```
Name       : pubsub-to-bq-safe
Region     : europe-west9
Trigger    : Pub/Sub → transactions-topic
Runtime    : Python 3.13
Entry point: on_pubsub_message

Variables d'environnement :
  PROJECT_ID = <id de votre projet>
  BQ_DATASET = ecom_dataset
  BQ_TABLE   = orders_streaming_safe
```

**`requirements.txt` :**

```
functions-framework==3.*
google-cloud-bigquery>=3.17.0
```

**Test de la pipeline :**

```bash
gcloud pubsub topics publish transactions-topic \
  --message='{
    "order_id":   42,
    "event_time": "2024-09-10T12:00:00Z",
    "country":    "FR",
    "category":   "electronics",
    "quantity":   2,
    "price":      199.99,
    "status":     "PAID",
    "email":      "customer.secret@example.com"
  }' \
  --attribute=event_type=order_created,source=web,schema_version=v1

# Vérifier dans BigQuery :
# 1. La transaction est présente
# 2. L'email est masqué (ex: c***@example.com)
```

---

## 🔧 Semaine 4 — CLI, Secret Manager & ML

### Jour 23 · Maîtriser la CLI gcloud & bq

```bash
# ─── BigQuery ────────────────────────────────────────────────────
bq mk --dataset retail_bq_cli                          # créer un dataset
bq ls                                                  # lister les datasets
bq show retail_bq_cli                                  # inspecter
bq rm -r -f retail_bq_cli                             # supprimer avec contenu

# Charger un CSV depuis GCS
bq load \
  --autodetect \
  --source_format=CSV \
  retail_bq_cli.orders_cli \
  gs://$PROJECT_ID-retail-data/orders_clean_no_issues.csv

bq head -n 3 retail_bq_cli.orders_cli                 # aperçu rapide

# Lancer une requête SQL depuis le terminal
bq query --use_legacy_sql=false \
  'SELECT category, SUM(amount) AS revenue
   FROM retail_bq_cli.orders_cli
   GROUP BY category ORDER BY revenue DESC'


# ─── Google Cloud Storage ────────────────────────────────────────
gcloud storage buckets create gs://$PROJECT_ID-retail-data --location=europe-west9
gcloud storage cp orders_clean_no_issues.csv gs://$PROJECT_ID-retail-data/
gcloud storage ls gs://$PROJECT_ID-retail-data/


# ─── Cloud SQL ───────────────────────────────────────────────────
gcloud sql instances create bootcamp-pg \
  --database-version=POSTGRES_17 --cpu=2 --memory=4GB \
  --region=europe-west9 --root-password='StrongPassw0rd!' --edition=ENTERPRISE

gcloud sql databases create retail --instance=bootcamp-pg
gcloud sql users create bootuser --instance=bootcamp-pg --password='AnotherStrongPassw0rd!'
gcloud sql connect bootcamp-pg --user=postgres --quiet
```

---

### Jour 24 · Secret Manager

**Secret Manager** = coffre-fort pour les credentials, API keys, mots de passe. Chaque accès est loggué dans Cloud Audit Logs.

| Rôle IAM | Usage |
|:--|:--|
| `roles/secretmanager.secretAccessor` | Lire un secret (app, service) |
| `roles/secretmanager.admin` | Gérer les secrets (DevOps, admin) |

**Comment ça marche :**

1. **Créer** le secret → GCP chiffre automatiquement
2. **Versionner** : chaque mise à jour = nouvelle version (v1, v2…)
3. **Configurer les permissions** via IAM (qui peut lire/modifier/rotation)
4. **Accéder** depuis les apps via l'API, les librairies Python GCP, la CLI, ou des variables d'environnement dans Cloud Run/Cloud Functions
5. **Audit** : chaque accès est enregistré dans Cloud Logging

**Accès depuis Python :**

```python
from google.cloud import secretmanager

def get_secret(project_id: str, secret_name: str, version: str = "latest") -> str:
    client = secretmanager.SecretManagerServiceClient()
    name   = f"projects/{project_id}/secrets/{secret_name}/versions/{version}"
    resp   = client.access_secret_version(request={"name": name})
    return resp.payload.data.decode("UTF-8")

# Usage dans une Cloud Function
api_key = get_secret("mon-projet", "VICTORIA")
```

**CLI :**

```bash
# Créer un secret
gcloud secrets create VICTORIA --replication-policy="automatic"
echo -n "ma-valeur-secrete" | gcloud secrets versions add VICTORIA --data-file=-

# Lire
gcloud secrets versions access latest --secret=VICTORIA

# Révoquer l'accès d'un service account à un secret précis
gcloud secrets remove-iam-policy-binding VICTORIA \
  --member="serviceAccount:ma-fonction@projet.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"

# Donner accès à tous les secrets du projet (IAM global)
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:ma-fonction@projet.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

---

### Jour 25–28 · 🏁 Mini-Projet Final — Pipeline Serverless E2E + BigQuery ML

**Architecture complète :**

```
CSV uploadé sur GCS
    ↓  (trigger automatique)
Cloud Function Gen2 (Python 3.11)
    ↓  (lit credentials via Secret Manager)
Transformation des données
    ↓
BigQuery → table retail.orders_raw
    ↓
BigQuery ML → entraînement + prédictions
    ↓
Tout orchestré via gcloud CLI
```

**BigQuery ML depuis la CLI :**

```bash
# Entraîner un modèle de régression linéaire directement dans BigQuery
bq query --use_legacy_sql=false '
  CREATE OR REPLACE MODEL retail.revenue_model
  OPTIONS (model_type="linear_reg", input_label_cols=["amount"])
  AS
  SELECT
    category,
    EXTRACT(MONTH FROM order_date) AS month,
    quantity,
    amount
  FROM retail.orders_raw
  WHERE status = "PAID"
'

# Évaluer le modèle
bq query --use_legacy_sql=false '
  SELECT * FROM ML.EVALUATE(MODEL retail.revenue_model)
'

# Faire des prédictions
bq query --use_legacy_sql=false '
  SELECT *
  FROM ML.PREDICT(
    MODEL retail.revenue_model,
    (SELECT "electronics" AS category, 6 AS month, 3 AS quantity)
  )
'
```

---

## 🚀 Projets personnels

> *Section à compléter — les projets personnels arrivent ici.*

---

## 🛠️ Stack & Services GCP

| Service | Rôle | Semaines |
|:--|:--|:--|
| **Cloud Storage (GCS)** | Stockage objet, data lake, triggers | S1 S3 S4 |
| **BigQuery** | Data warehouse, SQL analytique, ML | S1 S2 S3 S4 |
| **Compute Engine** | VMs Linux, PostgreSQL, Flask | S2 |
| **Cloud SQL** | PostgreSQL managé, Federated Queries | S2 |
| **Looker Studio** | Dashboards BI connectés à BigQuery | S2 |
| **Pub/Sub** | Messaging asynchrone, streaming | S3 |
| **Cloud Functions / Cloud Run** | Serverless event-driven | S3 S4 |
| **Secret Manager** | Credentials sécurisés, audit | S4 |
| **IAM** | Gestion des accès, service accounts, moindre privilège | S1→S4 |
| **BigQuery ML** | Entraînement de modèles ML en SQL | S4 |
| **gcloud CLI / bq CLI** | Automatisation, scripting | S4 |

---

## 🧠 Ce que j'ai appris

**Cloud & Architecture**
- Organiser un projet GCP (projets, folders, IAM, billing)
- Comprendre OLTP vs OLAP et choisir le bon outil selon le use case
- Concevoir des architectures batch et streaming sur GCP
- Sécuriser les accès avec le principe du moindre privilège

**BigQuery & SQL**
- Tables externes vs natives, partitionnement, clustering
- SQL analytique avancé : window functions, CTEs, EXPLAIN ANALYZE, `generate_series`
- Federated Queries pour joindre des sources hétérogènes (Cloud SQL ↔ BigQuery)
- BigQuery ML : entraîner et inférer des modèles sans sortir de SQL

**Infrastructure & Serverless**
- Déployer et administrer des VMs Linux sur Compute Engine
- Configurer PostgreSQL depuis zéro (rôles, BDD, tables, séquences)
- Déployer une app Flask + SQLAlchemy sur VM GCP
- Créer des Cloud Functions déclenchées par GCS et Pub/Sub
- Stocker et accéder à des secrets via Secret Manager

**CLI & Automatisation**
- Maîtriser `gcloud`, `bq`, `gcloud storage` pour tout faire sans interface graphique
- Automatiser des pipelines data complets en ligne de commande

---

## 📬 Contact

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/TON-PROFIL)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/TON-USERNAME)

---

<p align="center">
  <sub>Bootcamp Cloud GCP 28 jours · Data Engineering</sub>
</p>
