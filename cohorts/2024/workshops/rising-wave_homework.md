
# QUESTION 1 AND 2

```sql
DROP MATERIALIZED VIEW IF EXISTS trip_time_stats;
```

```sql
CREATE MATERIALIZED VIEW trip_time_stats AS
SELECT t1.zone AS pickup_zone, 
        t2.zone AS dropoff_zone,
        AVG(EXTRACT(EPOCH FROM (tpep_dropoff_datetime - tpep_pickup_datetime))) / 60 AS avg_trip_time,
        MIN(EXTRACT(EPOCH FROM (tpep_dropoff_datetime - tpep_pickup_datetime))) / 60 AS min_trip_time,
        MAX(EXTRACT(EPOCH FROM (tpep_dropoff_datetime - tpep_pickup_datetime))) / 60 AS max_trip_time,
        count(*) as trips
FROM trip_data 
LEFT JOIN taxi_zone AS t1 ON trip_data.pulocationid = t1.location_id 
LEFT JOIN taxi_zone AS t2 ON trip_data.dolocationid = t2.location_id
GROUP BY
        t1.zone, 
        t2.zone;
```

```sql
SELECT * FROM trip_time_stats ORDER BY avg_trip_time DESC LIMIT 1;        
```

```bash
  pickup_zone   | dropoff_zone | avg_trip_time | min_trip_time | max_trip_time | trips 
----------------+--------------+---------------+---------------+---------------+-------
 Yorkville East | Steinway     |   1439.550000 |   1439.550000 |   1439.550000 |     1
 ```

# BONUS
```sql
SELECT * FROM trip_time_stats WHERE max_trip_time > 10 AND avg_trip_time < 2;
```
```bash
 pickup_zone | dropoff_zone |         avg_trip_time          | min_trip_time | max_trip_time | trips 
-------------+--------------+--------------------------------+---------------+---------------+-------
 NaN         | NaN          | 1.9414855072463768115942028985 |      0.050000 |     29.200000 |    92
```



# QUESTION 3
```sql
CREATE MATERIALIZED VIEW latest_pickup AS
    SELECT
        max(tpep_pickup_datetime) AS latest_pickup_time
    FROM
        trip_data
            JOIN taxi_zone
                ON trip_data.PULocationID = taxi_zone.location_id
```

```sql
CREATE MATERIALIZED VIEW pickups_17hr_before AS
    SELECT
        taxi_zone.zone,
        count(*) AS cnt
    FROM
        trip_data
            JOIN latest_pickup
                ON trip_data.tpep_pickup_datetime > latest_pickup.latest_pickup_time - interval '17 hour'
            JOIN taxi_zone
                ON trip_data.PULocationID = taxi_zone.location_id
    GROUP BY taxi_zone.zone;
```

```sql
SELECT * FROM pickups_17hr_before ORDER BY cnt DESC LIMIT 3;
```

```bash
        zone         | cnt 
---------------------+-----
 LaGuardia Airport   |  19
 JFK Airport         |  17
 Lincoln Square East |  17
(3 rows)
```