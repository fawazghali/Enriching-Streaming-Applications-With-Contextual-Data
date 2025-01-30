
## 🚀 Overview
This project demonstrates how to enrich streaming data pipelines with contextual information from various sources (e.g., databases, APIs, or static reference data). By augmenting real-time data with relevant context, applications can gain deeper insights, improve personalization, and enable more intelligent automation.

## 🔥 Features
- **Real-Time Data Enrichment** – Merge incoming event streams with contextual data dynamically.
- **Flexible Integration** – Support for multiple data sources, including REST APIs, databases, and in-memory caches.
- **Streaming Framework Compatibility** – Works with Apache Kafka, Apache Flink, Spark Streaming, and more.
- **Low Latency Processing** – Optimized for high-performance, real-time workloads.
- **Scalability & Fault Tolerance** – Designed to handle large-scale data processing with resilience.

## 🎯 Use Cases
- **Personalized Recommendations** – Enrich user interactions with historical and behavioral data.
- **Fraud Detection** – Enhance transaction monitoring with external risk indicators.
- **Real-Time Analytics** – Combine live data with business intelligence sources.
- **IoT & Sensor Data Processing** – Add location, weather, or device metadata to raw sensor feeds.

  
## 📌 Getting Started
1. Clone the repository:
   ```sh
   git clone https://github.com/your-repo/enrich-streaming-data.git
   cd enrich-streaming-data
   ``` 
2. Start a Local Cluster with the CLI: https://docs.hazelcast.com/hazelcast/latest/getting-started/get-started-cli
  ```sh
   hz start
   ``` 
3. Start SQL with the CLI
  ```sh
   hz-cli sql
   ``` 
4. Generate a stream (this can be replaced by Kafka, Redpanda, Apache Pulsar, etc):
 ```sql
 CREATE OR REPLACE VIEW orders AS
  SELECT id,
       CASE WHEN userRand BETWEEN 0 AND 0.1 THEN 'Neapolitan'
            WHEN userRand BETWEEN 0.1 AND 0.2 THEN 'Greek'
            WHEN userRand BETWEEN 0.2 AND 0.3 THEN 'Chicago'
            WHEN userRand BETWEEN 0.3 AND 0.4 THEN 'Margherita'
            WHEN userRand BETWEEN 0.4 AND 0.5 THEN 'Hawaiian'
            WHEN userRand BETWEEN 0.5 AND 0.6 THEN 'Pepperoni'
            WHEN userRand BETWEEN 0.6 AND 0.7 THEN 'BBQ'
            WHEN userRand BETWEEN 0.7 AND 0.8 THEN 'Sicilian'
            ELSE 'Johnson'
       END as customer,
       CASE WHEN userRand BETWEEN 0 and 0.1 then userRand*50+1
            WHEN userRand BETWEEN 0.1 AND 0.2 THEN userRand*75+.6
            WHEN userRand BETWEEN 0.2 AND 0.3 THEN userRand*60+.2
            WHEN userRand BETWEEN 0.3 AND 0.4 THEN userRand*30+.3
            WHEN userRand BETWEEN 0.4 AND 0.5 THEN userRand*43+.7
            WHEN userRand BETWEEN 0.5 AND 0.6 THEN userRand*100+.4
            WHEN userRand BETWEEN 0.6 AND 0.7 THEN userRand*25+.8
            WHEN userRand BETWEEN 0.6 AND 0.7 THEN userRand*80+.5
            WHEN userRand BETWEEN 0.7 AND 0.8 THEN userRand*10+.1
            ELSE userRand*100+4
       END as price,
       order_ts,
       amount
FROM
    (SELECT v as id,
           RAND(v*v) as userRand,
           TO_TIMESTAMP_TZ(v*10 + 1645484400000) as order_ts,
           ROUND(RAND()*100, 0) as amount
     FROM TABLE(generate_stream(100)));
   ``` 
5. Run real-time query:
 ```sql
SELECT customer AS Customer, ROUND(price,2) AS Price, amount AS "Sold"
FROM orders
WHERE customer = 'Margherita';
   ``` 
6. Contextual data enrichment:
 ```sql
CREATE or REPLACE MAPPING extras (
__key BIGINT,
customer VARCHAR,
extra1 VARCHAR,
extra2 VARCHAR,
extra3 VARCHAR )
TYPE IMap
OPTIONS (
'keyFormat'='bigint',
'valueFormat'='json-flat');
   ``` 
7. Add contextual data enrichment:
 ```sql
  INSERT INTO extras VALUES
(1, 'Neapolitan', 'Fruit','Fries','vegetables'),
(2, 'Greek', 'Meatballs', 'Fries', 'soup'),
(3, 'Chicago', 'salad','Fries', 'soup'),
(4, 'Margherita', 'Meatballs','Fries', 'vegetables'),
(5, 'Hawaiian', 'salad', 'Fries', 'vegetables'),
(6, 'Pepperoni', 'vegetables', 'Meatballs', 'Fruit'),
(7, 'Martin', 'vegetables', 'Meatballs', 'Fruit'),
(8, 'BBQ', 'vegetables', 'Meatballs', 'Fruit'),
(9, 'Sicilian', 'vegetables','Meatballs','Fruit');
   ``` 
8. JOIN to combine the static information with the streaming data:
```sql 
    SELECT
    orders.customer AS Symbol,
    extras.extra1 as extra1,
    extras.extra2 as extra2,
    extras.extra3 as extra3,
     ROUND(orders.price,2) AS Price,
     orders.amount AS "Sold"
    FROM orders
    JOIN extras
    ON extras.customer = orders.customer 
    AND extras.extra2 = 'Fries';
   ```

9. Watermarking and Windowing:
```sql 
CREATE OR REPLACE VIEW pizza_ordered AS
SELECT *
  FROM TABLE(IMPOSE_ORDER(
  TABLE orders,
  DESCRIPTOR(order_ts),
  INTERVAL '0.5' SECONDS));
   ```
10. Aggregation:
```sql 
  SELECT
     window_start,
     window_end,
     id,
     ROUND(MAX(price),2) AS high,
     ROUND(MIN(price),2) AS low
FROM TABLE(TUMBLE(
     TABLE pizza_ordered,
     DESCRIPTOR(order_ts),
     INTERVAL '5' SECONDS
))
GROUP BY 1,2,3
;
   ``` 

