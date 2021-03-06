# Elasticsearch `analysis-emoji` plugin

This plugin create a new Tokenizer called `emoji_tokenizer` based on `icu_tokenizer` and the ICU 59.1.

## Installation

Make sure the change the version number in the download URL, or just copy and paste the latest zip full URL from the [release page](https://github.com/jolicode/emoji-search/releases). 

The plugin version must match your Elasticsearch version.

```
bin/elasticsearch-plugin install URL

# For 6.2.4:
bin/elasticsearch-plugin install https://github.com/jolicode/emoji-search/releases/download/6.2.4/analysis-emoji-6.2.4.zip
```

## Versions

ICU is _always_ up to date to the latest data in this plugin, so upgrading may require you to re-index your data.

We _may_ jump some Elasticsearch release from time to time... feel free to open an issue if you need one.

analysis-emoji version and ES version  | Install URL
-----------|-----------
6.2.4 | https://github.com/jolicode/emoji-search/releases/download/6.2.4/analysis-emoji-6.2.4.zip
6.1.3 | https://github.com/jolicode/emoji-search/releases/download/6.1.3/analysis-emoji-6.1.3.zip
6.1.2 | https://github.com/jolicode/emoji-search/releases/download/6.1.2/analysis-emoji-6.1.2.zip
5.6.6 | https://github.com/jolicode/emoji-search/releases/download/5.6.6/analysis-emoji-5.6.6.zip
5.5.0 | https://github.com/jolicode/emoji-search/releases/download/5.5.0/analysis-emoji-5.5.0.zip
5.4.3 | https://github.com/jolicode/emoji-search/releases/download/5.4.3/analysis-emoji-5.4.3.zip
5.3.3 **(ICU 59.1)** | https://github.com/jolicode/emoji-search/releases/download/5.3.3/analysis-emoji-5.3.3.zip
5.3.0 | https://github.com/jolicode/emoji-search/releases/download/5.3.0/analysis-emoji-5.3.0.zip
5.2.2 | https://github.com/jolicode/emoji-search/releases/download/5.2.2.1/analysis-emoji-5.2.2.1.zip
5.2.1 | https://github.com/jolicode/emoji-search/releases/download/5.2.1/analysis-emoji-5.2.1.zip
5.2.0 | https://github.com/jolicode/emoji-search/releases/download/5.2.0/analysis-emoji-5.2.0.zip
5.1.2 **(ICU 58.2)** | https://github.com/jolicode/emoji-search/releases/download/5.1.2/analysis-emoji-5.1.2.zip
5.1.1 | https://github.com/jolicode/emoji-search/releases/download/5.1.1/analysis-emoji-5.1.1.zip
5.0.2 | https://github.com/jolicode/emoji-search/releases/download/5.0.2/analysis-emoji-5.0.2.zip
5.0.1 | https://github.com/jolicode/emoji-search/releases/download/5.0.1/analysis-emoji-5.0.1.zip
5.0.0 **(ICU 58.1)** | https://github.com/jolicode/emoji-search/releases/download/5.0.0/analysis-emoji-5.0.0.zip

## How to use

Build your own analyzer and use the new tokenizer. Download the emoji and emoticon file you want from this repository and store them in `PATH_ES/config/analysis`.

```
config
├── analysis
│   ├── cldr-emoji-annotation-synonyms-en.txt
│   └── emoticons.txt
├── elasticsearch.yml
...
```

And build an analyzer:

```json
PUT /en-emoji
{
  "settings": {
    "analysis": {
      "char_filter": {
        "emoticons_char_filter": {
          "type": "mapping",
          "mappings_path": "analysis/emoticons.txt"
        }
      },
      "filter": {
        "english_emoji": {
          "type": "synonym",
          "synonyms_path": "analysis/cldr-emoji-annotation-synonyms-en.txt" 
        }
      },
      "analyzer": {
        "english_with_emoji": {
          "char_filter": ["emoticons_char_filter"],
          "tokenizer": "emoji_tokenizer",
          "filter": [
            "lowercase",
            "english_emoji"
          ]
        }
      }
    }
  }
}
```

Try it:

```json
GET /en-emoji/_analyze
{
  "analyzer": "english_with_emoji",
  "text": "I live in 🇫🇷 and I'm 👩‍🚀"
}
# Result: i live in 🇫🇷 france and i'm 👩‍🚀 astronaut rocket woman

GET /en-emoji/_analyze
{
  "analyzer": "english_with_emoji",
  "text": "Hi mom :)"
}
# Result:  hi mom 😃 face mouth open smile
```

## How to contribute / build

Building an Elasticsearch plugin is a mess and a pain. Here are some tips:

- follow [this example](https://github.com/spinscale/cookiecutter-elasticsearch-ingest-processor), it's better than the non existing documentation;
- install Gradle on your system, make sure you know how to switch to multiple versions;

Run the tests:

```
gradle clean check
```

Package for release:

```
gradle updateSHAs
gradle clean check assemble
```
