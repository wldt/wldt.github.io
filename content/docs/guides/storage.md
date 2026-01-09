---
title: "Storage Layer"
description: ""
summary: ""
date: 2024-08-29T17:46:07+02:00
lastmod: 2024-08-29T17:46:07+02:00
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "storage-51ce7ba4d9befe482fbddf0a9757d6b2"
weight: 901
toc: true
---

The storage layer has been integrated into the core WLDT library, enabling Digital Twins to manually and automatically store data related to the evolution of their state, generated events (as illustrated in [DT Events Page](/docs/introduction/dt-events)), and any variations involving properties, events, actions, relationships, and life cycle. 

The WLDT Storage Layer consists of two main components:

- **Storage Manager**: This is the central component of the storage system, facilitating the structured and modular storage and retrieval of information. It allows developers to create and utilize various storage systems (e.g., in-memory, file-based, or DBMS) simultaneously. The Storage Layer is accessible in both read and write modes internally by the DT's Model, and in read-only mode via the Query System by Digital Adapters.
- **Query System**: To delegate and encapsulate the responsibility of data storage within the DT's model, a query system has been integrated. This system enables Digital Adapters to retrieve stored data and expose it according to their specific logic and implementation.

The storage layer is designed for easy extension, allowing developers to create and share new storage layers (e.g., using Redis, MySQL, or MongoDB). The provided in-memory implementation serves _only_ for basic development and testing purposes. Similarly, the Query Manager can be extended and customized by developers to implement additional query management features or to enhance the default functionalities provided by the library.

### Storage Manager

The main module of the Storage Layer is the one associated to Storage Capabilities and it is composed by two main classes: `StorageManager` and `WldStorage` with the following characteristics and main methods:

- `StorageManager`: The StorageManager class is a class that represents the storage manager for a DigitalTwin.  It is responsible for managing the storage of the data related to the DigitalTwin. It is an observer of the `WldtEventBus`, and it is able to save the data in the available storages. The class extends a `DigitalTwinWorker`, in order to allow the component to work in a structure and integrated way on a different thread that the core of a DT can coordinate starting and stopping it when required. The manager allow the usage of different storage systems at the same time in order to allow the developers to memorize the information accordingly to their need in the right storage system at the same time (e.g., REDIS for quick cached information and MongDB for historical data). Main associated methods are: 
	- `putStorage(WldtStorage storage)`: Add a new WldtStorage to the StorageManager
	- `getStorageIdList()`: Returns the list of id of the WldtStorage in the StorageManager
	- `isStorageAvailable(String storageId)`: Checks if a target Storage Id is available in the Storage Manager
	- `getStorage(String storageId)`: Get the target WldtStorage by id from the Storage Manager
	- `removeStorage(String storageId)`: Remove an existing WldtStorage by id from the StorageManager
