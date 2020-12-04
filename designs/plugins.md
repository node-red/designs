---
state: draft
---


# Node-RED Plugins

## Summary

We want to make it easier to extend Node-RED through the use of plugins.

A plugin is an npm module that is loaded by Node-RED and used to provide custom
functionality to either the runtime and/or editor.


## Authors

 - Nick O'Leary

## Details

A lot can already be done using the model we have for nodes; they are installed
as npm modules, loaded into the runtime and the editor. The runtime manages
their lifecycle.

Whilst this works for getting custom JavaScript loaded into the editor, we want
to have a bit more structure in place to help plugins do the right thing.

The main points this design needs to cover are:

 - An editor API that plugins use to register themselves - similar to
   `RED.nodes.registerType` used by nodes. This API would then let the editor
   manage the plugin lifecycle

 - We know there are cases like nrlint where the core linter plugin will want
   to load its own plugins.

 - Loading additional resources - a plugin might not want to provide all of its
   content in the single .html file. We've long thought about having a way for
   nodes to declare additional resources that it will want to load in the editor.
   The same is true for plugins.

 - If a plugin is runtime-only, it should not be required to provide a .js file

 - **Question:** should it be possible to dynamically remove/disable a plugin as you can nodes? That would
   require *all* UI customisations to be undoable.
   If plugins are there in order to provide customisations of the editor, it may not be practical to
   fully support removing them without a reload of the editor.
   This will require more work to figure out whilst implementing some real plugins.


### Plugin registration

```
RED.plugins.register("plugin-identifier", {
    type: "plugin-type",
    onadd: function() {},
    onremove: function() {}

})
```

 - `plugin-identifier`: a unique identifier for the plugin.
 - `type`: an optional field to identify the type of the plugin (see below)
 - `onadd`: called when the plugin is registered
 - `onremove`: called when the plugin is removed due to the module being disabled or uninstalled.

When this register function is called, the following things will happening

 - The plugin is added to the internal plugin registry
 - If provided, the plugin's `onadd` function is called
 - The `"registry:plugin-added"` event is emitted with the name of the plugin as its payload

The `onremove` function is called when the plugin is removed due to the module being
disabled or uninstalled.



### Loading plugins

Plugins are loaded from node modules in the same way as nodes are.

However, there is a decision to be made about whether the same `package.json` structure
should be used, or if it should identify plugins explicitly. For example:

```
"node-red": {
    "plugins": {
        "foo": "foo.js"
    }
}
```

If the plugin only provides editor resources, the entry can point to just an HTML
file.

One advantage of this is it will be ignored by older versions of Node-RED - so installing
a plugin won't break things.

It would also allow the runtime to provide plugin html content to the editor separately
to the node html content. This means plugins can be loaded into the runtime before
it loads the palette of nodes.

This does mean a new admin api will be needed in order to serve the plugins.

### HTTP Admin API

#### `GET /plugins`

 - `Content-Type: application/json` - returns a list of the installed plugins
 - `Content-Type: text/html` - returns the html content of all installed plugins

TODO: need to figure out what other admin endpoints are needed.


### Editor APIs

As mentioned above, `RED.plugins` will be a new module in the Editor. It will expose the functions:

 - `RED.plugins.register(id,definition)` - called by plugins to register themselves
 - `RED.plugins.get(id)` - get a plugin by identifier
 - `RED.plugins.getByType(type)` - get all plugins of a particular type (using the optional `type` field in the plugin definition)

TODO: there may be other APIs to expose

### Editor customisations

For plugins that add to the UI, we need to identify what UI elements they may want
to extend and have a plan in place to have proper APIs in place for them.

 - Adding a custom sidebar - already available
 - Adding custom menu items - the `RED.menu` api needs overhauling to allow easier customisation of the main menu
 - Adding library types - this will be covered in a separate design note on the improved library api
 - Adding custom widgets to the workspace footer - work in progress, need to formalise the existing API


### Example plugin - nrlint

*I am using nrlint as an example to demonstrate how it could work. This is not meant to be the formal design for nrlint's plugin system. All of the names/types/ids I've used here are placeholders*

 - The Flow Linter tool will register a plugin called `nrlint`.
 - It's `onadd` function will do the following:
     - add a custom sidebar to the editor and all the logic to do its work.
     - it will listen for the `registry:plugin-added` event so it knows when other plugins are added.
       When that is called, it will check if the new plugin has a `type` of `nrlint-rule`.
     - It will also call `RED.plugins.getByType("nrlint-rule")` to get a list of existing plugins.
     - For the existing plugins, or newly added ones, it can then do whatever internal
       initialisation/registration it needs of those plugins into the lint tool.





## History

- 2020-12-xx - Initial proposal submitted
