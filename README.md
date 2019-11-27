# springboot-ELK

This is a poc having integartion with springboot and ELK. Steps to run:

1) Elasticsearch Set up:

Download latest version of Elasticsearch from this download page and unzip it any folder.
Run bin\elasticsearch.bat from command prompt.
By default, it would start at http://localhost:9200

2) Kibana set up:
 
Download the latest distribution from download page and unzip into any folder.
Open config/kibana.yml in an editor and set elasticsearch.url to point at your Elasticsearch instance. In our case as we will use the local instance just uncomment elasticsearch.url: "http://localhost:9200"
Run bin\kibana.bat from command prompt.
Once started successfully, Kibana will start on default port 5601 and Kibana UI will be available at http://localhost:5601

3) Logstash set up:

Download the latest distribution from download page and unzip into any folder.
Create one file logstash.conf under bin folder
then paste below line of codes:
{code}
input { stdin { } }
output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}

{code}

Now run bin/logstash -f logstash.conf to start logstash

4) For this project, you need to change logstash.conf file. please add below lines of code to logstash.conf file





input {
  file {
    type => "java"
    path => "C:/Users/Lenovo/workspace1/spring-boot-elk.log"
    codec => multiline {
      pattern => "^%{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME}.*"
      negate => "true"
      what => "previous"
    }
  }
}
 
filter {
  #If log line contains tab character followed by 'at' then we will tag that entry as stacktrace
  if [message] =~ "\tat" {
    grok {
      match => ["message", "^(\tat)"]
      add_tag => ["stacktrace"]
    }
  }
 
}
 
output {
   
  stdout {
    codec => rubydebug
  }
 
  # Sending properly parsed log events to elasticsearch
  elasticsearch {
    hosts => ["localhost:9200"]
  }
}



I have added this file to project level. You can refer:

start logstash server again.

Run Application:

Go to springboot-elk folder

run:  mvn clean install

Go to IDE
open main file "HelloWorldSpringBootApplication". Right click: Run as java application

Open below url in browser:
http://localhost:8080/elk


and below for exception:

http://localhost:8080/exception

Find index in kibana:
http://localhost:9200/_cat/indices/?v

health status index                      uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   logstash-2019.11.26-000001 gGREYyviQy66_vt7G46h5g   1   1         77            0     91.4kb         91.4kb
green  open   .kibana_task_manager_1     al0BDvexTIaVfUPtrR3s4A   1   0          2            0     12.5kb         12.5kb
green  open   .apm-agent-configuration   ZyDwCLuZTwi1J2W0s3T3fQ   1   0          0            0       283b           283b
green  open   .kibana_1                  CgqGrknJT0a-xhOOglJa8Q   1   0         18            2     37.7kb         37.7kb


http://localhost:9200/logstash-2019.11.26-000001/_search
 You will get logs.
 
 You can refer kibana to find out logs and exception.

