# Import using Neo4J admin

- The neo4j-admin import tool allows you to import CSV data to an empty database by specifying node files and relationship files.
- Prior to populated the graph, we have to format our data in a very particular way.
- The data
  - The data set contains information about movies, actors, and roles
  - Data for movies and actors are stored as nodes and the roles are stored as relationships.
- The files you want to import data from are:
  - movies.csv
  - actors.csv
  - roles.csv
- movies.csv
```
movieId:ID,title,year:int,:LABEL
tt0133093,"The Matrix",1999,Movie
tt0234215,"The Matrix Reloaded",2003,Movie;Sequel
tt0242653,"The Matrix Revolutions",2003,Movie;Sequel
```

- actors.csv
```
personId:ID,name,:LABEL
keanu,"Keanu Reeves",Actor
laurence,"Laurence Fishburne",Actor
carrieanne,"Carrie-Anne Moss",Actor
```

- Roles are represented by relationship data that connects actor nodes with movie nodes.
- There are three mandatory fields for relationship data:
  - :START_ID — ID refering to a node.
  - :END_ID — ID refering to a node.
  - :TYPE — The relationship type.
- In order to create a relationship between two nodes
  - the IDs defined in actors.csv and movies.csv are used for the :START_ID and :END_ID fields
- You also need to provide a relationship type (in this case ACTED_IN) for the :TYPE field.
- roles.csv
```
:START_ID,role,:END_ID,:TYPE
keanu,"Neo",tt0133093,ACTED_IN
keanu,"Neo",tt0234215,ACTED_IN
keanu,"Neo",tt0242653,ACTED_IN
laurence,"Morpheus",tt0133093,ACTED_IN
laurence,"Morpheus",tt0234215,ACTED_IN
laurence,"Morpheus",tt0242653,ACTED_IN
carrieanne,"Trinity",tt0133093,ACTED_IN
carrieanne,"Trinity",tt0234215,ACTED_IN
carrieanne,"Trinity",tt0242653,ACTED_IN
```

- Create New Database from Browser
```
:use system
create database atindbimport
:use atindbimport
```

- Importing the data
 - Paths to node data is defined with the --nodes option.
 - Paths to relationship data is defined with the --relationships option.


```
CALL apoc.import.csv([{fileName: 'https://raw.githubusercontent.com/atingupta2005/Training-Neo4J-Basics-APOC/master/data/movies.csv', labels: []}], [], {})
CALL apoc.import.csv([{fileName: 'https://raw.githubusercontent.com/atingupta2005/Training-Neo4J-Basics-APOC/master/data/actors.csv', labels: []}], [], {})

```

- Query the data
  - Now you can query the data

### Using separate header files
 - When dealing with very large CSV files it is more convenient to have the header in a separate file.
- movies3-header.csv
```
movieId:ID,title,year:int,:LABEL
```

- movies3.csv
```
tt0133093,"The Matrix",1999,Movie
tt0234215,"The Matrix Reloaded",2003,Movie;Sequel
tt0242653,"The Matrix Revolutions",2003,Movie;Sequel
```

- actors3-header.csv
```
personId:ID,name,:LABEL
```

- actors3.csv
```
keanu,"Keanu Reeves",Actor
laurence,"Laurence Fishburne",Actor
carrieanne,"Carrie-Anne Moss",Actor
```

- roles3-header.csv
```
:START_ID,role,:END_ID,:TYPE
```

- roles3.csv
```
keanu,"Neo",tt0133093,ACTED_IN
keanu,"Neo",tt0234215,ACTED_IN
keanu,"Neo",tt0242653,ACTED_IN
laurence,"Morpheus",tt0133093,ACTED_IN
laurence,"Morpheus",tt0234215,ACTED_IN
laurence,"Morpheus",tt0242653,ACTED_IN
carrieanne,"Trinity",tt0133093,ACTED_IN
carrieanne,"Trinity",tt0234215,ACTED_IN
carrieanne,"Trinity",tt0242653,ACTED_IN
```

- Importing the data
```
neo4j-admin import --database=neo4j --nodes=import/movies3-header.csv,import/movies3.csv --nodes=import/actors3-header.csv,import/actors3.csv --relationships=import/roles3-header.csv,import/roles3.csv
```

### Multiple input files
 - In addition to using a separate header file you can also provide multiple nodes or relationships files

- movies4-header.csv
```
movieId:ID,title,year:int,:LABEL
```

- movies4-part1.csv
```
tt0133093,"The Matrix",1999,Movie
tt0234215,"The Matrix Reloaded",2003,Movie;Sequel
```

- movies4-part2.csv
```
tt0242653,"The Matrix Revolutions",2003,Movie;Sequel
```

- actors4-header.csv
```
personId:ID,name,:LABEL
```

- actors4-part1.csv
```
keanu,"Keanu Reeves",Actor
laurence,"Laurence Fishburne",Actor
```

- actors4-part2.csv
```
carrieanne,"Carrie-Anne Moss",Actor
```

- roles4-header.csv
```
:START_ID,role,:END_ID,:TYPE
```

- roles4-part1.csv
```
keanu,"Neo",tt0133093,ACTED_IN
keanu,"Neo",tt0234215,ACTED_IN
keanu,"Neo",tt0242653,ACTED_IN
laurence,"Morpheus",tt0133093,ACTED_IN
laurence,"Morpheus",tt0234215,ACTED_IN
```

- roles4-part2.csv
```
laurence,"Morpheus",tt0242653,ACTED_IN
carrieanne,"Trinity",tt0133093,ACTED_IN
carrieanne,"Trinity",tt0234215,ACTED_IN
carrieanne,"Trinity",tt0242653,ACTED_IN
```

### Importing the data
```
neo4j-admin import --database=neo4j --nodes="import/movies4-header.csv,import/movies4-part.*" --nodes="import/actors4-header.csv,import/actors4-part.*" --relationships="import/roles4-header.csv,import/roles4-part.*"
```
