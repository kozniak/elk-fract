This pipeline monitors an Elasticsearch index and sends new documents to a webhook.  
Each document, after being sent, is updated in Elasticsearch with the fields `webhook_sent` and `webhook_sent_at` to avoid duplicates.

---

### Instalation 

1. Add `send_alert.conf` to `/etc/logstash/conf.d`
2. Add the three environment variables to `/etc/default/logstash`
