一、Java 链接redis

	DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to    clients.......  

	sentinel.conf中加入protected-mode no 

redis.conf 去掉bind 127.0.0.1

redis目前处于受保护模式，不允许非本地客户端链接，我们可以通过给redis设置密码

	./redis-cli
	config set requirepass ***
	quit
	./requirepass ***