Wazuh Docker Setup Guide
Pre-Setup: Steps Required Before Running Docker Compose
1. Create the Azure Credentials Folder
After pulling the repository, manually create the following folder structure:

wazuh-wodles/
└── credentials/
    └── log_analytics_and_graph
Instructions:

mkdir -p wazuh-wodles/credentials
touch wazuh-wodles/credentials/log_analytics_and_graph
Note: The file log_analytics_and_graph contains the Azure credentials used for Log Analytics.

Post-Setup: Steps Required After Running Docker Compose
2. Create the AWS Credentials Folder
After running docker compose up, a new folder will be created inside wazuh-config/: .aws . create manually its files inside it:

wazuh-config/
└── .aws/
    ├── config
    └── credentials
Instructions:

touch wazuh-config/.aws/config
touch wazuh-config/.aws/credentials
File Contents
wazuh-config/.aws/config
This file defines the AWS profile and region:

[default]
region = <your-aws-region>
wazuh-config/.aws/credentials
This file contains the AWS profile credentials:

[default]
aws_access_key_id = <your-access-key-id>
aws_secret_access_key = <your-secret-access-key>
Note: after setting up aws credentials, restart docker compose, so it can found aws credentials and start monitoring it.

Full Folder Structure Summary
.
|  docker-compose.yml
|  README.md
├── wazuh-wodles/                     # Created manually before docker compose up
│   └── credentials/
│       └── log_analytics_and_graph   # Azure credentials file
│
└── wazuh-config/                     # Created after docker compose up
    └── .aws/                         # Created manually after docker compose up
        ├── config                    # AWS profile and region
        └── credentials               # AWS profile credentials
Important Notes
All folder and file names are case-sensitive and must match exactly as shown above.
Do not rename any of the files or folders listed in this guide.


docker compose is the follozing : 