- `WldtStorage`: Defines an abstract class allowing the Digital Twin developer to implement its internal storage system for the Digital Twin instance.  
	- The class defines methods for the management of:  
		- Digital Twin State storage and retrieval with the associated change list;  
		* Generated State Digital Events;  
		* Life Cycle State storage and retrieval;  
		* Physical Asset Description storage and retrieval;  
		* Physical Asset Property Variation storage and retrieval;  
		* Physical Asset Relationship Instance storage and retrieval;  
		* Digital Action Request storage and retrieval;  
		* Physical Asset Action Request storage and retrieval;  
		* Physical Asset Event Notification storage and retrieval;
	* Each WldtStorage instance can be configured (using the right constructor method) to: 
		* Observe all Wldt events (`stateEvents`,  `physicalAssetEvents`,  `physicalAssetActionEvents`,  `physicalAssetDescriptionEvents`,  `digitalActionEvents`,  `lifeCycleEvents`)
		* Filter only for specific class of events
		* Once the WldtStorage has been properly configured to receive target events the `StorageManager` automatically save information of interest for that specific storage. For example we can have a `StorageA` (e.g, REDIS) configured to receive all the generated events and a `StorageB` (e.g., MongoDB) in charge of saving only DT's state variation over time.
	* The default implementation of the `WldtStorage` is the class `DefaultWldtStorage`. This class provides a simple storage solution for digital twin states,  digital twin state changes, physical asset events, and digital twin events. The class provides **ONLY** a memory based  approach for storage using ArrayLists and HashMaps and more advanced solution should be implemented for production oriented Digital Twins for examples using external storage and memorization solutions.
	* Each Record written and returned by methods available through the `WldtStorage` implementations are extension of the `StorageRecord` used to represents a single record in the storage with a unique id
	* Methods available and implemented by WldtStorage implementations are the following grouped by categories:
		* **Digital Twin State**:
			* `saveDigitalTwinState(DigitalTwinState digitalTwinState, List<DigitalTwinStateChange> digitalTwinStateChangeList)`: Save a new computed instance of the DT State in the Storage together with the list of the changes with respect  to the previous state
			* `getLastDigitalTwinState()`: Returns the latest computed Digital Twin State of the target Digital Twin instance
			* `getDigitalTwinStateCount()`: Returns the number of computed and stored Digital Twin States
			* `getDigitalTwinStateInTimeRange(long startTimestampMs, long endTimestampMs)`: Retrieves a list of DigitalTwinState objects within the specified time range
			* `getDigitalTwinStateInRange(int startIndex, int endIndex)`: Retrieves a list of Digital Twin states within the specified range of indices
		* **Digital Twin State Event Notification**:
			* `saveDigitalTwinStateEventNotification(DigitalTwinStateEventNotification<?> digitalTwinStateEventNotification)`: Save the Digital Twin State Event Notification
			* `getDigitalTwinStateEventNotificationCount()`: Get the number of Digital Twin State Event Notification
			* `getDigitalTwinStateEventNotificationInTimeRange(long startTimestampMs, long endTimestampMs)`: Get the Digital Twin State Event Notification in the specified time range
			* `getDigitalTwinStateEventNotificationInRange(int startIndex, int endIndex)`: Get the Digital Twin State Event Notification in the specified range of indices
		* **Life Cycle State Variation**:
			* `saveLifeCycleState(LifeCycleStateVariation lifeCycleStateVariation)`: Save the LifeCycleState of the Digital Twin
			* `getLastLifeCycleState()`:  Get the last LifeCycleState of the Digital Twin
			* `getLifeCycleStateCount()`: Get the number of LifeCycleState of the Digital Twin
			* `getLifeCycleStateInTimeRange(long startTimestampMs, long endTimestampMs)`: Get the last LifeCycleState of the Digital Twin
			* `getLifeCycleStateInRange(int startIndex, int endIndex)`: Get the LifeCycleState of the Digital Twin in the specified range of indices
		* **Physical Asset Event Notification**:
			* `savePhysicalAssetEventNotification(PhysicalAssetEventNotification physicalAssetEventNotification)`: Save the Physical Asset Event Notification
			* `getPhysicalAssetEventNotificationCount()`: Get the number of Physical Asset Event Notification
			* `getPhysicalAssetEventNotificationInTimeRange(long startTimestampMs, long endTimestampMs)`: Get the Physical Asset Event Notification in the specified time range
			* `getPhysicalAssetEventNotificationInRange(int startIndex, int endIndex)`: Get the Physical Asset Event Notification in the specified range of indices
		* **Physical Action Request**: 
			* `savePhysicalAssetActionRequest(PhysicalAssetActionRequest physicalAssetActionRequest)`: Save Physical Asset Action Request
			* `getPhysicalAssetActionRequestCount()`: Get the number of Physical Asset Action Request
			* `getPhysicalAssetActionRequestInTimeRange(long startTimestampMs, long endTimestampMs)`: Get the Physical Asset Action Request in the specified time range
			* `getPhysicalAssetActionRequestInRange(int startIndex, int endIndex)`: Get the Physical Asset Action Request in the specified range of indices
		* **Digital Action Request**:
			* `saveDigitalActionRequest(DigitalActionRequest digitalActionRequest)`: Save a Digital Action Request
			* `getDigitalActionRequestCount()`: Get the number of Digital Action Request Stored
			* `getDigitalActionRequestInTimeRange(long startTimestampMs, long endTimestampMs)`: Get the Digital Action Request in the specified time range
			* `getDigitalActionRequestInRange(int startIndex, int endIndex)`: Get the Digital Action Request in the specified range of indices
		* **Physical Asset Description (PAD) Notification**
			* New PAD Notification
				* `saveNewPhysicalAssetDescriptionNotification(PhysicalAssetDescriptionNotification physicalAssetDescriptionNotification)`: Save a new Physical Asset Description Available
				* `getNewPhysicalAssetDescriptionNotificationCount()`: Get the number of New Physical Asset Description Notifications available
				* `getNewPhysicalAssetDescriptionNotificationInTimeRange(long startTimestampMs, long endTimestampMs)`: Get the New Physical Asset Description Available in the specified time range
				* `getNewPhysicalAssetDescriptionNotificationInRange(int startIndex, int endIndex)`: Get the New Physical Asset Description Available in the specified range of indices
			* Updated PAD Notification
				* `saveUpdatedPhysicalAssetDescriptionNotification(PhysicalAssetDescriptionNotification physicalAssetDescriptionNotification)`: Save the updated Physical Asset Description Notification
				* `getUpdatedPhysicalAssetDescriptionNotificationCount()`: Get the number of Updated Physical Asset Description
				* `getUpdatedPhysicalAssetDescriptionNotificationInTimeRange(long startTimestampMs, long endTimestampMs)`: Get the Updated Physical Asset Description in the specified time range
				* `getUpdatedPhysicalAssetDescriptionNotificationInRange(int startIndex, int endIndex)`: Get the Updated Physical Asset Description in the specified range of indices
		* **Physical Asset Property Variation**: 
			* `savePhysicalAssetPropertyVariation(PhysicalAssetPropertyVariation physicalAssetPropertyVariation)`: Save the Physical Asset Property Variation
			* `getPhysicalAssetPropertyVariationCount()`: Get the number of Physical Asset Property Variation
			* `getPhysicalAssetPropertyVariationInTimeRange(long startTimestampMs, long endTimestampMs)`: Get the Physical Asset Property Variation in the specified time range
			* `getPhysicalAssetPropertyVariationInRange(int startIndex, int endIndex)`: Get the Physical Asset Property Variation in the specified range of indices
		* **Physical Asset Relationship Instance Notification**
			* Created Relationship Instance
				* `savePhysicalAssetRelationshipInstanceCreatedNotification(PhysicalRelationshipInstanceVariation physicalRelationshipInstanceVariation)`: Save the Physical Asset Relationship Instance Created Event
				* `getPhysicalAssetRelationshipInstanceCreatedNotificationCount()`: Get the number of Physical Asset Relationship Instance Created Event
				* `getPhysicalAssetRelationshipInstanceCreatedNotificationInTimeRange(long startTimestampMs, long endTimestampMs)`: Get the Physical Asset Relationship Instance Created Event in the specified time range
				* `getPhysicalAssetRelationshipInstanceCreatedNotificationInRange(int startIndex, int endIndex)`: Get the Physical Asset Relationship Instance Created Event in the specified range of indices
			* Deleted Relationship Instance
				* `savePhysicalAssetRelationshipInstanceDeletedNotification(PhysicalRelationshipInstanceVariation physicalRelationshipInstanceVariation)`: Save the Physical Asset Relationship Instance Updated Event
				* `getPhysicalAssetRelationshipInstanceDeletedNotificationCount()`: Get the number of Physical Asset Relationship Instance Updated Event
				* `getPhysicalAssetRelationshipInstanceDeletedNotificationInTimeRange(long startTimestampMs, long endTimestampMs)`: Get the Physical Asset Relationship Instance Updated Event in the specified time range
				* `getPhysicalAssetRelationshipInstanceDeletedNotificationInRange(int startIndex, int endIndex)`: Get the Physical Asset Relationship Instance Updated Event in the specified range of indices
		* **Storage Statistics**:
			* Retrieve and returns storage statistics in terms of the number of stored records for each type and the associated time range of the stored records (start and end timestamp in milliseconds). Storage Statistics are mapped and modeled using the classes `StorageStats` and `StorageStatsRecord`.

