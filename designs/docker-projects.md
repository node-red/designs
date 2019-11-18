---
state: draft
---

# Docker-enable Projects

## Summary

This design makes Node-RED Projects easier to use with Docker.

In summary, this involves:

1. add a default `Dockerfile` when a Node-RED project is created
2. add a default `settings.js` file to provide a 'production-ready' configuration
   for the project
3. add an `npm start` script to `package.json`

The `Dockerfile` and `settings.js` files will be editable from within the editor.

A new `build` sidebar will be added that can be used to run `docker build` and
see its output. It will also support `docker push` to publish built images to dockerhub.

The project settings view will have a docker section where tags/arch options can
be set.

## Authors

 - @knolleary

## Details

### Enabling this feature

Another option will be added to `editorTheme` to either enable or disable this
feature.

It will be disabled if:
 - projects is not enabled
 - the `docker` cli is not foun

Whether it is enabled by default if the prereqs are met is TBD.

### Dockerfile

The default Dockerfile will be based on the `nodered/node-red` image. This means
the user won't have to add `node-red` in as an explicit dependency of the project.

The user will be able to edit the Dockerfile via the editor.

**Todo: Add default Dockerfile here**

### settings.js

The default `settings.js` file will look like:

```
const path = require("path");

module.exports = {
    uiPort: process.env.PORT || 1880,
    credentialSecret: process.env.NODE_RED_CREDENTIAL_SECRET,
    httpAdminRoot: false,
}
```

This means:
 - the port defaults to 1880 but can be overridden by the `PORT` env var
 - the credential secret can be provided by an env var
 - the admin api (inc editor) is disabled

The user will be able to edit the settings file via the editor.

### Build sidebar

**More design work is needed for the sidebar.**

The build sidebar will allow the user to run a Docker build of their project.
The log output will be available via the existing Event Log.

For example, would it be useful to detect and list the images it has built?

The sidebar would also allow the user to publish a built image to Dockerhub.


The design needs to consider how this sidebar could be repurposed for other build
targets in the future. For example, building an Electron-based application rather
than a Docker image.


## History

 - 2019-11-15 - first pass
