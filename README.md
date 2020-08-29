# vault-101

vault-101

## Basics

### Your First Secret

```sh
docker run -it --rm --network vault-101_vault-demo vault sh

export VAULT_ADDR=http://vault-server:8200
export VAULT_TOKEN=myroot
vault kv put secret/hello foo=world

/ # vault kv put secret/hello foo=world
Key              Value
---              -----
created_time     2020-08-29T20:14:01.074204851Z
deletion_time    n/a
destroyed        false
version          1

# Multiple values
vault kv put secret/hello foo=world excited=yes
/ # vault kv put secret/hello foo=world excited=yes
Key              Value
---              -----
created_time     2020-08-29T20:14:54.886666818Z
deletion_time    n/a
destroyed        false
version          2

```

### Getting a Secret

```sh
/ # vault kv get secret/hello
====== Metadata ======
Key              Value
---              -----
created_time     2020-08-29T20:14:54.886666818Z
deletion_time    n/a
destroyed        false
version          2

===== Data =====
Key        Value
---        -----
excited    yes
foo        world

/ # vault kv get -field=excited secret/hello
yes

apk add jq
/ # vault kv get -format=json secret/hello | jq -r .data.data.excited
yes
```

### Deleting a Secret

```sh
/ # vault kv delete secret/hello
Success! Data deleted (if it existed) at: secret/hello
```

## Secrets Management

### Dynamic Secrets: Database Secrets Engine

#### Step 1: Enable the database secrets engine

```sh
docker run -it --rm --network vault-101_vault-demo vault sh

export VAULT_ADDR=http://vault-server:8200
export VAULT_TOKEN=myroot
vault secrets enable database

vault write database/config/my-mysql-database \
    plugin_name=mysql-database-plugin \
    connection_url="root:root@tcp(mysql:3306)/" \
    allowed_roles="plataforma" \
    username="vaultuser" \
    password="vaultpass"

vault write database/roles/plataforma \
    db_name=my-mysql-database \
    creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT SELECT ON *.* TO '{{name}}'@'%';" \
    default_ttl="1h" \
    max_ttl="24h"

vault read database/creds/role
```


```sh
docker run -it --rm --network vault-101_vault-demo mysql:8.0 bash
mysql -h mysql -uroot -proot
```