---
state: draft
---

# Node Property Typing

## Summary

A node's `defaults` object in its HTML definition provides a list of its properties
with some metadata associated with them.

If a property is intended to be a reference to a configuration node, its entry
in the defaults object will include the `type` property that identifies the type
of config node it should point to.

This allows the editor to automatically generate the Config Node select box UI
and manage the relationship between the nodes.

This design note explores extending this property to allow for a richer set of
relationships between nodes.

## Authors

 - @knolleary

## Details


### Examples uses

#### `mqtt in` nodes references a `mqtt-broker` node

This is the existing use of the `type` property.
```
broker: { type: "mqtt-broker"}
```

#### `link out` node references multiple `link in` nodes

In the current code, the editor is aware of the `link` nodes and their `links`
property that is an array of ids to partner link nodes.

This design proposes the following syntax could be used to tell the editor
this property is an array of references to `link in` nodes:

```
links: { type: "link in[]"}
```


#### `catch` node references multiple nodes of any type

As with the `link` nodes, the editor knows it has to handle the `scope` properties
of the `catch`, `status` and `complete` nodes as special cases.

Unlike the link nodes, these nodes can reference any node type. This design proposes
this could be expressed as follows.

```
scope: { type: "*[]"}
```

*Note: an earlier iteration of this design proposed `:any[]`. That was inspired
by TypeScript's `any` syntax, but the `:` was added to make it look like a keyword -
as `any` would be a valid node-type. On second though, this didn't look right, so
has been changed to the above syntax.*

#### Node references a single node of multiple possible types

The current model requires one node property per type of config node the node
may have a relationship to. We have had cases where a node needs to reference a
config node that could be of different types (for example, the WebSocket nodes).

The following sytanx could used to list the candidate types:

```
ui_container: { type: "ui_group | ui_widget"}
```

#### Node references multiple nodes of multiple possible types

```
ui_container: { type: "(ui_group | ui_widget)[]"}
```


## History

- 2021-01-07 Updated syntax
- 2020-09-29 Initial design drafted