```yaml 
# Wazuh 4.14.0 — Single Node Stack
# Manager + Indexer + Dashboard
# Includes Azure and AWS credential mounts

services:

  wazuh.manager:
    image: wazuh/wazuh-manager:4.14.0
    hostname: wazuh.manager
    container_name: wazuh-manager
    restart: always
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 655360
        hard: 655360
    ports:
      - "1514:1514"
      - "1515:1515"
      - "514:514/udp"
      - "55000:55000"
    environment:
      - INDEXER_URL=https://wazuh.indexer:9200
      - INDEXER_USERNAME=admin
      - INDEXER_PASSWORD=${INDEXER_PASSWORD}
      - FILEBEAT_SSL_VERIFICATION_MODE=full
      - SSL_CERTIFICATE_AUTHORITIES=/etc/ssl/root-ca.pem
      - SSL_CERTIFICATE=/etc/ssl/filebeat.pem
      - SSL_KEY=/etc/ssl/filebeat.key
      - API_USERNAME=wazuh-wui
      - API_PASSWORD=${API_PASSWORD}
    volumes:
      # Wazuh internals
      - ./wazuh-config:/var/ossec/etc
      - wazuh_api_configuration:/var/ossec/api/configuration
      - wazuh_logs:/var/ossec/logs
      - wazuh_queue:/var/ossec/queue
      - wazuh_var_multigroups:/var/ossec/var/multigroups
      - wazuh_active_response:/var/ossec/active-response/bin
      - wazuh_agentless:/var/ossec/agentless
      - filebeat_etc:/etc/filebeat
      - ./ingest_pipeline:/usr/share/filebeat/module/wazuh/alerts/ingest/
      - filebeat_var:/var/lib/filebeat
      # TLS certs for Filebeat → Indexer
      - ./wazuh-certificates/root-ca.pem:/etc/ssl/root-ca.pem:ro
      - ./wazuh-certificates/wazuh.manager.pem:/etc/ssl/filebeat.pem:ro
      - ./wazuh-certificates/wazuh.manager-key.pem:/etc/ssl/filebeat.key:ro
      # Azure credentials
      - ./wazuh-wodles/credentials:/var/ossec/wodles/credentials
      # AWS credentials
      - ./wazuh-config/.aws:/root/.aws
    networks:
      - wazuh-net
    depends_on:
      wazuh.indexer:
        condition: service_healthy

  wazuh.indexer:
    image: wazuh/wazuh-indexer:4.14.0
    hostname: wazuh.indexer
    container_name: wazuh-indexer
    restart: always
    ports:
      - "9200:9200"
    environment:
      - OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g
      - bootstrap.memory_lock=true
      - INDEXER_PASSWORD=${INDEXER_PASSWORD}
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - wazuh_indexer_data:/var/lib/wazuh-indexer
      - ./wazuh-certificates/root-ca.pem:/usr/share/wazuh-indexer/config/certs/root-ca.pem:ro
      - ./wazuh-certificates/wazuh.indexer.pem:/usr/share/wazuh-indexer/config/certs/indexer.pem:ro
      - ./wazuh-certificates/wazuh.indexer-key.pem:/usr/share/wazuh-indexer/config/certs/indexer-key.pem:ro
      - ./wazuh-certificates/admin.pem:/usr/share/wazuh-indexer/config/certs/admin.pem:ro
      - ./wazuh-certificates/admin-key.pem:/usr/share/wazuh-indexer/config/certs/admin-key.pem:ro
    networks:
      - wazuh-net
    healthcheck:
      test: ["CMD-SHELL", "curl -sku admin:${INDEXER_PASSWORD} https://localhost:9200/_cluster/health | grep -q 'green\\|yellow'"]
      interval: 30s
      timeout: 15s
      retries: 20
      start_period: 60s

  # wazuh.dashboard:
  #   image: wazuh/wazuh-dashboard:4.14.0
  #   hostname: wazuh.dashboard
  #   container_name: wazuh-dashboard
  #   restart: always
  #   ports:
  #     - "443:5601"
  #   environment:
  #     - INDEXER_USERNAME=admin
  #     - INDEXER_PASSWORD=${INDEXER_PASSWORD}
  #     - WAZUH_API_URL=https://wazuh.manager
  #     - DASHBOARD_USERNAME=kibanaserver
  #     - DASHBOARD_PASSWORD=${DASHBOARD_PASSWORD}
  #     - API_USERNAME=wazuh-wui
  #     - API_PASSWORD=${API_PASSWORD}
  #     - SERVER_SSL_ENABLED=true
  #     - SERVER_SSL_CERTIFICATE=/usr/share/wazuh-dashboard/config/certs/dashboard.pem
  #     - SERVER_SSL_KEY=/usr/share/wazuh-dashboard/config/certs/dashboard-key.pem
  #     - OPENSEARCH_SSL_CERTIFICATE_AUTHORITIES=/usr/share/wazuh-dashboard/config/certs/root-ca.pem
  #   volumes:
  #     - ./wazuh-certificates/wazuh.dashboard.pem:/usr/share/wazuh-dashboard/config/certs/dashboard.pem:ro
  #     - ./wazuh-certificates/wazuh.dashboard-key.pem:/usr/share/wazuh-dashboard/config/certs/dashboard-key.pem:ro
  #     - ./wazuh-certificates/root-ca.pem:/usr/share/wazuh-dashboard/config/certs/root-ca.pem:ro
  #     - wazuh_dashboard_custom:/usr/share/wazuh-dashboard/plugins/wazuh/public/assets/custom
  #   networks:
  #     - wazuh-net
  #   depends_on:
  #     wazuh.indexer:
  #       condition: service_healthy

networks:
  wazuh-net:
    driver: bridge

volumes:
  wazuh_api_configuration:
  wazuh_logs:
  wazuh_queue:
  wazuh_var_multigroups:
  wazuh_active_response:
  wazuh_agentless:
  filebeat_etc:
  filebeat_var:
  wazuh_indexer_data:
  wazuh_dashboard_custom:
```
