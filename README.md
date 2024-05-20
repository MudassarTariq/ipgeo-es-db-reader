This repository provides a streamlined and efficient solution for importing data from CSV files into an Elasticsearch instance.

## Prerequisites

Before you begin, please ensure you have the following prerequisites in place:

1) IP Geolocation Database Files: Download the necessary IP geolocation database files.
2) Elasticsearch: Install and ensure Elasticsearch is up and running. Refer to the [Elasticsearch installation guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html) if you need assistance.
3) Logstash: Install and ensure Logstash is up and running. Refer to the [Logstash installation guide](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html) if you need assistance.

Once the prerequisites are met now you can proceed with indexing the data into an ES instance.

## Setting Up ES Instance.

#### Setting Up place_db index
This index will contains data from 'db-place.csv' file.
- Create an index called 'place_db'.
```
curl -X PUT "localhost:9200/place_db"
```

- Define the mapping for 'place_db' index.
```
curl -s -X PUT "localhost:9200/place_db/_mapping" -H 'Content-Type: application/json' --data-binary "@index_mapping/place_db_mapping.json"
```

- Use logstash to push data from .csv file to an ES instance.
```
/usr/share/logstash/bin/logstash -f {path_where_repo_clone}/ipgeo-es-db-reader/logstash_config/place_db.conf --path.data /var/lib/logstash/place/
```

Just make sure that data dir `/var/lib/logstash/place/` used above must have writeable permissions.

#### Setting Up country_db index
This index will contains data.

- Create an index called 'country_db'.
```
curl -X PUT "localhost:9200/country_db"
```

- Define the mapping for 'country_db' index.
```
curl -X PUT "localhost:9200/country_db/_mapping" -H 'Content-Type: application/json' --data-binary "@index_mapping/country_db_mapping.json"
```

- Create an enrich policy.
```
curl -X PUT "localhost:9200/_enrich/policy/place-db-enrich-policy" -H 'Content-Type: application/json' --data-binary "@enrich_policies/country_db_enrich_policy.json"
```

- Execute the `place-db-enrich-policy`.
```
curl -X POST "localhost:9200/_enrich/policy/place-db-enrich-policy/_execute"
```

- Create a pipeline for pushing data.
```
curl -X PUT "localhost:9200/_ingest/pipeline/country-db-enrich-pipeline" -H 'Content-Type: application/json' --data-binary "@pipelines_processors/country_db_pipeline_processor.json"
```

- Use logstash to push data from .csv file to an ES instance.
```
/usr/share/logstash/bin/logstash -f {path_where_repo_clone}/ipgeo-es-db-reader/logstash_config/country_db.conf --path.data /var/lib/logstash/country/
```
Just make sure that data dir `/var/lib/logstash/country/` used above must have writeable permissions.

#### Setting up geolocation_db index
This will be our main index that will store information about the geolocation of an ip.

- create an index called 'geolocation_db'
```
curl -X PUT "localhost:9200/geolocation_db"
```

- Define the mapping for 'geolocation_db' index.
```
curl -X PUT "localhost:9200/geolocation_db/_mapping" -H 'Content-Type: application/json' --data-binary "@index_mapping/geolocation_db_mapping.json"
```

- Create an enrich policy.
```
curl -X PUT "localhost:9200/_enrich/policy/country-db-enrich-policy" -H 'Content-Type: application/json' --data-binary "@enrich_policies/geolocation_db_enrich_policy.json"
```

- Execute the `country-db-enrich-policy`.
```
curl -X POST "localhost:9200/_enrich/policy/country-db-enrich-policy/_execute"
```

- Create a pipeline for pushing data.
```
curl -X PUT "localhost:9200/_ingest/pipeline/geolocation-db-enrich-pipeline" -H 'Content-Type: application/json' --data-binary "@pipelines_processors/geolocation_db_pipeline_processor.json"
```

- Use logstash to push data from .csv file to an ES instance.
```
/usr/share/logstash/bin/logstash -f {path_where_repo_clone}/ipgeo-es-db-reader/logstash_config/geolocation_db.conf --path.data /var/lib/logstash/geolocation/ -w 8 -b 250
```
Just make sure that data dir `/var/lib/logstash/geolocation/` used above must have writeable permissions.
