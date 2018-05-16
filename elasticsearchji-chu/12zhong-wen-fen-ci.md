# 1.2中文分词

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
  "analyzer": "ik_max_word",       #字段的文本进行分词的分词器
  "search_analyzer": "ik_max_word" #搜索词进行分词的分词器
}
```


ik_max_word 和 ik_smart 什么区别?
>ik_max_word: 会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合；
ik_smart: 会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”。


详细使用参考：
[GitHub开源ik分词：elasticsearch-analysis-ik中文分词](https://github.com/medcl/elasticsearch-analysis-ik)