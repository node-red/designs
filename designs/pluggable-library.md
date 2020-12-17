---
state: draft
---

# Library Plugins

## Summary

Building on the Node-RED Plugins design, we can now look at supporting Library Plugins.

A library plugin acts as an alternative source of content for the libraries shown
in the editor.

For example, a library hosted on a remote URL, or an additional library on the local
filesystem.

An example use case is where a team of developers want a shared library of reusable
assets. They have a shared drive mounted on each of their computers (or could be
via a service like Dropbox). They want to be able to access the content of that
shared drive when loading/saving flows and other assets within the editor.

## Authors

 - Nick O'Leary

## Details

A library plugin consists of two parts:

1. a runtime plugin that implements a defined API to act as a library source
2. an optional editor plugin that allows the user to dynamically configure new instances of the plugin

This design is currently focussed on statically adding new library sources via
the settings file. More work is needed to design how a user may add/config new
sources via the editor.


### Runtime plugin

The following is the skeleton of an example runtime plugin

```
module.exports = function(RED) {

    class FileStorePlugin {
        constructor(config) {
            this.config = config;

        }
        async init() {
            console.log("FileStorePlugin.init")

        }
        async getEntry(type,path) {
            console.log("FileStorePlugin.getLibraryEntry",type,path)
            return []
        }
        async saveEntry(type,path,meta,body) {
            console.log("FileStorePlugin.saveLibraryEntry",type,path)

        }
    }

    RED.plugins.registerPlugin("node-red-library-filestore", {
        type: "node-red-library-source",
        class: FileStorePlugin
    })
}
```
1. The plugin defines a class (`FileStorePlugin`) that implements a standard api:
  - `constructor(config)` : passed in the configuration for the plugin instance
  - `init()` : optional async function to perform any initialisation of the plugin.
  - `getEntry(type,path)` : get an entry from the library
  - `saveEntry(type,path,meta,body)` : save an entry to the library
  - `deleteEntry(type,path)` : remove an entry from the library
2. It registers itself as a plugin with a type of `node-red-library-source` and
   provides a reference to its class

*TODO* add more details on `get/save/deleteEntry` functions. Their behaviour should
match the existing file system store we already have... although `deleteEntry` is
a new option that hasn't existed before.

### Editor plugin

*This needs more work to fill out the details*

To allow the user to add and configure new instances of the plugin via the editor,
it needs to provide an editor plugin.

In a similar way to nodes, it provides a list of configuration properties. I don't
know yet if we need to provide more custom edit functionality - or whether auto-generating
a config screen of key/values pairs based on these defaults is sufficient.

This is an area where the design may evolve over future versions.

```
<script type="text/javascript">
    RED.plugins.registerPlugin("node-red-library-filestore", {
        type: "node-red-library-source",
        name: "Local File-System Library",
        icon: "font-awesome/fa-hdd-o",
        defaults: {
            name: { value: "" },
            path: { value: ""}
        }
    })
</script>
```

### HTTP Admin API

The list of configured libraries is added to the `/settings` end point. The default
value is:

```
{
    "libraries": [
        {
            "id": "local",
            "label": "editor:library.types.local",
            "user": false,
        },
        {
            "id": "examples",
            "label": "editor:library.types.examples",
            "user": false,
            "readOnly": true,
            "types": [
                "flows"
            ]
        }
    ]
}
```

This default shows the definition of the built-in libraries - the local lib and
examples lib.

 - `id` : unique identifier for this library.
 - `label`: the label to show for the library. It gets passed through `RED._()` so can be a message catalog reference
 - `user` : whether this is a user-configurable library
 - `readOnly`: whether this is a read-only library.
 - `types`: an array of asset types the library supports.
 - (not shown) `icon`: an optional FA-icon identifier.



*TODO: define the APIs to support adding/configuring new library sources in the editor*

### `@node-red/runtime` API

*TODO: define the APIs to support adding/configuring new library sources in the editor*

### Settings file

A user can define new library sources via their settings file using the new `editorTheme.library`
setting:

```
editorTheme: {
    library: {
        sources: [
            {
                id:   "team-dir",
                type: "node-red-library-filestore",
                name: "team-dir",
                path: "/tmp/nr-lib/"

            }
        ]
    }
}
```

For each source:
 - `id` : unique identifier for the library source instance
 - `type`: the type of library plugin this is an instance of
 - ... anything else is then plugin-specific configuration.

Any source defined in the settings file will get its `user` property set to `false`
as it cannot be reconfigured in the editor.




## History

- 2020-12-xx - Initial proposal submitted
