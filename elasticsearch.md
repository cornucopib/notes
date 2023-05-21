# Elasticsearch

Elasticsearch学习总结笔记



## 安装Elasticsearch

### 下载安装包

进入Elasticsearch官网：https://www.elastic.co/cn/downloads/past-releases#elasticsearch, 如下图：

![下载Kibana](/Users/cornucopib/Documents/develop/java/es/notes/resources/下载Kibana.png)

### 指定Elasticsearch的jdk版本

Elasticsearch启动时，会根据`/bin/elastchsearch-env`文件中设置的Java环境变量指定Elasticsearch的jdk版本。

```shell
if [ ! -z "$ES_JAVA_HOME" ]; then
  JAVA="$ES_JAVA_HOME/bin/java"
  JAVA_TYPE="ES_JAVA_HOME"
elif [ ! -z "$JAVA_HOME" ]; then
  # fallback to JAVA_HOME
  echo "warning: usage of JAVA_HOME is deprecated, use ES_JAVA_HOME" >&2
  JAVA="$JAVA_HOME/bin/java"
  JAVA_TYPE="JAVA_HOME"
else
  # use the bundled JDK (default)
  if [ "$(uname -s)" = "Darwin" ]; then
    # macOS has a different structure
    JAVA="$ES_HOME/jdk.app/Contents/Home/bin/java"
  else
    JAVA="$ES_HOME/jdk/bin/java"
  fi
  JAVA_TYPE="bundled JDK"
fi

if [ ! -x "$JAVA" ]; then
  echo "could not find java in $JAVA_TYPE at $JAVA" >&2
  exit 1
fi
```

依次读取：

- ES_JAVA_HOME
- JAVA_HOME
- ES_HOME

一般情况下，我们使用Elasticsearch根目录下提供的jdk即可。我们可以修改上述shell脚本，将上述三个环境变量任意一个，指定为根目录下的jdk即可。mac系统，举个例子：

- 设置`ES_HOME`环境变量为ES的根目录

  ```
  vim ~/.bash_profile
  ```

  增加环境变量

  ```
  export ES_HOME=/Users/cornucopib/Documents/develop/java/es/install_package/elasticsearch-7.17.9
  ```

  ```
  source ~/.bash_profile
  ```

### 修改Elastichsearch配置文件

```
vim ${ES_HOME}/config/elasticsearch.yml
```

### 启动Elasticsearch

```
${ES_HOME}/bing/elasticsearch
```



## 安装Kibana

### 下载Kibana

进入:https://www.elastic.co/cn/downloads/past-releases#elasticsearch,选择指定版本的Kibana。

![下载Kibana](/Users/cornucopib/Documents/develop/java/es/notes/resources/下载Kibana.png)

### 修改Kibana配置文件

`vim ${KIBANA_HOME}/config/kibana.yml`

设置中文

```
i18n.locale: "zh-CN" 
```

### 启动Kibana

```
${KIBANA_HOME}/bin/kibana
```



## Elasticsearch安装插件

Elasticsearch提供了在线安装和离线安装。

### 在线安装

下面以安装`analysis-icu`插件为例说明。

```shell
#查看已安装插件
${ES_HOME}/bin/elasticsearch-plugin list
#安装插件
${ES_HOME}/bin/elasticsearch-plugin install analysis-icu
#卸载插件
${ES_HOME}/bin/elasticsearch-plugin remove analysis-icu
```

### 离线安装

本地下载相应插件，解压，传入到elasticsearch的根目录下的plugins目录，然后重启ES。

下面以安装`elasticsearch-analysis-ik`为例说明：

```
https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.17.6/elasticsearch-analysis-ik-7.17.6.zip
```

下载后，解压到`${ES_HOME}/plugins`,然后重启。



### 插件使用

创建索引时，指定ik分词器作为默认分词器

```
post {{es_url}}/es_db
{
 "settings":{
   "index":{
     "analysis.analyzer.default.type":"ik_max_word"
   }
 }
}

```

查询指定分词

默认分词

```
{{es_url}}/_analyze

{
    "analyzer":"standard",
    "text":"中华人民共和国"
}
```

ik插件分词，最粗粒度分词

```
{{es_url}}/_analyze

{
    "analyzer":"ik_smart",
    "text":"中华人民共和国"
}
```

ik插件分词，最细粒度分词

```
{{es_url}}/_analyze

{
    "analyzer":"ik_max_word",
    "text":"中华人民共和国"
}
```



## Elasticsearch基本概念

