### 1. 引入maven依赖、

```xml
        <!-- kafka -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
```

### 2. yml文件配置

```yaml
spring:
  kafka:
    bootstrap-servers: http://192.168.2.97:9092
    listener:
      type: batch
      # concurrency = 主题分区数，即一个消费线程消费一个partition的数据
      concurrency: 3
      # MANUAL：当每一批poll()的数据被消费者监听器（ListenerConsumer）处理之后, 手动调用Acknowledgment.acknowledge()后提交
      # MANUAL_IMMEDIATE：手动调用Acknowledgment.acknowledge()后立即提交
      ack-mode: MANUAL
    consumer:
      # earliest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
      # latest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据
      # none:topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      group-id: raonecloud_dataservices
      # 关闭自动提交
      enable-auto-commit: false
      # 一次最大拉取数据的数量
      max-poll-records: 5
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      # 提交规则【0,1，all】
      acks: 0
      # 失败重试次数
      retries: 3
```

### 3.实例

<!-- tabs:start -->

#### **消费者**

```java
@Component
@Slf4j
public class Consumer {

	/**
	 * 按分区消费
	 */
    @KafkaListener(id = "c0",topicPartitions = {@TopicPartition(topic = "original_data_test",partitions = {"0"})},containerFactory = "kafkaListenerContainerFactory")
    void c0(List<ConsumerRecord<?, ?>> records, Acknowledgment ack){
        for (ConsumerRecord record:records){
            log.info("分区0："+ JSONObject.parseObject((String) record.value()).toString());
        }
    }

    @KafkaListener(id = "c1",topicPartitions = {@TopicPartition(topic = "original_data_test",partitions = {"1"})},containerFactory = "kafkaListenerContainerFactory")
    void c1(List<ConsumerRecord<?, ?>> records, Acknowledgment ack){
        for (ConsumerRecord record:records){
            log.info("分区1："+ JSONObject.parseObject((String) record.value()).toString());
        }
    }
    @KafkaListener(id = "c2",topicPartitions = {@TopicPartition(topic = "original_data_test",partitions = {"2"})},containerFactory = "kafkaListenerContainerFactory")
    void c2(List<ConsumerRecord<?, ?>> records, Acknowledgment ack){
        for (ConsumerRecord record:records){
            log.info("分区2："+ JSONObject.parseObject((String) record.value()).toString());
        }
    }

	/**
	 * 按主题消费
	 */
    @KafkaListener(id = "c",topics = "original_data_test",containerFactory = "kafkaListenerContainerFactory")
    void c(List<ConsumerRecord<?, ?>> records, Acknowledgment ack){
        for (ConsumerRecord record:records){
            log.info("分区2："+JSONObject.parseObject((String) record.value()).toString());
        }
        ack.acknowledge();
    }
}
```

#### **生产者**

```java
@SpringBootTest
public class Product {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;
    @Test
    void producer(){
        
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("name","xiaohua"+i);
        
        kafkaTemplate.send("original_data_test",jsonObject.toString());
    }
}
```

<!-- tabs:end -->