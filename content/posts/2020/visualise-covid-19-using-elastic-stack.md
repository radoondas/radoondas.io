---
title: "Visualise Covid-19 data Using Elastic Stack (Slovakia)"
date: 2020-07-31T08:00:00+01:00
draft: false
comments: true
share: true
categories: ["hands-on"]
tags: [ "kibana", "logstash", "maps", "covid-19" ]
featured: true
categoryFeatured: false
---
Covid-19 and its implications are the topic number one for quite some time. The situation naturally provides us a new dataset which any data analyst can process with different tools.

Despite this is not a happy dataset and one I would rather not be part of, it is also an opportunity to use tools, learn new applications, and hope that anything we do will help in some way to other people around us. Whether it is a scientist looking for a different approach to solve the problem or a student learning new tools using interesting data right now, everyone can benefit. Because I believe that we learn by doing 'things', I am presenting a complete hands-on example based on Slovakia's data. The same methodology can be applied for similar use cases or just as a proof of concept when needed.

<!--more-->

The live dashboard of the setup is located at {{< a_blank "covid-19.radoondas.io" "https://covid-19.radoondas.io" >}} and the source code in the github {{< a_blank "repository" "https://github.com/radoondas/covid-19-slovakia" >}}.

{{< figure src="/images/2020/07/dashboard-view.png" title="Covid-19 Dashboard" >}}

{{< title-h4 >}}The Data{{< /title-h4 >}}

All over the world, many people, organizations, and companies provide different datasets about the Covid-19 outbreak. Accessibility and quality or
completeness depend on each source. Because I live in Slovakia, I focus on local data. Unfortunately, we have no official complete and
machine-readable data source provided by government authorities.

Since the beginning of the outbreak, we had only a few sources available, which were technically merged into one source over time. Government webpage dedicated to {{< a_blank "covid-19" "https://korona.gov.sk/" >}}. Data are provided by {{< a_blank "National health information centre" "http://www.nczisk.sk/en/Pages/default.aspx" >}}. Access to any machine-readable and open data source is impossible, or I am not aware of it. I consider this as unfortunate. Another source is news that scrapes the official data and possibly enhances with other reports from their resources. This approach does not make for a good and reliable data source - primarily if not published freely. Everyone keeps data for themselves as far as I know about (please correct me if I am wrong).

Because there is no official data source, I decided to put together my own and open it to the public. I believe that the data can be
reused, distributed, and enhanced if anyone is willing to do so. The data file {{< a_blank "file" "https://github.com/radoondas/covid-19-slovakia/blob/master/data/covid-19-slovensko.csv" >}} is available in the GitHub repository, and I am happy to accept Issues or PR's with data enhancements.

It is important to mention other visualizations available in Slovakia, which you can check.
- {{< a_blank "korona.gov.sk" "https://korona.gov.sk/koronavirus-na-slovensku-v-cislach/" >}}
- {{< a_blank "arcgis" "https://experience.arcgis.com/experience/3430195d620344c38e81d307c252c14f/" >}}
- each significant media also has a form of visualization available for readers.

#####  Data set description
The data set consists CSV formatted rows with following header.
`date;city;infected;gender;note_1;note_2;healthy;died;region;age;district`

