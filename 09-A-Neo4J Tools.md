# Neo4J Tools

## Import Data
```
neo4j-admin import --nodes import/movies_header.csv,import/movies.csv \
--nodes import/actors_header.csv,import/actors.csv \
--relationships import/roles_header.csv,import/roles.csv
```

```
## cypher-shell
neo4j-home> bin/cypher-shell -u neo4j -p <password>
```

## cypher-shell commands
```
:help
MATCH (n) RETURN n;
```
