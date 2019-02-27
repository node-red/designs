# Subflow Node Modules

**State:** draft

## Summary

The goal of this feature is to allow a subflow to be packaged as an installable
npm module and loaded into the runtime.

It will appear in the palette in the same way as any regular node; the end-user
will not need to know the node is implemented as a subflow.

Node-RED Nodegen should be updated to supported creating such a module.

Topics this design must cover:

 - how the runtime registry knows a module provides a subflow type node
 - how the module declares its node dependencies
 - how the node's help template and edit template are provided

## Authors

 - @knolleary

## Details

## History

  - 2019-02-26 - migrated from Design note wiki
