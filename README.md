# Node-RED Project Designs

This repository is used to track proposals for new features within Node-RED. It
covers both the core Node-RED and any of the assets the project maintains.

The full contribution process for the project is documented [here](https://nodered.org/about/contribute/).

All feature requests or ideas **must** be discussed on the [Node-RED forum](https://discourse.nodered.org)
or [Slack](https://nodered.org/slack) to check it fits with the project's plans.

If the discussion results in an agreed plan to create a design note, a pull
request should be opened in this repository to add a new design note to the `designs`
folder.

The design note *must* use the [proposal-template](proposal-template.md) provided.

### Design proposals

#### Draft

All design proposals start in the `draft` state. This means they are still under
discussion and review.

At this stage, the proposal should set out the high-level goals of the feature
with enough detail to review the intent and direction of the feature.

 - [adminAuth User Management](designs/adminAuth-user-management)
 - [Function Library Node](designs/function-library-node)
 - [Subflow Node Modules](designs/subflow-node-modules)

#### In-progress

Through consensus, the design can moved to the `in-progress` state. This is
for designs that are actively being worked on beyond the high-level detail.

 - [Dashboard Layout Tool](designs/dashboard-layout-tool/dashboard-layout-tool)
 - [Environment Variables](designs/env-vars/env-vars)

#### Complete

Once a design has been implemented it should be updated to the `complete` state.
The design should be updated to include references to where it has been implemented -
such as the release it is included in.

 - *none*
