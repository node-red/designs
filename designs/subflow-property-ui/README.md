---
state: in-progress
---

## Subflow Property UI

### Summary

0.20 introduces subflow instance properties. They are exposed as a list of key/value
pairs in the subflow edit dialog.

There is a requirement to be able to generate a richer UI interface for the user
to set these properties.


### Authors

 - @HiroyasuNishiyama

### Details

### UI definition

Next figure shows a process to define new parameter settings UI for a SUBFLOW.

![Subflow-dev-process](images/Subflow-dev-process.png)

**UI Edit Panel** is added to **SUBFLOW Template Edit Panel** of a SUBFLOW.  Simple parameter setting UI for the SUBFLOW can be created using this panel by adding UI items on UI items list.  Created UI can be previewed by clicking **preview** button or selecting preview tab.

Parameter settings UI of a SUBFLOW can be show by double clicking the SUBFLOW (and selecting parameter edit tab) on flow edit pane.  SUBFLOW customization parameters can be set using this UI.

#### SUBFLOW Parameter Representation

Each UI item defined in UI Edit Panel can store the defined value into two kinds of targets: SUBFLOW environment variable and node property within the SUBFLOW.

#### Edit UI Panel

**Edit UI Panel** have **add** button at the bottom.  By clicking this button, new UI item  can be added to UI items list.  UI items are shown from top to bottom in generated parameter settings UI.  Order of UI items can be rearragned by dragging an item in the list.

![UI-edit-panel](images/UI-edit-panel.png)

UI item definition consists of following sub-items:

![UI-element](images/UI-element01.png)

- **name** - name of defined UI item. Used to access input value.

- **label** - label of defined UI item.  HTML tags can be used in label definition.  For internationalization (i18n) of label text, locale of the text can be selected from menu.

  **label** field appearance is as follows:

  ![field-label](images/field-label.png)

- **type** - type of defined item. Currently following types are supported:

  | type              | input item                                          | description                                                  |
  | ----------------- | --------------------------------------------------- | ------------------------------------------------------------ |
  | *simple type*     | - initial value                                     | simple input for basic types supported by TypedInput such as string, number, etc. |
  | **any**           | - selection of candidate types<br />- initial value | set of basic types supported by TypedInput                   |
  | **label**         | -                                                   | only show label                                              |
  | **menu**          | - menu items<br />- initial value                   | select one item from list of menu items                      |
  | **checkbox**      | - label<br />- initial value                        | checkbox to select from items                                |
  | **spinner**       | - initial value                                     | spinner input of numeric value                               |
  | **node**          | - target node                                       | select from set of nodes                                     |
  | **node property** | - target property of a node                         | select from set of node properties                           |
  | **ui_group**      | - group of UI widget                                | select group of Node-RED Dashboard                           |
  | **ui_size**       | - size of UI widget                                 | set size of Node-RED Dashboard UI Widget                     |

  **type** field appearance is as follows:

  ![field-type](images/field-type.png)

- **tgt. type** - type of target to which input value is stored.  Curtly supported tgt. types are as follow:

  | name          | description                             |
  | ------------- | --------------------------------------- |
  | env var       | environment variable                    |
  | node property | property of a specified node in SUBFLOW |

  There are some kind of UI input types of nodes in SUBFLOW that can not be specified using environment variable.  For such input type, we allow directly specifying property of nodes from node settings UI.  

  **tgt. type** field appearance is as follows:

  ![field-tgt](images/field-tgt.png)

- **target** - specification of target for selected **tgt. type**.

  If **tgt. type** is **env var**, **target** field appearance is as follows:

  ![field-target-envvar](images/field-target-envvar.png)

  If **tgt. type** is **node property**, **target** field appearance is as follows:

  ![field-target-target](images/field-target-target.png)

- **type specific inputs** - input items specific to each UI item type.

#### UI Input Elements

This section describes UI definition, UI appearance, and environment variable defined by UI element for each UI input element type.  

##### *Simple Type*

- UI Definition

  ![UI-def-simple](images/UI-def-simple.png)

  - **initial value** accepts a value of type specified by **<type>** field.

- UI Appearance

  ![UI-app-simple](images/UI-app-simple.png)

