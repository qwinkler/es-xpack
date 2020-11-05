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

# Complete example

```bash
(
  apt-get update && apt-get install -y git
  curl https://get.docker.com | bash
  curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  chmod +x /usr/local/bin/docker-compose
  ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
)
git clone https://github.com/qwinkler/es-xpack.git
cd es-xpack
cat <<EOT >> .env
COMPOSE_PROJECT_NAME=es 
CERTS_DIR=/usr/share/elasticsearch/config/certificates 
ELASTIC_PASSWORD=changeme
ELASTIC_VERSION=7.6.1
EOT
docker-compose pull
docker-compose -f create-certs.yml run --rm create_certs
docker-compose up -d
```

# Watcher

Add the webhook url to the keystore:  
```bash
echo "https://hooks.slack.com/services/xxx/yyy/zzz" | bin/elasticsearch-keystore add -f --stdin xpack.notification.slack.account.default.secure_url
```

## Example

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
        "account": "default",
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
