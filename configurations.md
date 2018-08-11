# Single File Logstash

```
input {
  beats {
    port => 5044
  }
}

filter{
  grok {
		  match => { "message" => "%{MONTH:month} %{MONTHDAY:day} %{HOUR:hour}:%{MINUTE:minute}:%{SECOND:second} %{HOSTNAME} %{DATA:servicename}: %{GREEDYDATA}" }
  	}
 }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => ["messages-%{+YYYY.MM.dd}"]
  }
}

```

# Single File Filebeat

```
- input_type: log
  paths:
    - /var/log/messages
```

# Multifile Logstash

```
input {
  beats {
    port => 5044
  }
}

filter{
  if [fields][logtype] == "syslog"  {
  	grok {
		match => { "message" => "%{MONTH:month} %{MONTHDAY:day} %{HOUR:hour}:%{MINUTE:minute}:%{SECOND:second} %{HOSTNAME} %{DATA:servicename}: %{GREEDYDATA}" }
  	}
        mutate { 
                add_field => [ "index_name", "%{[fields][]logtype}" ]
        }
  }
  else if [fields][logtype] == "tomcat" {
       grok {
                match => { "message" => "%{MONTHDAY:mday}-%{MONTH:month}-%{YEAR:year} %{HOUR:hour}:%{MINUTE:minute}:%{SECOND:second}\.%{INT:mseconds} %{WORD:severity} %{GREEDYDATA:data}" }
       }
       mutate {
                add_field => [ "index_name", "%{[fields][]logtype}" ]
        }

  }
  
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => ["%{index_name}-%{+YYYY.MM.dd}"]
  }
}

```

# Multifile Filebeat

```
- input_type: log
  paths:
    - /opt/apache-tomcat-9.0.10/logs/catalina.out 
  fields: 
    logtype: tomcat

- input_type: log
  paths:
    - /var/log/messages
  fields:
    logtype: syslog

```
