---
title: "Change Log 0.3.0"
description: ""
summary: ""
date: 2024-02-09T12:23:33+01:00
lastmod: 2024-02-09T12:23:33+01:00
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "change-log-0.3.0-7a14234a44ea9a2cb648e66c783d1e2e"
weight: 901
toc: false
---

## Digital Adapters

- The following methods have been discontinued and removed from the `DigitalAdapter` class:
	- `onStateChangePropertyCreated`
	- `onStateChangePropertyUpdated`
	- `onStateChangePropertyDeleted`
	- `onStatePropertyUpdated`
	- `onStatePropertyDeleted`
	- `onStateChangeActionEnabled`
	- `onStateChangeActionUpdated`
	- `onStateChangeActionDisabled`
	- `onStateChangeEventRegistered`
	- `onStateChangeEventRegistrationUpdated`
	- `onStateChangeEventUnregistered`
	- `onStateChangeRelationshipInstanceDeleted`
	- `onStateChangeRelationshipDeleted`
	- `onStateChangeRelationshipInstanceCreated`
	- `onStateChangeRelationshipCreated`
	- `onDigitalTwinStateEventNotificationReceived`
- The Signature of the following methods have been changed:
	- `onDigitalTwinSync(IDigitalTwinState currentDigitalTwinState)` -> `onDigitalTwinSync(DigitalTwinState currentDigitalTwinState)`
	- `onDigitalTwinUnSync(IDigitalTwinState currentDigitalTwinState)` ->  `onDigitalTwinUnSync(DigitalTwinState currentDigitalTwinState)`
- New methods that have been added are: 
	- `onStateUpdate(DigitalTwinState newDigitalTwinState, DigitalTwinState previousDigitalTwinState, ArrayList<DigitalTwinStateChange> digitalTwinStateChangeList)`
	- `onEventNotificationReceived(DigitalTwinStateEventNotification<?> digitalTwinStateEventNotification)`
