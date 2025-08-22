## Struktura templatingu w Kibanie
Kibana używa **Mustache** jako systemu szablonów.  
Każda akcja ( "Add to index", "Webhook", "Email") otrzymuje **context**, czyli zestaw danych związanych z uruchomionym alertem.

Zmienne dzielą się na:

**Globalne** - dostępne zawsze (`rule`, `date`, `alert`, `context`).

**Specyficzne dla rule type** -  `context.hits` dla reguły `Elasticsearch query`.

---

## Używane zmienne

### `{{date}}`
- Data uruchomienia akcji.
- Format: ISO8601 (`2025-08-23T11:34:22.123Z`).
- Generowane automatycznie przy triggerze alertu.

### `{{rule.id}}`
- ID reguły, wygenerowane przez Kibana.
- Pomocne do śledzenia źródła alertu.

### `{{rule.name}}`
- Nazwa reguły, ustawiona przy tworzeniu.

### `{{context.hits}}`
- Tablica dokumentów (hity) zwróconych przez zapytanie (`Elasticsearch query rule`).
- Każdy element wygląda jak:
  ```json
  {
    "_index": "example-logs-2025.08.23",
    "_id": "abcd1234",
    "_source": {
      "event_tag": "EVENT_TAG",
      "src_ip": "192.0.2.5",
      "dst_port": 22,
      "msg": "Example message"
    }
  }
  ```

### `{{context.hits.0._source.field}}`
- Dostęp do **pierwszego** dokumentu z wyników (`hits[0]`).
- Przykład:
  - `{{context.hits.0._source.src_ip}}` - `192.0.2.5`
  - `{{context.hits.0._source.msg}}` - `Example message`


---

## Typowy szablon dla Add to index

```json
{
  "@timestamp": "{{date}}",
  "rule": {
    "id": "{{rule.id}}",
    "name": "{{rule.name}}"
  },
  "event_tag": "{{context.hits.0._source.event_tag}}",
  "src_ip": "{{context.hits.0._source.src_ip}}",
  "dst_port": "{{context.hits.0._source.dst_port}}",
  "msg": "{{context.hits.0._source.msg}}"
}
```

---