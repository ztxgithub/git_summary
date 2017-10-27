
# sengine注意事项

- sengine的 <MQTT_KEEP_ALIVE_INTERVAL>60</MQTT_KEEP_ALIVE_INTERVAL>

```shell
    
    最好设置为 60s,使得emqttd能够

```

- 在通信过程中,一端接受最好判断一下数据的有效性(看会不会异常的数据出现段错误)

