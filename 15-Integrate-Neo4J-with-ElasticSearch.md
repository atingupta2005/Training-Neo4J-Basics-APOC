# Integrate Neo4J with ElasticSearch
## Install Elastic Search on Ubuntu
```
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install elasticsearch
```

## Allow all hosts to connect
```
sudo nano /etc/elasticsearch/elasticsearch.yml
```

- Add below lines in last
```
network.host: 127.0.0.1
http.host: 0.0.0.0
```

## Restart
```
sudo systemctl enable elasticsearch
sudo systemctl restart elasticsearch
```

## Test
```
curl -X GET 'http://localhost:9200'
```


## Sample Commands to integrate Neo4J with APOC
```
CALL apoc.es.stats("vmneo4j.eastus2.cloudapp.azure.com")
call apoc.es.post("vmneo4j.eastus2.cloudapp.azure.com","tweets","users",null,{name:"Chris"})
call apoc.es.put("vmneo4j.eastus2.cloudapp.azure.com","tweets","users","1",null,{name:"Chris"})
call apoc.es.get("vmneo4j.eastus2.cloudapp.azure.com","tweets","users","1",null,null)
call apoc.es.stats("vmneo4j.eastus2.cloudapp.azure.com")
```

## Install Kibana
- Installation
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install kibana
```

- Run Kibana
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable kibana.service
```

- Kibana can be started and stopped as follows:
```
sudo systemctl start kibana.service
sudo systemctl stop kibana.service
```


- Configure Kibana via the config file
```
sudo nano /etc/kibana/kibana.yml
```

- Add below line in last
```
server.host: "0.0.0.0"
```

- Access Kibana
  - localhost:5601


## Acces Kibana from Neo4J
```
call apoc.es.post("localhost","tweets","users",null,{name:"Chris"})
call apoc.es.put("localhost","tweets","users","1",null,{name:"Chris"})
call apoc.es.get("localhost","tweets","users","1",null,null)
call apoc.es.stats("localhost")
```

```
// It's important to create an index to improve performance
CREATE INDEX ON :Document(id)
// First query: get first chunk of data + the scroll_id for pagination
CALL apoc.es.query('localhost','test-index','test-type','name:Neo4j&size=1&scroll=5m',null) yield value with value._scroll_id as scrollId, value.hits.hits as hits
// Do something with hits
UNWIND hits as hit
// Here we simply create a document and a relation to a company
MERGE (doc:Document {id: hit._id, description: hit._source.description, name: hit._source.name})
MERGE (company:Company {name: hit._source.company})
MERGE (doc)-[:IS_FROM]->(company)
// Then call for the other docs and use the scrollId value from previous query
// Use a range to count our chunk of data (i.e. i want to get chunks from 2 to 10)
WITH range(2,10,1) as list, scrollId
UNWIND list as count
CALL apoc.es.get("localhost","_search","scroll",null,{scroll:"5m",scroll_id:scrollId},null) yield value with value._scoll_id as scrollId, value.hits.hits as nextHits
// Again, do something with hits
UNWIND nextHits as hit
MERGE (doc:Document {id: hit._id, description: hit._source.description, name: hit._source.name})
MERGE (company:Company {name: hit._source.company})
MERGE (doc)-[:IS_FROM]->(company) return scrollId, doc, company
```
