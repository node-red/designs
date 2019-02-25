# Node-RED Project Designs

This repository is used to track proposals for new features within Node-RED. It
covers both the core Node-RED and any of the assets the project maintains.

In the first instance, all feature requests or ideas **must** be discussed on the
[Node-RED forum](https://discourse.nodered.org) or [Slack](https://nodered.org/slack)
to check it fits with the project's plans. This discussion should result in an
agreed next step to either:

 - proceed straight to a pull-request
 - submit a design proposal.

### Simple enhancements

Simple enhancements to existing nodes, or minor incremental changes will likely
progress straight to a pull-request.

### Design proposals

#### 0-initial

If the enhancement requires further design work before it can be agreed to, a design
proposal should be filled in and a pull request opened to add it to the `0-initial`
folder.

At this stage, the proposal should set out the high-level goals of the feature
with enough detail to review the intent and direction of the feature.

#### 1-in-progress

Through consensus, the design can moved to the `1-in-progress` folder. This is
for designs that are actively being worked on.

If an in-progress design stalls, it should be moved back to the `0-initial` folder.


#### 2-complete

Once a design has been implemented it can be moved to the `2-complete` folder. The
design should be updated to include references to where it has been implemented -
such as the release it is included in.

#### Archive

If a design reaches a point where the decision is to abandon it, it can be moved
to the `archive` folder.


### Creating a design proposal

Details of how to create a design proposal are provided in the [proposal-template](proposal-template.md).
