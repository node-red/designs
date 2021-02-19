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

The following applies to both editor and runtime plugins.

```
RED.plugins.registerPlugin("plugin-identifier", {
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
disabled or uninstalled. *TODO*: not clear if we'll support removing plugins dynamically.



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

### Settings

Plugins provide a way to custom and extended both the Node-RED editor and runtime.

It may not always be a good thing to allow a user to install random plugins. For example,
when Node-RED is embedded into another application, such as the commercial adopters.

In those scenarios, they may want to take advantage of plugins to provide customisations,
but not allow the end user to install/remove those plugins.

For this feature then, there are some use cases to consider:

 - enable/disable loading of plugins entirely
 - disable dynamic loading of plugins - only those loaded at startup are loaded
 - equivalent of `nodeExcludes`/`nodeIncludes` to identify specific modules that
   either may or may not be loaded. This applies beyond just plugins - we've
   needed a way to allow/deny modules by name for some time.

The details of this are being [discussed here](https://github.com/node-red/designs/discussions/40).

### Plugin Settings

A plugin may expect some settings to be provided via `settings.js`. As it stands,
a runtime plugin has full access to the `RED.settings` object so will be able to
access anything in there.

There is a question over how an editor plugin is able to access settings.

We already provide a [mechanism for nodes to do this](https://nodered.org/docs/creating-nodes/node-js#exposing-settings-to-the-editor)
when their runtime code calls `RED.nodes.registerType`.

Plugins should use a similar mechanism for consistency. Although there is a question
of whether it can be improved to provide some more flexibility.

The key requirement is how to tell the runtime to make certain settings available
to the editor.

To help organise the settings file, it is probably better to group plugin settings
together.

Lets say the plugin with id 'my-plugin' has three settings - 'color', 'shape' and 'size'

With the existing node setting scheme, you'd end up with three settings `myPluginColor`,
`myPluginShape` and  `myPluginSize`.

It would be cleaner to have a single top-level property that matches the plugin id
and then group the settings under that:
```
"my-plugin": {
    color: ...
    shape: ...
    size:
}
```

the `settings` property in the plugin definition could then look like:
```
settings: {
    color: { value: "red", exportable: true},
    shape: { value: "square", exportable: true},
    size: { value: "big", exportable: true},
}
```

However, this does require the plugin to pre-declare all possible settings. In the
case of something like `nrlint`, the single top-level setting may used to collect
settings of sub-plugins as well.

So the following would be supported:

```
settings: {
    "*": { exportable : true},
    "color": { exportable: false}
}
```

This will cause all settings under the plugin name to be exported, but also allow
for individual settings to be held back.

Another example:
```
settings: {
    "editor": { exportable : true},
    "runtime": { exportable: false}
}
```

would allow for a clear separation of settings for the runtime and editor.

We should consider updating the node settings naming to allow for this pattern
as well.

### HTTP Admin API

 - `GET /plugins`
   - `Content-Type: application/json` - returns a list of the loaded plugins (see below for format)
   - `Content-Type: text/html` - returns the html content of all loaded plugins
 - `GET /plugins/messages` - returns the message catalogs for all loaded plugins

TODO: need to figure out what other admin endpoints are needed.

### `@node-red/runtime` API

The following functions will be added to the external runtime API:

 - `plugins.getPluginList` - returns a promise that resolves to the list of loaded plugins (see below for format)
 - `plugins.getPluginConfigs` - returns a promise that resolves to the HTML content for loaded plugins
 - `plugins.getPluginCatalogs` - returns a promise that resolves to the message catalogs for all loaded plugins

The following functions will be added to the internal runtime API:

- `RED.plugins.registerPlugin` - register a new runtime plugin
- `RED.plugins.getPlugin` - get a plugin by id
- `RED.plugins.getPluginsByType` - get all plugins of a given type
- `RED.plugins.getPluginList` - get a list of the loaded plugins (see below for format)
- `RED.plugins.getPluginConfigs` - get the message catalogs for all loaded plugins

These functions all pass through to the `@node-red/registry` module...


### `@node-red/registry` API

 - `registerPlugin` - register a new runtime plugin
 - `getPlugin` - get a plugin by id
 - `getPluginsByType` - get all plugins of a given type
 - `getPluginList` - get a list of the loaded plugins (see below for format)
 - `getPluginConfigs` - get the message catalogs for all loaded plugins

### Node/Plugin Runtime API

The `RED` object passed to nodes/plugins will now include a `plugins` object with the functions:
 - `registerPlugin` - register a new runtime plugin
 - `getPlugin` - get a plugin by id
 - `getPluginsByType` - get all plugins of a given type


#### Plugin List format

This follows the same basic structure as the node list format:

```
[
    {
        "id": "test-plugin/test",
        "name": "test",
        "enabled": true,
        "local": true,
        "module": "test-plugin",
        "plugins": [
            {
                "type": "foo",
                "id": "my-test-plugin",
                "module": "test-plugin"
            }
        ],
        "editor": true,
        "runtime": true,
        "version": "1.0.0"
    }
]
```

Each entry in the array represents a separate set of plugins - ie a separate entry in
the `package.json`. The properties are:

 - `id` - the module/set
 - `name` - the set name
 - `enabled` - whether the plugin is enabled (*TODO* this is a hangover from nodes. Not sure if we'll
 support dynamic enable/disable of nodes)
 - `local` - whether the module is installed in `~/.node-red` or not - only local modules can be removed. (*TODO* another hangover
     from nodes)
 - `module` - the module name
 - `plugins` - an array of the plugins provided by this set
 - `editor` - boolean flag to say if this is an editor plugin (ie, provides an html file)
 - `runtime` - boolean flag to say if this is a runtime plugin (ie, provides a js file)
 - `version` - npm module version

### Editor APIs

`RED.plugins` will be a new module in the Editor. It will expose the functions:

 - `RED.plugins.registerPlugin(id,definition)` - called by plugins to register themselves
 - `RED.plugins.getPlugins(id)` - get a plugin by identifier
 - `RED.plugins.getPluginsByType(type)` - get all plugins of a particular type (using the optional `type` field in the plugin definition)

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
