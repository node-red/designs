# Node-RED Project Designs

This repository is used to track proposals for new features within Node-RED. It
covers both the core Node-RED and any of the assets the project maintains.

The full contribution process for the project is documented [here](https://nodered.org/about/contribute/).

All feature requests or ideas **must** be discussed on the [Node-RED forum](https://discourse.nodered.org)
or [Slack](https://nodered.org/slack) to check it fits with the project's plans.

If the discussion results in an agreed plan to create a design note, a pull
request should be opened in this repository to add a new design note to the `designs`
folder.

If the design requires images, create a subfolder for the proposal.

The design note *must* use the [proposal-template](proposal-template.md) provided.

### Design proposals

#### Draft

All design proposals start in the `draft` state. This means they are still under
discussion and review.

At this stage, the proposal should set out the high-level goals of the feature
with enough detail to review the intent and direction of the feature.

 - [adminAuth User Management](designs/adminAuth-user-management.md)
 - [Function Library Node](designs/function-library-node.md)
 - [Subflow Node Modules](designs/subflow-node-modules.md)
 - [Runnable Projects](designs/runnable-projects.md)
 - [Dynamic MQTT Node](designs/dynamic-mqtt-node.md)
 - [Exportable Subflow](designs/exportable-subflow/README.md)
 - [Node Timeout API](designs/timeout-api.md)
 - [Overwrite Values in settings.js](designs/overwrite-settings.md)
 - [Add Health and Status Endpoints to Admin API](designs/admin-api-health-status.md)

#### In-progress

Through consensus, the design can moved to the `in-progress` state. This is
for designs that are actively being worked on beyond the high-level detail.

 - [Subflow Property UI](designs/subflow-property-ui)
 - [Dashboard Layout Tool](designs/dashboard-layout-tool)
 - [Flow Manipulation API](designs/flow-manipulation-api)
 - [Function Node Lifecycle Model](designs/function-node-lifecycle/README.md)

#### Complete

Once a design has been implemented it should be updated to the `complete` state.
The design should be updated to include references to where it has been implemented -
such as the release it is included in.

 - [Node Messaging API](designs/node-messaging-api.md)
 - [Environment Variables](designs/env-vars)
 - [Admin API Authentication](designs/admin-api-authentication.md)
