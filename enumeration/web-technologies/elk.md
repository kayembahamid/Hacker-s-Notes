# ELK

## Elasticsearch

### Check status

{% code title="curl" %}
```bash
curl -X GET "ELASTICSEARCH-SERVER:9200/"
```
{% endcode %}

### Check Auth enabled

{% code title="curl" %}
```bash
curl -X GET "ELASTICSEARCH-SERVER:9200/_xpack/security/user"
```
{% endcode %}

### Users

* elastic:changeme
* kibana\_system
* logstash\_system
* beats\_system
* apm\_system
* remote\_monitoring\_user

### Other endpoints

{% code title="curl" %}
```bash
/_cluster/health
/_cat/indices
/_cat/health
```
{% endcode %}

### Interesting endpoints (BE CAREFUL)

<details>

<summary>Show endpoints</summary>

```
/_shutdown
/_cluster/nodes/_master/_shutdown
/_cluster/nodes/_shutdown
/_cluster/nodes/_all/_shutdown
```

</details>

***

## With creds

### Using the API key

{% code title="curl" %}
```bash
curl -H "Authorization: ApiKey <API-KEY>" ELASTICSEARCH-SERVER:9200/
```
{% endcode %}

### Get more information about the rights of a user

{% code title="curl" %}
```bash
curl -X GET "ELASTICSEARCH-SERVER:9200/_security/user/<USERNAME>"
```
{% endcode %}

### List all users on the system

{% code title="curl" %}
```bash
curl -X GET "ELASTICSEARCH-SERVER:9200/_security/user"
```
{% endcode %}

### List all roles on the system

{% code title="curl" %}
```bash
curl -X GET "ELASTICSEARCH-SERVER:9200/_security/role"
```
{% endcode %}

***

## Internal config files

* Elasticsearch configuration: /etc/elasticsearch/elasticsearch.yml
* Kibana configuration: /etc/kibana/kibana.yml
* Logstash configuration: /etc/logstash/logstash.yml
* Filebeat configuration: /etc/filebeat/filebeat.yml
* Users file: /etc/elasticsearch/users\_roles

***

## Kibana

### Basic

* Port: 5601
* Config file & users: /etc/kibana/kibana.yml
* Try also with user: kibana\_system

{% hint style="warning" %}
Version < 6.6.0 = RCE (https://github.com/LandGrey/CVE-2019-7609/)
{% endhint %}

***

## Logstash

### Basic

* Pipelines config: /etc/logstash/pipelines.yml
* Check pipelines for this property: "config.reload.automatic: true"

If a file wildcard is specified, a sample pipeline that executes a command and writes to a file might look like:

{% code title="logstash pipeline example" %}
```
input {
  exec {
    command => "whoami"
    interval => 120
  }
}

output {
  file {
    path => "/tmp/output.log"
    codec => rubydebug
  }
}
```
{% endcode %}
