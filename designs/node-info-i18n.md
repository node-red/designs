---
state: draft
---

# i18n of Node Info

Current Node-RED editor allow nodes or flows has their own description written in markdown.

Nodes can have internationalized info help text but markdown descriptions are restricted to one language.

This proposal covers how node info (shown on the right pane of Node-RED side bar) can be internationalized. 

The interface for specifying locale should follow design for locale selection interface for subflow enhancements.

### Authors

 - @HiroyasuNishiyamann

### Details

#### Interface

Current locale for info text can be selected by menu at the bottom of description tab of a node.

![Info Text Editor](images/i18n-info-text.png)

#### Internal Representation

In order to keep compatibility with old flow format, `info` property of nodes hold info text for `en-US` locale.

Info texts for other locales are stored in `i18nInfo` property.  The `i18nInfo` property points to an object with `locale` as a key and corresponding text as a value.

```json
{
    "info": "Info text in English",
    "i18nInfo": {
        "ja": "Info text in Japanese",
        "de": "Info text in German"
    }
}
```




## History

  - 2019-08-28 - initial revision
