# Neo4J-DB-Management
```
:use system
show databases
create database atindb
:use atindb
```

```
SHOW DATABASES
```


```
DROP DATABASE atindb
```

```
CALL db.schema.visualization()
```

## Set the password for the neo4j user
- Using Browser
```
:help server
:SERVER change-password
```

- Using Console
```
cypher-shell -u neo4j -p <password>

> SHOW USERS;
> ALTER USER atin_s SET PASSWORD 'abc123' CHANGE NOT REQUIRED SET STATUS ACTIVE;
```
