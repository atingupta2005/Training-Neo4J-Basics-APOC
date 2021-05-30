# Graph Algorithms
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
    'myGraphNodeSimilarity',
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
)
````

- Match the similarity between Persons
```
CALL gds.nodeSimilarity.stream('myGraphNodeSimilarity')
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
  'myGraphDegreeOfCentrality',
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
   'myGraphDegreeOfCentrality',
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
match (n:User) DETACH DELETE n
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
gds.graph.drop("BetweennessCentrality")
CALL gds.graph.create('BetweennessCentrality', 'User', 'FOLLOWS')
```

- Call Betweenness Algorithm
```
CALL gds.betweenness.stream('BetweennessCentrality')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score
ORDER BY name ASC
```


## Page Rank Algorithm
- Measures the importance of each node within the graph, based on the number incoming relationships and the importance of the corresponding source nodes

### Create Sample Graph
```
CREATE
  (home:Page {name:'Home'}),
  (about:Page {name:'About'}),
  (product:Page {name:'Product'}),
  (links:Page {name:'Links'}),
  (a:Page {name:'Site A'}),
  (b:Page {name:'Site B'}),
  (c:Page {name:'Site C'}),
  (d:Page {name:'Site D'}),
  (home)-[:LINKS {weight: 0.2}]->(about),
  (home)-[:LINKS {weight: 0.2}]->(links),
  (home)-[:LINKS {weight: 0.6}]->(product),
  (about)-[:LINKS {weight: 1.0}]->(home),
  (product)-[:LINKS {weight: 1.0}]->(home),
  (a)-[:LINKS {weight: 1.0}]->(home),
  (b)-[:LINKS {weight: 1.0}]->(home),
  (c)-[:LINKS {weight: 1.0}]->(home),
  (d)-[:LINKS {weight: 1.0}]->(home),
  (links)-[:LINKS {weight: 0.8}]->(home),
  (links)-[:LINKS {weight: 0.05}]->(a),
  (links)-[:LINKS {weight: 0.05}]->(b),
  (links)-[:LINKS {weight: 0.05}]->(c),
  (links)-[:LINKS {weight: 0.05}]->(d);
```

- This graph represents eight pages, linking to one another
- Each relationship has a property called weight, which describes the importance of the relationship.

- Create Graph
```
CALL gds.graph.create(
  'myGraphPageRank',
  'Page',
  'LINKS',
  {
    relationshipProperties: 'weight'
  }
)
```

- Run Page Rank Algorithm
```
CALL gds.pageRank.stream('myGraphPageRank')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score
ORDER BY score DESC, name ASC
```