Elasticsearch官方文档，可以参考[网址](https://www.elastic.co/guide/en/elasticsearch/reference/7.16/index.html)



## Elasticsearch增删改查

下面介绍一下Elasticsearch的增删改查



## 新建索引

```
PUT /class
{
  "mappings": {
    "properties": {
      "class_name": {
        "type": "text"
      },
      "student_count": {
        "type": "integer"
      },
      "members": {
        "type": "text"
      },
      "start_date": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "students": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text"
          },
          "age": {
            "type": "integer"
          },
          "birthdate": {
            "type": "date",
            "format": "yyyy-MM-dd HH:mm:ss"
          },
          "group": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

## 更新索引

```
# 增加新的索引
PUT /class
{
  "mappings": {
    "properties": {
      "class_name": {
        "type": "text"
      },
      "student_count": {
        "type": "integer"
      },
      "members": {
        "type": "text"
      },
      "start_date": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "students": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text"
          },
          "age": {
            "type": "integer"
          },
          "birthdate": {
            "type": "date",
            "format": "yyyy-MM-dd HH:mm:ss"
          },
          "group": {
            "type": "text"
          }
        }
      }
    }
  }
}

#将索引数据迁移到新的
POST /_reindex
{
  "source": {
    "index": "class_index"
  },
  "dest": {
    "index":"class"
  }
}

#删除旧的索引
DELETE /class_index
```

## 删除索引

```
DELETE /_all
DELETE /class_index
```



## 插入数据

```
POST /class/_doc
{
  "name": "Class A",
  "student_count": 30,
  "start_date": "2022-01-01 00:00:00",
  "students": [
    {
      "name": "John",
      "age": 20,
      "birthdate": "1990-05-15 00:00:00"
    },
    {
      "name": "Alice",
      "age": 22,
      "birthdate": "1999-09-30 00:00:00"
    },
    {
      "name": "Mike",
      "age": 21,
      "birthdate": "1998-03-10 00:00:00"
    }
  ]
}
```

## 批量插入数据

```
POST /class/_bulk
{ "index" : {} }
{ "class_name": "Class A", "student_count": 25, "members": "John Doe, Jane Smith, Alex Johnson", "start_date": "2022-01-15 09:00:00", "students": [{ "name": "John Doe", "age": 18, "birthdate": "2004-03-10 00:00:00", "group": "A,B" }, { "name": "Jane Smith", "age": 17, "birthdate": "2005-07-21 00:00:00", "group": "B,C" }, { "name": "Alex Johnson", "age": 19, "birthdate": "2003-11-05 00:00:00", "group": "C" }] }
{ "index" : {} }
{ "class_name": "Class B", "student_count": 30, "members": "Emily Brown, Michael Wilson, Olivia Davis", "start_date": "2022-02-20 10:30:00", "students": [{ "name": "Emily Brown", "age": 17, "birthdate": "2005-01-07 00:00:00", "group": "B" }, { "name": "Michael Wilson", "age": 18, "birthdate": "2004-08-14 00:00:00", "group": "C" }, { "name": "Olivia Davis", "age": 16, "birthdate": "2006-04-30 00:00:00", "group": "A" }] }
{ "index" : {} }
{ "class_name": "Class C", "student_count": 20, "members": "William Johnson, Sophia Miller, Ethan Anderson", "start_date": "2022-03-25 13:15:00", "students": [{ "name": "William Johnson", "age": 19, "birthdate": "2003-12-18 00:00:00", "group": "C" }, { "name": "Sophia Miller", "age": 17, "birthdate": "2005-05-02 00:00:00", "group": "A,B" }, { "name": "Ethan Anderson", "age": 18, "birthdate": "2004-09-28 00:00:00", "group": "B,C" }] }
{ "index" : {} }
{ "class_name": "Class D", "student_count": 22, "members": "Daniel Lee, Olivia Johnson, James Brown", "start_date": "2022-04-10 14:45:00", "students": [{ "name": "Daniel Lee", "age": 17, "birthdate": "2005-09-12 00:00:00", "group": "A,B,C" }, { "name": "Olivia Johnson", "age": 18, "birthdate": "2004-02-28 00:00:00", "group": "B,C" }, { "name": "James Brown", "age": 16, "birthdate": "2006-06-16 00:00:00", "group": "A,B" }] }
```

## 删除数据

```
#根据条件删除
POST /class/_delete_by_query
{
  "query": {
    "term": {
      "name.keyword": "ClassC"
    }
  }
}

#清理索引下所有数据
POST /class/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

更新数据

```
# 根据索引名称以及数据id来更新
POST _bulk
{"update":{"_index":"class","_id":"3"}}
{"doc":{"name":"刘达达"}}

# 根据条件更新
POST /class/_update_by_query
{
  "query": {
    "term": {
      "name.keyword": "Class A"
    }
  },
  "script": {
    "source": "ctx._source.name = 'Class 1'"
  }
}

```

## 查询索引全部的数据

```
GET /class/_search
{
  "query": {
    "match_all": {}
  }
}
```

## 日期时间搜索

