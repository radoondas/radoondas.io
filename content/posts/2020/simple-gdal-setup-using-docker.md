---
title: "Simple GDAL Setup Using Docker"
date: 2020-07-20T08:00:00+01:00
draft: false
comments: true
share: true
categories: ["TIL", "hands on"]
tags: ["docker", "gdal"]
featured: true
categoryFeatured: false
---
I often work with different geospatial data sets, including geolocation information, and I want to present documents in the context of the Map's position. To get the job done, I use Elastic {{< a_blank "Kibana Maps" "https://www.elastic.co/guide/en/kibana/current/maps.html" >}} for my visualizations. Elastic provides {{< a_blank "Elastic map service" "https://www.elastic.co/elastic-maps-service" >}} (ESMS), which anyone can use for geospatial visualizations. The service includes different {{< a_blank "vector layers" "https://maps.elastic.co/#file/world_countries" >}} out of the
box and is kept up to date and improved continuously. Only recently, all the {{< a_blank "Europe regions were completed"
"https://www.elastic.co/blog/europe-regions-are-complete-on-elastic-maps-service" >}} on ESMS and released to the public.
<!--more-->

{{< title-h4 >}}The issue - dependencies for GDAL latest version{{< /title-h4 >}}
Elastic posted a **fantastic** blog post on {{< a_blank "How to Ingest Geospatial Data Into Elasticsearch with GDAL" "https://www.elastic.co/blog/how-to-ingest-geospatial-data-into-elasticsearch-with-gdal" >}}
which explains how to use {{< a_blank "GDAL library" "https://gdal.org/" >}} in the context of Elasticsearch and geospatial data. The blog post is self-explanatory, but when I run into dependency issues using `brew` to install the latest version of the library, I started to look for a different solution to set up the GDAL library with the latest features available.

{{< title-h4 >}}The solution - simple wrapper using Docker{{< /title-h4 >}}
While I was working on an interesting problem and once again run into dependency issues, I got help from a colleague (Thanks to Jose!) who gave me a tip to use wrapper script using Docker with the latest version of GDAL library. The implementation is straight forward - especially when you are already familiar with Docker.

#### Shell script
The first step is to create a shell script and save it into your favorite location. I prepared a script version for `Linux` and `macOS`, which should work
without any further changes. The only difference between versions is the mount for the home directory.

**macOS**
{{< box-start >}}
 ```bash
#!/bin/bash
docker run --rm --network=host -u $(id -u ${USER}):$(id -g ${USER}) \
-v /Users:/Users \
osgeo/gdal:alpine-small-latest \
ogr2ogr "$@"
```
{{< box-end >}}

**Linux**
{{< box-start >}}
```bash
#!/bin/bash
docker run --rm --network=host -u $(id -u ${USER}):$(id -g ${USER}) \
-v /home:/home \
osgeo/gdal:alpine-small-latest \
ogr2ogr "$@"
```
{{< box-end >}}

#### Execute the script
The next task is to use and execute your new wrapper for the GDAL library. Either with the full path to your script or update the script's path in `$PATH`
variable and use it as any other CLI command. I'll leave it up to you, and I have put together a few examples to help you out with the first run.

Note: The name of the saved script from the previous step is `dogr2ogr` in my case.

1. Ingest content of the GeoJSON file directly into Elasticsearch

{{< box-start >}}
```bash
dogr2ogr -lco INDEX_NAME=index-name  "ES:http://elastic:changeme@localhost:9200"  \
  "$(pwd)/file-name.geojson"
```
{{< box-end >}}
2. Ingest Shape file directly into Elasticsearch

{{< box-start >}}
```bash
dogr2ogr -lco INDEX_NAME=index-name  "ES:http://elastic:changeme@localhost:9200"  \
  "$(pwd)/file-name.shp"
```
{{< box-end >}}
3. Convert Shape file to GeoJSON format using full path to the script

{{< box-start >}}
```bash
~/opt/bin/dogr2ogr -f GeoJSON "$(pwd)/file-name.geojson" "$(pwd)/file-name.shp" \
  -lco RFC7946=YES
```
{{< box-end >}}

{{< notebox color="is-warning is-light" >}}Note: <code>$(pwd)</code> variable in above examples is important as you must use full paths to the file in
the context of the Docker to work correctly with the input.{{< /notebox >}}

----
This simple trick will save you lots of headaches when dealing with the dependencies of a specific library or version of application not only for the GDAL
library but also with other applications. Let me know if the setup works for you down in the comments or suggest any improvements you use daily.

