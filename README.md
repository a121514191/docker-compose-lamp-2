# docker-compose-lamp-2

參考網址

lamp: https://hackmd.io/@titangene/docker-lamp

php文件:https://registry.hub.docker.com/_/php  (重要)

不同架法:https://www.thearmchaircritic.org/tech-journals/creating-a-lamp-stack-using-docker-compose

## 延續上次的試著加入CI專案

進入 http://192.168.99.100:8080/ phpmyadmin 把專案的sql丟入

在進去專案更改database密碼

```
$db['default'] = array(
    'dsn' => '',
    'hostname' => 'localhost',
    'username' => 'root',
    'password' => 'admin',
    'database' => 'ngbc',
    'dbdriver' => 'mysqli',
    'dbprefix' => '',
    'pconnect' => false,
    'db_debug' => (ENVIRONMENT !== 'production'),
    'cache_on' => false,
    'cachedir' => '',
    'char_set' => 'utf8',
    'dbcollat' => 'utf8_general_ci',
    'swap_pre' => '',
    'encrypt' => false,
    'compress' => false,
    'stricton' => false,
    'failover' => array(),
    'save_queries' => true,
);
```
之後查看網頁

### 錯誤1

![](https://github.com/a121514191/docker-compose-lamp-2/blob/master/error1.PNG)

網路上解決辦法

![](https://github.com/a121514191/docker-compose-lamp-2/blob/master/error1_ok.PNG)

但不確定正不正確，因為沒問題就先跳過

### 錯誤2

![](https://github.com/a121514191/docker-compose-lamp-2/blob/master/error2.PNG)

網路上的解決辦法都是跟php.ini檔 有關

所以我開始查詢php.ini檔的去向

我們一般 php:

php.ini 路徑: /etc/php.ini

PHP 模組設定檔目錄: /etc/php.d/

PHP 模組目錄: /usr/lib64/php/modules/

![](https://github.com/a121514191/docker-compose-lamp-2/blob/master/etc-phpini.PNG)

但由於這次安裝是按照他人的docker-compose 去裝所以配置文件的放置並不一樣

之後只好查詢dockerfile 裡的php是怎麼建的

![](https://github.com/a121514191/docker-compose-lamp-2/blob/master/php-apache-ini.PNG)

![](https://github.com/a121514191/docker-compose-lamp-2/blob/master/find_php_ini.PNG)

於是我在php的docker-file裡 加入一些新的指令

```
FROM php:7.1-apache
LABEL maintainer="titangene.tw@gmail.com"

# Install the PHP extensions we need
RUN set -ex; \
	apt-get update; \
	apt-get install vim -y; \  
	mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"; \
	docker-php-ext-install -j$(nproc) pdo_mysql; \
	rm -rf /var/lib/apt/lists/*

RUN a2enmod rewrite
```
因為我是進入container 裡去修改php.ini

本身container裡並沒有 vim ，所以要安裝 並加入-y (apt-get install vim -y; \ )

然後加入那串官網裡找到的 (mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"; \)

執行:

找到mv那一句

![](https://github.com/a121514191/docker-compose-lamp-2/blob/master/find_php_ini.PNG)

之後進入container裡面 找到路徑下的php.ini




## 後續問題
1.目前ip 192.168.99.100 是我docker-toolbox vm的 
  如果要能多IP 是設在此docker-toolbox vm裡面嗎

預計要完成如同vargrant 的一鍵建立本機測試網站
