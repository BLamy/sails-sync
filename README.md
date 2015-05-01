# sails-sync
Sails Sync is a project aimed at syncing rest endpoints into an indexed db using a simple api. 

**Models are defined declaratively** using polymer web components. An `io.models` property is added to the global object created by `sails.io.js`. 

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
  > Note: sails-sync cannot track destroyed records when it is not connected. Therefore it is recommended to employ a "soft delete". 
  
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


## Querying
The goal of sails-sync is to develop an API which represents waterline. This will allow developers to use the same query on the frontend and the backend. 

*Querying is currently limited*

Find One - find one object using a Unique index.
```
  document.querySelector("indexeddb-cache").addEventListener("indexedDB-opened", function(e) {
    io.models["User"].findOne({
      name: "BLamy"
    }, function(data){
      console.log(data);
    });
  });
  ```
  
Find - find all objects using non unique index
```
  document.querySelector("indexeddb-cache").addEventListener("indexedDB-opened", function(e) {
    io.models["User"].find({
      usergroup: "admin"
    }, function(data){
      console.log(data);
    });
  });
  ```

  
Models within an `indexeddb-cache` must wait until `indexedDB-opened` before sending request to the database. This is unfortuantly some time after domReady because we need to wait for polymer to parse the tags. 


Models within `sails-socket` will hit the server on a find request. Models within an `indexeddb-cache` will query the database.



# Usage:
**1) Create a new sails project**
> sails new projectName


**2) Create a `.bowerrc` file in your project root**
```
{
  "directory": "assets/bower_components"
}
```


**3) Install Polymer 0.8**
> bower install --save webcomponentsjs

> bower install --save Polymer/polymer#^0.8



**4) Install sails-sync**
> bower install --save https://github.com/BLamy/sails-sync.git



**5) Create a sails API to sync**
> sails generate api user



**6) Add data to your endpoint**

At CLI:

> sails lift

In Webbrowser visit:

> http://localhost:1337/user/create?name=foo



**7) Import all webcomponents in `layout.ejs`, Add the following between `<head>`**
```
  <script src="/bower_components/webcomponentsjs/webcomponents.min.js"></script>
  <link rel="import" href="/bower_components/sails-sync/sails-socket.html">
  <link rel="import" href="/bower_components/sails-sync/rest-api.html">
  <link rel="import" href="/bower_components/sails-sync/indexeddb-cache.html">
```


**8) Declaratively define your model.**
```
<sails-socket>
  <indexeddb-cache version="1">
    <rest-api name="User" indexes='[{"name":"name", "attributes":{"unique":true}}]'></rest-api>
  </indexeddb-cache>
</sails-socket>
```
All users should be syncing to an indexedDB named `sails-sync-cache`, in an object store names `User` with an index on `name`. The `key-path` for user will be assumed to be `id` since it was not supplied.


**9) Run Grunt**
> grunt


**10) Query your models**
```
  document.querySelector("indexeddb-cache").addEventListener("indexedDB-opened", function(e) {
    io.models["User"].findOne({
      name: "BLamy"
    }, function(data){
      console.log(data);
    });
  });
  ```
Models within an `indexeddb-cache` must wait until `indexedDB-opened` before sending request to the database. This is unfortuantly some time after domReady because we need to wait for polymer to parse the tags. 

Models within `sails-socket` will hit the server on a find request. Models within an `indexeddb-cache` will query the database.



## Roadmap
1) Querying is limited. The objective is to get querying as close to waterline as possible. That way the same query can be used on server side as can be used on client side. 

2) Test Coverage

3) No relationships. 
