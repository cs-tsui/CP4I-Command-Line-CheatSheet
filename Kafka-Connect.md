## Working with Kafka Connect REST API


### Create Connector
```
# Prep - After Kafka Connent Deployment is created and service is created

# Get Kafka Connect REST endpoint (if exposed through route)
KC_ROUTE=$(oc get routes kafkaconnect-service -n kafka-connect -ojsonpath='{.spec.host}')

# Test the curl
curl $KC_ROUTE/connectors

# Curl from 
curl -X POST $KC_ROUTE/connectors \
	-H "Content-Type: application/json"  \
	-d @mq-source.json

curl -X POST $KC_ROUTE/connectors \
	-H "Content-Type: application/json"  \
	-d @mq-sink.json

# Save as mq-source.json
{
	"name":"mq-source02",
	"config": {
            "mq.message.body.jms": true,
	    "connector.class":"com.ibm.eventstreams.connect.mqsource.MQSourceConnector",
	    "tasks.max":"1",
	    "mq.user.name":"admin", 
	    "mq.password":"passw0rd", 
	    "topic":"mq-es",
	    "mq.queue.manager":"mqes1",
	    "mq.connection.name.list":"mq-instance.mq.svc(1414)",
	    "mq.channel.name":"DEV.APP.SVRCONN",
	    "mq.queue":"FROM.MQ.TO.ES",
	    "mq.record.builder":"com.ibm.eventstreams.connect.mqsource.builders.DefaultRecordBuilder",
	    "value.converter":"org.apache.kafka.connect.converters.StringConverter"
    }
}

# Save as mq-sink.json
{
	"name":"mq-sink01",
	"config": {
	    "connector.class":"com.ibm.eventstreams.connect.mqsink.MQSinkConnector",
	    "tasks.max":"1",
	    "mq.user.name":"admin", 
	    "mq.password":"passw0rd", 
	    "topics":"es-mq",
	    "mq.queue.manager":"mqe2e2",
	    "mq.connection.name.list":"mq-instance.mq.svc(1414)",
	    "mq.channel.name":"DEV.APP.SVRCONN",
	    "mq.queue":"FROM.ES.TO.MQ",
	    "mq.message.builder": "com.ibm.eventstreams.connect.mqsink.builders.DefaultMessageBuilder",
	    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
	    "value.converter": "org.apache.kafka.connect.storage.StringConverter"
    }
}

# Single command vs. reading from file
curl -X POST \
  $KC_ROUTE/connectors \
  -H 'Content-Type: application/json' \
  -d '{
	"name":"mq-source-no-jms1",
	"config": {
		  "mq.message.body.jms": true,
	    "connector.class":"com.ibm.eventstreams.connect.mqsource.MQSourceConnector",
	    "tasks.max":"1",
	    "mq.user.name":"admin", 
	    "mq.password":"passw0rd", 
	    "topic":"mq-es",
	    "mq.queue.manager":"mqes1",
	    "mq.connection.name.list":"mq-instance.mq.svc(1414)",
	    "mq.channel.name":"DEV.APP.SVRCONN",
	    "mq.queue":"FROM.MQ.TO.ES",
	    "mq.record.builder":"com.ibm.eventstreams.connect.mqsource.builders.DefaultRecordBuilder",
	    "value.converter":"org.apache.kafka.connect.converters.StringConverter"
    }
}'
```


### Other actions 
```
# Get status
curl $KC_ROUTE/connectors/<connector_name>/status

# Pause
curl -X PUT $KC_ROUTE/connectors/<connector_name>/pause

# Update
curl -x PUT $KC_ROUTE/connectors/<connector_name>/config \
	-H 'Content-Type: application/json' \
	-d @updated_connector.json

# Delete
curl -X DELETE $KC_ROUTE/connectors/<connector_name>
```
