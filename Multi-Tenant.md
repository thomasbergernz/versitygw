# IAM
The gateway can support multi-tenant mode using an IAM service. Currently the only functional IAM service is the internal service that stores the accounts info in a local file. The gateway allows for 3 different classes of users: root, admin, user. The root account is the primary management account specified on the cli when running the versitygw command. The admin and user accounts are stored in the IAM service.

The root and admin accounts can create/delete admin/user accounts, create buckets, see all buckets. The user accounts can only access buckets that have been created for them.

| Role | See All Buckets | Create New Buckets | Create New Users | Assign Bucket Ownership | See Only Owned Buckets |
| :--- | :---: | :---: | :---: | :---: | :---: |
| root | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | |
| admin | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | |
| user | | | | | :white_check_mark: |

The admin and user accounts are managed through the versitygw admin API.  The easiest way to access this is with the versitygw command itself. The following commands assume the root or admin access key is `myaccess` and secret key is `mysecret`.  Adjust these and account details as needed. You can alternatively set `ADMIN_ACCESS_KEY`, `ADMIN_SECRET_KEY`, and `ADMIN_ENDPOINT_URL` environment variables instead of having to specify `--access`, `--secret`, and `--endpoint-url` respectively each time.

To create a new admin account with access key `myadmin` and secret key `mysecret`:
```
versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 create-user --access myadmin --secret mysecret --role admin
```

To create a new user account with access key `myuser` and secret key `mysecret`:
```
versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 create-user --access myuser --secret mysecret --role user
```

To list accounts:
```
versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 list-users
```

You can similarly delete accounts with the `delete-user` cli option.
```
versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 delete-user --access myuser
```

# Bucket Management
A user is only allowed access to bucket assigned to them. But they are unable to create new buckets. To create a bucket for a user account, an admin or root must create the bucket and assign ownership to the user.

This example extends the example above where the "myadmin" and "myuser" accounts were previously created.

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

The admin API allows for listing all buckets and their current owners. This is a special API call since this is not directly supported in the S3 API.  For example:
```
versitygw admin --access myaccess --secret mysecret --endpoint-url http://127.0.0.1:7070 list-buckets
Bucket   Owner
-------  ----
abucket  myaccess
bbucket  myadmin
cbucket  myuser
```

# ACLs
The gateway currently only supports bucket level ACLs defined with the same format as [AWS ACLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html).