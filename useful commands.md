# Elastic Search
Elastic search will run on port 5601 to open in web portal. To use it as api, port 9200 is used.

## _Useful commands_

### Create index 
This creates the index with the standard analyzer.
```
PUT contacts
```
We can create index with all the settings which mentioned below in a single command.
```
PUT /contacts
{
  "settings": {
    "index": {
      "max_ngram_diff": 30
    },
  "analysis": {
      "analyzer": {
        "ngram_analyzer": {
          "tokenizer": "contact_tokenizer",
          "filter": ["lowercase", "stop"]
        },
        "full_text_analyzer": {
          "type": "custom",
          "filter": [
             "lowercase"
          ],
          "tokenizer": "whitespace"
       }
      },
      "tokenizer": {
        "contact_tokenizer": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 4,
          "token_chars": [
            "letter",
            "digit"
          ]
        }
      }
    }
},
  "mappings": {
    "dynamic": "true",
    "dynamic_templates": [{
      "anything": {
        "match": "*",
        "mapping": {
          "index": true,
          "type": "text",
          "analyzer": "ngram_analyzer"
        }
      }
    }],
    "properties": {
      "field_1": {
         "type": "text",
         "analyzer": "full_text_analyzer"
      }
    }
  }
}
```

### Index Modifications
In order to do any changes with the index settings, we should always close the index.
```
POST dev-hc-contacts/_close
```

Once modifications are done, open the index to perform the operations.
```
POST dev-hc-contacts/_open
```

### ReIndexing 
Reindex will copy the documents from one index to another index. To perform re index operations in async way pass the query params. 
```
POST /_reindex?wait_for_completion=false
{
  "source": {
    "index": "fake"
  },
  "dest": {
    "index": "contacts"
  }
}
```
If we don't have much data we no need to pass the query params. wait_for_completion=false makes the command async
```
POST /_reindex
{
  "source": {
    "index": "fake"
  },
  "dest": {
    "index": "contacts"
  }
}
```

## Updating Index
To Update settings in the index use command
```
put dev-hc-contacts/_settings
{
    settings
}
```
Example settings is given below. In that settings we have two analyzer, one for ngram and another one for entire text.
> ngram index will perform index based on given characters

```
put dev-hc-contacts/_settings
{
  "index": {
      "max_ngram_diff": 30
    },
  "analysis": {
      "analyzer": {
        "ngram_analyzer": {
          "tokenizer": "contact_tokenizer",
          "filter": ["lowercase", "stop"]
        },
        "full_text_analyzer": {
          "type": "custom",
          "filter": [
             "lowercase"
          ],
          "tokenizer": "whitespace"
       }
      },
      "tokenizer": {
        "contact_tokenizer": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 10,
          "token_chars": [
            "letter",
            "digit"
          ]
        }
      }
    }
}
```

To update the mappings for the index. Dynamic templates will map the index based on the match. In our case all fileds expect mention in the properties section will be mapped.
```
put dev-hc-contacts/_mapping
{
    "dynamic": "true",
    "dynamic_templates": [{
      "anything": {
        "match": "*",
        "mapping": {
          "index": true,
          "type": "text",
          "analyzer": "ngram_analyzer"
        }
      }
    }],
    "properties": {
      "field_1": {
         "type": "text",
         "analyzer": "full_text_analyzer"
      }
    }
  }
```
> Note: Once the filed is indexed. The field cannot be changed to different index. Then we need to reindex it.

### Delete index
On index deletion, the documents also will be delete. To delete the index.
```
DELETE contacts_index
```

### Insert Record
```
POST /index_name/_doc/_id
{
  "test" : "data"
}
```

### Analyze Index
Based on our index creation, we can test how the input will be index.
- How ngram index will index the below text
```
POST contacts/_analyze
{
  "analyzer": "ngram_analyzer",
  "text": "16d479f8-53c1-4722-8d4d-149a48ca084c"
}
```

- How full text index will work
```
POST dev-hc-contacts/_analyze
{
  "analyzer": "full_text_analyzer",
  "text": "16d479f8-53c1-4722-8d4d-149a48ca084c"
}
```
> Above two analyze is based on the index we created above.

## Operations in index
We can perfom all sort of operations in the index

### Index informations
To get index details
```
GET contacts
```
To get index settings details
```
GET contacts/_settings
```
To get index mappings details
```
GET contacts/_mapping
```

### Search Index
To perform a simple index search with match
```
GET contacts/_search 
{
  "query": {
    "match": {
      "lastName": "unze"
    }
  }
}
```

Multi filed match
```
GET contacts/_search 
{
  "query": {
    "multi_match": {
        "query": "text"
    }
  }
}
```

Search with pagination
```
GET contacts/_search 
{
  "from": 0,
  "size": 100,
  "query": {
    "multi_match": {
        "query": "text",
    }
  }
}
```
To search with fields. ^ with number will give more specific on the field. firstName will give preference of 5 and lastName will be 3, other fields will be standard
```
GET contacts/_search 
{
  "from": 0,
  "size": 100,
  "query": {
    "multi_match": {
        "query": "text",
           "fields": ["firstName^5", "lastName^3", "*"]
    }
  }
}
```
To Search contacts with where condition. Filter will act as where condition. Term will match the exact value, we can use inrange, match too.
```
GET contacts/_search
{
  "from": 0,
  "size": 100,
  "query": { 
    "bool": { 
      "must": [
        {
          "multi_match" : {
            "query": "user",
            "fields": ["firstName^5", "lastName^5", "*"]
        }
        }
      ],
      "filter": [ 
        { "term":  
        { 
          "field_1" : "16d4-test-value" 
        }
        }
      ]
    }
  }
}
```

When speific fields are required use `_source` in the query
```
GET contacts/_search
{
  "from": 0,
  "size": 100,
  "query": { 
    "bool": { 
      "must": [
        {
          "multi_match" : {
            "query": "trekker",
            "fields": ["firstName^5", "lastName^5", "*"]
        }
        }
      ],
      "filter": [ 
        { "term":  
        { 
          "field_1" : "16d4-test-value" 
        }
        }
        ]
    }
  },
   "_source": [
        "firstName",
        "lastName",
        "contactID"
    ]
}

```
