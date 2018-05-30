添加pom文件

```
  <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>6.2.2</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.plugin</groupId>
            <artifactId>transport-netty4-client</artifactId>
            <version>6.2.2</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>transport</artifactId>
            <version>6.2.2</version>
            <exclusions>
                <exclusion>
                    <groupId>org.elasticsearch.plugin</groupId>
                    <artifactId>transport-netty4-client</artifactId>
                </exclusion>
            </exclusions>
 </dependency>
```
application.yml配置

```
#elasticsearch config
  data:
    elasticsearch:
       cluster-name: docker_es
       cluster-nodes: 172.16.16.179
       transport: 9300  #自定义属性
       search-sizes: 20 #线程池个数
```

集群连接初始化配置类:
package com.zczy.cloud.config;
/**
 * @Date：2018/4/8 9:15
 */


import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.transport.TransportAddress;
import org.elasticsearch.xpack.client.PreBuiltXPackTransportClient;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.net.InetAddress;

/**
 * @Author: 查恒
 * @Date: 2018/4/8 09:15
 * @Description:
 */
@Configuration
public class ElasticSearchConfig {
    private static final Logger LOGGER = LoggerFactory.getLogger(ElasticSearchConfig.class);

    @Value("${spring.data.elasticsearch.cluster-nodes}")
    private String clusterNodes ;
    @Value("${spring.data.elasticsearch.cluster-name}")
    private String clusterName;
    @Value("${spring.data.elasticsearch.x-pack}")
    private String xPack;
    @Value("${spring.data.elasticsearch.transport}")
    private int transport;
    @Value("${spring.data.elasticsearch.search-sizes}")
    private int searchSizes;

    @Bean
    public TransportClient init() {
        LOGGER.info("初始化开始。。。。。");
        TransportClient transportClient = null;
        try {
            // 配置信息
        esSetting = Settings.builder()
                    //3个主分片
                    //.put("number_of_shards", 3)
                    //副本分片
                    //.put("number_of_replicas", 2)
                    .put("client.transport.sniff", true)//增加嗅探机制，找到ES集群
                    .put("thread_pool.search.size", searchSizes)//增加线程池个数，暂时设为20
                    .put("cluster.name", clusterName)
                    //.put("xpack.security.transport.ssl.enabled", false)
                    //.put("xpack.security.user", xPack)
                    .build();
            //配置信息Settings自定义,下面设置为EMPTY
            transportClient = new PreBuiltXPackTransportClient(esSetting).addTransportAddress(new TransportAddress(InetAddress.getByName(clusterNodes), transport));
        } catch (Exception e) {
            LOGGER.error("elasticsearch TransportClient create error!!!", e);
        }

        return transportClient;
    }
}

```

javaClient操作ES的增刪改查:

```
package com.zczy.cloud.common;
/**
 * @Date：2018/4/9 11:24
 */

import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.zczy.cloud.base.QueryEntity;
import org.apache.commons.lang3.StringUtils;
import org.elasticsearch.action.admin.indices.create.CreateIndexResponse;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexResponse;
import org.elasticsearch.action.admin.indices.exists.indices.IndicesExistsRequest;
import org.elasticsearch.action.admin.indices.exists.indices.IndicesExistsResponse;
import org.elasticsearch.action.delete.DeleteResponse;
import org.elasticsearch.action.get.GetRequestBuilder;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.search.SearchRequestBuilder;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.search.SearchType;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.action.update.UpdateResponse;
import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.text.Text;
import org.elasticsearch.index.query.BoolQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.rest.RestStatus;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightBuilder;
import org.elasticsearch.search.sort.SortOrder;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.util.*;

/**
 * @Author: 查恒
 * @Date: 2018/4/9 11:24
 * @Description:
 */
@Component
public class ElasticsearchUtils {
    private static final Logger LOGGER = LoggerFactory.getLogger(ElasticsearchUtils.class);

