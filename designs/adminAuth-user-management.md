---
state: draft
---

# adminAuth User Management

## Summary

We need to make it much easier to secure Node-RED for first-time users. Today,
a user must hand-edit their settings file to manage the users.

This design note considers proposals for how to achieve this.

This is a very early stage proposal - it needs a lot more work to turn into a
proper proposal.


## Authors

 - @knolleary

## Details

We could provide an `adminAuth` implementation that is trivial to enable, which
uses an external file to maintain user information in. Once it is in a known
external file, it could become writable by the runtime - allowing for some
level of user-management UX in the editor.

This would be a feature than can be turned on/off for the OEM users who don't
want this feature.

It could also be possible to manage the users from the command-line. There are a
couple possible approaches:

 - What if the `node-red` command did more than just run node-red. With the right
   set of arguments to could be used as a cli tool to manage users.

 - `node-red-admin` already exists as a remote client for the admin api. If we
   were planning to add elements in the UI, they must come with additional admin
   api end points - so `node-red-admin` could also be used here. However, no-one
   installs `node-red-admin`. What if `node-red-admin` was installed as a dependency
    of node-red?

## History

 - 2019-02-26 - migrated from Design note wiki
