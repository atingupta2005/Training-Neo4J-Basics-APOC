# Import Relational Data Into Neo4j
### About the Data Domain
 - We will be using the NorthWind dataset, an often-used SQL dataset.
  - <img align="left" src="https://dist.neo4j.com/wp-content/uploads/Northwind_diagram.jpg"><br /><br />

### We can narrow our model down to these essential entities.
  - <img align="left" src="https://dist.neo4j.com/wp-content/uploads/Northwind_diagram_focus.jpg"><br /><br />

## Developing a Graph Model
- When deriving a graph model from a relational model, you should keep a couple of general guidelines in mind.
  - A row is a node.
  - A table name is a label name.
  - A join or foreign key is a relationship.

- With these principles in mind, we can map our relational model to a graph with the following steps:
- Rows to Nodes, Table names to labels
  - Each row on our Orders table becomes a node in our graph with Order as the label.
  - Each row on our Products table becomes a node with Product as the label.
  - Each row on our Suppliers table becomes a node with Supplier as the label.
  - Each row on our Categories table becomes a node with Category as the label.
  - Each row on our Employees table becomes a node with Employee as the label.

- Joins to relationships
  - Join between Suppliers and Products becomes a relationship named SUPPLIES (where supplier supplies product).
  - Join between Products and Categories becomes a relationship named PART_OF (where product is part of a category).
  - Join between Employees and Orders becomes a relationship named SOLD (where employee sold an order).
  - Join between Employees and itself (unary relationship) becomes a relationship named REPORTS_TO (where employees have a manager).
  - Join with join table (Order Details) between Orders and Products becomes a relationship named CONTAINS with properties of unitPrice, quantity, and discount (where order contains a product).


### If we draw our translation out on the whiteboard, we have this graph data model.
<img align="left" src="https://dist.neo4j.com/wp-content/uploads/northwind_graph_simple.svg"><br />
<br /><br /><br /><br /><br /><br /><br /><br /><br />

### How does the Graph Model Differ from the Relational Model?
  - There are no nulls
  - It describes the relationships in more detail

### Exporting Relational Tables to CSV
  - Take the data from the relational tables and put it in another format for loading to the graph

## Importing the Data using Cypher

### Create a blank database:
```
:USE system
CREATE DATABASE atin_northwind
SHOW DATABASES
:USE atin_northwind
CALL db.schema.visualization()
```

### Creating Order nodes
```
// Create orders
LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/jexp/054bc6baf36604061bf407aa8cd08608/raw/8bdd36dfc88381995e6823ff3f419b5a0cb8ac4f/orders.csv' AS row
MERGE (order:Order {orderID: row.OrderID})
  ON CREATE SET order.shipName = row.ShipName;
```

- This code creates 830 Order nodes in the database.
- You can view some of the nodes in the database by executing this code
```
MATCH (o:Order) return o LIMIT 5;
```

- Notice that you have not imported all of the field columns in the CSV file
- With your statements, you can choose which properties are needed on a node, which can be left out
- Also notice that you used the MERGE keyword, instead of CREATE
- Use MERGE as good practice for ensuring unique entities in our database

### Creating Product nodes
- Execute this code to create the Product nodes in the database:
```
// Create products
LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/jexp/054bc6baf36604061bf407aa8cd08608/raw/8bdd36dfc88381995e6823ff3f419b5a0cb8ac4f/products.csv' AS row
MERGE (product:Product {productID: row.ProductID})
  ON CREATE SET product.productName = row.ProductName, product.unitPrice = toFloat(row.UnitPrice);
```

- This code creates 77 Product nodes in the database.
- You can view some of these nodes in the database by executing this code:
```
MATCH (p:Product) return p LIMIT 5;
```

### Creating Supplier nodes
- Execute this code to create the Supplier nodes in the database:
```
// Create suppliers
LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/jexp/054bc6baf36604061bf407aa8cd08608/raw/8bdd36dfc88381995e6823ff3f419b5a0cb8ac4f/suppliers.csv' AS row
MERGE (supplier:Supplier {supplierID: row.SupplierID})
  ON CREATE SET supplier.companyName = row.CompanyName;
```

- This code creates 29 Supplier nodes in the database.
- You can view some of these nodes in the database by executing this code:
```
MATCH (s:Supplier) return s LIMIT 5;
```

### Creating Employee nodes
- Execute this code to create the Supplier nodes in the database:
```
// Create employees
LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/jexp/054bc6baf36604061bf407aa8cd08608/raw/8bdd36dfc88381995e6823ff3f419b5a0cb8ac4f/employees.csv' AS row
MERGE (e:Employee {employeeID:row.EmployeeID})
  ON CREATE SET e.firstName = row.FirstName, e.lastName = row.LastName, e.title = row.Title;
```

- This code creates 9 Employee nodes in the database.
- You can view some of these nodes in the database by executing this code:
```
MATCH (e:Employee) return e LIMIT 5;
```

### Creating Category nodes
```
// Create categories
LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/jexp/054bc6baf36604061bf407aa8cd08608/raw/8bdd36dfc88381995e6823ff3f419b5a0cb8ac4f/categories.csv' AS row
MERGE (c:Category {categoryID: row.CategoryID})
  ON CREATE SET c.categoryName = row.CategoryName, c.description = row.Description;
```

- This code creates 8 Category nodes in the database.
- You can view some of these nodes in the database by executing this code:
```
MATCH (c:Category) return c LIMIT 5;
```