    @Autowired
    private TransportClient transportClient;

    private static TransportClient client;

    @PostConstruct
    public void init() {
        client = this.transportClient;
    }

    /**
     * 创建索引
     *
     * @param index
     * @return
     */
    public static boolean createIndex(String index) {
        if (!isIndexExist(index)) {
            LOGGER.info("Index is not exits!");
        }
        CreateIndexResponse indexresponse = client.admin().indices().prepareCreate(index).execute().actionGet();
        LOGGER.info("执行建立成功？" + indexresponse.isAcknowledged());

        return indexresponse.isAcknowledged();
    }

    /**
     * 删除索引
     *
     * @param index
     * @return
     */
    public static boolean deleteIndex(String index) {
        if (!isIndexExist(index)) {
            LOGGER.info("Index is not exits!");
        }
        DeleteIndexResponse dResponse = client.admin().indices().prepareDelete(index).execute().actionGet();
        if (dResponse.isAcknowledged()) {
            LOGGER.info("delete index " + index + "  successfully!");
        } else {
            LOGGER.info("Fail to delete index " + index);
        }
        return dResponse.isAcknowledged();
    }

    /**
     * 判断索引是否存在
     *
     * @param index
     * @return
     */
    public static boolean isIndexExist(String index) {
        IndicesExistsResponse inExistsResponse = client.admin().indices().exists(new IndicesExistsRequest(index)).actionGet();
        if (inExistsResponse.isExists()) {
            LOGGER.info("Index [" + index + "] is exist!");
        } else {
            LOGGER.info("Index [" + index + "] is not exist!");
        }
        return inExistsResponse.isExists();
    }

    /**
     * 数据添加，正定ID
     *
     * @param jsonObject 要增加的数据
     * @param index      索引，类似数据库
     * @param type       类型，类似表
     * @param id         数据ID
     * @return
     */
    public static String addData(JSONObject jsonObject, String index, String type, String id) {

        IndexResponse response = client.prepareIndex(index, type, id).setSource(jsonObject).get();

        LOGGER.info("addData response status:{},id:{}", response.status().getStatus(), response.getId());

        return response.getId();
    }

    /**
     * 数据添加
     *
     * @param jsonObject 要增加的数据
     * @param index      索引，类似数据库
     * @param type       类型，类似表
     * @return
     */
    public static String addData(JSONObject jsonObject, String index, String type) {
        return addData(jsonObject, index, type, UUID.randomUUID().toString().replaceAll("-", "").toUpperCase());
    }

    /**
     * 通过ID删除数据
     *
     * @param index 索引，类似数据库
     * @param type  类型，类似表
     * @param id    数据ID
     */
    public static void deleteDataById(String index, String type, String id) {

        DeleteResponse response = client.prepareDelete(index, type, id).execute().actionGet();
        LOGGER.info("deleteDataById response status:{},id:{}", response.status().getStatus(), response.getId());
    }


