---
state: draft | in-progress | complete
---

# Add Health and Status Endpoints to Admin API

## Summary

In addition to the available methods in the Admin API laid out [here](https://nodered.org/docs/api/admin/methods/), this document proposes the creation of `health` and `status` endpoints. These endpoints would be useful for application monitoring but could find broader use as well.

While users can add this functionality manully to their own instances of Node-RED, this feature set feels sufficiently primitive and general that it would be useful to expose it as part of the Admin API.

## Authors

- [@szep](https://discourse.nodered.org/u/szep)

## Details

Two endpoints would be added to the Admin API:

1. `GET /health` -- on success returns status code 200 and payload `{"status": "ok"}`
   - Likely don't need to discuss what a fail condition would be since this endpoint is designed to indicate simply that Node-RED is up and running
1. `GET /status` -- on success returns status code 200 and payload:

```JSON
{
    "statuses": [
        <STATUS OBJECTS IDENTICAL TO
         THOSE FROM STATUS NODE>
    ]
}
```

## History

- 2021-03-04 - Initial proposal submitted