### Storage Manager Code

Some examples of usage for the Storage Layer are the following: 

Lets' create a new Digital Twin with a single Storage in charge of *automatically* observe and store all the event generated and going through the target DT instance

```java
// Create the Digital Twin Engine
DigitalTwinEngine digitalTwinEngine = new DigitalTwinEngine();  

// Create a new Digital Twin with a Demo Shadowing Function
DigitalTwin digitalTwin = new DigitalTwin(TEST_DIGITAL_TWIN_ID, new DemoShadowingFunction());

// Physical Adapter Configuration  
DemoPhysicalAdapter physicalAdapter = new DemoPhysicalAdapter(...);  
digitalTwin.addPhysicalAdapter(physicalAdapter);

// Digital Adapter Configuration
digitalAdapter = new DemoDigitalAdapter(...);  
digitalTwin.addDigitalAdapter(digitalAdapter);

// Create a new WldtStorage instance using the default implementation and observing all the events  
DefaultWldtStorage myStorage = new DefaultWldtStorage("test_storage", true);

// Add the new Default Storage Instance to the Digital Twin Storage Manager 
digitalTwin.getStorageManager().putStorage(myStorage);

// Add the Twin to the Engine  
digitalTwinEngine.addDigitalTwin(digitalTwin);  
  
// Start the Digital Twin  
digitalTwinEngine.startDigitalTwin(TEST_DIGITAL_TWIN_ID);
```

