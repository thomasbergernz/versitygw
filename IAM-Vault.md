## Hashicorp Vault as an IAM service

Vault IAM service stores IAM credentials in kv-v2 engine.

### Configuration
```
   --iam-vault-endpoint-url value                  vault server url [$VGW_IAM_VAULT_ENDPOINT_URL]
   --iam-vault-secret-storage-path value           vault server secret storage path [$VGW_IAM_VAULT_SECRET_STORAGE_PATH]
   --iam-vault-mount-path value                    vault server mount path [$VGW_IAM_VAULT_MOUNT_PATH]
   --iam-vault-root-token value                    vault server root token [$VGW_IAM_VAULT_ROOT_TOKEN]
   --iam-vault-role-id value                       vault server user role id [$VGW_IAM_VAULT_ROLE_ID]
   --iam-vault-role-secret value                   vault server user role secret [$VGW_IAM_VAULT_ROLE_SECRET]
   --iam-vault-server_cert value                   vault server TLS certificate [$VGW_IAM_VAULT_SERVER_CERT]
   --iam-vault-client_cert value                   vault client TLS certificate [$VGW_IAM_VAULT_CLIENT_CERT]
   --iam-vault-client_cert_key value               vault client TLS certificate key [$VGW_IAM_VAULT_CLIENT_CERT_KEY]
```

The gateway supports the following vault authentication methods:

* With root token (not recommended for production)
* Role based authentication (requires options: roleId and roleSecret)

This IAM service is intended to be managed through the versitygw admin commands similar to the internal IAM service. This uses the kv-v2 key/value secrets storage, and uses access key for the key and stores the JSON serialized account data as the value. The "mount" is `kv-v2` and secrets are stored below the path specified with `--iam-vault-secret-storage-path`.

### Example
This example is using a dev server with root token auth. These are not recommended for production use, but configuring production Vault is beyond scope of this manual. See Vault manual for production server configuration and role setup.

start Vault server:
```
# vault server -dev
```
The output of starting this dev server will provide the root token for the next commands.

use Vault cli to enable kv-v2 storage:
```
# export VAULT_ADDR='http://127.0.0.1:8200'
# export VAULT_TOKEN=hvs.dqw7YKwMlvXmsxICYA0N7Krs

# vault secrets enable kv-v2
Success! Enabled the kv-v2 secrets engine at: kv-v2/
```

Start versitygw with the iam-vault options. This example is storing secrets under `versitygw` path as an example. This can be set to anything that does not conflict with any other service. Set the `--iam-vault-root-token` to match the dev server output root token.
```
# versitygw --iam-vault-endpoint-url http://127.0.0.1:8200 --iam-vault-root-token hvs.dqw7YKwMlvXmsxICYA0N7Krs --iam-vault-secret-storage-path veristygw --access test --secret test posix /tmp/gw
```
or by setting env vars
```
# export VGW_IAM_VAULT_ENDPOINT_URL=http://127.0.0.1:8200
# export VGW_IAM_VAULT_ROOT_TOKEN=hvs.dqw7YKwMlvXmsxICYA0N7Krs
# export VGW_IAM_VAULT_SECRET_STORAGE_PATH=versitygw

# versitygw --access test --secret test posix /tmp/gw
```

Use versitygw to create and list users:
```
# export ADMIN_ACCESS_KEY=test
# export ADMIN_SECRET_KEY=test
# export ADMIN_ENDPOINT_URL= http://127.0.0.1:7070

# versitygw admin create-user --access myuser --secret upass --role user
The user has been created successfully

# versitygw admin create-user --access myadmin --secret apass --role admin
The user has been created successfully

# versitygw admin list-users                              
Account  Role   UserID  GroupID
-------  ----   ------  -------
myadmin  admin  0       0
myuser   user   0       0
```

The secrets that versitygw stored can be directly accessed with Vault cli:
```
# vault kv list --mount=kv-v2 versitygw                
Keys
----
myadmin
myuser

# vault kv get --mount=kv-v2 versitygw/myadmin
======== Secret Path ========
kv-v2/data/versitygw/myadmin

======= Metadata =======
Key                Value
---                -----
created_time       2024-06-09T16:08:43.710302Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

===== Data =====
Key        Value
---        -----
myadmin    map[access:myadmin groupID:0 role:admin secret:apass userID:0]
```

