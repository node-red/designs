---
state: draft
---

# Grouping Nodes

## Summary

This design note covers the ability to group nodes in the editor in a way that
is familiar to many drawing applications such as PowerPoint.

A group of nodes can be moved, copy/pasted as a single entity within the editor.

The group can have some visual properties applied, including a border width/color/
style and a background color/style.

The group can also have additional key/value pairs applied to it. These properties
are then available to the nodes within the group.

## Authors

 - Nick O'Leary

## Use Cases

### Documenting Flows

The visual appearance of the group can be used to help document the flows directly
in the workspace.

We already provide multiple ways to document flows - adding descriptions to flows
and nodes, and adding Comment nodes to the workspace. But none of these approaches
help to indicate what nodes a Comment node is related to.

By creating a group with a Comment node in the corner as a label, it is clearer
what the Comment node relates to.

![](images/groups-uc1.png)

### Adding metadata to parts of the flow

There is a long-term concept for Node-RED where a flow as drawn in the editor
is actually deployed across multiple runtimes.

By using groups, and then applying suitable metadata to each group, the runtime
would then be able to take appropriate action to deploy the different parts to the
require location.

![](images/groups-uc2.png)

### User-defined scalability of flows

Related to the previous use case, the metadata provided by a group could be used
by the runtime to scale different parts of the flow in different ways. The user
would be able to identify which parts of the flow is suitable for scaling and
which should not be scaled.


## Details

This design needs to cover:

 - The definition of a group
   - How is a group represented in the flow file?
   - Can a group contain groups? (hierarchy of groups)
   - Can a node exist in more than one group? (overlapping groups)
   - What properties does a group have?
 - The user interaction with groups in the editor:
   - How does a user create a group?
   - How are a group’s properties edited?

### Definition of a group

A group is defined as:

```
{
    "id": 123,
    "type": "group",
    "z": "container-id",
    "nodes": [ list of node/group ids]
    ...other metadata...
}
```
#### Group properties

 - **id** - unique identifier for the group
 - **type** - ``"group"``
 - **name** - a user-friendly name for the group
 - **z** - identify the container of the group. If this is a 'top-level' group,
   it will be the id of the `tab` or `subflow` it is in. If it is nested inside
   a group it will be the id of the `group`.
 - **nodes** - an array of node/group ids that are in the group.
 - **style** - an object containing properties related to the appearance of the group.
   These will be roughly consistent with SVG/CSS styles. The exact list of properties
   may change.
    - **stroke**
    - **stroke-width**
    - **fill**
    - **fill-opacity**
    - **width**/**height**
       - blank/`auto` - determine based on the bounding boxes of the group contents
       - `fill` - the group will cover the whole canvas in this direction
    - **padding** - for `auto` width/height, how much space is added around the
      content's bounding box
 - **meta** - (name tbd) - an object of user-defined metadata for the group


When a node is added to a group, its `z` property will be updated to be the id
that group.

I don't think the `x`/`y` properties will be updated - they would remain absolute
co-ordinates in the tab. This decision may change....

This model means:

 - a node cannot be in multiple groups - it is either not in a group, or has a
   single 'parent' group
 - a group can be in a group (nested groups)
 - groups cannot partially overlap

 <details><summary><b>Comments on backwards compatibility</b></summary>

 ***
 It is logically consistent for a node's `z` property to be updated to the id of
 the group it is in. That means the `z` is always the immediate container of an
 object.

 However that does reduce the backwards compatibility and more work is needed in
 order to identify exactly what nodes are on a given tab.

 Something to think more about...
 ***
 </details>


### Flow file representation

There are two options for how a group exists in the flow file.

##### Option 1. A `group` node type

 - A group would exist as a new node type in the flow file.

**Problem:** Older instances of NR will then refuse to run the flow as they
don't know what a `group` is.

**Solution:** Release a `1.0.x` that adds logic to filter/ignore this node type.
This won't help users on older versions, but would possibly help during the
transition between versions.

**Problem:** Need to check if there are any existing contrib nodes that use the
type `group`.

**Solution:** Pending the check of the Flow Library, the name of this node type
may have to change.


##### Option 2. Add a `groups` property to the flow/subflow nodes

 - The flow (`tab`) and `subflow` type nodes gain a new property called `groups`.
 - This is an array of `group` objects (as described above).

 - Each regular node will gain a new property called `g` that will be the `id` of
the group it is in. If it doesn't have this property then the node is not in a
group.

**Problem**: If the group definition is in the `tab` node, what happens when a user
selects a group and exports it? They are exporting a selection of nodes, not a full
flow - so there is no `tab` node in the exported JSON to hold the group definition.

#### Proposal - Option 1.

Whilst I'm in favour of maximising backwards compatibility (ie Option 2), I think
that would compromise the flow format and groups would be an obvious 'hack'.

**I propose we go with Option 1** - add a new `group` node type. We should also
look at what could be done in the 1.0.x stream to add basic support for importing
such a flow (obviously losing the group functionality when doing so).

### User Interaction

This is the most important part to get right and not all of it can be designed
on paper. Some of the fine detail will come from experimentation.

The intention is the grouping function is familiar to users who have used grouping
functions in other applications, for example, PowerPoint, Keynote and Inkscape.

More research and testing of those applications is needed to identify the standard
patterns of interaction with groups and their contents. They are all slightly
different, so we need to identify what feels right in the Node-RED context.

#### Creating a group

1. The user selects one or more nodes in the workspace
2. They then create the group by either:
   - Invoke the new `create-group` action
   - Use the keyboard shortcut assigned to that action
   - Select the `Create Group` option from the drop-down menu

#### Ungrouping

1. The user selects a group in the workspace
2. They then remove the group by either:
   - Invoke the new `remove-group` action
   - Use the keyboard shortcut assigned to that action
   - Select the `Remove Group` option from the drop-down menu

This will remove the group but leave its contents in place.

#### Deleting a group

1. The user selects the group.
2. They delete it by pressing the delete key
   - Invoke the existing `delete-section` action
   - Use the keyboard shortcut assigned to that action

#### Editing a group

1. The user double-clicks on the group to open its edit dialog
2. The edit dialog provides options to set its appearance:
   - border color, thickness and style
   - background color, opacity and style
   - size and padding

As with nodes, it will have a Description tab in the edit dialog.

It will also provide a properties table where user-defined key/value pairs can
be set. (This could be done in a later iteration - it isn't required for the
initial implementation)


#### Adding a node to an existing group

We don't want the groups to get in the way of creating flows. Once a group has
been created, it should be easy to add/remote nodes from the group without having
to ungroup, select the nodes, recreate group.

##### Dragging a node into a group

If the node is dragged over a group, and the user pauses the drag for a couple of
seconds, it will 'enter' the group. This will reduce the chance of a node being
accidentally added to a group when moving them around.

##### Merging selection

Another action will be provided called `merge-selection-to-group` (name tbd). This
will merge the current selection into a group.

If all of the selected 'things' are nodes, this is equivalent to the `create-group`
option.

If there are one or more groups in the selection, they will be merged to be a single
group. The new group will adopt the appearance of the first group in the selection.
This is different to the `create-group` action which would create a new group
containing the groups.

#### Removing a node from a group

There's not an obvious way to drag a node out of a group. Maybe adding a combined
keyboard meta-key + mouse option (We already use `Shift-Drag` to toggle `snap-to-grid`
whilst dragging). Not sure how intuitive that would be.

An action will be provide called `remove-selection-from-group` (name tbd) that
can be used to move the selection out of the group, with a keyboard shortcut
and menu item.


## History

- 2020-01-15 - Initial proposal
