# Identity and Access Management (IAM)
The gateway can support multi-tenant mode using an IAM service. The default is to operate in single tenant mode (root account only). To enable multi-tenant, one of the IAM services must be selected. The gateway allows for three different classes of users: root, admin, user. The root account is the primary management account specified on the cli when running the versitygw command. The admin and user accounts are stored in the IAM service.

The root and admin accounts can create/delete admin/user accounts, create buckets, see all buckets. The user accounts can only access buckets that have been created for them.

| Role | See All Buckets | Create New Buckets | Create New Users | Assign Bucket Ownership | See Only Owned Buckets |
| :--- | :---: | :---: | :---: | :---: | :---: |
| root | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | |
| admin | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | |
| user | | | | | :white_check_mark: |
| userplus | | :white_check_mark: | | | :white_check_mark: |

## IAM Internal
The Internal IAM stores tenant accounts in a file below the specified directory with the following option:
```
   --iam-dir value                         if defined, run internal iam service within this directory
```

If the gateway is running within a cluster for load balancing, this directory must be accessible on all hosts (such as NFS) so that the accounts are consistent across all gateways.

Note: The file is plan text JSON encoded fields and access is only protected with basic file permissions.

## IAM S3
```
   --s3-iam-access value                   s3 IAM access key [$VGW_S3_IAM_ACCESS_KEY]
   --s3-iam-secret value                   s3 IAM secret key [$VGW_S3_IAM_SECRET_KEY]
   --s3-iam-region value                   s3 IAM region (default: "us-east-1") [$VGW_S3_IAM_REGION]
   --s3-iam-bucket value                   s3 IAM bucket [$VGW_S3_IAM_BUCKET]
   --s3-iam-endpoint value                 s3 IAM endpoint [$VGW_S3_IAM_ENDPOINT]
   --s3-iam-noverify                       s3 IAM disable ssl verification (default: false) [$VGW_S3_IAM_NO_VERIFY]
   --s3-iam-debug                          s3 IAM debug output (default: false) [$VGW_S3_IAM_DEBUG]
```

The S3 IAM service is similar to the internal IAM service, but instead stores the account information JSON encoded in an S3 object. This should use a bucket that is not accessible to general users when using s3 backend to prevent access to account credentials. This IAM service is added for convenience, but is not considered as secure or scalable as a dedicated IAM service. This is generally useful when using s3proxy backend to not have other filesystem dependencies for the internal IAM service.

## IAM Vault
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
The details for Vault IAM storage are documented in [IAM Vault](./IAM-Vault).

## IAM LDAP
The LDAP IAM stores tenant accounts in an LDAP directory service. The following options are needed in order to access the LDAP service:
```
   --iam-ldap-url value                    ldap server url to store iam data
   --iam-ldap-bind-dn value                ldap server binding dn, example: 'cn=admin,dc=example,dc=com'
   --iam-ldap-bind-pass value              ldap server user password
   --iam-ldap-query-base value             ldap server destination query, example: 'ou=iam,dc=example,dc=com'
   --iam-ldap-object-classes value         ldap server object classes used to store the data. provide it as comma separated string, example: 'top,person'
```

The tenant access key, secret key, role, uid, and gid are mapped to LDAP attributes. These attributes are specified with the following options:
```
   --iam-ldap-access-atr value             ldap server user access key id attribute name
   --iam-ldap-secret-atr value             ldap server user secret access key attribute name
   --iam-ldap-role-atr value               ldap server user role attribute name
   --iam-ldap-user-id-atr value            ldap server user id attribute name
   --iam-ldap-group-id-atr value           ldap server user group id attribute name
```

Using the admin API to manage accounts is optional in the LDAP case. It is fine to manage users directly in LDAP, and only allow the gateway to have read permissions to the directory service. If the admin API is used for user management, then the gateway must have permissions to modify, add, and remove accounts within the directory service. 

## IAM cache
```
   --iam-cache-disable                     disable local iam cache (default: false)
   --iam-cache-ttl value                   local iam cache entry ttl (seconds) (default: 120)
   --iam-cache-prune value                 local iam cache cleanup interval (seconds) (default: 3600)
```
The IAM cache is intended to ease the load on the IAM service and increase the Gateway performance by caching accounts and credentials for the TTL time interval. Disabling this will cause a request to the configured IAM service for each incoming request in order to retrieve the corresponding account credentials. The cache is enabled by default. The TTL specifies how long to cache credentials, and the prune value determines the interval for expired entries to be removed from the cache. Increasing the TTL may lessen the load on the IAM service backend, but will increase the time the gateway will have out of date account info until the next interval. Decreasing the prune interval may reduce memory use at the cost of added CPU to check cache expirations.

