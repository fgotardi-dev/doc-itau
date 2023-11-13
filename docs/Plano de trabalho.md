Plano de Trabalho

Para esse plano de trabalho dividi as tarefas em tópicos:

1. Criar o ambiente em cloud pública, escolhi a AWS.

2. Utilizei a ferramenta TERRAFORM para provisionar as instâncias, cada instância foi criada em uma AZ.

3. Criei 5 instâncias, sendo elas:
    - 3 brokers;
    - 1 zookeeper;
    - 1 prometheus + grafana

4. Todos os serviços instalados estão configurados no systemd

5. Criei um tópico de nome test-topic, com 3 partições e 3 replicações.
bin/kafka-topics.sh --create --bootstrap-server 172.31.81.151:9092 --replication-factor 3 --partitions 3 --topic test-topic

6. Para produzir as mensagens fiz um "for" para mandar quantas mensagens forem definidas.
for x in {1..1000}; do echo $x; sleep 2; done | bin/kafka-console-producer.sh --topic test-topic --bootstrap-server 172.31.81.151:9092

7. Para consumir as mensagens executei o comando básico.
bin/kafka-console-consumer.sh --topic test-topic --from-beginning --bootstrap-server 172.31.81.151:9092

8. Criei uma instância prometheus+grafana, os serviços estão vinculados ao systemd.

Acesso WEB PROMETHEUS:
http://18.208.132.169:9090/

Acesso WEB GRAFANA:
http://18.208.132.169:3000/