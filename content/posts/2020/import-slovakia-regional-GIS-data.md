---
title: "Import Slovakia Regional GIS Data into Elastic Stack"
date: 2020-07-23T08:00:00+01:00
draft: false
comments: true
share: true
categories: [ "TIL", "hands on" ]
tags: [ "gdal", "GIS", "kibana", "maps" ]
featured: false
categoryFeatured: false
---
{{< a_blank "Elastic Map Service" "https://www.elastic.co/elastic-maps-service" >}} nowadays comes with {{< a_blank "Slovakia regions"
"https://maps.elastic.co/#file/slovakia_regions_v1" >}}, which you can happily use with your geospatial data. If you want to have higher precision of Region's shapes, or you do need to implement the level of 'districts' (okresy) or go to the detail of a single city/town, you must use custom map layers.
Using different layers is considered a basic task for any GIS application. Possibility to add custom layers in
[Kibana Maps](https://www.elastic.co/guide/en/kibana/current/maps.html) is available since the introduction and is improving with every release.
<!--more-->

{{< title-h4 >}}Intro and the data{{< /title-h4 >}}
{{< a_blank "Kibana Maps" "https://www.elastic.co/guide/en/kibana/current/maps.html" >}} allows
you to work with geospatial data in your documents. Besides showing the location on the map, layers with geo shapes will allow you to add more
functionality like aggregation over shapes, connect documents to specific locations, and more.

When I was working with geospatial data set from Slovakia, I came across the need to use better precision and present my documents in a map with
resolution as low as city shapes. This requirement is no issue when you have data at hand that are available in such format so they can be used in Kibana Layer. They can be in different formats, and typically are in the format of `shape`
[file](https://doc.arcgis.com/en/arcgis-online/reference/shapefiles.htm) `.shp` or `GeoJSON`
[format](https://geojson.org/).

Often, the task to find the right geospatial data is very hard, but I found Slovakia's geospatial data in
{{< a_blank "Geoportál" "https://www.geoportal.sk/sk/zbgis_smd/na-stiahnutie/" >}} free to download and use.

{{< figure src="/images/2020/07/geoportal.png" title="Geoportál" >}}

{{< title-h4 >}}How to import geospatial documents{{< /title-h4 >}}
Download the package for **First level of generalization (Prvá úroveň generalizácie)** using the link
[Esri SHP](https://www.geoportal.sk/files/zbgis/na_stiahnutie/shp/ah_shp_1.zip). The selected level of detail has a reasonable size and precision.

![Geoportál detail](/images/2020/07/geoportal-detail.png)

Once you `unzip` the package, the content consists of different file types. We are interested in `shape` files with `.shp` file extension. Make sure you can see all the shapefiles, as shown below. Note, there are more files that are the content of the zip package. Do not touch them or delete them.
Leave them as they are. They are essential for the GDAL library.
<div class="box ">

```bash
$ ls -l *.shp
-rw-rw-rw-@ 1 user  group   495124 Mar  2 10:15 kraj_1.shp
-rw-rw-rw-@ 1 user  group  8084120 Mar  2 10:15 ku_1.shp
-rw-rw-rw-@ 1 user  group  7325616 Mar  2 10:15 obec_1.shp
-rw-rw-rw-@ 1 user  group  1400224 Mar  2 10:15 okres_1.shp
-rw-rw-rw-@ 1 user  group   161820 Mar  2 10:15 sr_1.shp

```
</div>

To import geospatial data into Elasticsearch and to be able to use them in Kibana, use {{< a_blank "GDAL" "https://gdal.org/" >}} library. I have a  different [post](/posts/2020/simple-gdal-setup-using-docker/) dedicated to just how to set up the GDAL library using a wrapper script around the latest Docker image.
Please, read and follow the post if you do not have the latest version of GDAL or never used the library before.

We will use the latest GDAL library. You can read about options either in the official GDAL documentation or read Elastic
[blog post](https://www.elastic.co/blog/how-to-ingest-geospatial-data-into-elasticsearch-with-gdal) explaining in more depth how to work with the
library.

{{< notebox color="is-warning is-light" >}}Make sure that you have up and running Elasticsearch cluster with Kibana before importing shape data using
following commands.{{< /notebox >}}

<div class="box ">

```bash
$ ogr2ogr -lco INDEX_NAME=kraje -lco NOT_ANALYZED_FIELDS={ALL} \
  "ES:http://localhost:9200" "$(pwd)/kraj_1.shp"
  
$ ogr2ogr -lco INDEX_NAME=obce -lco NOT_ANALYZED_FIELDS={ALL} \
  "ES:http://localhost:9200" "$(pwd)/obec_1.shp"
    
$ ogr2ogr -lco INDEX_NAME=okresy -lco NOT_ANALYZED_FIELDS={ALL} \
  "ES:http://localhost:9200" "$(pwd)/okres_1.shp"
```
</div>

With the wrapper:
<div class="box ">

```bash
$ /path/to/dogr2ogr -lco INDEX_NAME=kraje -lco NOT_ANALYZED_FIELDS={ALL} \ 
  "ES:http://localhost:9200" "$(pwd)/kraj_1.shp"
  
$ /path/to/dogr2ogr -lco INDEX_NAME=obce -lco NOT_ANALYZED_FIELDS={ALL} \
  "ES:http://localhost:9200" "$(pwd)/obec_1.shp"
    
$ /path/to/dogr2ogr -lco INDEX_NAME=okresy -lco NOT_ANALYZED_FIELDS={ALL} \
  "ES:http://localhost:9200" "$(pwd)/okres_1.shp"
```
</div>

In case you need to connect to secured cluster do not hestitate and use authentication `http://user:passw@localhost:9200`.
<div class="box ">

```bash
$ ogr2ogr -lco INDEX_NAME=<index_name> -lco NOT_ANALYZED_FIELDS={ALL} \
  "ES:http://user:passw@localhost:9200" "$(pwd)/shape_file.shp"
```
</div>

The above commands will create three new indices in your Elasticsearch cluster. Use simple `_cat` API to check.
<div class="box ">

```bash
GET _cat/indices/obce,kraje,okresy?v
health status index  uuid   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   okresy uidA   1   1         79            0      5.1mb          5.1mb
yellow open   obce   uidB   1   1       2927            0     26.5mb         26.5mb
yellow open   kraje  uidC   1   1          8            0      1.8mb          1.8mb
```
</div>

{{< title-h4 >}}Layers in Kibana Maps{{< /title-h4 >}}
When the import is finished, you are ready to use your fresh geospatial data in {{< a_blank "Kibana Maps" "https://www.elastic.co/guide/en/kibana/current/maps.html" >}}

1. Navigate to Kibana `http://<IP or 127.0.0.1>:5601` and in the upper left menu go to `Management` application -> Stack management -> {{< a_blank "Index pattern" "https://www.elastic.co/guide/en/kibana/current/managing-fields.html" >}}.

2. Create
   [Index patterns](https://www.elastic.co/guide/en/kibana/current/tutorial-define-index.html) for all indices you just indexed. Do **not** use `Time Filter` for your index patterns. They are static data that do not change over time. Follow the example and create a dedicated Index pattern for each index.

{{< figure src="/images/2020/07/index-pattern-1.png" title="Create Index pattern" >}}
{{< figure src="/images/2020/07/index-pattern-2.png" title="Configure Index pattern" >}}
{{< figure src="/images/2020/07/index-pattern-3.png" title="Detail of the Index pattern" >}}
{{< figure src="/images/2020/07/index-pattern-4.png" title="List of Index patterns" >}}

3. Open [Maps](https://www.elastic.co/guide/en/kibana/current/maps.html) application in Kibana and you will see an empty map if this is your first one.

{{< figure src="/images/2020/07/open-kibana-maps.png" title="" >}}

4. On your new map, go ahead and `Add layer`.

{{< figure src="/images/2020/07/add-layer.png" title="Add layer" >}}

5. Choose `Documents` from the list to show on your Layer. Do not forget to **Save** the layer after  you `Add layer` before moving to the next layer.

{{< figure src="/images/2020/07/add-documents.png" title="" >}}
{{< figure src="/images/2020/07/layer-documents.png" title="" >}}

6. Repeat the process for all 3 indices and enjoy 3 layers of geospatial data.

{{< figure src="/images/2020/07/all-layers.png" title="All document layers" >}}

You are now ready to use your newly imported geospatial data with Kibana Maps. Please check the documentation for more information on how to navigate or configure in detail your layers. I plan to post more articles in the future on the topic, so come back and check the new content.

{{< notebox color="is-warning is-light" >}}
It is possible to import geospatial data using <a href="https://www.elastic.co/guide/en/kibana/current/geojson-upload.html">Geojson upload</a> functionality in Maps. This approach is a simplification of the above process, but also has limitations on how much data you are allowed to import (size limit) or how they look like. Geojson upload works for a small and simple dataset, and I do suggest to use import to get familiar with Maps application faster.
{{< /notebox >}}

----

Let me know if you have learned something new, or if the hands-on post is understandable. Feel free to comment or suggest me how to improve the content so you can enjoy it more.