# User Management
The admin and user accounts are managed through the versitygw admin API.  The easiest way to access this is with the versitygw command itself. The following commands assume the root or admin access key is `myaccess` and secret key is `mysecret`.  Adjust these and account details as needed. You can alternatively set `ADMIN_ACCESS_KEY`, `ADMIN_SECRET_KEY`, and `ADMIN_ENDPOINT_URL` environment variables instead of having to specify `--access`, `--secret`, and `--endpoint-url` respectively each time.

Options for creating users:
```
   --access value, -a value      access key id for the new user
   --secret value, -s value      secret access key for the new user
   --role value, -r value        role for the new user
   --user-id value, --ui value   userID for the new user (default: 0)
   --group-id value, --gi value  groupID for the new user (default: 0)
   --help, -h                    show help
```

Valid roles are: admin, user, userplus

The `user-id` and `group-id` can be set for individual accounts. Specific backends can enable use of these values to map user/group ids to backend access.  For example, in the posix and scoutfs backends, the options `--chuid` and `--chgid` can be enabled to set the user/group for newly created objects.

To create a new admin account with access key `myadmin` and secret key `mysecret`:
```
versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 create-user --access myadmin --secret mysecret --role admin
```

To create a new user account with access key `myuser` and secret key `mysecret`:
```
versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 create-user --access myuser --secret mysecret --role user
```

To list accounts:
Options for listing users:
```
   --help, -h                show help
```

example:
```
versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 list-users
```
```
Account  Role
-------  ----
myuser   user
myadmin  admin
```
note: The root account is not a permanently recorded in the IAM service, so not listed.  This account is always defined at gateway runtime.

Accounts can be deleted with the `delete-user` cli option.
Options for creating users:
```
   --access value, -a value  access key id of the user to be deleted
   --help, -h                show help
```

example:
```
versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 delete-user --access myuser
```

# Bucket Management
A user is only allowed access to bucket assigned to them. But they are unable to create new buckets. To create a bucket for a user account, an admin or root must create the bucket and assign ownership to the user.

This example extends the example above where the "myadmin" and "myuser" accounts were previously created.

Options for changing bucket owner:
```
   --bucket value, -b value  the bucket name to change the owner
   --owner value, -o value   the user access key id, who should be the bucket owner
   --help, -h                show help
```

Example of creating a bucket and assigning to "myuser" (Note: aws cli running with admin credentials):
```
export AWS_ACCESS_KEY_ID=myadmin
export AWS_SECRET_ACCESS_KEY=mysecret
export AWS_ENDPOINT_URL=http://127.0.0.1:7070
aws s3 mb s3://bucket1

versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 change-bucket-owner --bucket bucket1 --owner myuser
```
And now the user can list and access the bucket:
```
export AWS_ACCESS_KEY_ID=myuser
export AWS_SECRET_ACCESS_KEY=mysecret
export AWS_ENDPOINT_URL=http://127.0.0.1:7070
aws s3 ls s3://
```

The admin API allows for listing all buckets and their current owners. This is a special API call since this is not directly supported in the S3 API.

Options for list buckets:
```
   --help, -h  show help
```

example:
```
versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 list-buckets
```
```
Bucket   Owner
-------  ----
abucket  myaccess
bbucket  myadmin
cbucket  myuser
```

# ACLs
The gateway currently only supports the following bucket level ACLs defined with the same format as [AWS ACLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html):  **private**, **public-read**, and **public-read-write**.  **authenticated-read** is not supported.  And the ACLs may behave differently, as described below:

| ACL     | Direct to S3 | versitygw |
|:------:   | :------------: | :----------:|
| private | Allows only access by bucket owner | Same |
| public-read | Allows anyone to read from bucket using either AWS, or file retrieval tools (e.g. CURL, HTTP GET) | Allows non-owner users to read from the bucket, unless forbidden by a policy "Deny" |
| public-read-write | Allows anyone to read or write to the bucket using either AWS or other tools (e.g. CURL, HTTP GET/POST) | Allows non-owner users to read from or write to the bucket, unless forbidden by a policy "Deny" |
| authenticated-read | Allows anyone to read from the bucket with an AWS account | not supported |

# Policies
Policies are supported for **user** and **userplus** accounts.  No policy support is available for **root** accounts at this time, and policies set for **admin** accounts are currently ignored.