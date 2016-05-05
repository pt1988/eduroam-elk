วิธีสร้างรับ logs จาก radius เข้า ELK

Network Topology

 -----------------                             ----------------
 | radius server | -[syslog protocol:port]- >  | (elk server) |
 -----------------                             ----------------

ELK Topology

     --------------------------------------------------------------    ------------------      ------------
     | Syslog Listener -> logstash -> elasticsearch ouput plugin  | -> | Elasticsearch  |  <-> |  Kibana  |
     --------------------------------------------------------------    ------------------      ------------



Logstash Configuration 
```
input {
  syslog {
 #   port => 5544
  }
}
output {
  elasticsearch { host => localhost }
  stdout { codec => rubydebug }
}
filter {

  
  if [program] == "radiusd" {
    grok {
      match => { "message" => "Login %{WORD:Status}(?:SPACE|%{DATA:reason}): \[(?:%{USER:user}@%{HOSTNAME:realm}|%{DATA:user})/%{DATA:mothod}\] \(from client %{HOSTNAME:agent} %{DATA}\)" }
    }
    syslog_pri { }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }

  
  mutate {
    add_field => { "timestampX" => "%{year}-%{month}-%{day} %{time}" }
    add_field => { "timestamp" => "%{month} %{day} %{time}" }
    add_field => { "userID" => "%{user}@%{realm}" }
  }

  #find top level domain of entries
  if [realm] {
    grok {
      match => { "realm" => "(?:(?:%{WORD}.%{WORD}.%{WORD}.%{WORD}.|%{WORD}.%{WORD}.%{WORD}.|%{WORD}.%{WORD}.|%{WORD}.)(?:%{WORD:tld}))" }
    }
  }

  #filter logs
  if ![year] or ([realm]=="guest.ku.ac.th" and [user] == "cpcppt" ) or ([realm]=="uni.net.th" and [user] == "surachai") {
        drop{}
  }
}
```
