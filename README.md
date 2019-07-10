# Azure (Cognitive) Search Demo

## 0. Concepts 
- Data source. 
- Skillset / Cognitive Services. 
- Index. 
- Indexer. 
- Knowldge Store. 

## 1. Create an Azure Search Service  

## 2. Create an Azure Cognitive Service

## 3. Add a storage 
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

## 4. Create a skillset
Here pre-built cognitive skillsets:  
Language Detection   
Text Split   
Entity Recognition   
Key Phrase Extraction     

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

![alt text](https://docs.microsoft.com/en-us/azure/search/media/cognitive-search-tutorial-blob/skillset.png)

## 5. Create an Index
```shell
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

## 6. Create an Indexer
```shell
PUT https://[servicename].search.windows.net/indexers/demoindexer?api-version=2019-05-06
api-key: [api-key]
Content-Type: application/json
{
  "name":"demoindexer",	
  "dataSourceName" : "demodata",
  "targetIndexName" : "demoindex",
  "skillsetName" : "demoskillset",
  "fieldMappings" : [
    {
      "sourceFieldName" : "metadata_storage_path",
      "targetFieldName" : "id",
      "mappingFunction" :
        { "name" : "base64Encode" }
    },
    {
      "sourceFieldName" : "content",
      "targetFieldName" : "content"
    }
  ],
  "outputFieldMappings" :
  [
    {
      "sourceFieldName" : "/document/organizations",
      "targetFieldName" : "organizations"
    },
    {
      "sourceFieldName" : "/document/pages/*/keyPhrases/*",
      "targetFieldName" : "keyPhrases"
    },
    {
      "sourceFieldName": "/document/languageCode",
      "targetFieldName": "languageCode"
    }
  ],
  "parameters":
  {
    "maxFailedItems":-1,
    "maxFailedItemsPerBatch":-1,
    "configuration":
    {
      "dataToExtract": "contentAndMetadata",
      "imageAction": "generateNormalizedImages"
    }
  }
}

GET https://[servicename].search.windows.net/indexers/demoindexer/status?api-version=2019-05-06
api-key: [api-key]
Content-Type: application/json
```

## 7. Example Query
```
GET https://[servicename].search.windows.net/indexes/demoindex?api-version=2019-05-06
api-key: [api-key]
Content-Type: application/json

GET https://[servicename].search.windows.net/indexes/demoindex/docs?search=*&$select=organizations&api-version=2019-05-06
api-key: [api-key]
Content-Type: application/json
```

## 8. Knowldge Store
https://docs.microsoft.com/en-us/azure/search/knowledge-store-concept-intro
![alt text](https://docs.microsoft.com/en-us/azure/search/media/knowledge-store-concept-intro/annotationstore_sans_internalcache.png)


## Notes:
- How indexing and context works:  
https://medium.com/@Herger/chapter-2-azure-search-in-the-spotlight-72da0b4cf39c     

- How to query an index:  
https://docs.microsoft.com/en-us/azure/search/query-simple-syntax  
https://docs.microsoft.com/en-us/azure/search/query-lucene-syntax

- How to add a scoring profile  
https://docs.microsoft.com/en-us/azure/search/index-add-scoring-profiles

- How to add a suggester   
https://docs.microsoft.com/en-us/azure/search/index-add-suggesters

- How to use CORS   
https://clemenssiebler.com/azure-search-cors-configuration/   


# Useful Links
https://docs.microsoft.com/en-us/azure/search/
https://docs.microsoft.com/en-us/azure/search/cognitive-search-tutorial-blob   
https://github.com/Azure-Samples/azure-search-knowledge-mining  
https://github.com/Microsoft/cookiecutter-azure-search-cognitive-skill  
https://github.com/liamca/build2019aidemos  
https://github.com/Microsoft/SkillsExtractorCognitiveSearch  
https://github.com/Azure-Samples/azure-search-power-skills  
https://azure.github.io/LearnAI-KnowledgeMiningBootcamp/labs/lab-azure-search.html  

