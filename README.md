# Сервер EFK с доменной авторизацией через nginx. 

Так как в бесплатной версии elasticsearch нет доменной авторизации, поэтому делаем через nginx.

Перед запуском в файл /etc/sysctl.conf необходимо добавить запись vm.max_map_count=262144 и применить sysctl -p, в противном случае Elasticsearch может поругаться на нехватку этого значения.
В любом случае, отладка это важный шаг, по этому после запуска не лишним будет сделать docker logs elasticsearch. И так после добавления, запускаем наш контейнер.

```
docker compose up -d
```

 Заходим в контейнер, чтобы сгенерировать токен
```
docker exec -it efk-server-elasticsearch-1 bash
```

Копируем токен
```
bin/elasticsearch-create-enrollment-token -s kibana
```

Посмотреть пин-код для активации kibana
```
docker logs efk-stack-kibana-1
```

База у нас готова, можно проверить есть ли у нас индексы:
```
curl -X GET 'localhost:9200/_cat/indices?v'

health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   fluentd-20230417 SIFmYBdcSLK0L8iG_G9D4w   1   1         15            0     20.8kb         20.8kb
```

Порт кибаны 5601 закрыт, доступ к EFK проходит через nginx с авторизацией на 80 порту.
Порт 9200 у ElasticSearch доступен извне.
Порт 24224 у Fluentd тоже доступен извне.
Так же доступен порт 8888 у nginx, который отвечает за авторизцию в домене для проверки.

Проверка происходит таким образом:
```
curl --location --request GET 'http://localhost:8888' \
--header 'X-Ldap-URL: ldap://172.30.30.31:389' \
--header 'X-Ldap-BaseDN: OU=SBKUsers,DC=sbk,DC=lan' \
--header 'X-Ldap-BindDN: dfedorov@sbk.lan' \
--header 'X-Ldap-BindPass: password' \
--header 'X-Ldap-Template: (sAMAccountName=%(username)s)' -vv -u 'dfedorov:password'
```
Ответ в случае удачи приходит с кодом 200.

Так же можно проверить установив утилиту ldapsearch, но команда использовалась для настройки ldap.
```
ldapsearch -v -D "dfedorov@sbk.lan" -w "password" -b "OU=SBKUsers,DC=sbk,DC=lan" -H "ldap://172.30.30.31" "(sAMAccountName=efk)"
```