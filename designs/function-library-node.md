# Function Library node

**State:** draft

## Summary

We have seen a growing number of users who are inserting JavaScript into global
context so it can be readily reused amongst the Function nodes in a flow. This
is not a great developer experience and something we could improve.

The basic concept of this feature is to introduce a new Function Library
configuration node (from here on referred to as `func-lib` to save typing...
not necessarily the final name). This would be a node that can hold common code
accessible to regular Function nodes.

## Authors

 - @knolleary

## Details

Some initial thoughts:

 - each func-lib node contains some code. It should be structured as a node
   module - so it declares the functions/members it exports. It has a mandatory
   name field.
 - a function node can then `require` the module by its name - just as you would
   in node.js code. Whether this is done by introducing the `require` keyword
   needs thinking about. Maybe something like `node.require()` to reduce confusion
   over the top-level `require`.
 - this will mean the function has an *implicit* dependency on the func-lib config
   node - not an *explicit* dependency (unless we try to code-parse to spot the
   require statements). All of the func-lib config nodes will then appear as
   unused and trigger the associated warnings on deploy. This feature could
   introduce a new flag on config nodes that identifies them as 'userless' - that
   it is okay to not have any nodes explicitly depend on it.
 - however, not knowing the explicit dependency will mean exporting the function
   node won't include the func-lib nodes it needs... so maybe the function node
   does need to have someone to declare its dependencies...
 - the expanded JS editor introduced in 0.19 could be extended to a more useful
   IDE; with a list of all Function and func-lib nodes that can be used to
   quickly switch between them whilst editing. This would remove the need to
   close the dialog of one Function node and open another - reducing the
   disruption when trying edit multiple function nodes together.

## History

  - 2019-02-26 - migrated from Design note wiki
