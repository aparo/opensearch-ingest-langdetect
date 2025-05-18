# OpenSearch Langdetect Ingest Processor

This is a port of [spinscale's ElasticSearch Langdetect ingest plugin](https://github.com/spinscale/elasticsearch-ingest-langdetect).
The code was migrate using my migration script [ElasticSearch to OpenSearch Migration Scripts](https://github.com/aparo/elasticsearch-opensearch-migration-scripts)

Uses the [langdetect](https://github.com/YouCruit/language-detection/) plugin to try to find out the language used in a field.

Note that OpenSearch has native support for langdetection nowadays using the
`inference` ingest processor. See more in
[the documentation](https://www.elastic.co/guide/en/machine-learning/current/ml-lang-ident.html)

## Installation

| OS    | Command |
| ----- | ------- |
| 1.1.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-ingest-langdetect/releases/download/1.1.0/ingest-langdetect-1.1.0.zip` |
| 1.2.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-ingest-langdetect/releases/download/1.2.0/ingest-langdetect-1.2.0.zip` |
| 1.2.2 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-ingest-langdetect/releases/download/1.2.2/ingest-langdetect-1.2.2.zip` |
| 1.2.3 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-ingest-langdetect/releases/download/1.2.3/ingest-langdetect-1.2.3.zip` |

## Usage


```
PUT _ingest/pipeline/langdetect-pipeline
{
  "description": "A pipeline to do whatever",
  "processors": [
    {
      "langdetect" : {
        "field" : "my_field",
        "target_field" : "language"
      }
    }
  ]
}

PUT /my-index/my-type/1?pipeline=langdetect-pipeline
{
  "my_field" : "This is hopefully an english text, that will be detected."
}

GET /my-index/my-type/1

# Expected response
{
  "my_field" : "This is hopefully an english text, that will be detected.",
  "language": "en"
}
```

You could also set certain fields that use different analyzers for different languages

```
PUT _ingest/pipeline/langdetect-analyzer-pipeline
{
  "description": "A pipeline to index data into language specific analyzers",
  "processors": [
    {
      "langdetect": {
        "field": "my_field",
        "target_field": "lang"
      }
    },
    {
      "script": {
        "source": "ctx.language = [:];ctx.language[ctx.lang] = ctx.remove('my_field')"
      }
    }
  ]
}

PUT documents
{
  "mappings": {
    "doc" : {
      "properties" : {
        "language": {
          "properties": {
            "de" : {
              "type": "text",
              "analyzer": "german"
            },
            "en" : {
              "type": "text",
              "analyzer": "english"
            }
          }
        }
      }
    }
  }
}

PUT /my-index/doc/1?pipeline=langdetect-analyzer-pipeline
{
  "my_field" : "This is an english text"
}

PUT /my-index/doc/2?pipeline=langdetect-analyzer-pipeline
{
  "my_field" : "Das hier ist ein deutscher Text."
}

GET my-index/doc/1

GET my-index/doc/2
```

## Configuration

| Parameter | Use |
| --- | --- |
| field          | Field name of where to read the content from |
| target_field   | Field name to write the language to |
| max_length     | Max length of of characters to read, defaults to 10kb, requires a byte size value, like 1mb |
| ignore_missing | Ignore missing source field. Not throwing exception in that case. Expects for boolean value, defaults to false. |
| 2.1.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.1.0/opensearch-analysis-pinyin.zip` |
| 2.2.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.2.0/opensearch-analysis-pinyin.zip` |
| 2.3.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.3.0/opensearch-analysis-pinyin.zip` |
| 2.4.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.4.0/opensearch-analysis-pinyin.zip` |
| 2.6.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.6.0/opensearch-analysis-pinyin.zip` |
| 2.7.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.7.0/opensearch-analysis-pinyin.zip` |
| 2.8.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.8.0/opensearch-analysis-pinyin.zip` |
| 2.9.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.9.0/opensearch-analysis-pinyin.zip` |
| 2.10.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.10.0/opensearch-analysis-pinyin.zip` |
| 2.11.0 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.11.0/opensearch-analysis-pinyin.zip` |
| 2.11.1 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-analysis-pinyin/releases/download/2.11.1/opensearch-analysis-pinyin.zip` |
| 2.11.1 | `bin/opensearch-plugin install https://github.com/aparo/opensearch-ingest-langdetect/releases/download/2.11.1/opensearch-ingest-langdetect.zip` |

## Setup

In order to install this plugin, you need to create a zip distribution first by running

```bash
gradle clean check
```

This will produce a zip file in `build/distributions`.

After building the zip file, you can install it like this

```bash
bin/plugin install file:///path/to/ingest-langdetect/build/distribution/ingest-langdetect-0.0.1-SNAPSHOT.zip
```

## Side notes

In order to cope with the security manager, a special factory is used to load the languages from the classpath.
You can check out the `SecureDetectorFactory` class. This implementation also does not use jsonic to prevent the use of reflection when loading the languages.
