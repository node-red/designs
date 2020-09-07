---
state: draft
---


# Ability for Node Red projects to merge git branches locally

## Summary

At present, Node Red Editor provides a fantastic UI to perform the following 
git operations:
- git add .
- git commit -m "<commitmsg>"
- git push
- git checkout -b <newbranch>
- git pull
The Editor also provides a fine UI that shows changes for each node.

What it is lacking is the following feature:
- git merge "otherbranch"

### The problem
When multiple developers are working on the same NR project, due to the way 
flow.json is written by NR runtime, merge conflicts are very common. The issue 
is not the fact that they are common, rather that they are difficult to resolve 
when the underlying file is a json file. Git does not understand the object 
nature of the flow.json and treats the file as a line delimited text.

### Solution options
This can be solved in two ways:
1. Push and Merge on Remote: Use a custom git merge driver on the SCM/VCS to 
resolve conflicts
2. Merge locally and then Push: Use NR Editor (that may use a custom git driver) 
to resolve conflices

At the time of writing this (Sept 2020) Option 1 is not feasible. This is because 
most if not all SCM/VCS do not support use of custom git merge driver.

This leaves us with **option 2** as the viable solution

## Authors

 - @mannharleen
 - Need help!

## Details
Option 2 would introduce the git merge feature within NR via the following changes:
1. Editor needs to provide an option to perform git merge and resolve conflicts
2. Runtime should be able to execute the underlying git commands

### UI Mockups
TODO

### Admin API changes
TODO

## History

This should be a list of major milestones in the life of the proposal. For example:

- 2020-09-07 - Initial proposal submitted
