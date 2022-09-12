## Step 1 Prerequisite (Do these steps in all node)

```
yum install wget
```

> Stop & Disable Firewall by using these command
```
systemctl stop firewalld
systemctl disable firewalld
```

## Step 2 Install Elasticsearch on node1 node2 and node3
> Download Elasticssearch
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.0-x86_64.rpm
```
> Install Elasticsearch

```
rpm –install elasticsearch-8.4.0-x86_64.rpm
```

> Configure Elasticsearch service to start automatically
```
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
```

## Step 3 -	Edit network.host and discovery.seed_hosts in Node1

```
vi /etc/elasticsearch/elasticsearch.yml
```

```
network.host: IPNode
discovery.seed_hosts: ["IPnode1" , "IPnode2" , "IPnode3"]
```
> Start Elasticsearch service on Node1 to create cluster
```
systemctl start elasticsearch
```

> Check status of cluster
```
curl -k -XGET -u elastic "https://IPnode01:9200/_cluster/health"
```
## Step 4 Add node2 and node3 join the cluster

> Generate Token for make other nodes join the cluster 
>> on node1
```
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
```
> Add Node2 Node3 to cluster
```
/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token>
```

> Edit network.host and discovery.seed_hosts in Node2 Node 3
```
vi /etc/elasticsearch/elasticsearch.yml
```
```
network.host: IPnode
discovery.seed_host: ["IPnode1:9300" , "IPnode2" , "IPnode3"]
```
> Start Node2 Node3
```
systemctl start elasticsearch
```
> List node in cluster
```
curl -k -XGET -u elastic "https://IPnode1:9200/_cat/nodes?v=true"
```

## Step 5 Install Kibana
> Download Kibana on kibana node
```
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.4.0-x86_64.rpm
```

> Install Kibana
```
rpm --install kibana-8.4.0-x86_64.rpm
```

## Step 6 Configuration /etc/kibana/kibana.yml
- Generate elasticsearch.serviceAccountToken (Run below command in Node1)
```
curl -k -XPOST -u elastic  "https://IPnode1:9200/_security/service/elastic/kibana/credential/token/token1"
```
>> Copy /etc/elasticsearch/certs/http_ca.crt (node1) to /etc/kibana/certs/ (kibana node)

> on kibana node
```
vi /etc/kibana/kibana.yml
```

```
server.host: "IP kibananode"
elasticsearch.hosts: ["https://IPnode1:9200","https://IPnode2:9200","https://IPnode3:9200"]
elasticsearch.serviceAccountToken: “<serviceAccountToken:>”
elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/certs/http_ca.crt" ]
elasticsearch.ssl.verificationMode: none
logging:
  appenders:
    file:
      type: file
      fileName: /var/log/kibana/kibana.log
      layout:
        type: json
  root:
    appenders:
      - default
      - file
pid.file: /run/kibana/kibana.pid
```

- Generate kibana cert and enter password (Run below command in Node1)
```
/usr/share/elasticsearch/bin/elasticsearch-certutil cert -name kibana-server --self-signed
```
>> Copy /usr/share/elasticsearch/kibana-server.p12 (node1) to /etc/kibana/certs/ (kibana node)

- Create cert and key from .p12 file on kibana node
```
openssl pkcs12 -in  /etc/kibana/certs/kibana-server.p12 -out /etc/kibana/certs/kibana-cert.crt -nokeys
openssl pkcs12 -in /etc/kibana/certs/kibana-server.p12 -out /etc/kibana/certs/kibana-key.key -nodes -nocerts
```

> Change Own certs directory to kibana
```
chown -R kibana:kibana /etc/kibana/certs
```

> Edit config in /etc/kibana/kibana.yml
```
vi /etc/kibana/kibana.yml
```
```
server.ssl.enabled: true
server.ssl.certificate: /etc/kibana/certs/kibana-cert.crt
server.ssl.key: /etc/kibana/certs/kibana-key.key
```

> start kibana
```
systemctl start kibana
systemctl status -l kibana
```
