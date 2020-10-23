Ontop Set-Up
--------------------

To conduct the experiments you should set up an Ontop SPARQL endpoint which can be either done via CLI or Docker, but for reliability Docker is recommended.

The Docker image [ontop/ontop-endpoint](https://hub.docker.com/r/ontop/ontop-endpoint) is for fast setting up an Ontop SPARQL endpoint. One can either use this image directly, or create a dedicated image based on this image.

1. Go to the endpoint/ directory. You can download the following files and and paste them in `input/`:
   * <a href="https://github.com/alpano-unibz/LinkedGeoData/blob/ontop-dev/linkedgeodata-ontop/gdm_vkg.owl">this OWL Ontology file</a>
   * <a href="https://github.com/alpano-unibz/LinkedGeoData/blob/ontop-dev/linkedgeodata-ontop/gdm_vkg.obda">this mapping file</a>
   * <a href="https://github.com/alpano-unibz/LinkedGeoData/blob/ontop-dev/linkedgeodata-ontop/gdm_vkg.properties">this properties file</a>
2. Make sure to have the `jdbc/` directory and the JDBC driver inside.</li>

In addition, we need the PostgreSQL database running with data loaded like the LinkedGeoData project.

**NB**: Linux users have to modify the property `jdbc.url` in `input/university-complete.docker.properties`. Replace `host.docker.internal` with the IP address of your machine (you can see it running the `ifconfig` command).

**Create a dedicated Docker image**
We recommend to deploy a self-contained image, for which we can write a complete `Dockerfile`:
```
FROM ontop/ontop-endpoint
WORKDIR /opt/ontop
COPY input/university-complete.ttl input/university-complete.obda input/university-complete.docker.properties input/ 
COPY jdbc/h2-1.4.196.jar jdbc/
EXPOSE 8080
ENTRYPOINT java -cp ./lib/*:./jdbc/* -Dlogback.configurationFile=file:./log/logback.xml \
        it.unibz.inf.ontop.cli.Ontop endpoint \
        --ontology=input/university-complete.ttl \
        --mapping=input/university-complete.obda \
        --properties=input/university-complete.docker.properties \
        --cors-allowed-origins=http://yasgui.org \
        --lazy # if needed
```

Then, run the commands to build and run the Docker image:
```
$ docker build -t my-ontop-endpoint .
$ docker run -it --rm --name my-running-ontop-endpoint -p 8080:8080 my-ontop-endpoint
```

Further details can also be found on the official Ontop website: https://ontop-vkg.org/tutorial/endpoint/endpoint-docker.html

**Further details on Ontology and Mappings**
Ontology: Same as LinkedGeoData  
Mappings: Conversion of the Sparqlify mappings with some hand-crafted scripts and manual revision in Protege with the Ontop plugin Mapping Manager

Sample class mapping:
```
mappingId   lgd_node_tags_resource_kv_Airport
target      :node{node_id} a :Airport .
source      SELECT * FROM lgd_node_tags_resource_kv
                WHERE object=’http://linkedgeodata.org/ontology/Airport’
                AND ((property LIKE ’http://linkedgeodata.org/ontology/\%’) OR (property LIKE ’http://www.w3.org/\%’))
```

Ontop Experiments
--------------------

The sample datasets were retrieved on August 1st from Geofabrik at http://download.geofabrik.de/europe.html.
Datasets analyzed included North-East Italy, Italy, Germany (the datasets are too large to be uploaded here)

Table: Queries Performed

| Operation | Query | Description| 
| --------- | ----- | ---------- | 
| Q1 | Distance | Find OSM points within a distance from DBpedia location belonging to class| 
| Q2 | Distance | Find OSM points within a polygon belonging to class| 
| Q3 | Intersection | Find intersection between OSM points and predefined polygon| 
| Q4 | Intersection | Find intersection between OSM linestrings| 
| Q5 | Within | Find points within polygon| 
| Q6 | Contains | Find whether a polygon contains geometries| 
| Q7 | Buffer+Within | Find points within a 500 meter buffer of a location| 

With regard to the class filters, we defined two sets of classes which are analyzed across all datasets:
* Amenities which includes Bar, Restaurant, Pharmacy, Library, Bank (Q1, Q2, Q3, Q5, Q6, Q7)
* Networks which includes Motorway, Canal (Q4)

Table: Query Response Time (seconds)

| Dataset | Query | Sparqlify | Ontop | 
| ------- | ----- | --------- | ----- | 
| North-East | Q1 | 59.329 | 0.846| 
| North-East | Q2 | 3.616 | 0.311
| North-East | Q3 | 3.944 | 0.301
| North-East | Q4 | NA | 1320.733
| North-East | Q5 | NS | 0.199
| North-East | Q6 | NS | 0.18
| North-East | Q7 | NS | 0.179
| Italy | Q1 | 77.873 | 1.536
| Italy | Q2 | 4.131 | 0.54
| Italy | Q3 | 8.807 | 0.691
| Italy | Q4 | NA | 11312.91
| Italy | Q5 | NS | 0.742
| Italy | Q6 | NS | 0.74
| Italy | Q7 | NS | 0.735
| Germany | Q1 | 88.553 | 2.117
| Germany | Q2 | 4.564 | 1.485
| Germany | Q3 | 4.677 | 2.141
| Germany | Q4 | NA | 1632.594
| Germany | Q5 | NS | 1.496
| Germany | Q6 | NS | 1.461
| Germany | Q7 | NS | 1.496

where  
NA: Could not run due to memory constraints.  
NS: Could not run due to not being supported.