### Creating the indexes and constraints for the data in the graph
- After the nodes are created, you need to create the relationships between them
- Importing the relationships will mean looking up the nodes you just created and adding a relationship between those existing entities
- To ensure the lookup of nodes is optimized, you will create indexes for any node properties used in the lookups
- We also want to create a constraint (also creates an index with it) that will disallow orders with the same id from getting created, preventing duplicates
- Finally, as the indexes are created after the nodes are inserted, their population happens asynchronously, so we call db.awaitIndexes() to block until they are populated

- Execute this code block:
```
CREATE INDEX product_id FOR (p:Product) ON (p.productID);
CREATE INDEX product_name FOR (p:Product) ON (p.productName);
CREATE INDEX supplier_id FOR (s:Supplier) ON (s.supplierID);
CREATE INDEX employee_id FOR (e:Employee) ON (e.employeeID);
CREATE INDEX category_id FOR (c:Category) ON (c.categoryID);
CREATE CONSTRAINT order_id ON (o:Order) ASSERT o.orderID IS UNIQUE;
CALL db.awaitIndexes();
```

- After you execute this code, you can execute this code to view the indexes (and constraint) in the database:
```
CALL db.indexes();
```


### Creating the relationships between the nodes
- Next you will create relationships:
  - Between orders and employees.
  - Between products and suppliers and between products and categories.
  - Between employees.

#### Creating relationships between orders and employees
- With the initial nodes and indexes in place, you can now create the relationships for orders to products and orders to employees.
- Execute this code block:
```
// Create relationships between orders and products
LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/jexp/054bc6baf36604061bf407aa8cd08608/raw/8bdd36dfc88381995e6823ff3f419b5a0cb8ac4f/orders.csv' AS row
MATCH (order:Order {orderID: row.OrderID})
MATCH (product:Product {productID: row.ProductID})
MERGE (order)-[op:CONTAINS]->(product)
  ON CREATE SET op.unitPrice = toFloat(row.UnitPrice), op.quantity = toFloat(row.Quantity);
```

- This code creates 2155 relationships in the graph.
- You can view some of them by executing this code:
```
MATCH (o:Order)-[]-(p:Product)
RETURN o,p LIMIT 10;
```


- Then, execute this code block:
```
// Create relationships between orders and employees
LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/jexp/054bc6baf36604061bf407aa8cd08608/raw/8bdd36dfc88381995e6823ff3f419b5a0cb8ac4f/orders.csv' AS row
MATCH (order:Order {orderID: row.OrderID})
MATCH (employee:Employee {employeeID: row.EmployeeID})
MERGE (employee)-[:SOLD]->(order);
```

- This code creates 830 relationships in the graph.
- You can view some of them by executing this code:
```
MATCH (o:Order)-[]-(e:Employee)
RETURN o,e LIMIT 10;
```

#### Creating relationships between products and suppliers and between products and categories
- Next, create relationships between products, suppliers, and categories:
- Execute this code block:
```
// Create relationships between products and suppliers
LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/jexp/054bc6baf36604061bf407aa8cd08608/raw/8bdd36dfc88381995e6823ff3f419b5a0cb8ac4f/products.csv
' AS row
MATCH (product:Product {productID: row.ProductID})
MATCH (supplier:Supplier {supplierID: row.SupplierID})
MERGE (supplier)-[:SUPPLIES]->(product);
```
- This code creates 77 relationships in the graph.
- You can view some of them by executing this code:

```
MATCH (s:Supplier)-[]-(p:Product)
RETURN s,p LIMIT 10;
```

- Then, execute this code block:
```
// Create relationships between products and categories
LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/jexp/054bc6baf36604061bf407aa8cd08608/raw/8bdd36dfc88381995e6823ff3f419b5a0cb8ac4f/products.csv
' AS row
MATCH (product:Product {productID: row.ProductID})
MATCH (category:Category {categoryID: row.CategoryID})
MERGE (product)-[:PART_OF]->(category);
```

- This code creates 77 relationships in the graph.
- You can view some of them by executing this code:
```
MATCH (c:Category)-[]-(p:Product)
RETURN c,p LIMIT 10;
```

#### Creating relationships between employees
- Lastly, you will create the 'REPORTS_TO' relationship between employees to represent the reporting structure:
- Execute this code block:
```
// Create relationships between employees (reporting hierarchy)
LOAD CSV WITH HEADERS FROM 'https://gist.githubusercontent.com/jexp/054bc6baf36604061bf407aa8cd08608/raw/8bdd36dfc88381995e6823ff3f419b5a0cb8ac4f/employees.csv' AS row
MATCH (employee:Employee {employeeID: row.EmployeeID})
MATCH (manager:Employee {employeeID: row.ReportsTo})
MERGE (employee)-[:REPORTS_TO]->(manager);
```

- This code creates 8 relationships in the graph.
- You can view some of them by executing this code:
```
MATCH (e1:Employee)-[]-(e2:Employee)
RETURN e1,e2 LIMIT 10;
```

- Next, you will query the resulting graph to find out what it can tell us about our newly-imported data.

## Querying the Graph
```
//find a sample of employees who sold orders with their ordered products
MATCH (e:Employee)-[rel:SOLD]->(o:Order)-[rel2:CONTAINS]->(p:Product)
RETURN e, rel, o, rel2, p LIMIT 25;
```

```
//find the supplier and category for a specific product
MATCH (s:Supplier)-[r1:SUPPLIES]->(p:Product {productName: 'Chocolade'})-[r2:PART_OF]->(c:Category)
RETURN s, r1, p, r2, c;
```

- How are Employees Organized? Who Reports to Whom?
  - Execute this code block:
```
MATCH (e:Employee)<-[:REPORTS_TO]-(sub)
RETURN e.employeeID AS manager, sub.employeeID AS employee;
```

## Review the schema
```
CALL db.schema.visualization()
```
