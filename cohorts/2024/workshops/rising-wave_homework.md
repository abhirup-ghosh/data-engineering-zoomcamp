
# QUESTION 1 AND 2

```sql
DROP MATERIALIZED VIEW IF EXISTS trip_time_stats;
```

```sql
CREATE MATERIALIZED VIEW trip_time_stats AS
SELECT t1.zone AS pickup_zone, 
        t2.zone AS dropoff_zone,
        AVG(EXTRACT(EPOCH FROM (tpep_dropoff_datetime - tpep_pickup_datetime))) AS avg_trip_time,
        MIN(EXTRACT(EPOCH FROM (tpep_dropoff_datetime - tpep_pickup_datetime))) AS min_trip_time,
        MAX(EXTRACT(EPOCH FROM (tpep_dropoff_datetime - tpep_pickup_datetime))) AS max_trip_time,
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

```
  pickup_zone   | dropoff_zone | avg_trip_time | min_trip_time | max_trip_time | trips 
----------------+--------------+---------------+---------------+---------------+-------
 Yorkville East | Steinway     |  86373.000000 |  86373.000000 |  86373.000000 |     1
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