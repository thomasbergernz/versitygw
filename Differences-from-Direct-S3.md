## User creation/management/deletion

* Listing users is done by the **versitygw** **admin** tool as opposed to `iam list-users`
* User policies are not used.  If, for example, granting an account with **user** permissions access to certain files in a bucket, only the bucket policy or ACLs needs to be changed to allow the user access.