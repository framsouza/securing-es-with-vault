# Securing Elasticsearch with hashiCorp Vault

Are you using Elasticsearch and HashiCorp in your environment and were wondering how you could connect both?
In this concise article, you will discover the steps to effectively utilize hashiCorp Vault for the automated generation and revocation of credentials on Elasticsearch.

## Demo


## Requirements 
1. Create an account on [Elastic Cloud](https://www.elastic.co/cloud/)
2. Install and unseal vault in our Kubernetes/VMs

## Implementation
1. Enable vault database secret engine

`vault secrets enable -path=elasticsearch database`

2. Configure a Vault role that will be used by the elasticsearch secret engine
```
vault write database/roles/internally-defined-role \
      db_name=my-elasticsearch-database \
      creation_statements='{"elasticsearch_role_definition": {"indices": [{"names":["*"], "privileges":["read"]}]}}' \
      default_ttl="1h" \
      max_ttl="24h"
```

3. Configure the elasticsearch secret database to connect to your ES deployment
```
vault write database/config/my-elasticsearch-database \
    plugin_name="elasticsearch-database-plugin" \
    allowed_roles="internally-defined-role" \
    username=vault \
    password=myPa55word \
    url=<ES-URL>
```

4. Generate a new credentials on Elasticsearch by running:

`vault read database/creds/my-role`

This will generate user credentials on Elasticsearch using the role/permissions that was defined on step #2. The credentials will be automatically revoked once the ttl expires.

Easy, right? :)
