# logstash-output-kafka

## 配置

```sh
output {
  kafka {
    bootstrap_servers => ["localhost:9092"]
    codec => json{}
    topic_id =>  "my-topic"
  }
}
```