Now let's **suppose** to have two additional implementation of the WldtStorage class supporting Redis and MongDB and called `RedisWldtStorage` and `MongoDbWldtStorage`.
We would like to use Redis to *automatically* observe all the events and MongoDb only to store DT's state and life cycle variations. 

```java
[...]

// Create a new RedisWldtStorage instance using the default implementation and observing all the events  
RedisWldtStorage myRedisStorage = new RedisWldtStorage("redis_storage", true);
myRedisStorage.setRedisConfiguration(myRedisConfiguration);

// Add the new Redis Storage Instance to the Digital Twin Storage Manager 
digitalTwin.getStorageManager().putStorage(myRedisStorage);

// Create a new MongoDbWldtStorage instance using the default implementation and observing only State and LifeCycle Events
MongoDbWldtStorage myMongoDbStorage = new MongoDbWldtStorage("mongo_db_storage", true, false, false, false, false, true);
myMongoDbStorage.setMongoDbConfiguration(myMongoDbConfiguration);

// Add the new MongoDb Storage Instance to the Digital Twin Storage Manager 
digitalTwin.getStorageManager().putStorage(myRedisStorage);

[...]
```

Within the `ShadowingFunction` it is possible to have the reference to the `StorageManager` in order to access available Storage in both reading and writing mode.
This is an example of how to retrieve an available WldtStorage through its id and the use it to read Properties values in a time range of the last 5 minutes:

