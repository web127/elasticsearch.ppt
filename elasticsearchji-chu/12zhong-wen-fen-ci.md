Elasticsearch中，内置了很多分词器（analyzers），例如standard （标准分词器）、english （英文分词）和chinese （中文分词，但是分词效果不好）


安装中文分词ik分词器

```
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.2.2/elasticsearch-analysis-ik-6.2.2.zip
```
上面的版本必须和elasticsearch版本一致

Elastic 的分词器称为 analyzer。我们对每个字段指定分词器。
如：

```
"user": {
  "type": "text",
  "analyzer": "ik_max_word",
  "search_analyzer": "ik_max_word"
}
```

参考

[GitHub开源ik分词地址：elasticsearch-analysis-ik中文分词](https://github.com/medcl/elasticsearch-analysis-ik)