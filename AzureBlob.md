The Azure backend stores S3 objects within [Azure Blob Storage](https://azure.microsoft.com/en-us/products/storage/blobs).

🧪 The `azure` backend is currently experimental

# Command
```
versitygw azure [command options]
```

# Args
The gateway uses the official Azure SDK for access to Azure Blob Storage. This allows the gateway to automatically detect authentication  credentials and service URLs when running within an Azure instance. When running outside of an Azure instance, the following options can be used to supply authentication credentials to the gateway.

***
```
   --account value, -a value      azure account name
   --access-key value, -k value   azure account key
   --sas-token value, --st value  azure blob storage SAS token
   --url value, -u value          azure service URL
```
The authentication to Azure Blob Storage can use either `--access-key` or `--sas-token`. The storage account can be specified with `--account`. If `--url` is not specified, the default `https://<account>.blob.core.windows.net/` will be used.
***

# Testing
Microsoft provides a container for testing Azure endpoints locally called Azurite. See the [Azurite docs](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite?tabs=visual-studio%2Cblob-storage) for more information on using Azurite for blob storage testing. The versitygw project provides a docker file for quick setup and launch of both azurite and versitygw configured with azure backend. To run this, clone the git repo, and run the following in the top level directory:
```
make up-azurite
```
or the direct docker-compose command:
```
docker-compose -f tests/docker-compose.yml --env-file .env.dev --project-directory . up azurite azuritegw
```