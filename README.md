# Домашнее задание — «Система мониторинга Zabbix»

**Автор: Тутубалин Дмитрий**

---

## Задание 1

### Установите Zabbix Server с веб-интерфейсом

#### Процесс выполнения:

1. Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
2. Установите PostgreSQL. Для установки достаточна та версия, что есть в системном репозитороии Debian 11.
3. Пользуясь конфигуратором команд с официального сайта, составьте набор команд для установки последней версии Zabbix с поддержкой PostgreSQL и Apache.
4. Выполните все необходимые команды для установки Zabbix Server и Zabbix Web Server.

#### Требования к результатам 
1. Прикрепите в файл README.md скриншот авторизации в админке.
2. Приложите в файл README.md текст использованных команд в GitHub.

Развёртывание выполнено с помощью `docker-compose` (в силу актуальности и удобства в текущей ситуации).

#### Результаты:

**1) Скриншот авторизации в админке:**
![zabbix\_web](https://github.com/dtutu-tds/zabbix-hw/blob/main/screenshots/zabbix_web.png)

**2) Используемый `docker-compose.yml`:**

<details>
<summary>Развернуть для просмотра</summary>

```yaml
version: "3.8"

networks:
  zbx-net: {}

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - ./volumes/pgdata:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - zbx-net

  zabbix-server:
    image: zabbix/zabbix-server-pgsql
    environment:
      DB_SERVER_HOST: postgres
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      ZBX_STARTPOLLERS: 5
    depends_on:
      - postgres
    ports:
      - "10051:10051"
    restart: unless-stopped
    networks:
      - zbx-net

  zabbix-web:
    image: zabbix/zabbix-web-nginx-pgsql
    environment:
      DB_SERVER_HOST: postgres
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      PHP_TZ: ${PHP_TZ}
      ZBX_SERVER_HOST: zabbix-server
      ZBX_SERVER_NAME: ${ZBX_SERVER_NAME}
    depends_on:
      - zabbix-server
    ports:
      - "8080:8080"
      - "4433:8443"
    restart: unless-stopped
    networks:
      - zbx-net
```

</details>

---

## Задание 2

### Установите Zabbix Agent на два хоста

#### Процесс выполнения:
1. Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
2. Установите Zabbix Agent на 2 вирт.машины, одной из них может быть ваш Zabbix Server.
3. Добавьте Zabbix Server в список разрешенных серверов ваших Zabbix Agentов.
4. Добавьте Zabbix Agentов в раздел Configuration > Hosts вашего Zabbix Servera.
5. Проверьте, что в разделе Latest Data начали появляться данные с добавленных агентов.

#### Требования к результатам
1. Приложите в файл README.md скриншот раздела Configuration > Hosts, где видно, что агенты подключены к серверу
2. Приложите в файл README.md скриншот лога zabbix agent, где видно, что он работает с сервером
3. Приложите в файл README.md скриншот раздела Monitoring > Latest data для обоих хостов, где видны поступающие от агентов данные.
4. Приложите в файл README.md текст использованных команд в GitHub

#### Результаты:

**1) Скриншот раздела Configuration → Hosts:**
![agent's](https://github.com/dtutu-tds/zabbix-hw/blob/main/screenshots/agent's.png)

**2) Скриншот лога Zabbix Agent:**
![logs](https://github.com/dtutu-tds/zabbix-hw/blob/main/screenshots/logs.png)

**3) Скриншот Monitoring → Latest data:**
![wor\_agent](https://github.com/dtutu-tds/zabbix-hw/blob/main/screenshots/work_agent.png)

**4) Используемые команды и конфигурация для агентов:**

<details>
<summary>Развернуть для просмотра</summary>

```yaml
  agent_host:
    image: zabbix/zabbix-agent2
    container_name: agent_host
    network_mode: host
    pid: host
    environment:
      ZBX_SERVER_HOST: 127.0.0.1
      ZBX_SERVER_ACTIVE: 127.0.0.1
      ZBX_HOSTNAME: linux-host
    volumes:
      - /:/rootfs:ro
    restart: unless-stopped

  agent_container:
    image: zabbix/zabbix-agent2
    container_name: agent_container
    hostname: docker-host_2
    environment:
      ZBX_SERVER_HOST: zabbix-server
      ZBX_SERVER_ACTIVE: zabbix-server
      ZBX_HOSTNAME: docker-host_2
    expose:
      - "10050"
    restart: unless-stopped
    networks:
      - zbx-net
```

</details>

---

## Репозиторий

🔗 [https://github.com/dtutu-tds/zabbix-hw](https://github.com/dtutu-tds/zabbix-hw)
