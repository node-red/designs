---
state: draft
---

# Version information of developed flow
## Summary
This design note is about supporting version information in the package.json file created by the project feature.

Node-RED users can select two versions of Node-RED from upstream.
As of September 2022, v3.0.x and 2.2.x have been released by the Node-RED project.
There is no forward compatibility between the two versions.
Therefore, Node-RED sometimes executes the flow in unintentional behaviors when importing flow from a newer Node-RED to another older one.
For stable operations of Node-RED instance, some Node-RED services have used the older version of Node-RED but this situation leads to failure of the flow.

The version information of the flow and Node.js will be useful to solve this concern.

## Authors
[@kazuhitoyokoi](https://github.com/kazuhitoyokoi)

## Details
Because professional users tend to use project features in Node-RED flow development, the package.json file is suitable for storing the version information.

### Flow compatibility
Node-RED version information uses the dependencies section in the package.json file as follows.

```
"dependencies": {
    "node-red": ">=3.0.2",
    ...other dependencies...
},
```

The above example means the Node-RED flow can be executed correctly in greater than or equal to Node-RED v3.0.2.
If the environment is less than Node-RED v3.0.2, the flow editor shows the warning notification dialog as described later.
The old flow editor doesn't support this functionality but the version information will be valuable for users to investigate the environment version manually.

For example, The http request node in Node-RED version 3.0.x supports new functionality to set header information in the node property.
But the http request node in Node-RED version 2.2.x doesn't use the header information if the version 3.0.x flow is used.
Because of that, the process of the http request node fails when the REST API endpoint requires the header information.

__Alternative option__

The idea described above also fills the runnable project simultaneously.
But the following alternative way can be used if the version information should be separated.

```
"node-red": {
    "version": ">=3.0.2"
}
```

An alternative method is the same method as the custom node for Node-RED.

### Node.js compatibility
The Node.js API written in the function node depends on the version of Node.js.
Therefore, the following version information in the package.json file is useful to specify the version which the flow used.

```
"engines": {
    "node": ">=18.x"
},
```

For instance, the following code works in Node.js version 18.x correctly but Node.js version 16.x returns the error, "TypeError: array.findLastIndex is not a function".

```
var array = ['h', 'e', 'l', 'l', 'o'];
msg.payload = array.findLastIndex(function (i) {
    return i === 'l';
});
return msg;
```

The method is the same principle as the custom node and can be also used for the runnable project.

### UI designs
After cloning the Git repository, the Node-RED flow editor shows the notification dialog to warn when the versions of the Node-RED or Node.js are different from the version of the flow.
The dialog has the following information.

- Required: Node-RED v3.0.2, Node.js v18.x
- Actual: Node-RED v2.2.2, Node.js v16.17.0

In this example, the flow on the Git repository was developed on Node-RED v3.0.2 and Node.js v18.x.
But the flow editor is Node-RED v2.2.2 on Node.js v16.17.0.
From the dialog, the user will be aware of the version differentiation.

## History
- 2022-09-23 - Initial proposal submitted
