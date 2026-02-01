# Neo4j

## Neo4j

This is a living document that captures notes related to anything and all neo4j and cypher queries.

### List Databases

```
show databases 
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MQmj3sD2brWuBs4j-Md%2F-MQmspdShX2SGAvbKtl_%2Fimage.png?alt=media\&token=7fde7846-ea4c-41e3-b98a-c02ab8ba6f98)

### Create New Database

```graphql
create database spotless
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MQmj3sD2brWuBs4j-Md%2F-MQmsB1D_xqod7UuVon_%2Fimage.png?alt=media\&token=79af4dee-8bb9-4527-a042-e41810190563)

### Switch Database

```
:use spotless
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MQmj3sD2brWuBs4j-Md%2F-MQmt0ZFKXZ1tpaZITX6%2Fimage.png?alt=media\&token=008d5d84-9ceb-4ca4-acdc-7db5de374acc)

### Import Data from CSV and Define Relationships Between Nodes

#### Sample Data

Below is a sample CSV file with 3 columns, that represents Windows authentication information between different endpoints (think lateral movement detection/investigation/threat hunting):

| Column                | Meaning                                                                       |
| --------------------- | ----------------------------------------------------------------------------- |
| `SourceComputer`      | A computer that successfully authenticated to a DestinationComputer           |
| `DestinationComputer` | A computer that SourceComputer authenticated to                               |
| `DestinationUserName` | A user name that was used to logon from SourceComputer to DestinationComputer |

{% code title="lateral-movement.csv" %}
```scala
"SourceComputer","DestinationComputer","DestinationUserName"
"WS01","WS02","administrator"
"WS01","WS03","administrator"
"WS02","WS03","administrator"
"WS03","WS04","administrator"
"WS04","WS05","administrator"
"WS05","WS06","administrator"
"WS06","WS07","administrator"
"WS07","DB01","administrator"
"DB01","FS05","administrator"
"FS05","DC01","da-james"
"WS01","WS04","billy"
"WS02","WS04","sally"
"WS03","WS02","fred"
"WS03","WS02","james"
"WS01","WS02","james"
```
{% endcode %}

{% hint style="info" %}
The file needs to be saved to the `import` folder of your database folder. In my case, the path is C:\Users\User\AppData\Local\Neo4j\Relate\Data\dbmss\dbms-8320b8a8-e54d-4742-a432-c8014b5968ec\import\lateral-movement.csv
{% endhint %}

#### Importing Nodes from CSV and Creating Relationships

```graphql
LOAD CSV WITH HEADERS FROM 'file:///lateral-movement.csv' AS line
MERGE (a:Computer {Computer:line.SourceComputer} )
MERGE (b:Computer {Computer:line.DestinationComputer} )
MERGE (a) -[:LOGGED_IN {loggedAs:line.DestinationUserName}]-> (b)
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MQmj3sD2brWuBs4j-Md%2F-MQmvVrKo8StYL2LhX2o%2Fimage.png?alt=media\&token=35db1d70-8e1e-40c4-ba2d-3f54519e4217)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MQmj3sD2brWuBs4j-Md%2F-MQmvikI0Y4NhnZCTpp-%2Fimage.png?alt=media\&token=da546a8a-17fd-4478-bb56-41757c4e0b59)

### Clean Database

```graphql
match (a) -[r] -> () delete a, r; match (a) delete a
```

### Match Nodes WHERE DestinationComputer Contains "WS"

```graphql
MATCH p=()-[r:LOGGED_IN]->(m:Computer) where m.Computer CONTAINS "WS" RETURN p LIMIT 25
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MQmj3sD2brWuBs4j-Md%2F-MQmy5gdRPvkD0JpPngr%2Fimage.png?alt=media\&token=1cd08b7c-6435-4f49-be62-49ae946befaf)

### Match Nodes WHERE Relationship Contains "james"

```graphql
MATCH p=()-[r:LOGGED_IN]->() where (r.loggedAs contains "james") RETURN p LIMIT 25
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MQmj3sD2brWuBs4j-Md%2F-MQn1XROKOnqLyYsGjxZ%2Fimage.png?alt=media\&token=5ff78fc2-d0b8-4373-8482-beee71718ddb)

### Match Nodes with 3 Hops Between Them

```graphql
MATCH p=()-[r:LOGGED_IN*3]->() RETURN p LIMIT 25
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MQmj3sD2brWuBs4j-Md%2F-MQn1MApAsshGnr1AxED%2Fimage.png?alt=media\&token=8d2a0a25-29e9-497c-acfb-4d405bc6db19)
