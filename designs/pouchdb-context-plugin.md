---
state: draft
---

# PouchDB Context store plugin

The PouchDB Context store plugin holds context data in the PouchDB.

## Summary

This plugin uses the PouchDB database for all context scopes.  
PouchDB is an abstraction layer and the database to be used is selected by the adapter.  
This plugin applies a Node SQLite adapter and saves the data in SQLite.

pouchdb Adapters:
https://pouchdb.com/adapters.html

## Author
 - @KazuhiroIto

## Details

### Install

1. Run the following command in your Node-RED user directory - typically `~/.node-red`

    npm install git+https://github.com/node-red/node-red-context-pouchdb

2. Add a configuration in settings.js:

```javascript
contextStorage: {
    pouchdb: {
        module: require("node-red-context-pouchdb"),
        config: {
            // see below options
        }
    }
}
```

### Options

| Options  | Description                                                                   |
| -------- | ----------------------------------------------------------------------------- |
| base     | the base directory to use.                     `default: "context"`           |
| dir      | the directory to create the base directory in  `default: settings.userDir`    |

The file name of the database is `"context.db"`.  
Default database path:  `~/.node-red/context/context.db`

### Data Model

- This plugin uses a PouchDB database for all context scope.
- Use the Node SQLite adapter for the PouchDB adapter.
Context Data is stored in SQLite3 using PouchDB API.
- This plugin saves a JSON object of keys and values in a document for each scope.
  - The keys of `global context` will be id with `global` .Context data is stored in doc.data.
  - The keys of `flow context` will be id with `<id of the flow>` .Context data is stored in doc.data.
  - The keys of `node context` will be id with `<id of the node>` .Context data is stored in doc.data.

Structure of data stored in PouchDB:
```json
{
    "total_rows": 3,
    "offset": 0,
    "rows": [
        {
            "id": "2052fca8.312154:a77d79a4.d1a908",
            "key": "2052fca8.312154:a77d79a4.d1a908",
            "value": {
                "rev": "6-55e0513ffba64a8b8efec1ba8e43c90f"
            },
            "doc": {
                "data": {
                    "NODE-KEY-1": "NODE-DATA-1",
                    "NODE-KEY-2": "NODE-DATA-2"
                },
                "_id": "2052fca8.312154:a77d79a4.d1a908",
                "_rev": "6-55e0513ffba64a8b8efec1ba8e43c90f"
            }
        },
        {
            "id": "a77d79a4.d1a908",
            "key": "a77d79a4.d1a908",
            "value": {
                "rev": "61-2c2a457388db1c3859b79e4bb62e9375"
            },
            "doc": {
                "data": {
                    "FLOW-KEY-1": "FLOW-DATA-1",
                    "FLOW-KEY-2": "FLOW-DATA-2",
                },
                "_id": "a77d79a4.d1a908",
                "_rev": "61-2c2a457388db1c3859b79e4bb62e9375"
            }
        },
        {
            "id": "global",
            "key": "global",
            "value": {
                "rev": "73-ee260387e51ae20132076ccc83957600"
            },
            "doc": {
                "data": {
                    "GLOBAL-KEY-1": "GLOBAL-DATA-1",
                    "GLOBAL-KEY-2": "GLOBAL-DATA-2",
                },
                "_id": "global",
                "_rev": "73-ee260387e51ae20132076ccc83957600"
            }
        }
    ]
}
```

### Data Structure

- Data is saved in the JSON object format supported by PouchDB. The plugin does not convert JSON data to a string for storage.

- Code example that references database data:
```javascript
var pd = require('pouchdb');
pd.plugin(require('pouchdb-adapter-node-websql'));
var db;

db = new pd( "/tmp/pouchdb/context/context.db", { adapter: 'websql' });
db.allDocs({include_docs: true}, function(err, doc) {
   if (err) {
        return console.log(err);
   } else {
        var data = JSON.stringify(doc,null,4);
        console.log(data);
   }
});
```

## History

  - 2021-03-03 - Initial Design Note
