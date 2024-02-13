# Servidor Kafka Local

- [Servidor Kafka Local](#servidor-kafka-local)
  - [Introdução](#introdução)
  - [Tecnologias utilizadas](#tecnologias-utilizadas)
  - [Pré-requisitos](#pré-requisitos)
  - [Como usar](#como-usar)
    - [Acessando o Kafka-UI](#acessando-o-kafka-ui)
    - [Acessando o Kafka](#acessando-o-kafka)
    - [ATENÇÃO](#atenção)
  - [Comandos úteis](#comandos-úteis)
    - [Conectar na linha de comando do Kafka](#conectar-na-linha-de-comando-do-kafka)
    - [Criar tópico](#criar-tópico)
    - [Produzir mensagens sem chave](#produzir-mensagens-sem-chave)
    - [Consumir mensagens sem chave](#consumir-mensagens-sem-chave)
    - [Produzir mensagens com chave](#produzir-mensagens-com-chave)
    - [Consumir mensagens com chave](#consumir-mensagens-com-chave)
    - [Consumir mensagens em grupo](#consumir-mensagens-em-grupo)
    - [Consumir mensagens em grupo com chave](#consumir-mensagens-em-grupo-com-chave)
      - [Exemplo de mensagem](#exemplo-de-mensagem)
    - [Consumir mensagens com headers](#consumir-mensagens-com-headers)
      - [Example Messages:](#example-messages)
    - [Arquivos de Logs do Kafka](#arquivos-de-logs-do-kafka)
  - [Desligar o ambiente](#desligar-o-ambiente)
  - [Configurações](#configurações)
    - [min.insync.replicas](#mininsyncreplicas)
  - [Comandos adicionais](#comandos-adicionais)
    - [Listar tópicos](#listar-tópicos)
    - [Descrever tópico](#descrever-tópico)
    - [Alterar partições do tópico](#alterar-partições-do-tópico)
    - [Como ver grupos de consumidores](#como-ver-grupos-de-consumidores)
    - [Como ver detalhes de um grupo de consumidores](#como-ver-detalhes-de-um-grupo-de-consumidores)
    - [Como ver o log de commit](#como-ver-o-log-de-commit)

## Introdução

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

### Acessando o Kafka-UI

Para acessar o Kafka-UI, acesse `http://DOCKER_HOST_IP:8080` no navegador.

### Acessando o Kafka

Para acessar o Kafka, utilize os endereços `DOCKER_HOST_IP:9092`, `DOCKER_HOST_IP:9093` e `DOCKER_HOST_IP:9094` para os brokers 1, 2 e 3, respectivamente.

Exemplo:

Se o IP da máquina onde o Docker está rodando for `192.168.1.5`, os endereços para os brokers serão:

```properties
spring.kafka.bootstrap-servers=192.168.1.5:9092,192.168.1.5:9093,192.168.1.5:9094
```

### ATENÇÃO

Caso o IP da máquina onde o Docker está rodando seja alterado, é necessário alterar a propriedade `DOCKER_HOST_IP` no arquivo `.env` e reiniciar os containers.

Também será necessário alterar esse IP nos aplicativos que se conectam ao Kafka.

## Comandos úteis

### Conectar na linha de comando do Kafka

Para criar tópicos diretamente no Kafka, na linha de comando, primeiro é necessário se conectar a um dos brokers.

```sh
docker exec -it kafka1 bash
```

onde `kafka1` é o nome do container do broker 1. Se quiser se conectar a outro broker, substitua `kafka1` pelo nome do container do broker desejado (`kafka2` ou `kafka3`).

### Criar tópico

Para criar um tópico, execute o comando `kafka-topics` no broker desejado.

```bash
kafka-topics --bootstrap-server kafka1:19092 \
             --create \
             --topic test-topic \
             --replication-factor 3 \
             --partitions 3
```

### Produzir mensagens sem chave

Para produzir mensagens sem chave, execute o comando `kafka-console-producer` no broker desejado.

```bash
docker exec --interactive --tty kafka1  \
kafka-console-producer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic
```

### Consumir mensagens sem chave

Para consumir mensagens sem chave, execute o comando `kafka-console-consumer` no broker desejado.

```bash
docker exec --interactive --tty kafka1  \
kafka-console-consumer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --from-beginning
```

### Produzir mensagens com chave

Para produzir mensagens com chave, execute o comando `kafka-console-producer` no broker desejado, passando os parâmetros `--property "parse.key=true"` e `--property "key.separator=-"`.

```bash
docker exec --interactive --tty kafka1  \
kafka-console-producer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --property "parse.key=true" \
                       --property "key.separator=-"
```

### Consumir mensagens com chave

Para consumir mensagens com chave, execute o comando `kafka-console-consumer` no broker desejado, passando os parâmetros `--property "print.key=true"` e `--property "key.separator=-"`.

```bash
docker exec --interactive --tty kafka1  \
kafka-console-consumer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --from-beginning \
                       --property "print.key=true" \
                       --property "key.separator=-"
```

### Consumir mensagens em grupo

Para consumir mensagens em grupo, execute o comando `kafka-console-consumer` no broker desejado, passando o parâmetro `--group` com o nome do grupo.

```bash
docker exec --interactive --tty kafka1  \
kafka-console-consumer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --group my-group
```

### Consumir mensagens em grupo com chave

Para consumir mensagens em grupo com chave, execute o comando `kafka-console-consumer` no broker desejado, passando os parâmetros `--group` com o nome do grupo, `--property "print.key=true"` e `--property "key.separator=-"`.

```bash
docker exec --interactive --tty kafka1  \
kafka-console-consumer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --group my-group \
                       --property "print.key=true" \
                       --property "key.separator=-"
```

#### Exemplo de mensagem

Após executar o comando acima, vocÊ pode produzir mensagens com chave, bastando seguir o exemplo abaixo:

```bash
a-abc
b-bus
```

### Consumir mensagens com headers

Para consumir mensagens com headers, execute o comando `kafka-console-consumer` no broker desejado, passando os parâmetros `--property "print.headers=true"` e `--property "print.timestamp=true"`.

```bash
docker exec --interactive --tty kafka1  \
kafka-console-consumer --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 \
                       --topic test-topic \
                       --group my-group \
                       --property "print.headers=true" \
                       --property "print.timestamp=true"
```

#### Example Messages:

Após executar o comando acima, você pode produzir mensagens com headers, bastando seguir o exemplo abaixo:

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

Para listar todos os tópicos do Kafka, execute o comando `kafka-topics` no broker desejado.

```bash
docker exec --interactive --tty kafka1  \
kafka-topics --bootstrap-server kafka1:19092 --list
```

### Descrever tópico

Para descrever ou mais tópicos do Kafka, execute o comando `kafka-topics` no broker desejado.

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

Para alterar o número de partições de um tópico, execute o comando `kafka-topics` no broker desejado.

```bash
docker exec --interactive --tty kafka1  \
kafka-topics --bootstrap-server kafka1:19092 \
             --alter \
             --topic test-topic \
             --partitions 40
```

### Como ver grupos de consumidores

Parea ver os grupos de consumidores, execute o comando `kafka-consumer-groups` no broker desejado.

```bash
docker exec --interactive --tty kafka1  \
kafka-consumer-groups --bootstrap-server kafka1:19092 \
                      --list
```

### Como ver detalhes de um grupo de consumidores

Para ver detalhes de um grupo de consumidores, execute o comando `kafka-consumer-groups` no broker desejado, passando o parâmetro `--group`, com o nome do grupo, e o parâmetro `--describe`.

```bash
docker exec --interactive --tty kafka1  \
kafka-consumer-groups --bootstrap-server kafka1:19092 \
                      --describe \
                      --group my-group
```

### Como ver o log de commit

Para ver o log de commit, execute o comando `kafka-consumer-groups` no broker desejado, passando o parâmetro `--deep-iteration` e o parâmetro `--files`, com o caminho do arquivo de log.

```bash
docker exec --interactive --tty kafka1  \
kafka-run-class kafka.tools.DumpLogSegments \
                --deep-iteration \
                --files /var/lib/kafka/data/test-topic-0/00000000000000000000.log
```
