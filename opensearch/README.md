# Opensearch in docker

Run opensearch cluster in docker with custom certificates.

# TL;DR

Run Opensearch and Opensearch Dashboards:
```bash
# Pull docker images
docker-compose pull
# Generate certificates
bash generate-certs.sh
# Run Opensearch and Opensearch Dashboards
docker-compose up -d
# Wait for 30s or so and enable security plugin
docker-compose exec os01 bash -c "chmod +x plugins/opensearch-security/tools/securityadmin.sh && bash plugins/opensearch-security/tools/securityadmin.sh -cd plugins/opensearch-security/securityconfig -icl -nhnv -cacert config/certificates/ca/ca.pem -cert config/certificates/ca/admin.pem -key config/certificates/ca/admin.key -h localhost"
```

> Default username is admin and password is admin

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
cd es-xpack/opensearch
cat <<EOT >> .env
COMPOSE_PROJECT_NAME=os
OPENSEARCH_VERSION=1.3.0
EOT
docker-compose pull
bash generate-certs.sh
docker-compose up -d
sleep 30s
docker-compose exec os01 bash -c "chmod +x plugins/opensearch-security/tools/securityadmin.sh && bash plugins/opensearch-security/tools/securityadmin.sh -cd plugins/opensearch-security/securityconfig -icl -nhnv -cacert config/certificates/ca/ca.pem -cert config/certificates/ca/admin.pem -key config/certificates/ca/admin.key -h localhost"
```
