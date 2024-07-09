## User creation/management/deletion

* Listing users is done by the **versitygw** **admin** tool as opposed to `aws iam list-users`.
* With **versitygw**, user policies are not used.  If, for example, granting an account with **user** permissions access to certain files in a bucket that the user does not own, only the bucket policy or ACLs needs to be changed to allow the user access.
* If using **versitygw**, it is unnecessary to delete user fields such as access keys before deleting the user.
* The addition of the ID and password is required for user creation, and so running the `aws iam create-access-key` command isn't necessary to provide a key ID and password (or secret key) to be able to access buckets with the **aws** **s3** and **s3api** tools.  Also, in **versitygw**, there are not separate usernames and access key IDs, and only a single username which corresponds to both is used.
* Since **versitygw** uses a single username, the IAM access key ID change propagation delay isn't a concern, so no wait time is needed between IAM changes and policy/ACL additions or changes.

## Policies

* IAM policies are not supported in the gateway.
* **Condition** property is not supported in the bucket policy document.
* As of July 3, 2024, policies do not affect **versitygw** **admin** and **root** accounts.
* In **versitygw** policies, the Principal field must be a comma-separated list of raw usernames.
* There are 2 main supported structures for **Principal**:
```json
"Principal": "<user1_access_key_id, user2_access_key_id...>"
```
or
```json
"Principal": {
  "AWS": "user1_access_key_id, user2_access_key_id...>"
}
```
* A format similar to the following isn't necessary for **Principal**:

```
"Principal": {
  "AWS": "arn:aws:iam::<AWS account ID>:user/<username>"
}
```

## ACLs

* **versitygw** supports only bucket-level ACLs: object ACLs are not supported.
* Bucket-level ACLs follow the direct-to-S3 structure below with a few differences:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<AccessControlPolicy>
   <Owner>
      <DisplayName>string</DisplayName>
      <ID>string</ID>
   </Owner>
   <AccessControlList>
      <Grant>
         <Grantee>
            <DisplayName>string</DisplayName>
            <EmailAddress>string</EmailAddress>
            <ID>string</ID>
            <xsi:type>string</xsi:type>
            <URI>string</URI>
         </Grantee>
         <Permission>string</Permission>
      </Grant>
   </AccessControlList>
</AccessControlPolicy>

```

1. `DisplayName`, `EmailAddress`, `xsi:type` and `URI` properties are not used in **Grantee**.
2. The `DisplayName` property in **Owner** is omitted.
3. The `ID` property in **Owner** and **Grantee** is the user access key ID.

### Canned ACL translation

Only the following bucket canned ACLs are supported in the gateway: `private`, `public-read`, and `public-read-write`. These ACLs adhere to the translation rules specified below, where `all-users` indicates that the rules apply to all users.

**private**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<AccessControlList>
   <Grant>
      <Grantee>
         <ID><bucket_owner_access_key_id></ID>
      </Grantee>
      <Permission>FULL_CONTROL</Permission>
   </Grant>
</AccessControlList>
```

**public-read**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<AccessControlList>
   <Grant>
      <Grantee>
         <ID>all-users</ID>
      </Grantee>
      <Permission>READ</Permission>
   </Grant>
</AccessControlList>
```

**public-read-write**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<AccessControlList>
   <Grant>
      <Grantee>
         <ID>all-users</ID>
      </Grantee>
      <Permission>READ</Permission>
   </Grant>
   <Grant>
      <Grantee>
         <ID>all-users</ID>
      </Grantee>
      <Permission>WRITE</Permission>
   </Grant>
</AccessControlList>
```

### AWS CLI usage

The `put-bucket-acl` action from **aws s3api** works slightly differently in the gateway. The `--grant-full-control`, `--grant-read`, `--grant-read-acp`, `--grant-write` and `--grant-write-acp` flags expect comma-separated user access key IDs. For example:
```
aws --endpoint-url <gateway_endpoint_url> s3api put-bucket-policy --bucket <bucket_name> --grant-read <"user1_key, user2_key..."> --grant-write <"user1_key, user2_key...">
```

* In the case of the `--access-control-policy` flag, the only required field is the **Grantee ID**.

### Comparison to AWS S3

| ACL     | Direct to S3 | versitygw |
|:------:   | :------------: | :----------:|
| READ | Allows grantee to list the objects in the bucket | Grants a user access to all read/get/list operations in the bucket (e.g. ListObjects(V2), GetObject, HeadBucket ...) |
| WRITE | Allows grantee to create new objects in the bucket. For the bucket and object owners of existing objects, also allows deletions and overwrites of those objects | Grants a user access to all write operations in the bucket (e.g. PutObject, DeleteObject, initiate/abort multipart upload ...) |
| READ_ACP | Allows grantee to read the bucket ACL | Same |
| WRITE_ACP | Allows grantee to write the ACL for the applicable bucket | Same |
| FULL_CONTROL | Allows grantee the READ, WRITE, READ_ACP, and WRITE_ACP permissions on the bucket | Same |
