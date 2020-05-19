---
state: draft
---

# Standalone Node-RED

## Summary

The standalone Node-RED is a desktop version of Node-RED which includes Node-RED, Node.js, nodes, and browser engine using Chromium.

## Authors

 - @kazuhitoyokoi

## Use Cases

  - Easy installation

    With the Node-RED installer, it will be easy for users to set up the Node-RED environment.
    Especially, the targets are users who have no CLI skills to install Node-RED using commands like `npm install`.

  - Offline installation

    It is suitable for offline use cases because it contains all necessary components to execute Node-RED.

  - Integrated Development Environment (IDE)

    The standalone Node-RED behaves as IDE application like VS Code or Eclipse.
    Therefore, users may want to switch multiple projects for various environments on their single computer.
    Additionally, they need git functionality as default to proceed with team development or backup their flows.

## Details

### Flow editor window
  The window is used for the Node-RED flow editor. After executing the standalone Node-RED, the window will be opened automatically.

  - Dialog when closing window

    When closing the window without saving the flow, the window show dialog to select "save & exit" or "cancel".
    It is implemented using electron dialog or Node-RED notify UI.
    
  - Debug window, event log window

    To check the debug tab and event log in another window, users can open these UIs in a single window.
    (To realize this functionality, The event log UI in Node-RED core requires ”Open in new window” button which is same as debug tab)

### Runtime
  The Node-RED runtime will be executed when the standard Node-RED starts.

  - Admin API

    The URL to connect between window and runtime uses a random path and fixed port number because the flow editor needs to be prohibited from other users and allow access REST API endpoints using http-in nodes which require the fixed port number. The following URL is an example of the admin API which window accesses.

    http://localhost:1880/< random path >

  - settings.js file

    The standalone Node-RED has the default `settings.js` file. When there is no `settings.js` file under `~/.node-red/` directory, the standalone Node-RED saves the default `settings.js` file to load the custom settings in the following table.

    | key                          | value |
    | ---------------------------- | ----- |
    | editorTheme.projects.enabled | true  |

    If `~./node-red/` directory already has `settings.js` file which was created by common Node-RED, the standalone Node-RED uses it without overwriting the file to avoid changing the current environment.

### Logging
  Users can see log data in real-time on the event log window. On the other hand, users can also see historical log data from log files.
  If the `electron-log` module is used to implement logging functionality, the following paths are default file paths to store log data.

  - Windows: C:\Users\\< User name >\AppData\Roaming\node-red\logs\file.log
  - macOS: ~/Library/Logs/node-red/file.log
  - Linux: ~/.config/node-red/logs/file.log

### Building binaries
  `npm run build` command builds binaries for the following OS environments.

  - Windows (msi)
  - macOS (dmg)
  - Linux (tar.gz)

## History

  - 2019-05-19 - Initial proposal
