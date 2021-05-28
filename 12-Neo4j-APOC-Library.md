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

## Page Rank Algorithms
 - Let’s create some test data to run the PageRank algorithm on.
```
FOREACH (id IN range(0,1000) | CREATE (:Node {id:id}))
```

```
MATCH (n1:Node),(n2:Node) WITH n1,n2 LIMIT 1000000 WHERE rand() < 0.1
CREATE (n1)-[:KNOWS]->(n2)
```

- PageRank Procedure
  - PageRank is an algorithm used by Google Search to rank websites in their search engine results.
  - It is a way of measuring the importance of nodes in a graph.
  - PageRank counts the number and quality of relationships to a node to approximate the importance of that node.
  - PageRank assumes that more important nodes likely have more relationships.

```
MATCH (node:Node)
WHERE node.id %2 = 0
WITH collect(node) AS nodes
// compute over relationships of all types
CALL apoc.algo.pageRank(nodes) YIELD node, score
RETURN node, score
ORDER BY score DESC
```


## Install Graph Data Science Library (gds) library
```
cd /tmp
wget https://s3-eu-west-1.amazonaws.com/com.neo4j.graphalgorithms.dist/graph-data-science/neo4j-graph-data-science-1.6.0-standalone.zip
cd /var/lib/neo4j/plugins
unzip /tmp/neo4j-graph-data-science-1.6.0-standalone.zip
sudo chown neo4j:neo4j neo4j-graph-data-science-1.6.0.jar
sudo chmod 755 neo4j-graph-data-science-1.6.0.jar
```

- Add the following to your $NEO4J_HOME/conf/neo4j.conf file:

```
sudo vim /etc/neo4j/neo4j.conf
  dbms.security.procedures.unrestricted=gds.*
  dbms.security.procedures.whitelist=apoc.coll.*,apoc.load.*,apoc.*,gds.*
```
- Restart
```
sudo systemctl restart neo4j
```

- Verify Installation
```
RETURN gds.version()
CALL gds.list()
```

## Node Similarity
- Create Data
```
CREATE
  (alice:Person {name: 'Alice'}),
  (bob:Person {name: 'Bob'}),
  (carol:Person {name: 'Carol'}),
  (dave:Person {name: 'Dave'}),
  (eve:Person {name: 'Eve'}),
  (guitar:Instrument {name: 'Guitar'}),
  (synth:Instrument {name: 'Synthesizer'}),
  (bongos:Instrument {name: 'Bongos'}),
  (trumpet:Instrument {name: 'Trumpet'}),
  (alice)-[:LIKES]->(guitar),
  (alice)-[:LIKES]->(synth),
  (alice)-[:LIKES {strength: 0.5}]->(bongos),
  (bob)-[:LIKES]->(guitar),
  (bob)-[:LIKES]->(synth),
  (carol)-[:LIKES]->(bongos),
  (dave)-[:LIKES]->(guitar),
  (dave)-[:LIKES]->(synth),
  (dave)-[:LIKES]->(bongos);
```

```
match (p:Person)-[LIKES]- (i:Instrument) return p, i
```

- The following statement will create the graph and store it in the graph catalog
```
CALL gds.graph.create(
    'myGraph1',
    ['Person', 'Instrument'],
    {
        LIKES: {
            type: 'LIKES',
            properties: {
                strength: {
                    property: 'strength',
                    defaultValue: 1.0
                }
            }
        }
    }
);
````

- Match the similarity between Persons
```
CALL gds.nodeSimilarity.stream('myGraph1')
YIELD node1, node2, similarity
RETURN gds.util.asNode(node1).name AS Person1, gds.util.asNode(node2).name AS Person2, similarity
ORDER BY similarity DESCENDING, Person1, Person2
```


## Degree of Centrality
- Used to find popular nodes within a graph
- Create Data
```
CREATE
  (alice:User {name: 'Alice'}),
  (bridget:User {name: 'Bridget'}),
  (charles:User {name: 'Charles'}),
  (doug:User {name: 'Doug'}),
  (mark:User {name: 'Mark'}),
  (michael:User {name: 'Michael'}),
  (alice)-[:FOLLOWS {score: 1}]->(doug),
  (alice)-[:FOLLOWS {score: -2}]->(bridget),
  (alice)-[:FOLLOWS {score: 5}]->(charles),
  (mark)-[:FOLLOWS {score: 1.5}]->(doug),
  (mark)-[:FOLLOWS {score: 4.5}]->(michael),
  (bridget)-[:FOLLOWS {score: 1.5}]->(doug),
  (charles)-[:FOLLOWS {score: 2}]->(doug),
  (michael)-[:FOLLOWS {score: 1.5}]->(doug)
```

```
match (n:User) return n
```

- The following statement will create a graph using a reverse projection and store it in the graph catalog under the name 'myGraph'.
```
CALL gds.graph.create(
  'myGraph2',
  'User',
  {
    FOLLOWS: {
      orientation: 'REVERSE',
      properties: ['score']
    }
  }
)
```

- Now show the Degree of Centrality
```
CALL gds.degree.stream(
   'myGraph2',
   { relationshipWeightProperty: 'score' }
)
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score AS weightedFollowers
ORDER BY weightedFollowers DESC, name DESC
```


## Betweenness Centrality
- Betweenness centrality is a way of detecting the amount of influence a node has over the flow of information in a graph.
- It is often used to find nodes that serve as a bridge from one part of a graph to another.

- Create Data
```
CREATE
  (alice:User {name: 'Alice'}),
  (bob:User {name: 'Bob'}),
  (carol:User {name: 'Carol'}),
  (dan:User {name: 'Dan'}),
  (eve:User {name: 'Eve'}),
  (frank:User {name: 'Frank'}),
  (gale:User {name: 'Gale'}),
  (alice)-[:FOLLOWS]->(carol),
  (bob)-[:FOLLOWS]->(carol),
  (carol)-[:FOLLOWS]->(dan),
  (carol)-[:FOLLOWS]->(eve),
  (dan)-[:FOLLOWS]->(frank),
  (eve)-[:FOLLOWS]->(frank),
  (frank)-[:FOLLOWS]->(gale);
```

- Show Graph
```
match (n:User) return n
```

- Create and Save Graph
```
CALL gds.graph.create('myGraph5', 'User', 'FOLLOWS')
```

- Call Betweenness Algorithm
```
CALL gds.betweenness.stream('myGraph5')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score
ORDER BY name ASC
```
