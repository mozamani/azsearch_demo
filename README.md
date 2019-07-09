# Azure Search Demo

## Create the Search Service

## Add a storage 
```
POST https://[service name].search.windows.net/datasources?api-version=2019-05-06
Content-Type: application/json
api-key: [admin key]

{
  "name" : "<your name for storage>",
  "description" : "your description",
  "type" : "azureblob",
  "credentials" :
  { "connectionString" :
    "DefaultEndpointsProtocol=https;AccountName=<your account name>;AccountKey=<your account key>;"
  },
  "container" : { "name" : "<your blob container name>" }
}

curl --request POST \
  --url 'https://[service name].search.windows.net/datasources?api-version=2019-05-06' \
  --header 'Accept: */*' \
  --header 'Cache-Control: no-cache' \
  --header 'Connection: keep-alive' \
  --header 'Content-Type: application/json' \
  --header 'Host: [service name].search.windows.net' \
  --header 'accept-encoding: gzip, deflate' \
  --header 'api-key: [your api-key]' \
  --header 'cache-control: no-cache' \
  --header 'content-length: 304' \
  --data '{\n    "name" : "sample-data-json",\n    "type" : "azureblob",\n    "credentials" : { "connectionString" : "[connection string];" },\n    "container" : { "name" : "[containername]"}\n}'

```
