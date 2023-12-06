# Secure Docker Elastic cluster

An end-to-end fully secure Elasticsearch cluster (of 3 Elasticsearch instances) with Kibana and run by Docker. Using official images. Ever dreamed of the following ?

![Kibana Elasticsearch cluster](./images/kibana_elasticsearch_cluster.png)

First, you will need to raise your host's ulimits for Elasticsearch to be able to handle high I/O :

```console
sudo sysctl -w vm.max_map_count=500000
```

Now, we will generate the certificates for your cluster :

```console
docker-compose -f create-certs.yml run --rm create_certs
```

That's it ! Start the cluster with :

```console
docker-compose up -d
```

Access Kibana through [https://localhost:5601](https://localhost:5601)

> Default username is `elastic` and password is `changeme`

## Users management

To create a new user `ingest` with password `changeme` :

```bash
curl -k -X POST "https://localhost:9200/_security/user/ingest" -H "Content-Type: application/json" -u elastic:changeme -d '{
  "password" : "changeme",
  "full_name" : "Ingest User",
  "roles": [],
  "email" : "ingest@example.com",
  "metadata" : {
    "intelligence" : 7
  }
}'
```

To update a password :

```bash
docker exec -it secure-docker-elastic-cluster-es01-1 bin/elasticsearch-users passwd admin
```

Make it so `ingest` can write data in `*metric*` or `*logs*` indices :

```bash
curl -k -X PUT "https://localhost:9200/_security/role/ingest-role" -H "Content-Type: application/json" -u elastic:changeme -d'
{
  "cluster": ["manage_index_templates", "monitor", "manage_ilm"],
  "indices": [
    {
      "names": [ "*metric*", "*logs*" ],
      "privileges": ["read","write"]
    }
  ]
}'
curl -k -X PUT "https://localhost:9200/_security/user/ingest" -H "Content-Type: application/json" -u elastic:changeme -d '{
  "roles" : ["ingest-role"],
  "full_name" : "Ingest User",
  "email" : "ingest@example.com",
  "metadata" : {
    "intelligence" : 7
  }
}'
```

Test authentication :

```bash
curl -k -u ingest:changeme https://localhost:9200/_cluster/health
```