Columns description as of the publishing of this post. It may change over time. Please check for the latest description in Github
[repository](https://github.com/radoondas/covid-19-slovakia).

| Column name | Description                                                     |
|:------------|:----------------------------------------------------------------|
| date        | Date - the date of the record                                   |
| city        | City - the location of the person infected by covid-19          |
| infected    | Infected - number of infected                                   |
| gender      | Gender, `M` - male, `Ž` - female, `D` - children, `X` - unknown |
| note_1      | Note 1                                                          |
| note_2      | Note 2                                                          |
| healthy     | Healthy - number of people who recovered from the virus             |
| died        | Dead - number people who died                                   |
| region      | Region                                                          |
| age         | Age                                                             |
| district    | District                                                        |

{{< title-h4 >}}Architecture and Tools{{< /title-h4 >}}

The application stack of choice is
{{< a_blank "Elastic Stack" "https://www.elastic.co/elastic-stack" >}} to manage ingestion and analyze the data
- {{< a_blank "Logstash" "https://www.elastic.co/logstash" >}} for data ingestion from CSV file into
- {{< a_blank "Elasticsearch" "https://www.elastic.co/elasticsearch/" >}} and
- {{< a_blank "Kibana" "https://www.elastic.co/kibana" >}} for the visualization and geospatial data analysis.

For those with visual understanding, the pipeline of the data flow is very simple and straight forward.

Logstash reads CSV file and indexing documents into Elasticsearch. Then the user is working with Kibana connected to Elasticsearch to view and analyze and visualize documents.
```text
+----------+     +----------+     +---------------+     +--------+     +------+
|          |     |          |     |               |     |        |     |      |
| CSV file +-----> Logstash +-----> Elasticsearch <-----> Kibana <-----+ User |
|          |     |          |     |               |     |        |     |      |
+----------+     +----------+     +---------------+     +--------+     +------+
```

##### Geospatial tools
I use Slovakia geospatial data, and I published a different [post](/posts/2020/import-slovakia-regional-gis-data) on how to import Slovakia's GIS data into Elasticsearch. The post dives deeper into the details on how to index geospatial documents. Please read the article to get familiar with the setup, as it will help you with the following tutorial.

If you do not have the GDAL library already installed, then read also post on how to [setup latest GDAL](/posts/2020/simple-gdal-setup-using-docker) using simple wrapper script for Docker image.

#####  Optional
_Working Docker_ environment in case you need to set up GDAL wrapper and run Elasticsearch cluster using Docker.

{{< title-h4 >}}How to - tutorial{{< /title-h4 >}}

In the following section, I will describe the minimal steps required to build the same live dashboard on your local machine that will look like the dashboard at {{< a_blank "covid-19.radoondas.io" "https://covid-19.radoondas.io/" >}}.

1. Clone my {{< a_blank "repository" "https://github.com/radoondas/covid-19-slovakia" >}} from Github to get all necessary data and configuration files.
```bash
   $ git clone https://github.com/radoondas/covid-19-slovakia.git
```

2. Navigate to your local copy of the repository.
```bash
   $ cd covid-19-slovakia
```

3. Make sure that you have your Elasticsearch cluster together with Kibana up and running. It can be any cluster, but a single Elasticsearch node with Kibana will serve the purpose perfectly fine. If you do not have your cluster at hand, use prepared Docker configuration located in the repository root - `docker-compose.yml`.

   To use `docker-compose` environment run following command, and it will spin up 1 node cluster with version 7.8.1.

   Optionally, use {{< a_blank "Elastic cloud" "https://cloud.elastic.co/" >}} to spin up the cluster.

```bash
   $ docker-compose up -d
```

4. Verify that your cluster is up and running. Open your browser and go to Kibana url `http://127.0.0.1:5601/`. You now see a `Home page` of
   Kibana.

5. Ingest geospatial data into Elasticsearch. I generated `geojson` source files for all Towns/Cities/Villages (Mestá) together with Regions (Kraje) and Districts (Okresy). You can find those files in the {{< a_blank "data" "https://github.com/radoondas/covid-19-slovakia/tree/master/data" >}} folder of the repository (`obce.json`, `kraje.json`, `okresy.json`). JSON files are pre-generated from {{< a_blank "Geoportal" "https://www.geoportal.sk/sk/zbgis_smd/na-stiahnutie/" >}} using the GDAL library for format conversion.

   Read more in the section above dedicated to **geospatial tools**. After the import, you now have 3 indices in the cluster named `kraje` , `obce`, and `okresy`.

   Commands using the GDAL library for the reference (copy/paste might not work depending on your GDAL install method):
```bash
   $ ogr2ogr -lco INDEX_NAME=kraje "ES:http://localhost:9200" -lco NOT_ANALYZED_FIELDS={ALL} "$(pwd)/data/kraje.json"
   $ ogr2ogr -lco INDEX_NAME=obce "ES:http://localhost:9200" -lco NOT_ANALYZED_FIELDS={ALL} "$(pwd)/data/obce.json"
   $ ogr2ogr -lco INDEX_NAME=okresy "ES:http://localhost:9200" -lco NOT_ANALYZED_FIELDS={ALL} "$(pwd)/okresy.json"
```

```bash
   #Imported indices
   GET _cat/indices/obce,kraje,okresy?v
   health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
   yellow open   okresy 3LYqUkRcTdiBf1XQ0P7h-A   1   1         79            0      5.1mb          5.1mb
   yellow open   obce   T-k8wud0Q5eH2J75nLITmg   1   1       2927            0     26.5mb         26.5mb
   yellow open   kraje  2AA2LM0NRUO9vfqILaZtbg   1   1          8            0      1.8mb          1.8mb
```

6. Ingest data into Elasticsearch and stop Logstash Docker container when documents are indexed. This is a one-time index job. Run the following command from the root of the repository. If all works as expected, then you will see in the output no errors related to the ingest process and documents currently being index will also be printed on the screen.

   There are a few essential pieces in the ingest command.
   - we use a special template for correct mapping in elasticsearch `-v $(pwd)/template.json:/tmp/template.json`
   - we point input file to the local source file `-v $(pwd)/data/covid-19-slovensko.csv:/tmp/covid-19-slovensko.csv`
   - we use our custom Logstash configuration `-v $(pwd)/ls.conf:/usr/share/logstash/pipeline/logstash.conf`
 ```bash
    docker run --rm -it --network=host \
    -v $(pwd)/template.json:/tmp/template.json \
    -v $(pwd)/data/covid-19-slovensko.csv:/tmp/covid-19-slovensko.csv \
    -v $(pwd)/ls.conf:/usr/share/logstash/pipeline/logstash.conf \
    docker.elastic.co/logstash/logstash:7.8.1
```

```bash
   # Check the index
   GET _cat/indices/covid-19-sk?v
   health status index       uuid                   pri rep docs.count docs.deleted store.size pri.store.size
   yellow open   covid-19-sk -5VUg-y_QJmItQ2qY6zoVA   1   1       1751            0    557.5kb        557.5kb
```

5. Additionaly, import the template and actual data for annotations used in visualizations
```bash
   cd data
   # Import index template
   curl -s -H "Content-Type: application/x-ndjson" -XPUT "localhost:9208/_template/milestones" \
     --data-binary "@template_milestones.json"; echo
   # You should see message: {"acknowledged":true}
   # Index actual data
   curl -s -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/_bulk --data-binary "@milestones.bulk"; echo
```
```bash
   # List template
   GET _cat/templates/milestones?v
   name       index_patterns order version composed_of
   milestones [milestones]   0
   # List index
   GET _cat/indices/milestones?v
   health status index      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
   yellow open   milestones e3c-cNfaTg2JCyzGNZmX5Q   1   1         17            0      7.6kb          7.6kb
```

6. {{< a_blank "Import" "https://www.elastic.co/guide/en/kibana/current/managing-saved-objects.html#managing-saved-objects-export-objects" >}} `Saved Objects` from provided {{< a_blank "visualisations file" "https://github.com/radoondas/covid-19-slovakia/blob/master/data/visualisations.ndjson" >}} and adjust patterns appropriately if needed.

    _Note, if you named imported indices for geospatial data as in the example (kraje,okresy, obce), then there is no need to adjust index patterns._
    {{< figure src="/images/2020/07/import-saved-objects.png" title="" >}}

7. Navigate to `Dashboards` and find the one with name `[covid-19] Overview Dashboard`, open and check the content.
{{< figure src="/images/2020/07/list-dashboards.png" title="" >}}
If all goes right, this is the dashboard you will see (or similar as visualisations might develop over the time).
{{< figure src="/images/2020/07/dashboard-view.png" title="Covid-19 Dashboard" >}}
{{< figure src="/images/2020/07/dashboard-view-white.png" title="Covid-19 Dashboard in white" >}}

Note: The adjustment of all saved objects should not be necessary if you used `ogr2ogr` (or the wrapper) to import the geojson data. The mapping and index name will match those in `Saved objects`. If you use a different setup, then you will need to adjust ID's of pattern inside of the `visualisations.ndjson` file.

##### Additional documentation
- Logstash {{< a_blank "CSV filter plugin" "https://www.elastic.co/guide/en/logstash/current/plugins-filters-csv.html" >}}
- Elasticsearch {{< a_blank "templates" "https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html" >}}

---
Let me know if you have learned something new, or if you were able to follow this hands-on tutorial successfully. Feel free to comment or suggest me how to improve the content so you can enjoy it more.
