## User creation/management/deletion

* Listing users is done by the **versitygw** **admin** tool as opposed to `aws iam list-users`.
* With **versitygw**, user policies are not used.  If, for example, granting an account with **user** permissions access to certain files in a bucket that the user does not own, only the bucket policy or ACLs needs to be changed to allow the user access.
* If using **versitygw**, it is unnecessary to delete user fields such as access keys before deleting the user.
* The addition of the ID and password is required for user creation, and so running the `aws iam create-access-key` command isn't necessary to provide a key ID and password (or secret key) to be able to access buckets with the **aws** **s3** and **s3api** tools.  Also, in **versitygw**, there are not separate usernames and key IDs, and only a single username which corresponds to both is used.

## Policies

* As of July 3, 2024, policies don't have an effect on **versitygw** **admin** and **root** accounts.
* In **versitygw** policies, the **Principal** field can simply be the raw username or group of raw usernames, e.g. `"Principal": "<username>"`.  A format similar to the following isn't necessary:
```
"Principal": {
  "AWS": "arn:aws:iam::<AWS account ID>:user/<username>"
},
```