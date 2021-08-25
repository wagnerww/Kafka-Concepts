# Teoria na Prática

### Executando o container
    
    docker-compose up -d

### Para ver se terminou:

    docker-compose ps

    docker logs kafka-concepts_kafka_1

## Acessando o Control Center:

    http://localhost:9021/

### Entrando no container:
    
    docker exec -it kafka-concepts_kafka_1 bash

### Criando um Tópico:
    
    kafka-topics --create --topic=teste --bootstrap-server=localhost:9092 --partitions=3

### Listando os Tópicos:

  Obs: Quando sobe o control-center junto, ele cria um monte de tópicos para ele, então ignorar.

    kafka-topics --list --bootstrap-server=localhost:9092

### Detalhando o topico teste:

    kafka-topics --bootstrap-server=localhost:9092 --topic=teste --describe

### Criando um producer:

    kafka-console-producer --bootstrap-server=localhost:9092 --topic=teste

### Criando um consumer:

  Obs: Abrir um novo terminal, pq ele fica preso para processar as mensagens

    kafka-console-consumer --bootstrap-server=localhost:9092 --topic=teste

  Obs: caso ler do inicio, adicionar a tag: --from-beginning

### Criando um Consumer Group
  Obs: Abrir um novo terminal, e replicar o comando abaixo, para ver o efeito de round-robin
  
    kafka-console-consumer --bootstrap-server=localhost:9092 --topic=teste --group=x

### Detalhando um grupo:

     kafka-consumer-groups --bootstrap-server=localhost:9092 --group=x --describe