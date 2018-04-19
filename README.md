# Introduction

The pass-indexer keeps an [Elasticsearch](https://github.com/elastic/elasticsearch) index up to date with resources in a
[Fedora repository](https://fedorarepository.org/).

# Design

The pass-indexer monitors a JMS queue for messages about creation, deletion, and modification of Fedora resources.
(Fedora must be configured appropriately to setup this queue.)

The Elasticsearch index is created on startup if it does not exist with a set [configuration](pass-indexer-core/src/main/resources/esindex.json).
If the index does exist, the configuration is retrieved from the index. In either case the mapping must match the documents which will be indexed.
The Elasticsearch document is the compact JSON-LD representation of that resource without server triples.

When there is a message about a resource of a type being monitored, the indexer either creates a corresponding document in Elasticsearch 
from the Fedora resource, updates such a document, or deletes the document.  Only messages about a resource of a type which matches a
configured prefix, PI_TYPE_PREFIX, are handled. The id of the Elasticsearch document is the safe URL base64 encoding of resource path. This lets both the document be created and updated with the same PUT.

# Auto-completion (suggestion) support

Suggestion is supported on field of type complection. In order to allow auto-completion on the values of a field
NAME, add a mapping for NAME_suggest of type completion with a copy_to for NAME. The field NAME_suggest will
be automatically populated with the contents of NAME.


Then you can search like below to find awardNumbers beginning with "R".

```
curl -X POST "http://localhost:9200/pass/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "suggest": {
        "my-suggest" : {
            "prefix" : "R", 
            "completion" : { 
                "field" : "awardNumber_suggest" 
            }
        }
    }
}
'
```

# Command line tool

The command line tool runs the indexer with the configuration either specifies as environment variable or system
properties. 

Configuration properties:
* PI_FEDORA_USER=admin
* PI_FEDORA_PASS=admin
* PI_ES_INDEX=http://elasticsearch:9200/pass/
* PI_FEDORA_JMS_BROKER=tcp://fcrepo:61616
* PI_FEDORA_JMS_QUEUE=fedora
* PI_TYPE_PREFIX=http://example.org/pass/
* PI_LOG_LEVEL=debug
