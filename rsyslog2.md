# Rsyslog testing 

## only rsyslog client
### 多檔input， file output
>rsyslog client端</br>
>>`/etc/rsyslog.d/client_1.conf`
```conf
module(load="imfile" mode="inotify")
#*.log only run in inotify mode
# rsyslog Templates
template (name="DynFile" type="string" string="%msg%\n") 

#rsyslog Input Modules
input(type="imfile"
      File="/var/log/test-0419*.log"
      Tag="test-0419:"
      Ruleset="MyRuleSet")

#rsyslog RuleSets
ruleset(name="MyRuleSet") {
    action(type="omfile"
        File="/var/log/0419_output.log"
        template="DynFile"
        )
    stop
}
```

### udp input，file ouput
>logstash output -- 走udp
```
output {
    udp {
        host => "localhost"
        port => "514"
        codec => line { format => "%{message}"}	
   	}
```
>rsyslog client端</br>
-- udp input，file ouput</br>
>>`/etc/rsyslog.d/client_2.conf`
```conf
module(load="imudp" PollingInterval="1") 
template (name="DynFile" type="string" string="%msg%\n") 
input(type="imudp" 
      Address="localhost"
      port="514"
      ruleset="MyRuleSet")
ruleset(name="MyRuleSet") {
    action(type="omfile"
	    File="/var/log/output.log"
	    template="DynFile" )
    stop
}
```
-----
## rsyslog client & server

### client端
>在`/etc/rsyslog.conf`加server ip + port</br>

>>`/etc/rsyslog.conf`</br>
>>在最下方加`local3.*  @192.168.70.171:514`</br>
>>>*local3是可以自行更改的，這裡透過local3傳送input file中message*

>>`/etc/rsyslog.d/rsyslog_udp_ver.conf`</br>
```conf
module(Load="imfile" mode="inotify") 
#*.log only run in inotify mode
# rsyslog Templates
template (name="DynFile" type="string" string="%msg%\n") 

# rsyslog Input Modules
input(type="imfile"
     File="/var/log/test-0419*.log"
     Tag="test-0419:"
     Facility="local3")
```
### server端
>在`/etc/rsyslog.conf`打開udp設定</br>

>>`/etc/rsyslog.conf`</br>
```conf
# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514
```

### 檢查傳輸狀況
>`sudo tailf /var/log/messages`










 