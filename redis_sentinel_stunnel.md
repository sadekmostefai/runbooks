# Redis DIME with stunnel

This example takes one master/2 slaves with sentinel

## Install stunnel

```yum install stunnel```

Redis server will be listening on local interface

```bind 127.0.0.1```

Server certificates will be obtained via Vault.

## on MASTER node

```
/etc/stunnel/stunnel.conf:
	pid=/var/run/stunnel.pid
	debug = info
	#foreground = yes
	output = /var/log/stunnel.log
	 
	[redis-master-server]
	cert = /etc/stunnel/master.crt
	key = /etc/stunnel/master.key
	accept = master_exposed_ip:6379
	connect = 127.0.0.1:6379
	
	[redis-master-client]
	client = yes
	accept = 6380
	connect = master_exposed_ip:6379
	CAfile = /etc/stunnel/master.crt
	
	[redis-slave01-client]
	client = yes
	accept = 6381
	connect = slave01_exposed_ip:6379
	CAfile = /etc/stunnel/slave01.crt
	
	[redis-slave02-client]
	client = yes
	accept = 6382
	connect = slave02_exposed_ip:6379
	CAfile = /etc/stunnel/slave02.crt
```


## On SLAVE nodes

 
```
/etc/stunnel/stunnel.conf:
	pid=/var/run/stunnel.pid
	debug = info
	#foreground = yes
	output = /var/log/stunnel.log
	 
	[redis-slave01-server]
	cert = /etc/stunnel/slave01.crt
	key = /etc/stunnel/slave01.key
	accept = slave01_exposed_ip:6379
	connect = 127.0.0.1:6379
	
	[redis-master-client]
	client = yes
	accept = 6380
	connect = master_exposed_ip:6379
	CAfile = /etc/stunnel/master.crt
	
	[redis-slave01-client]
	client = yes
	accept = 6381
	connect = slave01_exposed_ip:6379
	CAfile = /etc/stunnel/slave01.crt
	
	[redis-slave02-client]
	client = yes
	accept = 6382
	connect = slave02_exposed_ip:6379
	CAfile = /etc/stunnel/slave02.crt
```

Repeat the same step on slave02 by replacing the server section.

```
	[redis-slave02-server]
	cert = /etc/stunnel/slave02.crt
	key = /etc/stunnel/slave02.key
	accept = slave02_exposed_ip:6379
	connect = 127.0.0.1:6379
```
 

Add to redis.conf on all master/slave nodes:

```
slave-announce-ip "exposed_ip" (e.g. slave02_ip)
slave-announce-port "stunnel_client_port" (e.g. 6382)
```


## Sentinel configuration

```sentinel monitor mymaster master_exposed_ip stunnel_master_client_port quorum```


## Clients set up

All clients/applications that need to connect to Redis will have stunnel running locally with client configuration only

```
[redis-master-client]
client = yes
accept = 6380
connect = master_exposed_ip:6379
CAfile = /etc/stunnel/master.crt

[redis-slave01-client]
client = yes
accept = 6381
connect = slave01_exposed_ip:6379
CAfile = /etc/stunnel/slave01.crt

[redis-slave02-client]
client = yes
accept = 6382
connect = slave02_exposed_ip:6379
CAfile = /etc/stunnel/slave02.crt
```