```java
String TARGET_STORAGE_ID = "test_storage";  
  
if(this.storageManager.isStorageAvailable(TARGET_STORAGE_ID)){  
  
    // Access the Storage Manager to store the last value of the property  
    WldtStorage targetStorage = this.storageManager.getStorage(TARGET_STORAGE_ID);  
  
    // Get the current time in milliseconds  
    long endTime = System.currentTimeMillis();  
  
    // Get the Time in the last 5 minutes  
    long startTime = endTime - (5 * 60 * 1000);  
  
    // Get the last Physical Asset Action Request in the last 5 minutes  
    List<PhysicalAssetPropertyVariationRecord> propertyVariationRecords = targetStorage.getPhysicalAssetPropertyVariationInTimeRange(startTime, endTime);  
        for(PhysicalAssetPropertyVariationRecord propertyVariationRecord : propertyVariationRecords){  
        logger.info("Property Variation Record: {}", propertyVariationRecord);  
        [...]  
    }  
}
```

**Note**: The `StorageManager`, as previously described, can automatically store DT-related events based on the configuration and setup of each `WldtStorage` instance added to the manager. However, since the `ShadowingFunction` has direct access to the `StorageManager` in both read and write modes, manual handling of data storage is also possible. To achieve this, you can disable automatic storage by setting it to `false` for specific event types or for all event types. This allows you to manually manage the storage of information within the `ShadowingFunction`.

### Custom Storage

As previously mentioned, the class `WldtStorage` is an abstract class allowing the developer to implement its internal storage system for the Digital Twin instance. The class defines a set of abstract methods that should be implemented by the developer to shape the management of reading and writing data from and to the target Storage and associated to the identified variations and changes: 

- Digital Twin State 
- Generated State Digital Events
- Life Cycle State 
- Physical Asset Description
- Physical Asset Property Variation 
- Physical Asset Relationship Instance 
- Digital Action Request 
- Physical Asset Action Request
- Physical Asset Event Notification
- Storage Statistics

According to the type of the managed resource the type of queries can be characterized in terms of: 

- Time Query: Returns available records in a time range made by a start time in ms and and end time in ms
- Index Query: Returns available records in a index range made a start and end index
- Last Value Query: Return the last available value for the target resource
- Count Query: Returns the number of element of that specific resource

All the stored and retrieve information and record as mapped into dedicated classes available in the package `storage.model` and associated to the different type of managed resources: `digital`, `lifecycle`, `physical`, `state`. Each record class extends the base class `StorageRecord`. 

A default Storage module denoted as `DefaultWldtStorage` is natively available in the library providing a simple in-memory storage solution but a developer can implement its own Storage module (e.g., to enable the support for MongoDb or REDIS).

In order to implement its own storage module the developer should extend the basic abstract class `WldtStorage` and implement all the supported method to handle data writing and reading. These new classes can extend also the constructor and the required information for example to handle and manager storage configuration (e.g., ip address and port of the storage system or the local folder where tha stored file should be written).

### Query System

Given the library's goal of maximizing modularity and decoupling responsibilities among the available components, the **Query System** has been introduced. This system allows components external to the core responsibilities of the Digital Twin (e.g., Digital Adapters and Augmentation Functions) to retrieve stored data and use or expose it according to their specific logic and implementation. For instance, an HTTP Digital Adapter could expose stored information about a DT's state variations over time, or a Monitoring Adapter could use available storage instances to retrieve events for a deeper understanding of the target DT instance's behavior. The query system has been implemented entirely through dedicated events in order to maximize the decoupling of the solution and and supports at the same time both **synchronous** and **asynchronous** queries.

The main classes associated to the Query System are the following: 

- `QueryManager`: This class represents the Query Manager responsible to handle the query request and manage the query execution and has been designed to be extended by the user to implement the desired query management logic (e.g., as with the `DefaultQueryManager`).
- `QueryRequest`: The class contains all the information needed to perform a query on the storage system
- `QueryRequestType`: This Enum represents the Query Request Type used to specify the type of query to be performed on the storage system supporting: 
	- TIME_RANGE
	- SAMPLE_RANGE
	- LAST_VALUE
	- COUNT
