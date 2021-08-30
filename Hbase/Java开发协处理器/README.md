#### 1.BulkListener

```java
import org.elasticsearch.action.bulk.BulkProcessor.Listener;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;

public class BulkListener implements BulkProcessor.Listener{
  public void beforeBulk(long l, BulkRequest bulkRequest)
  {
  }

  public void afterBulk(long l, BulkRequest bulkRequest, BulkResponse bulkResponse)
  {
  }

  public void afterBulk(long l, BulkRequest bulkRequest, Throwable throwable)
  {
  }
}
```

#### 2.ElasticSearchPoolUtil

```java
import org.apache.commons.pool2.impl.GenericObjectPool;
import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.elasticsearch.client.RestHighLevelClient;

public class ElasticSearchPoolUtil{
  private static GenericObjectPoolConfig<RestHighLevelClient> poolConfig = new GenericObjectPoolConfig();

  private static PoolableUserFactory poolableUserFactory = new PoolableUserFactory();

  private static GenericObjectPool<RestHighLevelClient> clientPool = new GenericObjectPool(poolableUserFactory, poolConfig);

  public static RestHighLevelClient getClient()
    throws Exception
  {
    RestHighLevelClient client = (RestHighLevelClient)clientPool.borrowObject();
    return client;
  }

  public static void returnClient(RestHighLevelClient client)
  {
    clientPool.returnObject(client);
  }
}
```

#### 3.ElasticSearchPoolUtil

```java
import org.apache.commons.pool2.impl.GenericObjectPool;
import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.elasticsearch.client.RestHighLevelClient;

public class ElasticSearchPoolUtil{
  private static GenericObjectPoolConfig<RestHighLevelClient> poolConfig = new GenericObjectPoolConfig();

  private static PoolableUserFactory poolableUserFactory = new PoolableUserFactory();

  private static GenericObjectPool<RestHighLevelClient> clientPool = new GenericObjectPool(poolableUserFactory, poolConfig);

  public static RestHighLevelClient getClient()
    throws Exception
  {
    RestHighLevelClient client = (RestHighLevelClient)clientPool.borrowObject();
    return client;
  }

  public static void returnClient(RestHighLevelClient client)
  {
    clientPool.returnObject(client);
  }
}
```

#### 4.PoolableUserFactory

```java
import org.apache.commons.pool2.PooledObject;
import org.apache.commons.pool2.PooledObjectFactory;
import org.apache.commons.pool2.impl.DefaultPooledObject;
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;

public class PoolableUserFactory implements PooledObjectFactory<RestHighLevelClient>
{
  public PooledObject<RestHighLevelClient> makeObject()
    throws Exception
  {
    RestHighLevelClient client = null;
    try {
      client = new RestHighLevelClient(RestClient.builder(new HttpHost[] { new HttpHost("10.0.1.175", 9200, "http") }));
    }
    catch (Exception e)
    {
      e.printStackTrace();
    }
    return new DefaultPooledObject(client);
  }

  public void destroyObject(PooledObject<RestHighLevelClient> pooledObject)
    throws Exception
  {
  }

  public boolean validateObject(PooledObject<RestHighLevelClient> pooledObject)
  {
    return false;
  }

  public void activateObject(PooledObject<RestHighLevelClient> pooledObject)
    throws Exception
  {
  }

  public void passivateObject(PooledObject<RestHighLevelClient> pooledObject)
    throws Exception
  {
  }
}
```

#### 5.ElasticSearchMethods

