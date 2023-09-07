# IAM
The gateway can support multi-tenant mode using an IAM service. Currently the only functional IAM service is the internal service that stores the accounts info in a local file. The gateway allows for 3 different classes of users: root, admin, user. The root account is the primary management account specified on the cli when running the versitygw command. The admin and user accounts are stored in the IAM service.

The root and admin accounts can create/delete admin/user accounts, create buckets, see all buckets. The user accounts can only access buckets that have been created for them.

| Role | See All Buckets | Create New Buckets | Create New Users | Assign Bucket Ownership | See Only Owned Buckets |
| :--- | :---: | :---: | :---: | :---: | :---: |
| root | X | X | X | X | |
| admin | X | X | X | X | |
| user | | | | | X |

The admin and user accounts are managed through the versitygw admin API.  The easiest way to access this is with the versitygw command itself. The following commands assume the root or admin access key is `myaccess` and secret key is `mysecret`.  Adjust these and account details as needed.

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
versitygw admin --access myaccess -s mysecret --endpoint-url http://127.0.0.1:7070 delete-user --access myuser
```

[Known Issues](https://github.com/versity/versitygw/labels/IAM)

# ACLs
The gateway currently only supports bucket level ACLs defined with the same format as [AWS ACLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html).