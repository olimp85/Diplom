Для выполнения дипломной работы были развернуты 7 ВМ
![](https://github.com/olimp85/Diplom/blob/main/%D0%B2%D0%B8%D1%80%D1%82%20%D0%BC%D0%B0%D1%88%D0%B8%D0%BD.jpg)
По условиям задания сервера web, Elasticsearch были размещены в приватные подсети. Сервера Zabbix, Kibana, application load balancer определены в публичную подсеть. Также сервера web1 и web2 размещены в разных зонах доступности.

На сервера web1 192.168.2.25 и web2 192.168.3.26 был установлен Nginx, для их работы настроен балансировщик
![](https://github.com/olimp85/Diplom/blob/main/web1.jpg)
![](https://github.com/olimp85/Diplom/blob/main/web%202.jpg)
![](https://github.com/olimp85/Diplom/blob/main/balanser.jpg)
![](https://github.com/olimp85/Diplom/blob/main/netology%20forever.jpg)

Были созданы Target Group и Backend Group
![](https://github.com/olimp85/Diplom/blob/main/backend.jpg)
![](https://github.com/olimp85/Diplom/blob/main/target.jpg)
Также был создан HTTP router
![](https://github.com/olimp85/Diplom/blob/main/router.jpg)
Создан Application load balancer
![](https://github.com/olimp85/Diplom/blob/main/test%20balanser.jpg)

На сервере 192.168.2.37 установлен ​​Zabbix и настроены два Дашборда на сервера WEB1 и WEB2.
![](https://github.com/olimp85/Diplom/blob/main/dashbord%20web1.jpg)
![](https://github.com/olimp85/Diplom/blob/main/dashbord%20web2.jpg)

На серверах web1 и web2 установлены и настроены zabbix-agent.
![](https://github.com/olimp85/Diplom/blob/main/zabbix-agent%20web1.jpg)
![](https://github.com/olimp85/Diplom/blob/main/zabbix-agent%20web2.jpg)

Cоздана ВМ, на которой развернут Elasticsearch 192.168.2.19. Установлен filebeat в ВМ к web1 и web2, настроена отправка access.log, error.log nginx в Elasticsearch.

Создана ВМ, на которой развернута Kibana 192.168.2.7, сконфигурировано соединение с Elasticsearch.
![](https://github.com/olimp85/Diplom/blob/main/elastic.jpg)
![](https://github.com/olimp85/Diplom/blob/main/kibana.jpg)
![](https://github.com/olimp85/Diplom/blob/main/elastic%201.jpg)

Настроены Security Groups на входящий трафик только к нужным портам
![](https://github.com/olimp85/Diplom/blob/main/olimpsec.jpg)
![](https://github.com/olimp85/Diplom/blob/main/olimpbas.jpg)

ВМ 192.168.2.36 реализует концепцию bastion host
![](https://github.com/olimp85/Diplom/blob/main/bastion.jpg)

Созданы снимки дисков всех ВМ
![](https://github.com/olimp85/Diplom/blob/main/snapshot.jpg)

