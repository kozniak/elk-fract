# Kibana Alerting – Add to Index: Skąd się biorą zmienne?

Ten plik opisuje dostępne zmienne w **Kibana Alerting → Actions → Add to index**  
oraz dlaczego w przykładach używamy `{{date}}`, `{{rule.id}}`, `{{context.hits.0._source.*}}`.

---

## 1. Struktura templatingu w Kibanie
Kibana używa **Mustache** jako systemu szablonów.  
Każda akcja (np. "Add to index", "Webhook", "Email") otrzymuje **context**, czyli zestaw danych związanych z uruchomionym alertem.

Zmienne dzielą się na:

- **Globalne** – dostępne zawsze (`rule`, `date`, `alert`, `context`).
- **Specyficzne dla rule type** – np. `context.hits` dla reguły `Elasticsearch query`.

---

## 2. Najczęściej używane zmienne

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
  - `{{context.hits.0._source.src_ip}}` → `192.0.2.5`
  - `{{context.hits.0._source.msg}}` → `Example message`

### Iteracja po wszystkich wynikach
- Mustache pozwala pisać:
  ```mustache
  {{#context.hits}}
  {{_source.src_ip}}
  {{/context.hits}}
  ```
- Wynik: wszystkie adresy IP wklejone w jeden string.  
- **Problem**: akcja „Add to index” oczekuje pojedynczego dokumentu, więc iteracja jest zwykle bezużyteczna.

---

## 3. Typowy szablon dla Add to index

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

## 4. Kiedy używać iteracji (`#context.hits`)?

- Do wysyłania maili, Slacka, webhooków – tam można wstawić listę.
- **Nie** przy „Add to index”, bo ta akcja tworzy **jeden dokument na trigger**.

---

## 5. Źródła dokumentacji

- [Kibana Alerting Action Variables](https://www.elastic.co/guide/en/kibana/current/alert-action-variables.html)  
- [Elasticsearch query rule](https://www.elastic.co/guide/en/kibana/current/rule-type-es-query.html)  
- [Mustache templating](https://mustache.github.io/mustache.5.html)

---

## TL;DR

- `{{date}}`, `{{rule.id}}`, `{{rule.name}}` – globalne zmienne Kibany.  
- `{{context.hits}}` – tablica dokumentów z query.  
- Dostęp do pól dokumentu → `{{context.hits.0._source.field}}`.  
- `Add to index` → zawsze jeden dokument na akcję.  
