# Neo4j APOC Library
 - APOC stands for Awesome Procedures on Cypher
 - Before APOC’s release, developers needed to write their own procedures and functions for common functionality that Cypher or the Neo4j database had not yet implemented for support
 - Each developer might write his own version of these functions, causing a lot of duplication.
 - So, one of our Neo4j developers created the APOC library as a standard utility library for common procedures and functions.
 - This allowed developers across platforms and industries to use a standard library for common procedures and only write their own functionality for business logic and use-case-specific needs.
 - It includes over 450 standard procedures

## List available Procedures
```
CALL dbms.procedures()
```

## Installing the APOC Library
```
cd /var/lib/neo4j/plugins
wget https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/download/4.1.0.3/apoc-4.1.0.3-all.jar
sudo chown neo4j:neo4j apoc-4.1.0.3-all.jar
sudo chmod 755 apoc-4.1.0.3-all.jar
sudo vim /etc/neo4j/neo4j.conf
//replace the line #dbms.security.procedures.whitelist=apoc.coll.*,apoc.load.* with dbms.security.procedures.whitelist=apoc.coll.*,apoc.load.*,apoc.*
// Also add below line in last of the file
apoc.import.file.enabled=true
//Save and quit the file
sudo systemctl restart neo4j
```

## Help
```
CALL apoc.help("apoc")
```
```
CALL apoc.help("apoc.algo")
```

## apoc.load.json
- With apoc.load.json, it’s now very easy to load JSON data from any file or URL.
```
WITH "https://api.stackexchange.com/2.2/questions?pagesize=100&order=desc&sort=creation&tagged=neo4j&site=stackoverflow&filter=!5-i6Zw8Y)4W7vpy91PMYsKM-k9yzEsSC1_Uxlf" AS url
CALL apoc.load.json(url) YIELD value
UNWIND value.items AS item
RETURN item.title, item.owner.user_id, item.creation_date
```

- Can create Graph from JSON
```
WITH 'https://raw.githubusercontent.com/atingupta2005/Training-Neo4J-Basics-APOC/master/data/person.json' AS url
CALL apoc.load.json(url) YIELD value as person
MERGE (p:Person {name:person.name})
   ON CREATE SET p.age = person.age, p.children = size(person.children)
```

```
match (p:Person) return p
```


## Dijkstra Algorithms
### Shortest Path Finder
- Create Database
```
:USE system
CREATE DATABASE apocpathatin
:USE apocpathatin
```

```
MATCH (n) DETACH DELETE n
```

- Generate Data
```
MERGE (a:Loc {name:'A'})
MERGE (b:Loc {name:'B'})
MERGE (c:Loc {name:'C'})
MERGE (d:Loc {name:'D'})
MERGE (e:Loc {name:'E'})
MERGE (f:Loc {name:'F'})
MERGE (a)-[:ROAD {distance:50}]->(b)
MERGE (a)-[:ROAD {distance:50}]->(c)
MERGE (a)-[:ROAD {distance:500}]->(f)
MERGE (a)-[:ROAD {distance:100}]->(d)
MERGE (b)-[:ROAD {distance:40}]->(d)
MERGE (c)-[:ROAD {distance:40}]->(d)
MERGE (c)-[:ROAD {distance:80}]->(e)
MERGE (d)-[:ROAD {distance:30}]->(e)
MERGE (d)-[:ROAD {distance:80}]->(f)
MERGE (e)-[:ROAD {distance:40}]->(f);
```

- Show Graph
```
match (n) return n
```

- Now let’s try to find shortest path from A to F
```
MATCH (c1:Loc { name: 'A' })
MATCH (c2:Loc { name: 'F' })
MATCH path = shortestPath((c1)-[*]-(c2))
RETURN path
```

- if we want to calculate shortest path with usage of distance
```
MATCH (c1:Loc { name: 'A' })
MATCH (c2:Loc { name: 'F' })
CALL apoc.algo.dijkstra(c1, c2, 'ROAD', 'distance') YIELD path, weight
RETURN path, weight
```
