---
state: draft
---

# metrics usage

## Summary

Metrics logs are useful for gathering and estimating execution statistics of Node-RED.  This design note summarizes metrics usage scenarios and expected enhancements on them.


## Authors

 - @HiroyasuNishiyama

## Details

### Supported Metrics Logs

Currently, Node-RED runtime and its core nodes support following metrics logs.

|      | Runtim/Node         | Name                                                   |
| ---- | ------------------- | ------------------------------------------------------ |
| 1    | Runtime (all nodes) | `send`, `receive`                                      |
| 2    | `Function` node     | `duration`                                             |
| 3    | `Httprequest` node  | `duration.millis`, `size.bytes`                        |
| 4    | `Http in` node      | `response.time.millis`, `response.content-length.size` |

### Usage Scenarios and Enhancements

Following table lists possible usage scenarios and enhancements of metrics logs:

|      | Purpose                                       | Used Metrics                                                 | Mrtrics Usage                                                | Desc.                                                        |
| ---- | --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | Capture Elapsed Time                          | `send`, `receive`                                            | *<elapsed time>*=*<time of recv>*-*<time of send>*           | \- current implementation (1.0-) can not be used for this purpose because of async message send [(Issue #2444)](https://github.com/node-red/node-red/issues/2444) <br />- processing between `recv` and `send` must be synchronous or new metrics that logs transfer of control is required |
| 2    | Trace Message Processing                      | *send*, *receive*, **done**                                  | use message ID to create directed graph to capture execution path | - current implementation do not record end of message processing ([Issue #2446])[https://github.com/node-red/node-red/issues/2446],<br />\- `send` and *done* must contain ID of contributing input messages |
| 3    | Statistics on Incoming/Outgoing  HTTP request | - `duration.millis`, `size.bytes`<br />- `response.time.millis`, `response.content-length.size` | use metrics logs to know statistics on HTTP requests         |                                                              |
| 4    | Processing time of `Function` node            | `duration`                                                   | use metrics log to know processing time of `Function` node   | - similar problem to #1 can occur if function body is not synchronous. |



## History

 - 2020-02-09 - initial note