```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.elasticsearch.action.bulk.BulkProcessor;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.script.Script;
import org.elasticsearch.script.ScriptType;

public class ElasticSearchMethods
{
  private static final Log LOG = LogFactory.getLog(ElasticSearchMethods.class);
  private static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

  static void putText(BulkProcessor bulkProcessor, String index, Map<String, Map<String, Object>> maps) {
    if ("".equals(index)) {
      return;
    }
    UpdateRequest updateRequest = null;
    Set rows = maps.keySet();
    try {
      for (localIterator1 = rows.iterator(); localIterator1.hasNext(); ) { rowkey = (String)localIterator1.next();
        originalRow = (Map)maps.get(rowkey);
        newRow = null;
        if (originalRow.size() > 0) {
          Set keys = originalRow.keySet();
          for (String key : keys) {
            newRow = new HashMap();
            if ("created_on".equals(key)) {
              Date d = simpleDateFormat.parse((String)originalRow.get("created_on"));
              newRow.put(key, d);
            }
            if ("di_id".equals(key)) {
              newRow.put(key, originalRow.get("di_id"));
            }
            if (newRow.size() > 0) {
              updateRequest = new UpdateRequest(index, rowkey);
              Script inline = new Script(ScriptType.INLINE, "painless", "ctx._source." + key + " = params." + key, newRow);
              updateRequest.script(inline);
              updateRequest.upsert(newRow);
              bulkProcessor.add(updateRequest);
            }
          }
        }
      }
    }
    catch (ParseException e)
    {
      Iterator localIterator1;
      String rowkey;
      Map originalRow;
      Map newRow;
      e.printStackTrace();
    }
  }

  public static void main(String[] args)
  {
  }
}
```

#### 6.ObserverUtil

```java
import com.obs.observerc.config.BulkBuilder;
import java.io.IOException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.NavigableMap;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.CoprocessorEnvironment;
import org.apache.hadoop.hbase.client.Durability;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.coprocessor.BaseRegionObserver;
import org.apache.hadoop.hbase.coprocessor.ObserverContext;
import org.apache.hadoop.hbase.coprocessor.RegionCoprocessorEnvironment;
import org.apache.hadoop.hbase.regionserver.wal.WALEdit;
import org.apache.hadoop.hbase.util.Bytes;
import org.elasticsearch.action.bulk.BulkProcessor;

public class ObserverUtil extends BaseRegionObserver
{
  private static final Log LOG = LogFactory.getLog(ObserverUtil.class);
  private BulkProcessor bulkProcessor = null;

  public void start(CoprocessorEnvironment e) throws IOException {
    BulkBuilder bulkBuilder = new BulkBuilder();
    this.bulkProcessor = bulkBuilder.getBulkProcessor();
  }

  public void stop(CoprocessorEnvironment e)
    throws IOException
  {
  }

  public void postPut(ObserverContext<RegionCoprocessorEnvironment> e, Put put, WALEdit edit, Durability durability)
  {
    Log log = LogFactory.getLog(ObserverUtil.class);

    String rowkey = Bytes.toString(put.getRow());

    NavigableMap familyMap = put.getFamilyCellMap();

    Map esData = new HashMap();
    Map data = new HashMap();

    for (Map.Entry entry : familyMap.entrySet()) {
      for (Cell cell : (List)entry.getValue()) {
        String key = Bytes.toString(CellUtil.cloneQualifier(cell));
        String value = Bytes.toString(CellUtil.cloneValue(cell));
        data.put(key, value);
      }
    }
    esData.put(rowkey, data);
    log.info("钩子函数接收到的数据：" + esData);
    try {
      if (this.bulkProcessor != null)
        ElasticSearchMethods.putText(this.bulkProcessor, "raonecloud", esData);
    }
    catch (Exception es) {
      es.printStackTrace();
    }
  }
}
```

#### 7.BulkBuilder

```java

import org.elasticsearch.action.ActionListener;
import org.elasticsearch.action.bulk.BackoffPolicy;
import org.elasticsearch.action.bulk.BulkProcessor;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.unit.ByteSizeUnit;
import org.elasticsearch.common.unit.ByteSizeValue;
import org.elasticsearch.common.unit.TimeValue;

public class BulkBuilder {
  private RestHighLevelClient Lclient = null;
  
  public BulkProcessor getBulkProcessor() {
    try {
      this.Lclient = ElasticSearchPoolUtil.getClient();
    } catch (Exception e) {
      e.printStackTrace();
    } 
    BulkProcessor.Builder builder = BulkProcessor.builder((request, bulkListener) -> this.Lclient.bulkAsync(request, RequestOptions.DEFAULT, bulkListener), new BulkListener());
    builder.setBulkActions(5000);
    builder.setBulkSize(new ByteSizeValue(5L, ByteSizeUnit.MB));
    builder.setConcurrentRequests(10);
    builder.setFlushInterval(TimeValue.timeValueSeconds(10L));
    builder.setBackoffPolicy(
        BackoffPolicy.constantBackoff(TimeValue.timeValueSeconds(10L), 3));
    return builder.build();
  }
}
```