- For additional details about Digital Adapters check Sub Section [[Change Log - v.0.3.0#Digital Adapter| Digital Adapters]] 

## Shadowing Function 

- `ShadowingModelFunction` is now  `ShadowingFunction`
- `this.digitalTwinState` is not directly accessible anymore and it is wrapped through the `DigitalTwinStateManager` using the variable `digitalTwinStateManager` (see next descriptions and changes)
- The method `addRelationshipInstance` now take only one parameter that is the `DigitalTwinStateRelationshipInstance`
- The same change for example should be applied in the point of the code where the Shadowing Function receive a variation from the Physical world through a target adapter and the callback method `onPhysicalAssetPropertyVariation(...)`
- When the Shadowing Function has to compute the new DT State it can now work with the following method to handle DT State Transition:
	- `this.digitalTwinStateManager.startStateTransaction()` 
	- DT State variation methods such as:
		- `digitalTwinStateManager.createProperty()`
		- `digitalTwinStateManager.updateProperty()`  
		- `digitalTwinStateManager.updatePropertyValue()`
		- `digitalTwinStateManager.deleteProperty()`
		- `digitalTwinStateManager.enableAction()`
		- `digitalTwinStateManager.updateAction()`
		- `digitalTwinStateManager.disableAction()`
		- `digitalTwinStateManager.registerEvent()`
		- `digitalTwinStateManager.updateRegisteredEvent()`
		- `digitalTwinStateManager.unRegisterEvent()`
		- `digitalTwinStateManager.createRelationship()`
		- `digitalTwinStateManager.addRelationshipInstance()`
		- `digitalTwinStateManager.deleteRelationship()`
		- `digitalTwinStateManager.deleteRelationshipInstance()`
	- At the end the transaction can be committed using the method: `digitalTwinStateManager.commitStateTransaction()`  
- - The method `notifyDigitalTwinStateEvent` is now available through the `digitalTwinStateManager`
- Additional Details associated to Shadowing Function Migration can be found in the dedicated section [[Change Log - v.0.3.0#Shadowing Function Changes | Shadowing Function Changes]]

## WLDT Engine & DT Creation

- `WldtEngine` is now `DigitalTwin` and model and structure a single Digital Twin and takes the following parameters:
	- `String digitalTwinId`
	- `ShadowingFunction shadowingFunction`
- The `startLifeCycle` has been removed from the DigitalTwin (previously WLDT Engine) and now `DigitalTwinEngine` should be used to start twins
- Once a new Digital Twin has been create it has to be added to the `DigitalTwinEngine`
- `DigitalTwinEngine` has dedicated method to start and stop twins such as:
	- `startAll()`
	- `startDigitalTwin(<DIGITAL_TWIN_ID>);`
	- `stopAll()`
	- `digitalTwinEngine.stopDigitalTwin(<DIGITAL_TWIN_ID>);`

# Digital Twin & Digital Twin Engine

With the following code we now create a new Digital Twin Instance

```java
// Create the new Digital Twin with its Shadowing Function  
DigitalTwin digitalTwin = new DigitalTwin(digitalTwinId, new DemoShadowingFunction());  
  
// Physical Adapter with Configuration  
digitalTwin.addPhysicalAdapter(  
        new DemoPhysicalAdapter(  
                String.format("%s-%s", digitalTwinId, "test-physical-adapter"),  
                new DemoPhysicalAdapterConfiguration(),  
                true));  

// Digital Adapter with Configuration  
digitalTwin.addDigitalAdapter(  
        new DemoDigitalAdapter(  
                String.format("%s-%s", digitalTwinId, "test-digital-adapter"),  
                new DemoDigitalAdapterConfiguration())  
);
```

In the new version the DT cannot be directly run but it should be added to the `DigitalTwinEngine` in order to be executed through the WLDT Library 

```java
// Create the Digital Twin Engine
DigitalTwinEngine digitalTwinEngine = new DigitalTwinEngine();  

// Add the Digital Twin to the Engine
digitalTwinEngine.addDigitalTwin(digitalTwin);
```

In order to start a DT from the Engine you can: 

```java
// Directly start when you add it passing a second boolean value = true
digitalTwinEngine.addDigitalTwin(digitalTwin. true);

// Starting the single DT on the engine through its id
digitalTwinEngine.startDigitalTwin(DIGITAL_TWIN_ID);

// Start all the DTs registered on the engine
digitalTwinEngine.startAll();
```

To stop a single twin or all the twin registered on the engine:

```java
// Stop a single DT on the engine through its id
digitalTwinEngine.stopDigitalTwin(DIGITAL_TWIN_ID);

// Stop all the DTs registered on the engine
digitalTwinEngine.stopAll();
```

It is also possible to remove a DT from the Engine with a consequent stop if it is active and the deletion of its reference from the engine:

```java
// Remove a single DT on the engine through its id
digitalTwinEngine.removeDigitalTwin(DIGITAL_TWIN_ID);

// Remove all the DTs registered on the engine
digitalTwinEngine.removeAll();
```

# Digital Twin State

- DT State now has the reference timestamp representing the evaluation instant of the digital twin state, this timestamp is computed through the `DigitalTwinStateManager` and cannot manually set by the developer
- The information available on the DT State are:   
	- `properties`: List of Properties with their values (if available)  
	- `actions`: List of Actions that can be called on the DT  
	- `events`: List of Events that can be generated by the DT  
	- `relationships`: List of Relationships and their instances (if available)  
	- `evaluationInstant`: The timestamp representing the evaluation instant of the DT state  
- Available main methods on that class instance are:   
	- Properties:   
		  - ```getProperty(String propertyKey)```: Retrieves if present the target DigitalTwinStateProperty by Key  
		  - ```containsProperty(String propertyKey)```: Checks if a target Property Key is already available in the current Digital Twin's State  
		  - ```getPropertyList()```: Loads the list of available Properties (described by the class DigitalTwinStateProperty) available on the Digital Twin's State  
		  - ```createProperty(DigitalTwinStateProperty<?> dtStateProperty)```: Allows the creation of a new Property on the Digital Twin's State through the class DigitalTwinStateProperty  
		  - ```readProperty(String propertyKey)```: Retrieves if present the target DigitalTwinStateProperty by Key  
		  - ```updateProperty(DigitalTwinStateProperty<?> dtStateProperty)```: Updates the target property using the DigitalTwinStateProperty and the associated Property Key field  
		  - ```deleteProperty(String propertyKey)```: Deletes the target property identified by the specified key  
	- Actions:   
		  - ```containsAction(String actionKey)```: Checks if a Digital Twin State Action with the specified key is correctly registered  
		  - ```getAction(String actionKey)```: Loads the target DigitalTwinStateAction by key  
		  - ```getActionList()```: Gets the list of available Actions registered on the Digital Twin's State  
		  - ```enableAction(DigitalTwinStateAction digitalTwinStateAction)```: Enables and registers the target Action described through an instance of the DigitalTwinStateAction class  
		  - ```updateAction(DigitalTwinStateAction digitalTwinStateAction)```: Update the already registered target Action described through an instance of the DigitalTwinStateAction class  
		  - ```disableAction(String actionKey)```: Disables and unregisters the target Action described through an instance of the DigitalTwinStateAction class  
	- Events:  
		  - ```containsEvent(String eventKey)```: Check if a Digital Twin State Event with the specified key is correctly registered  
		  - ```getEvent(String eventKey)```: Return the description of a registered Digital Twin State Event according to its Key  
		  - ```getEventList()```: Return the list of existing and registered Digital Twin State Events  
		  - ```registerEvent(DigitalTwinStateEvent digitalTwinStateEvent)```: Register a new Digital Twin State Event  
		  - ```updateRegisteredEvent(DigitalTwinStateEvent digitalTwinStateEvent)```: Update the registration and signature of an existing Digital Twin State Event  
		  - ```unRegisterEvent(String eventKey)```: Un-register a Digital Twin State Event  
		  - ```notifyDigitalTwinStateEvent(DigitalTwinStateEventNotification<?> digitalTwinStateEventNotification)```: Method to notify the occurrence of the target Digital Twin State Event  
	- Relationships:  
		  - ```containsRelationship(String relationshipName)```: Checks if a Relationship Name is already available in the current Digital Twin's State  
		  - ```createRelationship(DigitalTwinStateRelationship<?> relationship)```: Creates a new Relationships (described by the class DigitalTwinStateRelationship) in the Digital Twin's State  
		  - ```addRelationshipInstance(String name, DigitalTwinStateRelationshipInstance<?> instance)```: Adds a new Relationship instance described through the class DigitalTwinStateRelationshipInstance and identified through its name  
		  - ```getRelationshipList()```: Loads the list of existing relationships on the Digital Twin's State through a list of DigitalTwinStateRelationship  
		  - ```getRelationship(String name)```: Gets a target Relationship identified through its name and described through the class DigitalTwinStateRelationship  
		  - ```deleteRelationship(String name)```: Deletes a target Relationship identified through its name  
		  - ```deleteRelationshipInstance(String relationshipName, String instanceKey)```: Deletes the target Relationship Instance using relationship name and instance Key


# Digital Twin State Manager

The `DigitalTwinStateManager` is a Java class that serves as the default implementation of the `IDigitalTwinStateManager` interface within the White Label Digital Twin Java Framework (whitelabel-digitaltwin). This class allows developers to manage the state of a digital twin, including properties, actions, events, and relationships.
## Features

- **State Management:** Handles the creation, update, and deletion of properties, actions, events, and relationships associated with the digital twin state.
- **Transaction Support:** Allows developers to start, commit, or rollback transactions to manage changes to the digital twin state.
- **Event Notification:** Notifies listeners about updates to the digital twin state through the WLDT event bus.

When the Shadowing Function has to compute the new DT State it can now work with the following method to handle DT State Transition:
	- Start the DT State Transaction:  `startStateTransaction()` 
	- DT State variation methods such as:
		- `createProperty()`
		- `updateProperty()`  
		- `updatePropertyValue()`
		- `deleteProperty()`
		- `enableAction()`
		- `updateAction()`
		- `disableAction()`
		- `registerEvent()`
		- `updateRegisteredEvent()`
		- `unRegisterEvent()`
		- `createRelationship()`
		- `addRelationshipInstance()`
		- `deleteRelationship()`
		- `deleteRelationshipInstance()`

At the end the transaction can be committed using the method: `commitStateTransaction()`  
## Usage

To use the `DigitalTwinStateManager` within your digital twin implementation:

1. **Initialization:** Create an instance of the `DigitalTwinStateManager`.
    
```java    
DigitalTwinStateManager digitalTwinStateManager = new DigitalTwinStateManager();
```    
2. **State Transaction:**
    
    - Start a new transaction using `startStateTransaction()` to manage changes.
    - Make changes to the digital twin state.
    - Commit the transaction using `commitStateTransaction()` to apply the changes.
    
```java    
digitalTwinStateManager.startStateTransaction(); 
// Make changes to properties, actions, events, or relationships 
// [...]
digitalTwinStateManager.commitStateTransaction();
```
3. **Event Notification:**
    
    - DT State Updates after a commit action are automatically notified to Digital Adapter by the Manager
    - Once an event incoming from the physical or generated by the DT itself is handled by the Shadowing Function, the developer can use `notifyDigitalTwinStateUpdate` to notify Digital Adapter listening about events variations.
    
```java  
// Notify a specific event notification 
digitalTwinStateManager.notifyDigitalTwinStateEvent(digitalTwinStateEventNotification);
```
    
4. **Property, Action, Event, Relationship Management:**
    
    - Create, update, or delete properties, actions, events, or relationships as needed.
    
```java
// Begin Digital Twin State Transaction
digitalTwinStateManager.startStateTransaction(); 

// Create a new property 
digitalTwinStateManager.createProperty(dtStateProperty);  

// Update an existing property 
digitalTwinStateManager.updateProperty(dtStateProperty);  

// Delete a property 
digitalTwinStateManager.deleteProperty(propertyKey);`

// Commit DT State Update to apply all the changes and notify the Digital Adapters and other listeners about the variation
digitalTwinStateManager.commitStateTransaction();
```    

### Exception Handling

- The class throws `WldtDigitalTwinStateException` to indicate errors related to digital twin state management.
- Proper exception handling is advised to manage potential errors during state transactions.

## Shadowing Function Changes

Now that the DT State is managed through the `DigitalTwinStateManager` class all the changes and variation should be applied on the DT ShadowingFunction using the previously presented transaction management and the correct call of methods `startStateTransaction()` and `commitStateTransaction()`.

Here there is an example of the change with a simple and demo shadowing function on callback `onDigitalTwinBound`:

```java
@Override  
protected void onDigitalTwinBound(Map<String, PhysicalAssetDescription> adaptersPhysicalAssetDescriptionMap) {  
  
    try{    
  
		// NEW -> Start DT State Change Transaction  
		this.digitalTwinStateManager.startStateTransaction();  

		for(Map.Entry<String, PhysicalAssetDescription> entry : adaptersPhysicalAssetDescriptionMap.entrySet()){  

			String adapterId = entry.getKey();  
			PhysicalAssetDescription physicalAssetDescription = entry.getValue();  

			//In that simple case the Digital Twin shadow all the properties and actions available in the physical asset  
			for(PhysicalAssetProperty<?> p : physicalAssetDescription.getProperties())  
				this.digitalTwinStateManager.createProperty(new DigitalTwinStateProperty<>(p.getKey(), p.getInitialValue()));  

			for(PhysicalAssetAction a : physicalAssetDescription.getActions())  
				this.digitalTwinStateManager.enableAction(new DigitalTwinStateAction(a.getKey(),  
						a.getType(),  
						a.getContentType()));  

			for(PhysicalAssetEvent e: physicalAssetDescription.getEvents())  
				this.digitalTwinStateManager.registerEvent(new DigitalTwinStateEvent(e.getKey(), physicalAssetEvent.getType()));  
		}  

		// NEW -> Commit DT State Change Transaction to apply the changes on the DT State and notify about the change  
		this.digitalTwinStateManager.commitStateTransaction();      
  
        //Observer Target Physical Properties  
        for(Map.Entry<String, PhysicalAssetDescription> entry : adaptersPhysicalAssetDescriptionMap.entrySet()){  
		  [...]
        }  
  
        //Observe all the target available Physical Asset Events for each Adapter  
        for(Map.Entry<String, PhysicalAssetDescription> entry : adaptersPhysicalAssetDescriptionMap.entrySet()){  
		  [...]
        }  

		// Observer for Incoming Digital Actions
        observeDigitalActionEvents();  
	    
	    //Notify Shadowing Completed  
		notifyShadowingSync();
  
    }catch (Exception e){  
        e.printStackTrace();  
    }  
}
```

The same change for example should be applied in the point of the code where the Shadowing Function receive a variation from the Physical world through a target adapter and the callback method `onPhysicalAssetPropertyVariation(...)`

```java
@Override  
protected void onPhysicalAssetPropertyVariation(PhysicalAssetPropertyWldtEvent<?> physicalPropertyEventMessage) {  
  
    try {  
    
        if(physicalPropertyEventMessage != null && getPhysicalEventsFilter().contains(physicalPropertyEventMessage.getType())){  
   
            if(physicalPropertyEventMessage.getPhysicalPropertyId().equals(TestPhysicalAdapter.SWITCH_PROPERTY_KEY)  
                    && physicalPropertyEventMessage.getBody() instanceof String){  
			  [...]  
            }  
            else{  
    
                //Update Digital Twin State  
                //NEW -> Start State Transaction
                this.digitalTwinStateManager.startStateTransaction();  
                
                // Update State Property Value
                this.digitalTwinStateManager.updateProperty(  
                        new DigitalTwinStateProperty<>(  
                                physicalPropertyEventMessage.getPhysicalPropertyId(),  
                                physicalPropertyEventMessage.getBody()));

				//NEW -> Commit State Transaction
                this.digitalTwinStateManager.commitStateTransaction();  
            }  
        }  
        else  
            logger.error("WRONG Physical Event Message Received !");  
  
    }catch (Exception e){  
        e.printStackTrace();  
    }  
}
```
# Digital Adapter 

The Digital Adapter base class has been significantly extended and improved with respect to the previous version. 
In this new Version notifications that are received by the Adapter from the the DT core belongs to the following categories:

- **Digital Twin State Update** through the method `onStateUpdate(...)` providing information about the new state of the Digital Twin, the previous state, and a list of changes that occurred between these two states. In the previous version each variation of a property, relationships, actions or events were notified. In the new version only a committed DT'State variation is notified to listeners.
- **Event Notifications** through the method `onEventNotificationReceived(...)` whenever there is a notification about an event related to the Digital Twin's state coming from the physical world, generated by the twin and processed by the Shadowing Function. For example in the DT State we can have the declaration of the `over-heating-alert` structured and received in the DT State while the effective occurrence of the event and the associated notification is notified through this dedicated callback 

The `onStateUpdate` method is an abstract method that must be implemented by any class extending the `DigitalAdapter` class. This method is called whenever there is an update to the Digital Twin's state. It provides information about the new state of the Digital Twin, the previous state, and a list of changes that occurred between these two states.

Here is an explanation of the parameters:

1. `newDigitalTwinState`: This parameter represents the updated state of the Digital Twin. It is an instance of the `DigitalTwinState` class, which encapsulates the current state information.
    
2. `previousDigitalTwinState`: This parameter represents the state of the Digital Twin before the update. It is also an instance of the `DigitalTwinState` class.
    
3. `digitalTwinStateChangeList`: This parameter is an `ArrayList` containing `DigitalTwinStateChange` objects. Each `DigitalTwinStateChange` object encapsulates information about a specific change that occurred between the previous and new states. It includes details such as the property or aspect of the state that changed, the previous value, and the new value.

The DT State is automatically monitored by each Digital Adapter while for the `Events` potentially generated by the DT can be observed by each adapter using:   
  
- `observeAllDigitalTwinEventsNotifications`: Enable the observation of available Digital Twin State Events Notifications.  
- `unObserveAllDigitalTwinEventsNotifications`: Cancel the observation of Digital Twin State Events Notifications  
- `observeDigitalTwinEventsNotifications`: Enable the observation of the notification associated to a specific list of Digital Twin State events. With respect to event a notification contains the new associated value  
- `unObserveDigitalTwinEventsNotifications`: Cancel the observation of a target list of properties  
- `observeDigitalTwinEventNotification`: Enable the observation of the notification associated to a single Digital Twin State event. With respect to event a notification contains the new associated value  
- `unObserveDigitalTwinEventNotification`: Cancel the observation of a single target event
#### DigitalTwinStateChange Class

**DigitalTwinStateChange Class**

The `DigitalTwinStateChange` class is a representation of a change that occurred in the state of a Digital Twin. It encapsulates information about the type of operation, the resource type, and the affected resource within the Digital Twin.

**Enums:**

- `Operation`: Enumerates different types of operations that can be performed on a Digital Twin state. The possible operations are:
    - `OPERATION_UPDATE`: Represents an update operation on a resource.
    - `OPERATION_UPDATE_VALUE`: Represents an update operation specifically on the value of a resource.
    - `OPERATION_ADD`: Represents an addition operation of a new resource.
    - `OPERATION_REMOVE`: Represents a removal operation of an existing resource.
- `ResourceType`: Enumerates different types of resources within a Digital Twin. The possible resource types are:
    - `PROPERTY`: Represents a property of the Digital Twin.
    - `PROPERTY_VALUE`: Represents the value of a property within the Digital Twin.
    - `EVENT`: Represents an event associated with the Digital Twin.
    - `ACTION`: Represents an action that can be performed on the Digital Twin.
    - `RELATIONSHIP`: Represents a relationship between different components of the Digital Twin.
    - `RELATIONSHIP_INSTANCE`: Represents an instance of a relationship.

**Fields:**

- `operation`: Indicates the type of operation performed on the Digital Twin state (e.g., update, add, remove).
- `resourceType`: Represents the type of resource affected by the change (e.g., property, event, relationship).
- `resource`: The specific resource that has undergone the change, represented by an instance of the `DigitalTwinStateResource` class.

Available type of `DigitalTwinStateResource` are: 

- `DigitalTwinStateProperty<T>`: This class define a generic property associated to the Digital Twin State.  Each property is associated to a Key and a Value. Furthermore, it can also be associated to a type  to identify its nature and data structure. By default, it is associated to the type of the  Class (e.g., java.lang.String) but it can be directly changed by the developer  to associate it to a specific ontology or data type.
- `DigitalTwinStateEvent`: This class define a generic event associated to the Digital Twin State.  Events enable a mechanism for asynchronous messages to be sent by the digital twin (e.g., an overheating) . They are different from Properties that can change values according to the type of Digital Twin and may be associated also to telemetry patterns. Each event is associated to a Key and a Type used to identify its nature and data structure.  By default, it is associated to the type of the Class (e.g., java.lang.String) but it can be directly changed by the developer to associate it to a specific ontology or data type.
- `DigitalTwinStateAction`: This class define a generic action associated to the Digital Twin State.  Each action is by a key, an action type and a content type used to identify the expected input required by the action.  The type of the can be directly changed by the developer to associate it to a specific ontology or data type.
- `DigitalTwinStateRelationship<T>`: Structures and describes a Relationship in the Digital Twins's State.  This is just the description of the relationships while the effective values/instances are described through the  other class DigitalTwinStateRelationshipInstance
- `DigitalTwinStateRelationshipInstance<T>`: Structures and describes a Relationship Instance in the Digital Twins's State.  This is effective description of a relationship while its generic declaration is described through the class  DigitalTwinStateRelationship.

When there is a change in the DT State it is possibile to cast the received resource variation to the correct one. For example in the following code we detect and manage the variation on a Property Value: 

```java
// Get information from the state change  
DigitalTwinStateChange.Operation operation = stateChange.getOperation();  
DigitalTwinStateChange.ResourceType resourceType = stateChange.getResourceType();  
DigitalTwinStateResource resource = stateChange.getResource();  
  
// Search for property value variation  
if(resourceType.equals(DigitalTwinStateChange.ResourceType.PROPERTY_VALUE) && operation.equals(DigitalTwinStateChange.Operation.OPERATION_UPDATE) && resource instanceof DigitalTwinStateProperty){  
  
    DigitalTwinStateProperty<?> digitalTwinStateProperty = (DigitalTwinStateProperty<?>) resource;  
  
    if(getConfiguration().getPropertyUpdateTopics().containsKey(digitalTwinStateProperty.getKey())){  
        //Handle property value variation
    }  
}
```

**Constructors:**

- `DigitalTwinStateChange()`: An empty constructor that allows creating an instance of the class.
- `DigitalTwinStateChange(Operation operation, ResourceType resourceType, DigitalTwinStateResource resource)`: Constructs a `DigitalTwinStateChange` object with specified operation, resource type, and resource. Throws a `WldtDigitalTwinStateException` if any of the parameters is missing or null.

**Methods:**

- Accessor methods (`getOperation()`, `getResourceType()`, `getResource()`) to retrieve the values of the fields.
- Mutator methods (`setOperation()`, `setResourceType()`, `setResource()`) to update the values of the fields.

#### Usage Examples

Developers extending the `DigitalAdapter` class should implement the `onStateUpdate` method to define custom logic that needs to be executed whenever the state of the Digital Twin is updated. This could include tasks such as processing state changes, updating internal variables, triggering specific actions, or notifying other components about the state update.

Here's an example of how the method might be implemented in a concrete subclass of `DigitalAdapter`:

```java
@Override
protected void onStateUpdate(DigitalTwinState newDigitalTwinState,
                              DigitalTwinState previousDigitalTwinState,
                              ArrayList<DigitalTwinStateChange> digitalTwinStateChangeList) {

	// In newDigitalTwinState we have the new DT State
	System.out.println("New DT State is: " + newDigitalTwinState);

	// The previous DT State is available through the variable previousDigitalTwinState
	System.out.println("Previous DT State is: " + previousDigitalTwinState);

    // We can also check each DT's state change potentially differentiating the behaviour for each change
    if (digitalTwinStateChangeList != null && !digitalTwinStateChangeList.isEmpty()) {
        
        // Iterate through each state change in the list
        for (DigitalTwinStateChange stateChange : digitalTwinStateChangeList) {
            
            // Get information from the state change
            DigitalTwinStateChange.Operation operation = stateChange.getOperation();
            DigitalTwinStateChange.ResourceType resourceType = stateChange.getResourceType();
            DigitalTwinStateResource resource = stateChange.getResource();
            
            // Perform different actions based on the type of operation
            switch (operation) {
                case OPERATION_UPDATE:
                    // Handle an update operation
                    System.out.println("Update operation on " + resourceType + ": " + resource);
                    break;
                case OPERATION_UPDATE_VALUE:
                    // Handle an update value operation
                    System.out.println("Update value operation on " + resourceType + ": " + resource);
                    break;
                case OPERATION_ADD:
                    // Handle an add operation
                    System.out.println("Add operation on " + resourceType + ": " + resource);
                    break;
                case OPERATION_REMOVE:
                    // Handle a remove operation
                    System.out.println("Remove operation on " + resourceType + ": " + resource);
                    break;
                default:
                    // Handle unknown operation (optional)
                    System.out.println("Unknown operation on " + resourceType + ": " + resource);
                    break;
            }
        }
    } else {
        // No state changes
        System.out.println("No state changes detected.");
    }
}
```

In this example, the method iterates over the list of state changes, extracts information about each change, and performs custom actions based on the changes. Developers can adapt this method to suit the specific requirements of their Digital Twin application.
