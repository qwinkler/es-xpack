# Elasticsearch in docker

Run elasticsearch cluster in docker with X-Pack features (and free trial!)

# TL;DR

Create dotenv file:
```dotenv
COMPOSE_PROJECT_NAME=es 
CERTS_DIR=/usr/share/elasticsearch/config/certificates 
ELASTIC_PASSWORD=changeme
ELASTIC_VERSION=7.6.1
```

Run Elasticsearch and Kibana:
```bash
# Pull docker images
docker-compose pull
# Generate certificates
docker-compose -f create-certs.yml run --rm create_certs
# Run Elasticsearch with Kibana
docker-compose up -d
```


# Watcher example

```json
{
  "trigger": {
    "schedule": {
      "interval": "30m"
    }
  },
  "input": {
    "http": {
      "request": {
        "scheme": "https",
        "host": "localhost",
        "port": 9200,
        "method": "get",
        "path": "/_cluster/health",
        "params": {},
        "headers": {},
        "auth": {
          "basic": {
            "username": "elastic",
            "password": "changeme"
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.status": {
        "eq": "green"
      }
    }
  },
  "actions": {
    "notify-slack": {
      "slack": {
        "account": "monitoring",
        "message": {
          "from": "watcher",
          "to": ["#example"],
          "attachments": {
            "title": "Errors Not Found",
            "text": "Cluster '{{ctx.payload.cluster_name}}' is in {{ctx.payload.status}} state.",
            "color": "45BF89"
          }
        }
      }
    }
  }
}
```