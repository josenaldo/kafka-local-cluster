# Servdor Kafka Local

O intuito deste projeto é criar um ambiente local para testes de integração com o Kafka.

Ao executar esse projeto, será criado um cluster Kafka com 3 brokers, um Zookeeper e um Kafka-UI para visualização dos tópicos e mensagens.

## Tecnologias utilizadas

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Kafka](https://kafka.apache.org/)
- [Kafka-UI](https://github.com/provectus/kafka-ui)

## Pré-requisitos

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Como usar

1. Clone o repositório
2. Configure a propriedade `DOCKER_HOST_IP` no arquivo `.env` com o IP da máquina onde o Docker está rodando
3. Execute o comando `docker-compose up -d` na raiz do projeto
4. Aguarde a inicialização dos containers

## Acesso ao Kafka

### Kafka-UI

Para acessar o Kafka-UI, acesse `http://DOCKER_HOST_IP:8080` no navegador.

### Kafka

Para acessar o Kafka, utilize os endereços `DOCKER_HOST_IP:9092`, `DOCKER_HOST_IP:9093` e `DOCKER_HOST_IP:9094` para os brokers 1, 2 e 3, respectivamente.

## Comandos úteis

### Conectar na linha de comando do Kafka

Para criar tópicos diretamente no Kafka, na linha de comando, primeiro é necessário se conectar a um dos brokers.

```sh
docker exec -it kafka1 bash
```

### Criar tópico

```bash
kafka-topics --bootstrap-server kafka1:19092 \
             --create \
             --topic test-topic \
             --replication-factor 3 \
             --partitions 3
```

### Produzir mensagens sem chave

```bash
docker exec --interactive --tty kafka1  \
kafka-console-producer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic
```

### Consumir mensagens sem chave

```bash
docker exec --interactive --tty kafka1  \
kafka-console-consumer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --from-beginning
```

### Produzir mensagens com chave

```bash
docker exec --interactive --tty kafka1  \
kafka-console-producer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --property "parse.key=true" \
                       --property "key.separator=-"
```

### Consumir mensagens com chave

```bash
docker exec --interactive --tty kafka1  \
kafka-console-consumer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --from-beginning \
                       --property "print.key=true" \
                       --property "key.separator=-"
```

### Consumir mensagens em grupo

```bash
docker exec --interactive --tty kafka1  \
kafka-console-consumer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --group my-group
```

### Consumir mensagens em grupo com chave

```bash
docker exec --interactive --tty kafka1  \
kafka-console-consumer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --group my-group \
                       --property "print.key=true" \
                       --property "key.separator=-"
```

#### Exemplo de mensagem

```bash
a-abc
b-bus
```

### Consumir mensagens com headers

```bash
docker exec --interactive --tty kafka1  \
kafka-console-consumer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --group my-group \
                       --property "print.headers=true" \
                       --property "print.timestamp=true"
```

#### Example Messages:

```bash
a-abc
b-bus
```

### Arquivos de Logs do Kafka

Para acessar arquivos de logs do Kafka e o arquivo de configuração do broker, acesse o broker desejado.

- Para acessar o broker 1:

```bash
docker exec -it kafka1 bash
```

- Para acessar o broker 2:

```bash
docker exec -it kafka2 bash
```

- Para acessar o broker 3:

```bash
docker exec -it kafka3 bash
```

O arquivo de configuração do broker está localizado em `/etc/kafka/server.properties`.

Os arquivos de logs do Kafka estão localizados em `/var/lib/kafka/data` dentro de broker.

## Desligar o ambiente

Para desligar o ambiente, execute o comando `docker-compose down` na raiz do projeto.

```bash
docker-compose down
```

## Configurações

### min.insync.replicas

O `min.insync.replicas` é a quantidade mínima de réplicas que devem estar em sincronia para que a gravação de uma mensagem seja considerada bem-sucedida.

Para alterar o valor do `min.insync.replicas`, edite o arquivo `server.properties` de cada broker e adicione a propriedade `min.insync.replicas` com o valor desejado.

```properties
min.insync.replicas=2
```

Outra alternativa é alterar o valor do `min.insync.replicas` no tópico desejado.

```bash
kafka-topics --bootstrap-server kafka1:19092 \
             --entity-type topics \
             --entity-name test-topic \
             --alter \
             --add-config min.insync.replicas=2
```

## Comandos adicionais

### Listar tópicos

```bash
docker exec --interactive --tty kafka1  \
kafka-topics --bootstrap-server kafka1:19092 --list
```

### Descrever tópico

- Comando para descrever todos os tópicos do Kafka.

```bash
docker exec --interactive --tty kafka1  \
kafka-topics --bootstrap-server kafka1:19092 --describe
```

- Comando para descrever um tópico específico do Kafka.

```bash
docker exec --interactive --tty kafka1  \
kafka-topics --bootstrap-server kafka1:19092 \
             --describe \
             --topic test-topic
```

### Alterar partições do tópico

```bash
docker exec --interactive --tty kafka1  \
kafka-topics --bootstrap-server kafka1:19092 \
             --alter \
             --topic test-topic \
             --partitions 40
```

### Como ver grupos de consumidores

```bash
docker exec --interactive --tty kafka1  \
kafka-consumer-groups --bootstrap-server kafka1:19092 \
                      --list
```

### Como ver detalhes de um grupo de consumidores

```bash
docker exec --interactive --tty kafka1  \
kafka-consumer-groups --bootstrap-server kafka1:19092 \
                      --describe \
                      --group my-group
```

### Como ver o log de commit

```bash
docker exec --interactive --tty kafka1  \
kafka-run-class kafka.tools.DumpLogSegments \
                --deep-iteration \
                --files /var/lib/kafka/data/test-topic-0/00000000000000000000.log
```