    /**
     * 通过ID 更新数据
     *
     * @param jsonObject 要增加的数据
     * @param index      索引，类似数据库
     * @param type       类型，类似表
     * @param id         数据ID
     * @return
     */
    public static boolean updateDataById(JSONObject jsonObject, String index, String type, String id) {
        try {
            UpdateRequest updateRequest = new UpdateRequest();

            updateRequest.index(index).type(type).id(id).doc(jsonObject);

            UpdateResponse response = client.update(updateRequest).get();
            if (response.status() == RestStatus.OK) {
                return true;
            } else {
                return false;
            }
        } catch (Exception e) {
            LOGGER.error("错误更新Index {},Type{},Id {}，已存在!", index, type, id);
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 通过ID获取数据
     *
     * @param index  索引，类似数据库
     * @param type   类型，类似表
     * @param id     数据ID
     * @param fields 需要显示的字段，逗号分隔（缺省为全部字段）
     * @return
     */
    public static Map<String, Object> searchDataById(String index, String type, String id, String fields) {

        GetRequestBuilder getRequestBuilder = client.prepareGet(index, type, id);

        if (StringUtils.isNotEmpty(fields)) {
            getRequestBuilder.setFetchSource(fields.split(","), null);
        }

        GetResponse getResponse = getRequestBuilder.execute().actionGet();

        return getResponse.getSource();
    }


    /**
     * 使用分词查询
     *
     * @param index    索引名称
     * @param type     类型名称,可传入多个type逗号分隔
     * @param fields   需要显示的字段，逗号分隔（缺省为全部字段）
     * @param matchStr 过滤条件（xxx=111,aaa=222）
     * @return
     */
    public static List<Map<String, Object>> searchListData(String index, String type, String fields, String matchStr) {
        return searchListData(index, type, 0, 0, null, fields, null, false, null, matchStr);
    }

    /**
     * 使用分词查询
     *
     * @param index       索引名称
     * @param type        类型名称,可传入多个type逗号分隔
     * @param fields      需要显示的字段，逗号分隔（缺省为全部字段）
     * @param sortField   排序字段
     * @param matchPhrase true 使用，短语精准匹配
     * @param matchStr    过滤条件（xxx=111,aaa=222）
     * @return
     */
    public static List<Map<String, Object>> searchListData(String index, String type, String fields, String sortField, boolean matchPhrase, String matchStr) {
        return searchListData(index, type, 0, 0, null, fields, sortField, matchPhrase, null, matchStr);
    }


    /**
     * 使用分词查询
     *
     * @param index          索引名称
     * @param type           类型名称,可传入多个type逗号分隔
     * @param size           文档大小限制
     * @param fields         需要显示的字段，逗号分隔（缺省为全部字段）
     * @param sortField      排序字段
     * @param matchPhrase    true 使用，短语精准匹配
     * @param highlightField 高亮字段
     * @param matchStr       过滤条件（xxx=111,aaa=222）
     * @return
     */
    public static List<Map<String, Object>> searchListData(String index, String type, Integer size, String fields, String sortField, boolean matchPhrase, String highlightField, String matchStr) {
        return searchListData(index, type, 0, 0, size, fields, sortField, matchPhrase, highlightField, matchStr);
    }


    /**
     * 使用分词查询
     *
     * @param index          索引名称
     * @param type           类型名称,可传入多个type逗号分隔
     * @param startTime      开始时间
     * @param endTime        结束时间
     * @param size           文档大小限制
     * @param fields         需要显示的字段，逗号分隔（缺省为全部字段）
     * @param sortField      排序字段
     * @param matchPhrase    true 使用，短语精准匹配
     * @param highlightField 高亮字段
     * @param matchStr       过滤条件（xxx=111,aaa=222）
     * @return
     */
    public static List<Map<String, Object>> searchListData(String index, String type, long startTime, long endTime, Integer size, String fields, String sortField, boolean matchPhrase, String highlightField, String matchStr) {

        SearchRequestBuilder searchRequestBuilder = client.prepareSearch(index);
        if (StringUtils.isNotEmpty(type)) {
            searchRequestBuilder.setTypes(type.split(","));
        }
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();

        if (startTime > 0 && endTime > 0) {
            boolQuery.must(QueryBuilders.rangeQuery("processTime")
                    .format("epoch_millis")
                    .from(startTime)
                    .to(endTime)
                    .includeLower(true)
                    .includeUpper(true));
        }

        //搜索的的字段
        if (StringUtils.isNotEmpty(matchStr)) {
            for (String s : matchStr.split(",")) {
                String[] ss = s.split("=");
                if (ss.length > 1) {
                    if (matchPhrase == Boolean.TRUE) {
                        boolQuery.must(QueryBuilders.matchPhraseQuery(s.split("=")[0], s.split("=")[1]));
                    } else {
                        boolQuery.must(QueryBuilders.matchQuery(s.split("=")[0], s.split("=")[1]));
                    }
                }

            }
        }

        // 高亮（xxx=111,aaa=222）
        if (StringUtils.isNotEmpty(highlightField)) {
            HighlightBuilder highlightBuilder = new HighlightBuilder();

            //highlightBuilder.preTags("<span style='color:red' >");//设置前缀
            //highlightBuilder.postTags("</span>");//设置后缀

            // 设置高亮字段
            highlightBuilder.field(highlightField);
            searchRequestBuilder.highlighter(highlightBuilder);
        }


        searchRequestBuilder.setQuery(boolQuery);

        if (StringUtils.isNotEmpty(fields)) {
            searchRequestBuilder.setFetchSource(fields.split(","), null);
        }
        searchRequestBuilder.setFetchSource(true);

        if (StringUtils.isNotEmpty(sortField)) {
            searchRequestBuilder.addSort(sortField, SortOrder.DESC);
        }

        if (size != null && size > 0) {
            searchRequestBuilder.setSize(size);
        }

        //打印的内容 可以在 Elasticsearch head 和 Kibana  上执行查询
        LOGGER.info("\n{}", searchRequestBuilder);

        SearchResponse searchResponse = searchRequestBuilder.execute().actionGet();

        long totalHits = searchResponse.getHits().totalHits;
        long length = searchResponse.getHits().getHits().length;

        LOGGER.info("共查询到[{}]条数据,处理数据条数[{}]", totalHits, length);

        if (searchResponse.status().getStatus() == 200) {
            // 解析对象
            return setSearchResponse(searchResponse, highlightField);
        }

        return null;

    }

    /**
     * 使用分词查询,并分页
     *
     * @param index          索引名称
     * @param type           类型名称,可传入多个type逗号分隔
     * @param currentPage    当前页
     * @param pageSize       每页显示条数
     * @param timeField      时间字段
     * @param startTime      开始时间
     * @param endTime        结束时间
     * @param fields         需要显示的字段，逗号分隔（缺省为全部字段）
     * @param sortField      排序字段
     * @param matchPhrase    true 使用，短语精准匹配
     * @param highlightField 高亮字段
     * @param matchStr       过滤条件（xxx=111,aaa=222）
     * @return
     */
    public static EsPage searchDataPage(String index, String type, int currentPage, int pageSize, String timeField, long startTime, long endTime, String fields, String sortField, boolean matchPhrase, String highlightField, String matchStr) {
        SearchRequestBuilder searchRequestBuilder = client.prepareSearch(index);
        if (StringUtils.isNotEmpty(type)) {
            searchRequestBuilder.setTypes(type.split(","));
        }
        searchRequestBuilder.setSearchType(SearchType.QUERY_THEN_FETCH);

        // 需要显示的字段，逗号分隔（缺省为全部字段）
        if (StringUtils.isNotEmpty(fields)) {
            searchRequestBuilder.setFetchSource(fields.split(","), null);
        }

        //排序字段
        if (StringUtils.isNotEmpty(sortField)) {
            searchRequestBuilder.addSort(sortField, SortOrder.DESC);
        }

        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();

        if (StringUtils.isNoneBlank(timeField)) {
            if (startTime > 0 && endTime > 0) {
                boolQuery.must(QueryBuilders.rangeQuery(timeField)//对于某个字段的范围
                        .format("epoch_millis")
                        .from(startTime)
                        .to(endTime)
                        .includeLower(true)
                        .includeUpper(true));
            }
        }

        // 查询字段
        if (StringUtils.isNotEmpty(matchStr)) {
            for (String s : matchStr.split(",")) {
                String[] ss = s.split("=");
                if (matchPhrase == Boolean.TRUE) {
                    boolQuery.must(QueryBuilders.matchPhraseQuery(s.split("=")[0], s.split("=")[1]));
                } else {
                    boolQuery.must(QueryBuilders.matchQuery(s.split("=")[0], s.split("=")[1]));
                }
            }
        }

        // 高亮（xxx=111,aaa=222）
        if (StringUtils.isNotEmpty(highlightField)) {
            HighlightBuilder highlightBuilder = new HighlightBuilder();

            //highlightBuilder.preTags("<span style='color:red' >");//设置前缀
            //highlightBuilder.postTags("</span>");//设置后缀

            // 设置高亮字段
            highlightBuilder.field(highlightField);
            searchRequestBuilder.highlighter(highlightBuilder);
        }

        searchRequestBuilder.setQuery(QueryBuilders.matchAllQuery());
        searchRequestBuilder.setQuery(boolQuery);

        // 分页应用
        searchRequestBuilder.setFrom(currentPage).setSize(pageSize);

        // 设置是否按查询匹配度排序
        searchRequestBuilder.setExplain(true);

        //打印的内容 可以在 Elasticsearch head 和 Kibana  上执行查询
        LOGGER.info("\n{}", searchRequestBuilder);

        // 执行搜索,返回搜索响应信息
        SearchResponse searchResponse = searchRequestBuilder.execute().actionGet();

        long totalHits = searchResponse.getHits().totalHits;
        long length = searchResponse.getHits().getHits().length;

        LOGGER.debug("共查询到[{}]条数据,处理数据条数[{}]", totalHits, length);

        if (searchResponse.status().getStatus() == 200) {
            // 解析对象
            List<Map<String, Object>> sourceList = setSearchResponse(searchResponse, highlightField);

            return new EsPage(currentPage, pageSize, (int) totalHits, sourceList);
        }

        return null;

    }

    //复杂的分页查询
    public static EsPage searchDataPage(QueryEntity queryEntity) {
        if (queryEntity != null) {
            SearchRequestBuilder searchRequestBuilder = client.prepareSearch(queryEntity.getIndex());
            if (StringUtils.isNotEmpty(queryEntity.getType())) {
                searchRequestBuilder.setTypes(queryEntity.getType().split(","));
            }
            searchRequestBuilder.setSearchType(SearchType.QUERY_THEN_FETCH);

            // 需要显示的字段，逗号分隔（缺省为全部字段）
            if (StringUtils.isNotEmpty(queryEntity.getShowfields())) {
                searchRequestBuilder.setFetchSource(queryEntity.getShowfields().split(","), null);
            }

            //排序字段
            if (StringUtils.isNotEmpty(queryEntity.getSortField())) {
                //转换为json
                JSONArray parse = (JSONArray) JSONArray.parse(queryEntity.getSortField());
                if (parse.size() > 0) {
                    for (int i = 0; i < parse.size(); i++) {
                        JSONObject object = (JSONObject) parse.get(i);
                        Set<Map.Entry<String, Object>> entries = object.entrySet();
                        for (Map.Entry ee : entries) {
                            String sortField = (String) ee.getKey();
                            String sort = (String) ee.getValue();
                            if (sort.equalsIgnoreCase(SortOrder.ASC.toString())) {
                                searchRequestBuilder.addSort(sortField, SortOrder.ASC);
                            }
                            if (sort.equalsIgnoreCase(SortOrder.DESC.toString())) {
                                searchRequestBuilder.addSort(sortField, SortOrder.DESC);
                            }
                        }
                    }
                }
            }

            //查询范围
            BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();

            if (StringUtils.isNoneBlank(queryEntity.getRangField())) {
                //转换为json
                JSONArray parse = (JSONArray) JSONArray.parse(queryEntity.getRangField());
                if (parse.size() > 0) {
                    for (int i = 0; i < parse.size(); i++) {
                        JSONObject object = (JSONObject) parse.get(i);
                        String field = (String) object.get("field");
                        String start = (String) object.get("start");
                        String end = (String) object.get("end");
                        if (StringUtils.isNotBlank(field) && StringUtils.isNotBlank(start) && StringUtils.isNotBlank(end)) {
                            boolQuery.must(QueryBuilders.rangeQuery(field)//对于某个字段的范围
                                    .from(start)
                                    .to(end)
                                    .includeLower(true)
                                    .includeUpper(true));
                        }
                    }
                }
            }

            // 查询字段
            if (StringUtils.isNotEmpty(queryEntity.getMatchStr())) {
                for (String s : queryEntity.getMatchStr().split(",")) {
                    String[] ss = s.split("=");
                    if (queryEntity.getMatchPhrase() == Boolean.TRUE) {
                        boolQuery.must(QueryBuilders.matchPhraseQuery(s.split("=")[0], s.split("=")[1]));
                    } else {
                        boolQuery.must(QueryBuilders.matchQuery(s.split("=")[0], s.split("=")[1]));
                    }
                }
            }

            // 高亮（xxx=111,aaa=222）
            if (StringUtils.isNotEmpty(queryEntity.getHighlightField())) {
                HighlightBuilder highlightBuilder = new HighlightBuilder();

                //highlightBuilder.preTags("<span style='color:red' >");//设置前缀
                //highlightBuilder.postTags("</span>");//设置后缀

                // 设置高亮字段
                highlightBuilder.field(queryEntity.getHighlightField());
                searchRequestBuilder.highlighter(highlightBuilder);
            }

            searchRequestBuilder.setQuery(QueryBuilders.matchAllQuery());
            searchRequestBuilder.setQuery(boolQuery);

            // 分页应用
            searchRequestBuilder.setFrom(queryEntity.getCurrentPage()).setSize(queryEntity.getPageSize());

            // 设置是否按查询匹配度排序
            searchRequestBuilder.setExplain(true);

            //打印的内容 可以在 Elasticsearch head 和 Kibana  上执行查询
            LOGGER.info("\n{}", searchRequestBuilder);

            // 执行搜索,返回搜索响应信息
            SearchResponse searchResponse = searchRequestBuilder.execute().actionGet();

            long totalHits = searchResponse.getHits().totalHits;
            long length = searchResponse.getHits().getHits().length;

            LOGGER.debug("共查询到[{}]条数据,处理数据条数[{}]", totalHits, length);

            if (searchResponse.status().getStatus() == 200) {
                // 解析对象
                List<Map<String, Object>> sourceList = setSearchResponse(searchResponse, queryEntity.getHighlightField());
                return new EsPage(queryEntity.getCurrentPage(), queryEntity.getPageSize(), (int) totalHits, sourceList);
            }
        }

        return null;

    }






    /**
     * 高亮结果集 特殊处理
     *
     * @param searchResponse
     * @param highlightField
     */
    private static List<Map<String, Object>> setSearchResponse(SearchResponse searchResponse, String highlightField) {
        List<Map<String, Object>> sourceList = new ArrayList<Map<String, Object>>();
        StringBuffer stringBuffer = new StringBuffer();

        for (SearchHit searchHit : searchResponse.getHits().getHits()) {
            searchHit.getSourceAsMap().put("id", searchHit.getId());

            if (StringUtils.isNotEmpty(highlightField)) {

                System.out.println("遍历 高亮结果集，覆盖 正常结果集" + searchHit.getSourceAsMap());
                if(searchHit.getHighlightFields().get(highlightField)!=null){
                    Text[] text = searchHit.getHighlightFields().get(highlightField).getFragments();

                    if (text != null) {
                        for (Text str : text) {
                            stringBuffer.append(str.string());
                        }
                        //遍历 高亮结果集，覆盖 正常结果集
                        searchHit.getSourceAsMap().put(highlightField, stringBuffer.toString());
                    }
                }

            }
            sourceList.add(searchHit.getSourceAsMap());
        }

        return sourceList;
    }
}
```




