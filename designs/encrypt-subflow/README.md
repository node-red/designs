---
state: draft
---

# Encoding SUBFLOW

This proposal covers how a SUBFLOW can be encoded when exported as reusable node.

Node-RED flow definitions are currently distributed as JSON data.  This JSON representation exposes algorithms represented in flow.  Thus, flow can be directly or maniplated by other Node-RED instance that imported the JSON data or inspected manually.

In some cases, Node-RED users do not want their flows to be looked or manipulated by unauthorized users because of intelectual property or other reasons.

This design note aims to address this issue and proposes a flow encoding/decoding feature.

### Authors

- @HiroyasuNishiyama

### Details

The flow encoding feature is realized as an extension of the feature
for converting an exportable subflow to NPM node module proposed at
https://github.com/node-red-hitachi/designs/tree/subflow2npm/designs/subflow-to-npm .  In this SUBFLOW conversion feature proposal, users that import a node created from a  SUBFLOW is not possible to refer to the internals of the
SUBFLOW.  However, the internal structure of the SUBFLOW is exposed in
the JSON distribution format included in the NPM package.  Therefore, the purpose of this design note is to prevent an unauthorized person from referring to the internal representation of a SUBFLOW by encoding the internal structure of the SUBFLOW.

![encryption](encryption.png)

Since the flow encoding algorithm depends on use case, it should be selected by user setting.

#### Encoded SUBFLOW Representation

Exportable SUBFLOW adopts new flow format.  It embeds an array of nodes within a SUBFLOW in `flow` property.  When encrypting SUBFLOW, set the `flow` property to the following object instead of an array.

|     | name     | type   | description                  |
| --- | -------- | ------ | ---------------------------- |
| 1   | encoding | string | encoding method              |
| 2   | flow     | string | encoded nodes within SUBFLOW |

![encrypted-json](encrypted-json.png)

#### Encoding Mechanism

The template for packaging SUBFLOW JSON as NPM provides a means for specifying methods for encoding / decoding flows.  This is specified by `encodeSubflow` property of `settings.js`.  `encodeSubflow` property points to an object with following properties:

|     | name    | type   | description                         |
| --- | ------- | ------ | ----------------------------------- |
| 1   | methods | array  | encoding methods specifications     |
| 2   | default | string | default encoding specification name |

Encoding method specification is represented by following object:

|     | name        | type     | description                                                                                                                                                               |
| --- | ----------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | name        | string   | name of encoding method                                                                                                                                                   |
| 2   | requiresKey | booleak  | `true` if encoding/decoding requires key, otherwise `false`                                                                                                               |
| 3   | encode      | function | flow encode function that accepts an JavaScript object that represents encoded SUBFLOW's nodes array and an encoding key.  Returns string representation of encoded flow. |
| 4   | decode      | function | flow decode function that accepts string encoded flow and a decoding key.  Returns a JavaScript object that represents array of nodes.                                    |

When loading/packaging decoded/eboded flow, it uses the `encode`/`decode` function with specified name.  The `default` property specifies default encofing method specification. 

A tool or Editor UI for packaging NPM module from SUBFLOW JSON representation will provide selection interface or option for encoding method, or select default encoding method.

**Example:**

```
encodeSubflow: {
    methods: [
        {
            name: "method1",
            encode: function(flow, key) {
               // code for encoding flow
            },
            decode: function (flow, key) {
                // code for decoding flow
            },
        },
        ....
    ],
    default: "method1"
}
```



#### Limitations

To execute the flow, the Node-RED runtime needs to decrypt the flow.  Therefore, a flow can be acquired by using the corrected runtime.  This design note does not assume that such cases will be addressed.

### Remaining Issues

Editor interface for supporting encoding subflow needs following:

- UI for specifying encoding scheme,

- API for accessing `encodeSubflow` list
  
  These items will be updated after the details of SUBFLOW packaging by Editor have been decided.

## History

- 2020-02-22 - initial design note
- 2021-03-15 - update design