#### 8.测试

```java
import java.io.IOException;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import org.apache.http.HttpHost;
import org.elasticsearch.action.ActionListener;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.Cancellable;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.unit.TimeValue;
import org.elasticsearch.index.query.QueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.rest.RestStatus;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.sort.FieldSortBuilder;
import org.elasticsearch.search.sort.SortOrder;

public class TextSearch {
  public static void searchText(RestHighLevelClient client) {
    SearchRequest searchRequest = new SearchRequest();
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query((QueryBuilder)QueryBuilders.termQuery("name", "xiao"));
    searchSourceBuilder.query((QueryBuilder)QueryBuilders.termQuery("age", "12"));
    searchSourceBuilder.from(0);
    searchSourceBuilder.size(5);
    searchSourceBuilder.timeout(new TimeValue(60L, TimeUnit.SECONDS));
    searchRequest.indices(new String[] { "demo" });
    searchRequest.source(searchSourceBuilder);
    try {
      SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
      RestStatus status = searchResponse.status();
      TimeValue took = searchResponse.getTook();
      Boolean terminatedEarly = searchResponse.isTerminatedEarly();
      boolean timedOut = searchResponse.isTimedOut();
      int totalShards = searchResponse.getTotalShards();
      int successfulShards = searchResponse.getSuccessfulShards();
      int failedShards = searchResponse.getFailedShards();
      SearchHits hits = searchResponse.getHits();
      SearchHit[] searchHits = hits.getHits();
      for (SearchHit searchHit : searchHits) {
        Map<String, Object> sourceAsMap = searchHit.getSourceAsMap();
        System.out.println(sourceAsMap.toString());
      } 
    } catch (IOException e) {
      e.printStackTrace();
    } 
  }
  
  public static void searchAsyncText() {
    RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(new HttpHost[] { new HttpHost("192.168.2.72", 9200, "http") }));
    SearchRequest searchRequest = new SearchRequest();
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query((QueryBuilder)QueryBuilders.termQuery("name", "xiao"));
    searchSourceBuilder.from(0);
    searchSourceBuilder.size(5);
    searchSourceBuilder.timeout(new TimeValue(60L, TimeUnit.SECONDS));
    searchSourceBuilder.sort((new FieldSortBuilder("age")).order(SortOrder.ASC));
    searchRequest.indices(new String[] { "demo" });
    searchRequest.source(searchSourceBuilder);
    ActionListener<SearchResponse> listener = new ActionListener<SearchResponse>() {
        public void onResponse(SearchResponse searchResponse) {
          RestStatus status = searchResponse.status();
          TimeValue took = searchResponse.getTook();
          Boolean terminatedEarly = searchResponse.isTerminatedEarly();
          boolean timedOut = searchResponse.isTimedOut();
          int totalShards = searchResponse.getTotalShards();
          int successfulShards = searchResponse.getSuccessfulShards();
          int failedShards = searchResponse.getFailedShards();
          SearchHits hits = searchResponse.getHits();
          SearchHit[] searchHits = hits.getHits();
          for (SearchHit hit : searchHits) {
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();
            String name = (String)sourceAsMap.get("name");
            String str1 = (String)sourceAsMap.get("age");
          } 
        }
        
        public void onFailure(Exception e) {}
      };
    Cancellable cancellable = client.searchAsync(searchRequest, RequestOptions.DEFAULT, listener);
  }
}
```