- `QueryResourceType`: This Enum represents the Query Resource Type used to specify the type of resource to be queried  on the storage system supporting the following resource types mapping those available and managed by the storage manager (and the supported and associated RequestType):
	- PHYSICAL_ASSET_PROPERTY_VARIATION
		- TIME_RANGE
		- SAMPLE_RANGE
		- COUNT
	- PHYSICAL_ASSET_EVENT_NOTIFICATION
		- TIME_RANGE
		- SAMPLE_RANGE
		- COUNT
	- PHYSICAL_ACTION_REQUEST
		- TIME_RANGE
		- SAMPLE_RANGE
		- COUNT
	- DIGITAL_ACTION_REQUEST
		- TIME_RANGE
		- SAMPLE_RANGE
		- COUNT
	- DIGITAL_TWIN_STATE
		- TIME_RANGE
		- SAMPLE_RANGE
		- COUNT
		- LAST_VALUE
	- NEW_PAD_NOTIFICATION
		- TIME_RANGE
		- SAMPLE_RANGE
		- COUNT
	- UPDATED_PAD_NOTIFICATION
		- TIME_RANGE
		- SAMPLE_RANGE
		- COUNT
	- PHYSICAL_RELATIONSHIP_INSTANCE_CREATED_NOTIFICATION
		- TIME_RANGE
		- SAMPLE_RANGE
		- COUNT
	- PHYSICAL_RELATIONSHIP_INSTANCE_DELETED_NOTIFICATION
		- TIME_RANGE
		- SAMPLE_RANGE
		- COUNT
	- LIFE_CYCLE_EVENT
		- TIME_RANGE
		- SAMPLE_RANGE
		- COUNT
		- LAST_VALUE
	- STORAGE_STATS
		- LAST_VALUE
- `QueryExecutor`: This class represents the Query Executor used to execute queries on the storage system supporting both synchronous and asynchronous query execution. Internally is implemented through an event-based mechanism to handle the query request and response
- `QueryResult`: This class represents the Query Result returned by the Query Executor containing the query results and the query status (successful or not) and error message (if any) together with also the original request
- `IQueryResultListener`: This interface represents the Query Result Listener used to receive the query results

### Query System Code 

An example of Synchronous query is: 

```java
QueryExecutor queryExecutor = new QueryExecutor(TEST_DIGITAL_TWIN_ID, "query-executor");  
  
// Create Query Request to the Storage Manager for the Last Digital Twin State  
QueryRequest queryRequest = new QueryRequest();  
queryRequest.setResourceType(QueryResourceType.DIGITAL_TWIN_STATE);  
queryRequest.setRequestType(QueryRequestType.LAST_VALUE);  
  
// Send the Query Request to the Storage Manager for the target DT  
QueryResult<?> queryResult = queryExecutor.syncQueryExecute(queryRequest);
```

Following the same approach an Asynchrounouse query can be executed as follows:

```java
QueryExecutor queryExecutor = new QueryExecutor(TEST_DIGITAL_TWIN_ID, "query-executor");  
  
// Create Query Request to the Storage Manager for the Last Digital Twin State  
QueryRequest queryRequest = new QueryRequest();  
queryRequest.setResourceType(QueryResourceType.DIGITAL_TWIN_STATE);  
queryRequest.setRequestType(QueryRequestType.LAST_VALUE);  
    
// Send the Query Request to the Storage Manager for the target DT  
queryExecutor.asyncQueryExecute(queryRequest, new IQueryResultListener() {  
    @Override  
    public void onQueryResult(QueryResult<?> queryResult) {  
        [...]  
    }  
});
```

The class `DigitalAdapter` has been updated adding also an internal reference to a `QueryExecutor` in order to simplify the interaction with the query system directly from an adapter like in the following example where we use the query Executor of the Digital Adapter `invokeAction` callback through its internal variable accessible through `this.queryExecutor` without creating a new executor: 

