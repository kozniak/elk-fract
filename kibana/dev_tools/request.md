### Check index
---

GET _cat/indices?v

GET _cat/indices?h=index


### Create new index
---

PUT my-new-index

```http
PUT my-new-index
{
  "mappings": {
    "properties": {
      "timestamp": { "type": "date" },
      "message":   { "type": "text" }
    }
  }
}
```

### Delete index
---

DELETE my-old-index