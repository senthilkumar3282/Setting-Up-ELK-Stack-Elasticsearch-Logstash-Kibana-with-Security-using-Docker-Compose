## 🚀 Setting Up ELK Stack (Elasticsearch, Logstash, Kibana) with Security using Docker Compose

**Author**: Senthilkumar Sugumar
**Date**: June 2025
**Tech Stack**: Docker, ELK (v8.13.4), Beats, Security, API Keys

---

### 🧩 Overview

In this post, we’ll walk through setting up a secure ELK stack using Docker Compose. We’ll configure:

* Elasticsearch with security (basic auth + API key)
* Kibana with secure login and encryption keys
* Logstash with monitoring and authenticated output
* Beats support via TCP input on Logstash

By the end, you’ll have a fully working, production-ready ELK stack.

---

### 📁 Project Structure

```
.
├── docker-compose.yml
├── elasticsearch.yml
├── kibana.yml
├── logstash.conf
```

---

### 🐳 1. Start Elasticsearch First

```bash
docker-compose up -d elasticsearch
```

> ⏳ **Wait 10–15 minutes** until Elasticsearch fully initializes.

---

### 🔑 2. Set Passwords for System Users

After Elasticsearch is up, run the following commands to **manually reset passwords**:

```bash
docker exec -it elasticsearch bin/elasticsearch-reset-password -u kibana_system -i
```

```bash
docker exec -it elasticsearch bin/elasticsearch-reset-password -u logstash_system -i
```

> ✅ Set them as:
>
> * `kibana_system` → `kibana123`
> * `logstash_system` → `logstash123`

---

### 🟢 3. Start Kibana and Logstash

Once passwords are set:

```bash
docker-compose up -d kibana
docker-compose up -d logstash
```

---

### ⚙️ Configuration Files

#### `docker-compose.yml`

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.13.4
    container_name: elasticsearch
    environment:
      - node.name=es01
      - cluster.name=elk-cluster
      - discovery.type=single-node
      - ELASTIC_PASSWORD=elastic123
      - xpack.security.enabled=true
      - xpack.security.authc.api_key.enabled=true
      - network.host=0.0.0.0
    volumes:
      - esdata:/usr/share/elasticsearch/data
      - ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - 9200:9200
    networks:
      - elk

  kibana:
    image: docker.elastic.co/kibana/kibana:8.13.4
    container_name: kibana
    depends_on:
      - elasticsearch
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=kibana123
      - xpack.encryptedSavedObjects.encryptionKey=fX!@8J!39Qm<ye3.!{Gp*:ph;8ji)}tu
    volumes:
      - ./kibana.yml:/usr/share/kibana/config/kibana.yml
    ports:
      - 5601:5601
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:8.13.4
    container_name: logstash
    depends_on:
      - elasticsearch
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - 5044:5044
    environment:
      - xpack.monitoring.enabled=true
      - xpack.monitoring.elasticsearch.username=logstash_system
      - xpack.monitoring.elasticsearch.password=logstash123
    networks:
      - elk

volumes:
  esdata:

networks:
  elk:
    driver: bridge
```

---

#### `elasticsearch.yml`

```yaml
xpack.security.enabled: true
xpack.security.authc.api_key.enabled: true
```

---

#### `kibana.yml`

```yaml
server.name: kibana
server.host: "0.0.0.0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
elasticsearch.username: "kibana_system"
elasticsearch.password: "kibana123"
xpack.security.encryptionKey: "fX!@8J!39Qm<ye3.!{Gp*:ph;8ji)}tu"
xpack.reporting.encryptionKey: "fX!@8J!39Qm<ye3.!{Gp*:ph;8ji)}tu"
xpack.security.session.idleTimeout: "1h"
```

---

#### `logstash.conf`

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  # your filters here
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    user => "logstash_system"
    password => "logstash123"
    ssl => false
    index => "logstash-%{+YYYY.MM.dd}"
  }
}
```

---

### ✅ Final Check

* Visit Kibana: [http://localhost:5601](http://localhost:5601)
* Login using:

  * **Username:** `elastic`
  * **Password:** `elastic123`

---

### 📎 Optional: Filebeat Example

Add Filebeat to send logs via `5044` to Logstash (skip for now unless needed).

---

### 📌 Conclusion

You now have a secured ELK stack ready for observability and log ingestion in a Dockerized setup.

---

### 🔗 Share & Explore

**Follow me on LinkedIn**: [linkedin.com/in/senthilkumar-sugumar-11030715](https://www.linkedin.com/in/senthilkumar-sugumar-11030715/)

---
