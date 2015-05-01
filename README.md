# sails-sync
Sails Sync is a project aimed at syncing rest endpoints into an indexed db using a simple api. 

**Models are defined declaratively** using polymer web components. A `io.models` property is added to the global object created by `sails.io.js`. 

> sails-socket has a dependency on the sails.io.js file included in sails.js. If you used `sails new ` to create your project you already have it.

Sails-sync contains 3 web components which can be used together to sync data from a REST API directly into a IndexedDB.
* `sails-socket` - establishes a socket connection to your server. This component is opinionated at Sails but if a similar API is mimicked could work with any REST API.
* `indexeddb-cache` - Traditionally put inside of sails socket. All models within `indexeddb-cache` will sync all data at `get /`, and subscribe for changes
* `rest-api` - These are your models. They do a lot! Read on.

## Pub / Sub
Models placed inside a socket can take advantage of sails Pub/Sub system.
```
<sails-socket>
  <rest-api name="User"></rest-api>
</sails-socket>
```

Can be accessed from `io.models["User"]`. 
  ```
  io.models["User"].subscribe(this, { 
      created: function(data){},
      updated: function(data){},
      destroyed: function(data){}
      });
  ```
  **Any webcomponent can register for model changes.** 
  
  
  The `data` object being the following for each:
  
  **created**
  ```
  {
    id: 1,
    verb: "created",
    data:{ 
      // Object that was created
    }
  }
  ```
  
  **updated**
  ```
  {
    id: 1,
    verb: "updated",
    data:{
      // Changed deltas
    },
    previous:{
      // Old Object
    }
  }
  ```
  
  **destroyed**
  ```
  {
    id: 1,
    verb: "destroyed",
    previous:{
      // Object that was just deleted
    }
  }
  ```
  > Note: sails-sync cannot track destroyed records when it is not connected. There for it is recommended to employ a "soft delete". 
  
## Indexeddb-cache
`indexeddb-cache` can wrap `rest-api` models. Upon socket connection the models inside of `indexeddb-cache` will sync all data at ```get /?where={updatedAt:{'>': `$last_updated`}}```
  


```
<sails-socket>
    <rest-api name="User"></rest-api>
    <indexeddb-cache version="26">
      <rest-api name="Device" key-path="UUID" indexes='[{"name":"name", "attributes":{"unique":true}}]'></rest-api>
    </indexeddb-cache>
  </sails-socket>
  ```
  
`rest-api's` take on a couple new attributes when they are in an `indexeddb-cache`.
* `key-path` - AKA Primary Key of the indexeddb ObjectStore. Default is `id`.
* `indexes` - A **double quoted json** string, Config object for `store.createIndex(name, name, args)`


