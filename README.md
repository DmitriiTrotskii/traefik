#   TRAEFIK
##  Simple default config
---
##  Описание

```bash
.
├── configs                 #   Каталог конфигураций
│   ├── dynamic.yml         #   Основнной файл динамической конфигурации
│   ├── teamplate.yml       #   Шаблон дополнительной динамической конфигурации
│   └── traefik.yml         #   Основнной файл статической конфигруации
├── README.md               #   
└── traefik-swarm.yml       #   docker-compose для запуска в swarm режиме 
```

##  Перед началом работы
1.  Настроить letsencrypt  
    файл: `traefik.yml`

```yaml
certificatesResolvers:
  letsencrypt:
    acme:
      email: {EMAIL}
      storage: "/var/traefik/certificates/acme.json"
      # caServer: "https://acme-v02.api.letsencrypt.org/directory"
      # caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
      tlsChallenge: true
```

Необходимо заменить `{EMAIL}` на адрес почтового ящика для регистрации.  
Раскоментировать одну из строк caServer

| Сервер                                                 | Пояснение |
| :----------------------------------------------------- | :-------- |
| https://acme-v02.api.letsencrypt.org/directory         | боевой    |
| https://acme-staging-v02.api.letsencrypt.org/directory | тестовый  |

Во время отладки рекомендуется использовать тестовый сервер, во избежания блокировки letsencrypt.

2.  Настроить middleware basic authentification  
    файл: `dynamic.yml`

```yaml
defaultAuth:
    basicAuth:
    users:
        - "{BASICAUTH}"
```
Необходимо заменить `{BASICAUTH}` на `логин:хеш пароля`  
Сгнерировать можно утилитой htpasswd

```bash
htpasswd -nb user password

user:$apr1$zvd7ilq2$4vbvE2rE96X.xyvRpO.ih.
```
**Если в получившемся хэше присутсвуют символы $ - их необходимо экранировать заменив на $$**

3.  Указать lable с доменным именем для traefik  
    файл: `traefik-swarm.yml`

```yaml
- traefik.http.routers.traefik.rule=Host(`{FQDN}`)
```

Необходимо заменить `{FQDN}` на имя по которому будет доступен traefik.

4.  *Опционально*. Настрить revers-proxy для хоста.  
    файл: `teamplate.yml`

##  Запуск

```bash
docker node update --label-add traefik=true         # Добавить lable на нодну, на этой недле будет запущен контейнер
docker network create traefik-network -d overlay    # Создать сеть
docker stack deploy traefik -c traefik-swarm.yml    # Запустить сервис
```