- Environment Variable

  | name        | value                                                        |
  | ----------- | ------------------------------------------------------------ |
  | *name*      | input value represented by specified type                    |
  | *name*_type | type of input value                                          |
  | *name*_info | JavaScript object that represents input information.  It contains following properties:<br />- name: name of field<br />- label: label of field<br />- value: input value<br />- type: type of UI element<br />- target_type: “env var” or “node property”<br />- target: name or [id, name] |

##### **any**

- UI Definition

  ![UI-def-any](images/UI-def-any0.png)

  - A set of candidate types can be selected by checkbox.
  - Initial value can be defined using **initial value** field.

- UI Appearance

  ![UI-app-any](images/UI-app-any.png)

  - A value can be input using **TypedInput** interface.

- Environment Variable

  same as *simple type*

##### **label**

- UI Definition

  ![UI-def-label](images/UI-def-label.png)

- UI Appearance

  ![UI-app-label](images/UI-app-label.png)

- Environment Variable

  | name        | value                                                        |
  | ----------- | ------------------------------------------------------------ |
  | *name*_type | “label”                                                      |
  | *name*_info | JavaScript object that represents input information.  It contains following properties:<br />- name: name of field<br />- label: label of field<br />- type: “label" |

##### **menu**

- UI Definition

  ![UI-def-menu](images/UI-def-menu.png)

  - menu items are defined by adding *item label* (with locale) and its *value* pair.
  - initial value of menu can be defined using **initial value** menu.

- UI Appearance

  ![UI-app-menu](images/UI-app-menu.png)

- Environment Variable

  | name        | value                                                        |
  | ----------- | ------------------------------------------------------------ |
  | *name*      | *value* of selected item                                     |
  | *name*_type | "menu"                                                         |
  | *name*_info | JavaScript object that represents input information.  It contains following properties:<br />- name: name of field<br />- label: label of field<br />- value: value of selected item<br />- type: “menu"<br />- target_type: “env var”, “node”, or “node property”<br />- target: name or [id, name]<br />- items: list of objects with following properties:<br />  + labels: list of objects with following properties:<br />     * lang: locale of label<br />     * label: label<br />  + value: value of item |

##### **checkbox**

- UI Definition

  ![UI-def-checkbox](images/UI-def-checkbox.png)

  - initial value (**true** or **false**) can be defined by **initial value** field.

- UI Appearance

  ![UI-app-checkbox](images/UI-app-checkbox.png)

  - item label is shown after check box.

- Environment Variable

  | name        | value                                                        |
  | ----------- | ------------------------------------------------------------ |
  | *name*      | **true** or **false**                                        |
  | *name*_type | “checkbox”                                                   |
  | *name*_info | JavaScript object that represents input information.  It contains following properties:<br />- name: name of field<br />- label: label of field<br />- value: **true** or **false**<br />- type: “checkbox"<br />- target_type: “env var” or “node property”<br />- target: name or [id, name] |

##### spinner

- UI Definition

  ![UI-def-spinner](images/UI-def-spinner.png)

  - initial value can be defined by **initial value** field.

- UI Appearance

  ![UI-app-spinner](images/UI-app-spinner.png)

- Environment Variable

  | name        | value                                                        |
  | ----------- | ------------------------------------------------------------ |
  | *name*      | numeric value                                                |
  | *name*_type | “spinner”                                                    |
  | *name*_info | JavaScript object that represents input information.  It contains following properties:<br />- name: name of field<br />- label: label of field<br />- value: numeric value<br />- type: “spinner"<br />- target_type: “env var” or “node property”<br />- target: name or [id, name] |

##### node

- UI Definition

  ![UI-def-node](images/UI-def-node.png)

  - **filter** specifies a regular expression that filters node type.

- UI Appearance

  ![UI-app-node](images/UI-app-node.png)

- environment variable

  | name        | node ID                                                      |
  | ----------- | ------------------------------------------------------------ |
  | *name*_type | "node"                                                  |
  | *name*_info | JavaScript object that represents input information.  It contains following properties:<br />- name: name of field<br />- label: label of field<br />- value: numeric value<br />- type: “node"<br />- target_type: “env var” or “node property”<br />- target: name or [id, name] |

##### node property

*T.B.D.* (will be implemented as an extension of **TypedInput** interface)

## History

  - 2019-02-27 - migrated from Design note wiki
