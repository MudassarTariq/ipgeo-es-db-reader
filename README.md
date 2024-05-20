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

- Define the mapping for 'place_db' index. In Elasticsearch, a "mapping" is similar to a schema definition in a traditional database. It defines how documents and their fields are stored and indexed, including their data types, field-specific indexing options, analyzers, and other settings. I you want to see or modify the mappings just update the corresponding .json file [here](/index_mapping/).
```
curl -s -X PUT "localhost:9200/place_db/_mapping" -H 'Content-Type: application/json' --data-binary "@index_mapping/place_db_mapping.json"
```

- Use logstash to push data from .csv file to an ES instance. You can find the corresponding configuration files [here](/logstash_config/). These files contains necessary configuration to map csv data onto an index.

Before running below command make sure to add your place file(downloaded from ipgeolocation.io) path inside [geolocaiton_db.conf](/logstash_config/place_db.conf).


```
/usr/share/logstash/bin/logstash -f {path_where_repo_clone}/ipgeo-es-db-reader/logstash_config/place_db.conf --path.data /var/lib/logstash/place/
```

Just make sure that data dir `/var/lib/logstash/place/` have writeable permissions.

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

- Create an enrich policy. Enrichment policies are used to enrich documents with additional data from external sources before indexing them. This enrichment process enhances the search capabilities by adding relevant information to the documents, which may not be present in the original dataset. We're enhancing the `country_db` index by incorporating data from the `place_db` index.
If you want, you can have the enrich policies lited [here](/enrich_policies/)
```
curl -X PUT "localhost:9200/_enrich/policy/place-db-enrich-policy" -H 'Content-Type: application/json' --data-binary "@enrich_policies/country_db_enrich_policy.json"
```

- Execute the `place-db-enrich-policy`.
```
curl -X POST "localhost:9200/_enrich/policy/place-db-enrich-policy/_execute"
```

- Create an ingest pipeline for pushing data. An ingest pipeline will be helpful during document indexing. It will allows us to pre-process documents before they are indexed, enabling various transformations and enrichments of the data. This pipeline will use the `place-db-enrich-policy` enrich policy for pre-processing.
```
curl -X PUT "localhost:9200/_ingest/pipeline/country-db-enrich-pipeline" -H 'Content-Type: application/json' --data-binary "@pipelines_processors/country_db_pipeline_processor.json"
```

- Use logstash to push data from .csv file to an ES instance. You can find the corresponding configuration files [here](/logstash_config/). These files contain the necessary configurations to map CSV data onto an Elasticsearch index. Additionally, they include pipelines used to enrich the data.

Before running below command make sure to add your country file(downloaded from ipgeolocation.io) path inside [country_db.conf](/logstash_config/country_db.conf).


```
/usr/share/logstash/bin/logstash -f {path_where_repo_clone}/ipgeo-es-db-reader/logstash_config/country_db.conf --path.data /var/lib/logstash/country/
```
Just make sure that data dir `/var/lib/logstash/country/` have writeable permissions.

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

- Use logstash to push data from .csv file to an ES instance. Here you will see few extra options:
    - `-w` will be used for number of pipeline worker, which could improve processing throughput at the cost of increased resource usage.
    - `-b` sets the pipeline batch size, allowing each worker to collect and process n number of events before sending them to outputs.

Before running below command make sure to add your geolocation file(downloaded from ipgeolocation.io) path inside [geolocaiton_db.conf](/logstash_config/geolocation_db.conf).
```
/usr/share/logstash/bin/logstash -f {path_where_repo_clone}/ipgeo-es-db-reader/logstash_config/geolocation_db.conf --path.data /var/lib/logstash/geolocation/ -w 8 -b 250
```
Just make sure that data dir `/var/lib/logstash/geolocation/` have writeable permissions.
