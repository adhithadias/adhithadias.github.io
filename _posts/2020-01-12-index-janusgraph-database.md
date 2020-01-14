---
title:  "Index JanusGraph Database"
search: true
categories: 
  - Jekyll
last_modified_at: 2020-01-13T08:06:00-05:00
toc: true
---

### Lifecycle

#### States (SchemaStatus)

States | Description
------------ | -------------
**INSTALLED** | The index is installed in the system but not yet registered with all instances in the cluster
**REGISTERED** | The index is registered with all instances in the cluster but not (yet) enabled
**ENABLED** | The index is enabled and in use
**DISABLED** | The index is disabled and no longer in use

#### Actions (SchemaAction)

SchemaAction | Description
------------ | -------------
**REGISTER_INDEX** | Registers the index with all instances in the graph cluster. After an index is installed, it must be registered with all graph instances
**REINDEX** | Re-builds the index from the graph
**ENABLE_INDEX** | Enables the index so that it can be used by the query processing engine. An index must be registered before it can be enabled.
**DISABLE_INDEX** | Disables the index in the graph so that it is no longer used.
**REMOVE_INDEX** | Removes the index from the graph (optional operation). Only on composite index.

### Scripts to index the graph database

First you have to kill the gremlin server and change some parameters in the server script
```bash
ps aux | grep gremlin # finds the gramlin process id
sudo kill -9 8532

vi conf/gremlin-server/gremlin-server.yaml
# then change scriptEvaluationTimeout -> scriptEvaluationTimeout: 300000 
# save and exit

# run the gramlin server again
cd janusgraph-0.4.0-hadoop2 # change directory to JanusGraph download location
sudo bin/gremlin-server.sh conf/gremlin-server/gremlin-server.yaml &
# make sure to run the command with & sign so that you can exit from the the command without killing it

```

Login to the gremlin console after restarting the server
```bash
sudo ./bin/gremlin.sh

# run inside the console to connect with the remote gremlin server
:remote connect tinkerpop.server conf/remote.yaml session
:remote console
```

Let's assume the database is empty. Now we have to first create a node for indexing.
Let's create an account node

```groovy
g.addV('Account').property(single, 'id', '11111111').next()
g.tx().commit()
```

Check for open transactions and close them
```groovy
graph.getOpenTransactions()
// to remove any open transactions <- remove all open transactions by repeating the command
graph.getOpenTransactions().getAt(0).rollback()
```

Check for any open sessions and close them
```groovy
mgmt = graph.openManagement()
mgmt.getOpenInstances()
// close all instances except the current instance
mgmt.forceCloseInstance('<_id_>')
mgmt.commit()
```
Printing schemas -- this helps to view the internal organization of the database
```groovy
mgmt = graph.openManagement()
mgmt.printSchema()
mgmt.commit()
```

Now let's register the index with the graph. This commands adds an index with both the vertex label and a key property
```groovy
graph.tx().rollback()  //Never create new indexes while a transaction is active
mgmt = graph.openManagement()
accountId = mgmt.getPropertyKey('id')
account = mgmt.getVertexLabel('Account')
mgmt.buildIndex('byAccountIdAndLabel', Vertex.class).addKey(accountId).indexOnly(account).buildCompositeIndex()
mgmt.commit()
```

Now if you look at the schemas, you will see a Vertex Index Name by 'byAccountIdAndLabel' in the REGISTERED state
```groovy
//Wait for the index to become available
ManagementSystem.awaitGraphIndexStatus(graph, 'byAccountIdAndLabel').call()
```

As the final step reindex the data in the graph
```groovy
//Reindex the existing data
mgmt = graph.openManagement()
mgmt.updateIndex(mgmt.getGraphIndex("byAccountIdAndLabel"), SchemaAction.REINDEX).get()
mgmt.commit()
```

Now if you do a print schema command, you will see that the 'byAccountIdAndLabel' index status is turned to ENABLED

### Troubleshooting

* [Notes on stuck indexes](https://stackoverflow.com/questions/40585417/titan-db-ignoring-index/40591478#40591478)
* [Script to describe indexes](https://gist.github.com/robertdale/ad4c63910009dd1118abe67b33ce41e1)

If you are unsuccessful in executing this procedure due to some interruption and the index gets 
stuck at the INSTALLED state there is a workaround to ENABLE the index


```groovy
//Step1: Clear all transactions
graph.getOpenTransactions()
graph.getOpenTransactions().getAt(0).rollback()

//Step2: Clear all management instances
mgmt=graph.openManagement()
mgmt.getOpenInstances() 

//After doing step 1 and 2
//Step3: Force change from Installed to Registered
mgmt = graph.openManagement()
mgmt.updateIndex(mgmt.getGraphIndex("byAccountIdAndLabel"), SchemaAction.REGISTER_INDEX).get()
mgmt.commit()

//Wait for the index to become available
ManagementSystem.awaitGraphIndexStatus(graph, 'byAccountIdAndLabel').call()

//Reindex the existing data
mgmt = graph.openManagement()
mgmt.updateIndex(mgmt.getGraphIndex("byAccountIdAndLabel"), SchemaAction.REINDEX).get()
mgmt.commit()
```


### References

* https://docs.janusgraph.org/index-management/index-performance/
* https://groups.google.com/forum/#!topic/janusgraph-users/E0speKxetgM