```java
public <T> void invokeAction(String actionKey, T body){  
    try {  
          
        // Create Query Request to the Storage Manager for the Last Digital Twin State  
        QueryRequest queryRequest = new QueryRequest();  
        queryRequest.setResourceType(QueryResourceType.DIGITAL_TWIN_STATE);  
        queryRequest.setRequestType(QueryRequestType.LAST_VALUE);  
  
        // Send the Query Request to the Storage Manager for the target DT  
        QueryResult<?> queryResult = this.queryExecutor.syncQueryExecute(queryRequest);  
        
        // Do Something with the Query Result  
        for(Object result : queryResult.getResults()){  
	        // Check the type of the Resulting class accordingly to the query
		    if(result instanceof DigitalTwinState)  
		        logger.info("LAST DT STATE: {}", result);  
		    else
			    logger.error("INVALID RESULT TYPE: {}", result.getClass().getName());  
		}
          
        logger.info("INVOKING ACTION: {} BODY: {}", actionKey, body);  
        publishDigitalActionWldtEvent(actionKey, body);  
    } catch (EventBusException e) {  
        e.printStackTrace();  
    }  
}
```

### Custom Query Manager

The class in charge of managing an incoming query is called `QueryManager` and it is characterized by two core methods: 

- `handleQuery(QueryRequest queryRequest, Map<String, WldtStorage> storageMap)`: Handle Query Request allowing its management through the storage map and the associated storage objects. Uses the method `getTargetStorage` to select the target storage to be used to handle the query.
- `getTargetStorage(QueryRequest queryRequest, Map<String, WldtStorage> storageMap)`: The method has been designed to return the desired storage object from the storage map to be used for the query management starting from the target QueryRequest and the StorageMap of the DT. In the default implementation, the method returns the first storage object available in the storage map.

The `QueryManager` class has a list of methods that structure the type of available queries and that return
an instance of `QueryResult` with an error and a message of *"Query not supported by the current implementation !"*. The list of these methods is the following: 

- `handleLifeCycleEventQuery(...)`: Handle Life Cycle Event Query Request
- `handlePhysicalRelationshipInstanceDeletedNotificationQuery(...)`: Handle Physical Relationship Instance Deleted Notification Query Request
- `handlePhysicalRelationshipInstanceCreatedNotificationQuery(...)`: Handle Physical Relationship Instance Created Notification Query Request
- `handleUpdatedPadNotification(...)`: Handle Updated Pad Notification Query Request
- `handleNewPadNotification(...)`: Handle New Pad Notification Query Request
- `handleDigitalActionRequestQuery(...)`: Handle Digital Action Request Query Request
- `handlePhysicalActionRequestQuery(...)`: Handle Physical Action Request Query Request
- `handlePhysicalAssetEventNotificationQuery(...)`: Handle Physical Asset Event Notification Query Request
- `handlePhysicalAssetPropertyVariationQuery(...)`: Handle Physical Asset Property Variation Query Request
- `handleStateQuery(...)`: Handle Digital Twin State Query Request
- `handleStorageStatsQuery(...)`: Handle Storage Stats Query

The library provides a default implementation that is ready to use and automatically activated in every DT instance and called `DefaultQueryManager`. This class extends the `QueryManager` class and implements the default behavior for the query management implementing each of the previous listed methods. In the default implementation the method `getTargetStorage` returns the first storage object available in the storage map. *The behavior can be changed by overriding the method in the custom query manager implementation. In the custom implementation, the method can be used to select different storage according to the query.*

In order to extend and customize the adopter QueryManager a developer can:

- Create a new Class extending the `QueryManager`
- Implement (if required) a custom `getTargetStorage` method to return the correct available storage according to the type of request (e.g., the State of the DT are stored on MongoDB while the variation of the Physical Asset on REDIS)
- If required the developer can implement o change the behavior of the query manager in terms of the query management methods

One a custom QueryManager has been defined it can be set and configured through the following method `setQueryManager` on the `StorageManager` of each Digital Twin:

```java
digitalTwin.getStorageManager().setQueryManager(myQueryManager);
```