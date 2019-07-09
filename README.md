# Azure Search Demo

## Create the Search Service

## Add a storage 
```shell
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
```
or 
```shell
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

## Create a skillset
Here Language Detection, Text Split, Entity Recognition & Key Phrase Extraction 

```shell
PUT https://[servicename].search.windows.net/skillsets/demoskillset?api-version=2019-05-06
api-key: [admin key]
Content-Type: application/json

{
  "description":
  "Extract entities, detect language and extract key-phrases",
  "skills":
  [
    {
      "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
      "categories": [ "Organization" ],
      "defaultLanguageCode": "en",
      "inputs": [
        {
          "name": "text", "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "organizations", "targetName": "organizations"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
      "inputs": [
        {
          "name": "text", "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "languageCode",
          "targetName": "languageCode"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
      "textSplitMode" : "pages",
      "maximumPageLength": 4000,
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        },
        {
          "name": "languageCode",
          "source": "/document/languageCode"
        }
      ],
      "outputs": [
        {
          "name": "textItems",
          "targetName": "pages"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
      "context": "/document/pages/*",
      "inputs": [
        {
          "name": "text", "source": "/document/pages/*"
        },
        {
          "name":"languageCode", "source": "/document/languageCode"
        }
      ],
      "outputs": [
        {
          "name": "keyPhrases",
          "targetName": "keyPhrases"
        }
      ]
    }
  ]
}
```
```shell
curl --request PUT \
  --url 'https://[servicename].search.windows.net/skillsets/demoskillset?api-version=2019-05-06' \
  --header 'Accept: */*' \
  --header 'Cache-Control: no-cache' \
  --header 'Connection: keep-alive' \
  --header 'Content-Type: application/json' \
  --header 'Host: [servicename].search.windows.net' \
  --header 'accept-encoding: gzip, deflate' \
  --header 'api-key: [api-key]' \
  --header 'cache-control: no-cache' \
  --header 'content-length: 1672' \
  --data '{\n  "description":\n  "Extract entities, detect language and extract key-phrases",\n  "skills":\n  [\n    {\n      "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",\n      "categories": [ "Organization" ],\n      "defaultLanguageCode": "en",\n      "inputs": [\n        {\n          "name": "text", "source": "/document/content"\n        }\n      ],\n      "outputs": [\n        {\n          "name": "organizations", "targetName": "organizations"\n        }\n      ]\n    },\n    {\n      "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",\n      "inputs": [\n        {\n          "name": "text", "source": "/document/content"\n        }\n      ],\n      "outputs": [\n        {\n          "name": "languageCode",\n          "targetName": "languageCode"\n        }\n      ]\n    },\n    {\n      "@odata.type": "#Microsoft.Skills.Text.SplitSkill",\n      "textSplitMode" : "pages",\n      "maximumPageLength": 4000,\n      "inputs": [\n        {\n          "name": "text",\n          "source": "/document/content"\n        },\n        {\n          "name": "languageCode",\n          "source": "/document/languageCode"\n        }\n      ],\n      "outputs": [\n        {\n          "name": "textItems",\n          "targetName": "pages"\n        }\n      ]\n    },\n    {\n      "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",\n      "context": "/document/pages/*",\n      "inputs": [\n        {\n          "name": "text", "source": "/document/pages/*"\n        },\n        {\n          "name":"languageCode", "source": "/document/languageCode"\n        }\n      ],\n      "outputs": [\n        {\n          "name": "keyPhrases",\n          "targetName": "keyPhrases"\n        }\n      ]\n    }\n  ]\n}'
```

![alt text](https://docs.microsoft.com/en-us/azure/search/media/cognitive-search-tutorial-blob/skillset.png)

## Create Index
```
PUT https://[servicename].search.windows.net/indexes/demoindex?api-version=2019-05-06
api-key: [api-key]
Content-Type: application/json

{
  "fields": [
    {
      "name": "id",
      "type": "Edm.String",
      "key": true,
      "searchable": true,
      "filterable": false,
      "facetable": false,
      "sortable": true
    },
    {
      "name": "content",
      "type": "Edm.String",
      "sortable": false,
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "languageCode",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "keyPhrases",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "organizations",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "sortable": false,
      "filterable": false,
      "facetable": false
    }
  ]
}
```
