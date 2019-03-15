---
state: draft
---

# Atomic Context API

## Summary

To avoid concurrency headaches, we should add some utility functions to the Context api to allow atomic updates to properties.  
This is important for when you want to have multiple instances running and interacting with the same context properties.

## Authors

 - @HirokiUchikawa

## Details

### Use Cases

1. Multiple Node-RED instances share context without causing data inconsistency or conflicts.  
   When multiple instances share a context and try to update it with the current API, data conflicts may occur.  
   For example, two instances share a context and use it as a counter. When they increment it at the same time, operations may interleave as following.
   
   | Time | value of `count` in store | Instance A                  | Instance B                  |
   |:----:|:-------------------------:|-----------------------------|-----------------------------|
   |   1  |             0             | var c = global.get("count") |                             |
   |   2  |             0             | c = c + 1                   | var c = global.get("count") |
   |   3  |             1             | global.set("count", c)      | c = c + 1                   |
   |   4  |             1             |                             | global.set("count", c)      |
   |   5  |             1             |                             |                             |
   
   Each instance increments the counter, so the expected value of 'count' is 2, but the result is 1.

### Atomic Context Store API

These proposed API will be added to the [Context Store API](https://nodered.org/docs/api/context/methods/).  
The context plugin (e.g. memory, localfilesysytem) should create that implements these API.

#### incr/decr

- ContextStore.incr(scope, key, [amount], [callback])  
  
  | Name      | Type       | Description                                                            |
  | --------- | :--------: | ---------------------------------------------------------------------- |
  | scope     | `string`   | the scope of the key                                                   |
  | key       | `string`   | the key to increment the value for.                                    |
  | amount    | `number`   | _optional_ the amount to increment by. Default : `1`                   |
  | callback  | `function` | _optional_ a callback function to invoke when the value is incremented |
 
- ContextStore.decr(scope, key, [amount], [callback])
  
  | Argument  | Type       | Description                                                            |
  | --------- | :--------: | ---------------------------------------------------------------------- |
  | scope     | `string`   | the scope of the key                                                   |
  | key       | `string`   | the key to decrement the value for.                                    |
  | amount    | `number`   | _optional_ the amount to decrement by. Default : `1`                   |
  | callback  | `function` | _optional_ a callback function to invoke when the value is decremented |


##### Description

These APIs increments/decrements the value by one or passed amount. If the key does not exist, it is set to 0 before performing the operation.  

If no callback is provided, and the store supports synchronous access, the incr/decr function should return the value after the increment/decrement. If the store does not support synchronous access it should throw an error.
```javascript
incr(scope, key, amount) // return the value after the increment.
decr(scope, key, amount) // return the value after the decrement.
```

If callback argument is provided, it must be a function that takes two arguments:
```javascript
incr(scope, key, amount, (error, value) => {
    // If an error occurred in process, pass the error object as first argument.
    // If the process was succeed, pass null as first argument and the incremented value as second argument.
});
decr(scope, key, amount, (error, value) => {
    // If an error occurred in process, pass the error object as first argument.
    // If the process was succeed, pass null as first argument and the decremented value as second argument.
});
```

if the value of the key or `amount` argument is not a number, throw an error.

##### Uasge in a Function node
<details>

```javascript
global.incr("count");
// Increase the value of `count` by 1 in `default` store.
// And return the value after increment.

global.incr("count", 5);
// Increase the value of `count` by 5 in `default` store.
// And return the value after increment.

global.incr("count", "file");
// Increase the value of `count` by 1 in `file` store.
// And return the value after increment.

global.incr("count", 5, "file");
// Increase the value of `count` by 5 in `file` store.
// And return the value after increment.

global.incr("count", (err, value) => {
// Increase the value of `count` by 1 in `default` store.
// Then invoke callback and pass an error object or the value after increment.
});

global.incr("count", 5, (err, value) => {
// Increase the value of `count` by 5 in `default` store.
// Then invoke callback and pass an error object or the value after increment.
});

global.incr("count", "file", (err, value) => {
// Increase the value of `count` by 1 in `file` store.
// Then invoke callback and pass an error object or the value after increment.
});

global.incr("count", 5, "file", (err, value) => {
// Increase the value of `count` by 5 in `file` store.
// Then invoke callback and pass an error object or the value after increment.
});
```

</details>

##### Uasge in Other Nodes

TBD

It would be useful that some nodes (e.g. Change Node) can call atomic api.    
Currently change node can incr/decr with JSONata but it is not atomic.  
 
For example:
  ```json
  [{"id":"cd1ed2c0.4f3e7","type":"change","z":"92e079a4.8d9f78","name":"Increment global.count","rules":[{"t":"set","p":"count","pt":"global","to":"$globalContext(\"count\") ? $globalContext(\"count\") + 1 : 1","tot":"jsonata"}],"action":"","property":"","from":"","to":"","reg":false,"x":420,"y":60,"wires":[[]]},{"id":"af7472b9.69e3f","type":"inject","z":"92e079a4.8d9f78","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":220,"y":60,"wires":[["cd1ed2c0.4f3e7"]]}]
  ```

##### Discussion

1. Is `decr` needed ?  
   Some databases(e.g. MongoDB, Couchbase see [here](#Atomic%20API%20provided%20by%20NoSQL%20databases)) do not provide `decr` but can decrease the value using `incr` with negative number. Is `decr` needed  for Context API ? 


### Next Phase

* Add atomic API for Array  
  This is useful for representing queue and stack with array.  
* Add atomic API for String

### Atomic API provided by NoSQL databases

This proposal refers to atomic api provided by popular NoSQL databases.

<table>
  <tr>
    <th rowspan="2">#</th>
    <th rowspan="2">Name</th>
    <th colspan="3">Number Operations</th>
    <th colspan="3">String Operations</th>
    <th colspan="3">Array Operations</th>
  </tr>
  <tr>
    <th>Increment the number</th>
    <th>Decrement the number</th>
    <th>Others</th>
    <th>Append data to the string</th>
    <th>Prepend data to the string</th>
    <th>Others</th>
    <th>Add element to the array</th>
    <th>Remove element from the array</th>
    <th>Others</th>
  </tr>
  <tr>
    <td>1</td>
    <td>
    
[Redis](https://redis.io/)
    </td>
    <td>

[INCR](https://redis.io/commands/incr)  
[INCRBY](https://redis.io/commands/incrby)  
[INCRBYFLOAT](https://redis.io/commands/incrbyfloat)  
    </td>
    <td>

[DECR](https://redis.io/commands/decr)  
[DECRBY](https://redis.io/commands/decrby)  
    </td>
    <td>-</td>
    <td>
    
[APPEND](https://redis.io/commands/append)
    </td>
    <td>-</td>
    <td>-</td>
    <td>

[LPUSH](https://redis.io/commands/lpush)  
[RPUSH](https://redis.io/commands/rpush)  
[LINSERT](https://redis.io/commands/linsert)
    </td>
    <td>

[LPOP](https://redis.io/commands/lpop)  
[RPOP](https://redis.io/commands/rpop)  
[LREM](https://redis.io/commands/lrem)  
    </td>
    <td>

[RPOPLPUSH](https://redis.io/commands/rpoplpush)
[All list commnads...](https://redis.io/commands#list)
    </td>
  </tr>
    <tr>
    <td>2</td>
    <td>

[memcached](https://memcached.org/)
    </td>
    <td>

[incr](https://github.com/memcached/memcached/wiki/Commands#incrdecr)
    </td>
    <td>
    
[decr](https://github.com/memcached/memcached/wiki/Commands#incrdecr)
    </td>
    <td>-</td>
    <td>
    
[append](https://github.com/memcached/memcached/wiki/Commands#append)
    </td>
    <td>

[prepend](https://github.com/memcached/memcached/wiki/Commands#prepend)
    </td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
  </tr>
  <tr>
    <td>3</td>
    <td>
    
[MongoDB](https://www.mongodb.com/)
    </td>
    <td>

[$inc](https://docs.mongodb.com/manual/reference/operator/update/inc/)
    </td>
    <td>Pass negative value to `$inc`</td>
    <td>

[$mul](https://docs.mongodb.com/manual/reference/operator/update/mul/)
    </td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
    <td>

[$push](https://docs.mongodb.com/manual/reference/operator/update/push/)
    </td>
    <td>

[$pop](https://docs.mongodb.com/manual/reference/operator/update/pop/)  
[$pull](https://docs.mongodb.com/manual/reference/operator/update/pull/)  
[$pullall](https://docs.mongodb.com/manual/reference/operator/update/pullall/)  
    </td>
    <td>-</td>
  </tr>
    <tr>
    <td>4</td>
    <td>

[Couchbase](https://www.couchbase.com/)
    </td>
    <td>

[Bucket#counter](http://docs.couchbase.com/sdk-api/couchbase-node-client-2.6.1/Bucket.html#counter__anchor)
    </td>
    <td>Pass negative value to `Bucket#counter`</td>
    <td>-</td>
    <td>

[Bucket#append](http://docs.couchbase.com/sdk-api/couchbase-node-client-2.6.1/Bucket.html#append__anchor)
    </td>
    <td>

[Bucket#prepend](http://docs.couchbase.com/sdk-api/couchbase-node-client-2.6.1/Bucket.html#prepend__anchor)
    </td>
    <td>-</td>
    <td>

[Bucket#listAppend](http://docs.couchbase.com/sdk-api/couchbase-node-client-2.6.1/Bucket.html#listAppend__anchor)  
[Bucket#listPrepend](http://docs.couchbase.com/sdk-api/couchbase-node-client-2.6.1/Bucket.html#listPrepend__anchor)  
    </td>
    <td>
    
[Bucket#listRemove](http://docs.couchbase.com/sdk-api/couchbase-node-client-2.6.1/Bucket.html#listRemove__anchor)
    </td>
    <td>-</td>
  </tr>
  </tr>
    <tr>
    <td>5</td>
    <td>

[Cassandra](https://cassandra.apache.org/)
    </td>
    <td>

[UPDATE table_name SET column_name = column_name + value](https://cassandra.apache.org/doc/latest/cql/dml.html#update-statement)
    </td>
    <td>

[UPDATE table_name SET column_name = column_name - value](https://cassandra.apache.org/doc/latest/cql/dml.html#update-statement)
    </td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
    <td>

[UPDATE table_name SET column_name = \[ value (, value)* \] + column_name](https://cassandra.apache.org/doc/latest/cql/types.html#lists)  
[UPDATE table_name SET column_name = column_name + \[ value (, value)* \]](https://cassandra.apache.org/doc/latest/cql/types.html#lists)
    </td>
    <td>
    
[DELETE column_name\[index\] FROM table_name WHERE where_clause](https://cassandra.apache.org/doc/latest/cql/types.html#lists)
    </td>
    <td>-</td>
  </tr>
  </tr>
    <tr>
    <td>6</td>
    <td>
    
[HBase](https://hbase.apache.org/)
    </td>
    <td>
    
[Increment](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/client/Increment.html)
    </td>
    <td>Pass negative value to `Increment`</td>
    <td>-</td>
    <td>

[Append](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/client/Append.html)
    </td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
  </tr>
</table>

- Some of the above databases support transaction. So they can perform above operations by set of operations.  
  However, To focus on only single operation here, transaction operations are excluded.
- memcached and HBase do not support Array(or List) data type.

## Related Links

- https://trello.com/c/lwMQX5kn/91-add-atomic-actions-to-context-api
- https://github.com/node-red/node-red/wiki/Design%3A-Persistable-Context

## History

- 2019-03-27 - Initial proposal submitted