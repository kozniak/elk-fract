### Check index
---

```http
GET _cat/indices?v
```

```http
GET _cat/indices?h=index
```

```http
GET <index>/_count
```

```http
GET <index>/_search
{
  "size": 5,
  "sort": [{ "@timestamp": "desc" }]
}
```

### Create new index
---

```http
PUT <my-new-index>
```

```http
PUT <my-new-index>
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

```http
DELETE my-old-index
```

### Delete doc from index
---

```http
POST <index>/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```