```
GET /class/_search
{
  "query": {
    "range": {
      "start_date": {
        "gte": "2023-01-01 00:00:00",
        "lte": "now"
      }
    }
  }
}
```



## 模糊查询

```
GET /my_index/_search
{
  "query": {
    "match": {
      "class_name": {
        "query": "A",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

## 将字段进行分词搜索

### 设置字段分词

```
PUT /class
{
  "settings": {
    "analysis": {
      "analyzer": {
        "comma_analyzer": {
          "type": "custom",
          "tokenizer": "comma_tokenizer"
        }
      },
      "tokenizer": {
        "comma_tokenizer": {
          "type": "pattern",
          "pattern": ","
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "class_name": {
        "type": "text"
      },
      "student_count": {
        "type": "integer"
      },
      "members": {
        "type": "text",
        "analyzer": "comma_analyzer"
      },
      "start_date": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "students": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text",
            "analyzer": "comma_analyzer"
          },
          "age": {
            "type": "integer"
          },
          "birthdate": {
            "type": "date",
            "format": "yyyy-MM-dd HH:mm:ss"
          },
          "group": {
            "type": "text",
            "analyzer": "comma_analyzer"
          }
        }
      }
    }
  }
}

```

### 创建数据

```
```

### 进行分词搜索

```
GET /class/_search
{
    "query": {
        "bool":{
            "filter": [
                {
                    "match": {
                        "members": "Emily Brown"
                    }
                },
                {
                    "nested": {
                        "path": "students",
                        "query": {
                            "match": {
                                "students.group": "A"
                            }
                        }
                    }
                }
            ]
        }
    }
}
```

## 查询字段分词

```
GET /class/_analyze
{
  "field": "members",
  "text": "James Brown,Olivia Johnson,Daniel Lee"
}
```

## 非相关性评分搜索过滤

1. 使用 `filter` 查询：`filter` 查询只关注文档的匹配与否，不进行相关性评分。这适用于需要快速筛选匹配文档的场景，例如精确的过滤查询或布尔逻辑组合查询。它可以提高搜索性能，因为筛选器可以被缓存并重复使用，避免了相关性评分的计算。然而，由于不进行相关性评分，无法对结果进行排序。

   ```
   GET /index/_search
   {
     "query": {
       "bool": {
         "filter": {
           "term": {
             "field": "value"
           }
         }
       }
     }
   }
   ```



2. 使用 `constant_score` 查询：`constant_score` 查询将所有匹配的文档分配相同的固定分数，忽略相关性评分。这适用于只需要匹配文档而不关心相关性排序的场景，例如精确匹配或过滤查询。它可以确保所有匹配的文档获得相同的分数，方便筛选和过滤。

   ```
   GET /index/_search
   {
     "query": {
       "constant_score": {
         "filter": {
           "term": {
             "field": "value"
           }
         }
       }
     }
   }
   ```

   

3. 使用 `function_score` 查询并设置相关性评分部分为零：`function_score` 查询允许您自定义评分函数，并将相关性评分与其他评分函数结合。在不关心相关性排序的场景下，您可以将相关性评分部分设置为零，以便进行非相关性评分搜索。

   ````
   GET /index/_search
   {
     "query": {
       "function_score": {
         "query": {
           "match": {
             "field": "value"
           }
         },
         "functions": [
           {
             "filter": {
               "term": {
                 "other_field": "other_value"
               }
             },
             "weight": 0  // 设置相关性评分部分为零
           }
         ],
         "score_mode": "sum"  // 使用和模式将评分函数结果相加
       }
     }
   }
   ````

   

## 对字符串进行单字符分词，并进行模糊匹配查询

```
# 对字段设置单字符分词
PUT /class1
{
  "settings": {
    "analysis": {
      "analyzer": {
        "ngram_analyzer": {
          "tokenizer": "ngram_tokenizer"
        }
      },
      "tokenizer": {
        "ngram_tokenizer": {
          "type": "ngram",
          "min_gram": 1,
          "max_gram": 1
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "class_name": {
        "type": "text",
        "analyzer": "ngram_analyzer"
      }
    }
  }
}

# 增加模拟数据
POST /class1/_bulk
{ "index" : {} }
{ "class_name": "刘伟"}
{ "index" : {} }
{ "class_name": "刘杰"}
{ "index" : {} }
{ "class_name": "abc"}
{ "index" : {} }
{ "class_name": "abc123"}


# 进行模糊匹配
GET /class1/_search
{
  "query": {
    "match": {
      "class_name": {
        "query": "杰",
        "fuzziness": "auto"
      }
    }
  }
}

# 非相关性模糊匹配
{
    "query": {
        "bool": {
            "filter": [
                {
                    "match": {
                        "class_name": {
                            "query": "伟",
                            "fuzziness": "auto"
                        }
                    }
                }
            ]
        }
    }
}
```

