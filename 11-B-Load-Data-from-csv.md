# Import Data using CSV

## CSV Format/Requirements
- The character encoding must be UTF-8.
- The end line termination is system dependent, for example, \n on Unix or \r\n on Windows.
- The terminator must be a comma , unless specified otherwise using the FIELDTERMINATOR option.
- The character for string quotation is the double quote " (these are stripped off when the data is read in).
- Any characters that need to be escaped can be escaped with the backslash \ character.

## Create a blank database:
```
:USE system
CREATE DATABASE atindb_music
SHOW DATABASES
:USE atindb_music
CALL db.schema.visualization()
```


## Ways to Import CSV Files
- LOAD CSV Cypher command
  - Great starting point and handles small- to medium-sized data sets (up to 10 million records)
- neo4j-admin bulk import tool
  - Command line tool useful for straightforward loading of large data sets


### LOAD CSV command with Cypher
- Import a CSV File
```
LOAD CSV FROM 'https://www.quackit.com/neo4j/tutorial/genres.csv' AS line
CREATE (:Genre { GenreId: line[0], Name: line[1]})
```

- Show data
```
MATCH (n:Genre) RETURN n
```

- Using linenumber() with LOAD CSV
  - For certain scenarios, like debugging a problem with a csv file, it may be useful to get the current line number that LOAD CSV is operating on
```
LOAD CSV FROM 'https://www.quackit.com/neo4j/tutorial/tracks.csv' AS line
RETURN linenumber() AS number, line
```

- Important Tips for LOAD CSV
  - All data from the CSV file is read as a string
    - Use toInteger(), toFloat(), split() or similar functions to convert
  - Import a CSV file containing Headers

- Converting Data Values with LOAD CSV
  - Remember that Neo4j does not store null values
  - Null or empty fields in a CSV files can be skipped or replaced with default values in LOAD CSV.
  - Suppose we have CSV file with data as below:
```
Id,Name,Location,Email,BusinessType
1,Neo4j,San Mateo,contact@neo4j.com,P
2,AAA,,info@aaa.com,
3,BBB,Chicago,,G
```

- Here are some examples of importing this data.
```
//skip null values
LOAD CSV WITH HEADERS FROM 'file:///companies.csv' AS row
WITH row WHERE row.Id IS NOT NULL
MERGE (c:Company {companyId: row.Id});
// clear data
MATCH (n:Company) DELETE n;
//set default for null values
LOAD CSV WITH HEADERS FROM 'file:///companies.csv' AS row
MERGE (c:Company {companyId: row.Id, hqLocation: coalesce(row.Location, "Unknown")})
```

- Load CSV with headers
```
LOAD CSV WITH HEADERS FROM 'https://www.quackit.com/neo4j/tutorial/tracks.csv' AS line
CREATE (:Track { TrackId: line.Id, Name: line.Track, Length: line.Length})
```

- Show data
```
MATCH (n:Track) RETURN n
```

- Custom Field Delimiter
  - You can specify a custom field delimiter if required. For example, you could specify a semi-colon instead of a comma if that's how the CSV file is formatted.
```
LOAD CSV WITH HEADERS FROM 'https://www.quackit.com/neo4j/tutorial/tracks.csv' AS line FIELDTERMINATOR ';'
CREATE (:Track { TrackId: line.Id, Name: line.Track, Length: line.Length})
```


- Importing Large Files
  - If you're going to import a file with a lot of data, the PERODIC COMMIT clause can be handy.
  - Using PERIODIC COMMIT instructs Neo4j to commit the data after a certain number of rows.
  - The default is 1000 rows, so the data will be committed every thousand rows.
```
USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'https://www.quackit.com/neo4j/tutorial/tracks.csv' AS line
CREATE (:Track { TrackId: line.Id, Name: line.Track, Length: line.Length})
```


#### Example for importing MovieGraph
- Load the data from the persons.csv file
```
LOAD CSV WITH HEADERS FROM "file:///persons.csv" AS csvLine
CREATE (p:Person {id: toInteger(csvLine.id), name: csvLine.name})
```

- Load the data from the movies.csv file
```
LOAD CSV WITH HEADERS FROM "file:///movies.csv" AS csvLine
MERGE (country:Country {name: csvLine.country})
CREATE (movie:Movie {id: toInteger(csvLine.id), title: csvLine.title, year:toInteger(csvLine.year)})
CREATE (movie)-[:ORIGIN]->(country)
```

- Load the data from the roles.csv file
```
USING PERIODIC COMMIT 500
LOAD CSV WITH HEADERS FROM "file:///roles.csv" AS csvLine
MATCH (person:Person {id: toInteger(csvLine.personId)}), (movie:Movie {id: toInteger(csvLine.movieId)})
CREATE (person)-[:ACTED_IN {role: csvLine.role}]->(movie)
```

- Validate the imported data
```
MATCH (n)-[r]->(m) RETURN n, r, m
```



## neo4j-admin bulk import tool
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

- Importing the data
 - Paths to node data is defined with the --nodes option.
 - Paths to relationship data is defined with the --relationships option.
```
bin/neo4j-admin import --database=<database_name> --nodes=import/movies.csv --nodes=import/actors.csv --relationships=import/roles.csv
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
bin/neo4j-admin import --database=neo4j --nodes=import/movies3-header.csv,import/movies3.csv --nodes=import/actors3-header.csv,import/actors3.csv --relationships=import/roles3-header.csv,import/roles3.csv
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
bin/neo4j-admin import --database=neo4j --nodes="import/movies4-header.csv,import/movies4-part.*" --nodes="import/actors4-header.csv,import/actors4-part.*" --relationships="import/roles4-header.csv,import/roles4-part.*"
```
