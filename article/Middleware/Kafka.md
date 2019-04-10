# Kafka


### kafka-run-class
```Shell
获取topic的offset: -1(latest),-2(earliest)
kafka-run-class.sh  kafka.tools.GetOffsetShell --broker-list bd001:9092 --topic sea --time -1
```
