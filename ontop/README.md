Ontop Set-Up
--------------------

Ontology: Sames as LinkedGeoData  
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

Datasets analyzed: North-East Italy, Italy, Germany

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
