# Proccess AWS ALB Access logs to Opensearch

This README provides instructions for setting up an OpenSearch cluster with Logstash and OpenSearch Dashboards using Docker Compose.

## Prerequisites

- Docker
- Docker Compose

## Getting Started

1. Clone this repository or copy the `docker-compose.yml` file to your local machine.

2. Optionally, if you need to ingest data from an AWS S3 bucket, create an AWS credentials file (`~/.aws/credentials`) with your access key and secret key:

```
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
```

3. Run the following command to start the containers:

```
docker-compose up -d
```

This will start the following services:

- **opensearch-node**: The OpenSearch node that forms the cluster.
- **logstash**: The Logstash instance for ingesting data from an AWS S3 bucket (if configured).
- **opensearch-dashboards**: The OpenSearch Dashboards for visualizing and analyzing data.
- **healthcheck**: A simple container that keeps the Docker Compose instance running.

## Configuration

The `docker-compose.yml` file contains the following configuration:

### OpenSearch Node

- **image**: `opensearchproject/opensearch:latest`
- **environment**:
  - `cluster.name`: The name of the OpenSearch cluster.
  - `node.name`: The name of the OpenSearch node.
  - `discovery.seed_hosts`: The seed hosts for discovery.
  - `cluster.initial_cluster_manager_nodes`: The initial cluster manager nodes.
  - `bootstrap.memory_lock`: Enables memory locking for better performance.
  - `OPENSEARCH_JAVA_OPTS`: Sets the JVM heap size.
  - `DISABLE_INSTALL_DEMO_CONFIG`: Disables the installation of demo configurations.
  - `DISABLE_SECURITY_PLUGIN`: Disables the security plugin (for simplicity).
- **ports**: Exposes ports 9200 (for OpenSearch) and 9600 (for performance analyzer).
- **healthcheck**: Checks the health of the OpenSearch node.

### Logstash

- **build**: Builds the Logstash image from the `Dockerfile-logstash.yml`.
- **volumes**:
  - Mounts the AWS credentials file (`~/.aws/credentials`).
  - Mounts the Logstash configuration files (`logstash.conf` and `logstash.yml`).
- **environment**:
  - `OPENSEARCH_HOST`: The hostname of the OpenSearch node.
  - `OPENSEARCH_PORT`: The port of the OpenSearch node.
  - `BUCKET_NAME`: The name of the AWS S3 bucket to ingest data from.
  - `BUCKET_PREFIX`: The prefix of the AWS S3 bucket to ingest data from.
  - `BUCKET_REGION`: The AWS region of the S3 bucket.
  - `AWS_SHARED_CREDENTIALS_FILE`: The path to the AWS credentials file.
- **depends_on**: Ensures that the Logstash container starts after the OpenSearch node is healthy.

### OpenSearch Dashboards

- **image**: `opensearchproject/opensearch-dashboards:latest`
- **ports**: Exposes port 5601 for accessing the OpenSearch Dashboards.
- **environment**:
  - `OPENSEARCH_HOSTS`: The hostname and port of the OpenSearch node.
  - `DISABLE_SECURITY_DASHBOARDS_PLUGIN`: Disables the security plugin for the OpenSearch Dashboards.
- **depends_on**: Ensures that the OpenSearch Dashboards container starts after the OpenSearch node is healthy.
- **healthcheck**: Checks the health of the OpenSearch Dashboards.

### Healthcheck

A simple container that keeps the Docker Compose instance running by executing a sleep command in an infinite loop.

## Accessing OpenSearch and OpenSearch Dashboards

After starting the containers, you can access the following services:

- **OpenSearch**: `http://localhost:9200`
- **OpenSearch Dashboards**: `http://localhost:5601`

## Cleanup

To stop and remove the containers, run the following command:

```
docker-compose down
```

This will stop and remove all the containers created by the `docker-compose.yml` file.
