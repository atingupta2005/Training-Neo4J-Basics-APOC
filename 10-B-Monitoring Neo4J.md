# Monitoring Neo4J

## Create some nodes
- Connect to Ubuntu Console and run below commands:
```
cypher-shell -u neo4j -p secret -d neo4j --format plain
> :use neo4j;
> MATCH (n) DETACH DELETE n;
```

- Create some records
```
> FOREACH (i IN RANGE(1,200) | CREATE (:Person {name:'Person' + i}));
```

## Monitoring queries
```
cypher-shell -u neo4j -p secret -d neo4j --format plain
```

- Execute a query that runs for longer than 1000 ms
```
> MATCH (a), (b), (c), (d) RETURN count(id(a));
```

- On Browser: Viewing currently running queries
```
:queries
```

- Killing the query
  - Click on the icon to the Right to kill the query

## Automating monitoring of queries
- Automate the killing of long-running queries
- On Browser, run:
```
CALL dbms.listQueries() YIELD query, elapsedTimeMillis, queryId, username
WHERE  NOT query CONTAINS toLower('LOAD')
AND elapsedTimeMillis > 1000
WITH query, collect(queryId) AS q
CALL dbms.killQueries(q) YIELD queryId
RETURN query, queryId;
```

## Monitoring connections
- Can view the current connections to a Neo4j instance from cypher-shell
```
Call dbms.listConnections();
```

- Terminate the connection
```
dbms.killConnection()
```
