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
CREATE DATABASE atindbmusic
SHOW DATABASES
:USE atindbmusic
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
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/atingupta2005/Training-Neo4J-Basics-APOC/master/data/companies.csv' AS row
WITH row WHERE row.Id IS NOT NULL
MERGE (c:Company {companyId: row.Id});
//Show Data
MATCH (n:Company) return n;
// clear data
MATCH (n:Company) DELETE n;
//set default for null values
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/atingupta2005/Training-Neo4J-Basics-APOC/master/data/companies.csv' AS row
MERGE (c:Company {companyId: row.Id, hqLocation: coalesce(row.Location, "Unknown")})
MATCH (n:Company) return n;
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
:auto USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM 'https://www.quackit.com/neo4j/tutorial/tracks.csv' AS line
CREATE (:Track { TrackId: line.Id, Name: line.Track, Length: line.Length})
```

## apoc.import.csv tool

### How to install apoc library?
- Refer [12-Neo4j-APOC-Library](12-Neo4j-APOC-Library)

### Create a blank database:
```
:USE system
CREATE DATABASE atinapcimport
:USE atinapcimport
```
- Import Data
```
CALL apoc.import.csv([{fileName: 'https://raw.githubusercontent.com/atingupta2005/Training-Neo4J-Basics-APOC/master/data/movies.csv', labels: []},
{fileName: 'https://raw.githubusercontent.com/atingupta2005/Training-Neo4J-Basics-APOC/master/data/actors.csv', labels: []}],
[{fileName: 'https://raw.githubusercontent.com/atingupta2005/Training-Neo4J-Basics-APOC/master/data/roles.csv'}
], {})
```

- Query Data
```
match (n) return n
```

- Visualize Schema
```
CALL db.schema.visualization()
```
