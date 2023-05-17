# «Очереди RabbitMQ»
# Александр Широбоков
## Задание 1. Установка RabbitMQ.
### *Используя Vagrant или VirtualBox, создайте виртуальную машину и установите RabbitMQ. Добавьте management plug-in и зайдите в веб-интерфейс.*
**Устанавливаю RabbitMQ, плагины, затем подключаюсь:**

![Снимок экрана (194)](https://github.com/AleksandrShirobokov/RabbitMQ/assets/69298696/a7825260-ff59-44fd-88d7-bdf8eb079445)

## Задание 2. Отправка и получение сообщений.
### *Используя приложенные скрипты, проведите тестовую отправку и получение сообщения.*
**Запускаю первый скрипт:**

![Снимок экрана (197)](https://github.com/AleksandrShirobokov/RabbitMQ/assets/69298696/1559094e-0756-4f59-bc6a-4711a4ddac29)

**Запускаю второй скрипт:**

![Снимок экрана (198)](https://github.com/AleksandrShirobokov/RabbitMQ/assets/69298696/fce713cf-db8e-450d-9d70-4dcc336cb927)

**Также создал очередь test100 из 100 сообщений Hello World**

![Снимок экрана (199)](https://github.com/AleksandrShirobokov/RabbitMQ/assets/69298696/b120cf20-2b66-4be5-9fb2-3d8677ed49dc)

**Затем получаю сообщения из очереди в консоль:**

![Снимок экрана (200)](https://github.com/AleksandrShirobokov/RabbitMQ/assets/69298696/c2ebcb57-8f03-4895-a527-b94631a30b6c)

## * Задание 4. Ansible playbook.
### *Напишите плейбук, который будет производить установку RabbitMQ на любое количество нод и объединять их в кластер. При этом будет автоматически создавать политику ha-all.*
**Создаю плейбук для установки RabbitMQ и создания кластера с политикой ha-all:**
```
- name: Установка и настройка RabbitMQ
  hosts: rabbitmq_nodes
  become: true
  
  tasks:
    - name: Установка RabbitMQ
      apt:
        name: rabbitmq-server
        state: present
      notify:
        - Start RabbitMQ

    - name: Копирование конфигурационного файла
      template:
        src: rabbitmq.conf.j2
        dest: /etc/rabbitmq/rabbitmq.conf
      notify:
        - Restart RabbitMQ
      
  handlers:
    - name: Start RabbitMQ
      service:
        name: rabbitmq-server
        state: started
        enabled: true
        
    - name: Restart RabbitMQ
      service:
        name: rabbitmq-server
        state: restarted
        enabled: true
```

**Также создаю файл-шаблон rabbitmq.conf.j2, который включает конфиг для rabbitmq, настройку из трех кластеров, и политику ha-all:**
```
cluster_formation.peer_discovery_backend = rabbit_peer_discovery_classic_config
cluster_formation.classic_config.nodes.1 = rabbit@node1
cluster_formation.classic_config.nodes.2 = rabbit@node2
cluster_formation.classic_config.nodes.3 = rabbit@node3

cluster_partition_handling = autoheal

policy ha-all "^" '{"ha-mode":"all"}' \n

```

