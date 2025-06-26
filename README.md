Here's a complete and well-structured **technical blog draft** you can publish, titled:

---

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

### 📦 Folder Structure

```bash
elk-stack/
│
├── docker-compose.yml
├── elasticsearch.yml
├── kibana.yml
├── logstash.conf
└── esdata/          # Created automatically on first run
```

---

### 🔐 Elasticsearch Configuration

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
    ports:
      - 9200:9200
    volumes:
      - esdata:/usr/share/elasticsearch/data
      - ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    networks:
      - elk
```

#### `elasticsearch.yml`

```yaml
xpack.security.enabled: true
xpack.security.authc.api_key.enabled: true
```

---

### 🎨 Kibana Configuration

#### `docker-compose.yml`

```yaml
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
    ports:
      - 5601:5601
    volumes:
      - ./kibana.yml:/usr/share/kibana/config/kibana.yml
    networks:
      - elk
```

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

### 🔄 Logstash Configuration

#### `docker-compose.yml`

```yaml
  logstash:
    image: docker.elastic.co/logstash/logstash:8.13.4
    container_name: logstash
    depends_on:
      - elasticsearch
    ports:
      - 5044:5044
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    environment:
      - xpack.monitoring.enabled=true
      - xpack.monitoring.elasticsearch.username=logstash_system
      - xpack.monitoring.elasticsearch.password=logstash123
    networks:
      - elk
```

#### `logstash.conf`

```conf
input {
  beats {
    port => 5044
  }
}

filter {
  # Add parsing/filter logic here (e.g., grok, date)
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

### 🔑 Set System User Passwords

Run these after Elasticsearch is healthy (check with `curl localhost:9200 -u elastic:elastic123`):

```bash
# Reset password for kibana_system
docker exec -it elasticsearch bin/elasticsearch-reset-password -u kibana_system -i

# Reset password for logstash_system
docker exec -it elasticsearch bin/elasticsearch-reset-password -u logstash_system -i
```

Set the passwords to match your `docker-compose` config (e.g., `kibana123`, `logstash123`).

---

### 🚀 Running the Stack

Start the full stack:

```bash
docker-compose down -v    # optional: clean previous data
docker-compose up -d --build
```

✅ You can access:

* **Kibana**: [http://localhost:5601](http://localhost:5601)
* **Elasticsearch**: [http://localhost:9200](http://localhost:9200) (use basic auth)
* **Logstash**: Listens on port `5044` for beats input

---

### 📋 Verification Steps

1. **Elasticsearch**:
   Visit [http://localhost:9200](http://localhost:9200)
   Login with: `elastic / elastic123`

2. **Kibana**:
   Visit [http://localhost:5601](http://localhost:5601)
   Login with: `kibana_system / kibana123`

3. **Logstash**:
   Check logs:

   ```bash
   docker logs -f logstash
   ```

---

### 📘 Notes & Tips

* You **must reset** `kibana_system` and `logstash_system` passwords unless using default bootstrap or auto-generated secrets.
* Set a **strong encryptionKey** for Kibana.
* Don’t use the `elastic` user for services. It’s an admin.
* Always define custom roles/users in production using the Elasticsearch API.

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