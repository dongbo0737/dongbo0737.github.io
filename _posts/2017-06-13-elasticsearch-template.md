---
layout: post
title:  "Elasticsearch 索引模版"
date:   2017-06-13
categories: elasticsearch
tags: elasticsearch template
---

* content
{:toc}

使用Elasticsearch 存储数据的时候，每个单独的索引，或者一个类型的索引都可能有些自己特殊的属性，这样我们就要使用template了
通过指定指定的模版名字，来定义这个类型的索引多少个分片和副本，哪些字段不需要分词等等
使用*可以模糊匹配索引.

例如：
business-*  可以匹配到 business-2017.05.01






## business template

	{
	  "order": 0,
	  "template": "ule-business-*",
	  "settings": {
	    "index": {
	      "routing": {
	        "allocation": {
	          "require": {
	            "box_type": "hot"
	          }
	        }
	      },
	      "refresh_interval": "3s",
	      "analysis": {
	        "filter": {
	          "camel_filter": {
	            "split_on_numerics": "false",
	            "type": "word_delimiter",
	            "stem_english_possessive": "false",
	            "generate_number_parts": "false",
	            "protected_words": [
	              "iPhone",
	              "WiFi"
	            ]
	          }
	        },
	        "analyzer": {
	          "camel_analyzer": {
	            "filter": [
	              "camel_filter",
	              "lowercase"
	            ],
	            "char_filter": [
	              "html_strip"
	            ],
	            "type": "custom",
	            "tokenizer": "ik_smart"
	          }
	        },
	        "tokenizer": {
	          "my_tokenizer": {
	            "type": "whitespace",
	            "enable_lowercase": "false"
	          }
	        }
	      },
	      "number_of_shards": "30",
	      "number_of_replicas": "0"
	    }
	  },
	  "mappings": {
	    "_default_": {
	      "dynamic_templates": [
	        {
	          "message_field": {
	            "mapping": {
	              "analyzer": "camel_analyzer",
	              "index": "analyzed",
	              "omit_norms": true,
	              "type": "string"
	            },
	            "match_mapping_type": "string",
	            "match": "message"
	          }
	        },
	        {
	          "logid_field": {
	            "mapping": {
	              "type": "long"
	            },
	            "match_mapping_type": "string",
	            "match": "logid"
	          }
	        },
	        {
	          "string_fields": {
	            "mapping": {
	              "index": "not_analyzed",
	              "omit_norms": true,
	              "type": "string",
	              "fields": {
	                "raw": {
	                  "ignore_above": 256,
	                  "index": "not_analyzed",
	                  "type": "string"
	                }
	              }
	            },
	            "match_mapping_type": "string",
	            "match": "*"
	          }
	        }
	      ],
	      "_all": {
	        "omit_norms": true,
	        "enabled": true
	      },
	      "properties": {
	        "geoip": {
	          "dynamic": true,
	          "type": "object",
	          "properties": {
	            "location": {
	              "type": "geo_point"
	            }
	          }
	        },
	        "@version": {
	          "index": "not_analyzed",
	          "type": "string"
	        }
	      }
	    }
	  },
	  "aliases": {}
	}

## webaccess template

	{
	  "order": 0,
	  "template": "ule-webaccess-*",
	  "settings": {
	    "index": {
	      "routing": {
	        "allocation": {
	          "require": {
	            "box_type": "hot"
	          }
	        }
	      },
	      "refresh_interval": "3s",
	      "analysis": {
	        "analyzer": {
	          "ik": {
	            "type": "custom",
	            "tokenizer": "ik_smart"
	          }
	        }
	      },
	      "number_of_shards": "20",
	      "number_of_replicas": "0"
	    }
	  },
	  "mappings": {
	    "_default_": {
	      "dynamic_templates": [
	        {
	          "message_field": {
	            "mapping": {
	              "analyzer": "ik",
	              "index": "analyzed",
	              "omit_norms": true,
	              "type": "string"
	            },
	            "match_mapping_type": "string",
	            "match": "message"
	          }
	        },
	        {
	          "bytes_field": {
	            "mapping": {
	              "type": "long"
	            },
	            "match_mapping_type": "string",
	            "match": "bytes"
	          }
	        },
	        {
	          "string_fields": {
	            "mapping": {
	              "index": "not_analyzed",
	              "omit_norms": true,
	              "type": "string",
	              "fields": {
	                "raw": {
	                  "ignore_above": 256,
	                  "index": "not_analyzed",
	                  "type": "string"
	                }
	              }
	            },
	            "match_mapping_type": "string",
	            "match": "*"
	          }
	        }
	      ],
	      "_all": {
	        "omit_norms": true,
	        "enabled": true
	      },
	      "properties": {
	        "geoip": {
	          "dynamic": true,
	          "type": "object",
	          "properties": {
	            "location": {
	              "type": "geo_point"
	            }
	          }
	        },
	        "@version": {
	          "index": "not_analyzed",
	          "type": "string"
	        }
	      }
	    }
	  },
	  "aliases": {}
	}