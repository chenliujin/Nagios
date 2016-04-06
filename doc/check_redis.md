<h1>依赖包</h1>
<pre>
# yum install -y perl-Redis
</pre>

<h1>connected_clients</h1>
<h2>* NRPE</h2>
<pre>
# vim /etc/nrpe.d/check-redis.cfg
command[check_redis_connected_clients]=/usr/lib64/nagios/plugins/check_redis -H 127.0.0.1 -p 6379 -a 'connected_clients' -w ~ -c ~ -f

# 重启服务
# service nrpe restart
</pre>

<h2>* 检查代理是否工作</h2>
<pre>
# /usr/lib64/nagios/plugins/check_nrpe -H 127.0.0.1 -c check_redis_connected_clients
OK: REDIS 2.4.10 on 127.0.0.1:6381 has 1 databases (db0) with 10515 keys, up 14 days 16 hours - connected_clients is 26 | connected_clients=26
</pre>

<h2>* Service</h2>
<pre>
# vim /etc/nagios/conf.d/redis.cfg
define service {
    use                     generic-service,service-pnp
    service_description     Redis Clients
    check_command           check_nrpe!check_redis_connected_clients
    host_name               www.chenliujin.com
	normal_check_interval   5
	notification_interval   5
}
</pre>

<h1>Memory</h1>
<h2>* NRPE</h2>
<pre>
# vim /etc/nrpe.d/check-redis.cfg
command[check_redis_memory]=/usr/lib64/nagios/plugins/check_redis.pl -H 127.0.0.1 -p 6379 -a 'used_memory,used_memory_peak' -w ~,~ -c ~,~ -f
</pre>

<h2>* 检查代理是否工作</h2>
<pre>
# /usr/lib64/nagios/plugins/check_nrpe -H 127.0.0.1 -c check_redis_memory
OK: REDIS 2.8.19 on 172.16.64.150:6379 has 2 databases (db3,db15) with 61 keys, up 6 hours 58 minutes - used_memory is 854392, used_memory_peak is 855416 | used_memory=854392 used_memory_peak=855416
</pre>

<h2>* Service</h2>
<pre>
# vim /etc/nagios/conf.d/redis.cfg
define service {
    use                     generic-service,service-pnp
    service_description     Redis Memory
    check_command           check_nrpe!check_redis_connected_clients
    host_name               www.chenliujin.com
	normal_check_interval   5
	notification_interval   5
}
</pre>

<h1>Keyspace</h1>
<p>Session 使用 Redis 作为存储，监控对应 DB 的 key 个数，可以知道 Session 的变化趋势。check_redis.pl -H 127.0.0.1 -A 可以看到集成了db0_keys, db0_expires, db0_avg_ttl, db1_keys...，所以可以获取对应 DB 的 key 个数</p>

<h2>* NRPE</h2>
<pre>
# vim /etc/nrpe.d/check-redis.cfg
command[check_redis_db0_keys]=/usr/lib64/nagios/plugins/check_redis -H 127.0.0.1 -p 6379 -a 'db0_keys' -w ~ -c ~ -f

# 重启服务
# service nrpe restart
</pre>

<h2>* 检查代理是否工作</h2>
<pre>
# /usr/lib64/nagios/plugins/check_nrpe -H 127.0.0.1 -c check_redis_db0_keys
OK: REDIS 2.8.19 on 172.16.64.150:6379 has 2 databases (db3,db15) with 61 keys, up 4 days 23 hours - db0_keys is 9 | db0_keys=9
</pre>

<h2>* Service</h2>
<pre>
# vim /etc/nagios/conf.d/redis.cfg
define service {
    use                     generic-service,service-pnp
    service_description     Session
    check_command           check_nrpe!check_redis_db0_keys
    host_name               www.chenliujin.com
	normal_check_interval   1
	notification_interval   5
}
</pre>

## Redis KPI
* connected_clients：链接数
* instantaneous_ops_per_sec：每秒处理命令数。redis_version:2.8.19 有，redis_version:2.4.10 没有，使用前检查下是否能取到该值
* used_memory：内存使用
* used_memory_peak：内存使用（峰值）

### NRPE
```
command[check_redis]=/usr/lib64/nagios/plugins/check_redis.pl -H 172.16.64.150 -p 6379 -a 'connected_clients,instantaneous_ops_per_sec,used_memory,used_memory_peak' -w ~,~,~,~ -c ~,~,~,~ -f
```

### Nagios
```
```
