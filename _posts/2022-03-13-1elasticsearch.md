# ElasticSearch安装

![image-20220416115124262](F:\202109-pro\jacob-yang.github.io\images\posts\2022-03-13-1elasticsearch\image-20220416115124262.png)

![image-20220416115400237](F:\202109-pro\jacob-yang.github.io\images\posts\2022-03-13-1elasticsearch\image-20220416115400237.png)

![image-20220416115455977](F:\202109-pro\jacob-yang.github.io\images\posts\2022-03-13-1elasticsearch\image-20220416115455977.png)

1. 下载解压  https://www.elastic.co/cn/downloads/elasticsearch
2. 熟悉目录

![image-20210813142300419](C:\xy\Elasticsearch\image\image-20210813142300419.png)

3. 启动,访问9200

   |        | linux                                    | windeows                                 |
   | ------ | ---------------------------------------- | ---------------------------------------- |
   | 命令行 | cd elasticsearch/bin  ./elasticsearch -d | cd elasticsearch\bin  .\elasticsearch -d |
   | 图形化 | bin/ElasticSearch.bat                    | ---                                      |
   |        |                                          |                                          |
   
   
   
   ![image-20220416120626063](F:\202109-pro\jacob-yang.github.io\images\posts\2022-03-13-1elasticsearch\image-20220416120626063.png)

>可视化界面

下载 https://github.com/mobz/elasticsearch-head

```shell
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
npm run start
open http://localhost:9100/
```

配置跨域

```yml
http.cors.enabled: true
http.cors.allow-origin: "*"

```



![image-20210813143834787](C:\xy\Elasticsearch\image\image-20210813143834787.png)

# Kibana安装

![image-20210813143925194](C:\xy\Elasticsearch\image\image-20210813143925194.png)

1. 下载解压 https://www.elastic.co/cn/downloads/kibana

   Kibana版本要与ElasticSearch版本一致

2. 启动 /bin

3. 测试 也可以用post

4. 汉化 zh-CN

   

   ![image-20210814183758152](C:\xy\Elasticsearch\image\image-20210814183758152.png)

   ```
   config/kibana.yml
   
   i18n.locale: "zh-CN"
   ```

   



# ES

![image-20210813145133116](C:\xy\Elasticsearch\image\image-20210813145133116.png)

# IK分词器

![image-20210813145355908](C:\xy\Elasticsearch\image\image-20210813145355908.png)

由于 ElasticSearch 默认的分词器不支持中文分词，所以我们需要集成IK 分词器。

1. 下载 https://github.com/medcl/elasticsearch-analysis-ik/releases

   1. 从github上下载 对应es 版本的IK分词器zip包。
   2. 解压并重命名为IK 将整个文件夹上传到es 中的 plugins 目录中。重启es即可。
   3. 测试分词器 是否生效。

     

# 文档操作

![image-20210813160757276](C:\xy\Elasticsearch\image\image-20210813160757276.png)

## 修改 _update

![image-20210813161201994](C:\xy\Elasticsearch\image\image-20210813161201994.png)



# 集成springboot

>看文档

## 索引

```java
@Autowired
    @Qualifier("restHighLevelClient")
    private RestHighLevelClient client;
    //创建索引
    @Test
    void createIndexTest() throws IOException {
        //创建索引请求
        CreateIndexRequest request = new CreateIndexRequest("jacob001");
        //客户端执行请求
        CreateIndexResponse createIndexResponse = client.indices().create(request, RequestOptions.DEFAULT);
        System.out.println(createIndexResponse.index());
    }
    //获取索引是否存在
    @Test
    void existsIndexTest() throws IOException {
        GetIndexRequest request = new GetIndexRequest("jacob001");
        boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
        System.out.println(exists);
    }
    //删除索引
    @Test
    void delIndexTest() throws IOException {
        DeleteIndexRequest request = new DeleteIndexRequest("jacob001");
        AcknowledgedResponse delete = client.indices().delete(request, RequestOptions.DEFAULT);
        System.out.println(delete);
    }
```



## 文档

```java
	//添加文档
    @Test
    void addDocTest() throws IOException {
        User user = new User("ydd",23);
        //创建请求
        IndexRequest request = new IndexRequest("jacob002");
        request.id("1");
        request.timeout("1s");
        request.source(JSON.toJSONString(user), XContentType.JSON);
        IndexResponse index = client.index(request, RequestOptions.DEFAULT);
        System.out.println(index.toString());
    }
    //获取文档是否存在
    @Test
    void existsDocTest() throws IOException {
        GetRequest request = new GetRequest("jacob002","1");
        //不获取返回_source 的上下文
        request.fetchSourceContext(new FetchSourceContext(false));
        boolean exists = client.exists(request, RequestOptions.DEFAULT);
        System.out.println(exists);
    }
    //获取文档信息
    @Test
    void getDocTest() throws IOException {
        GetRequest request = new GetRequest("jacob002","1");
        GetResponse documentFields = client.get(request, RequestOptions.DEFAULT);
        System.out.println(documentFields.getSourceAsString());//打印文档内容
    }
    //跟新文档信息
    @Test
    void updateDocTest() throws IOException {
        UpdateRequest request = new UpdateRequest("jacob002","1");
        User user = new User("抗元元", 34);
        request.id("1");
        request.doc(JSON.toJSONString(user),XContentType.JSON);
        UpdateResponse update = client.update(request, RequestOptions.DEFAULT);
        System.out.println(update.getIndex());
    }
    //删除文档信息
    @Test
    void delDocTest() throws IOException {
        DeleteRequest request = new DeleteRequest("jacob002", "1");
        request.timeout("2s");
        DeleteResponse delete = client.delete(request, RequestOptions.DEFAULT);
        System.out.println(delete.getIndex());
    }
```

## 批量

```java
@Test
    void addBulkRequest() throws IOException {
        BulkRequest bulkRequest = new BulkRequest();
        bulkRequest.timeout("10s");
        ArrayList<User> users = new ArrayList<>();
        users.add(new User("xy",3));
        users.add(new User("亢媛媛",11));
        users.add(new User("宝宝",1));
        users.add(new User("微软",23));
        users.add(new User("qq",34));
        users.add(new User("Ew",12));
        for (int i = 0; i < users.size(); i++) {
            bulkRequest.add(
                    new IndexRequest("jacob002")
                    .id(""+(i+1))
                    .source(JSON.toJSONString(users.get(i)),XContentType.JSON));
        }
        BulkResponse bulk = client.bulk(bulkRequest, RequestOptions.DEFAULT);
        System.out.println(bulk.status()); //OK
    }

//查询
    @Test
    void searchTest() throws IOException {
        SearchRequest searchRequest = new SearchRequest("jacob002");
        //构建搜索条件
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
//        TermQueryBuilder builder = QueryBuilders.termQuery("name", "ydd"); //term 精确匹配
        MatchAllQueryBuilder builder = QueryBuilders.matchAllQuery(); //匹配所有
        sourceBuilder.query(builder);
        sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));
        searchRequest.source(sourceBuilder);
        SearchResponse search = client.search(searchRequest,RequestOptions.DEFAULT);
        System.out.println(JSON.toJSONString(search.getHits()));
    }
```



# 实战

jacobyang-es-jd