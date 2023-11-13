Criação de tópico

Nesta criação de tópico, define-se fator de replicação e número de partições:
 bin/kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 replication-factor 1 --partitions 3

Para certificar que o tópico foi criado podemos rodar o describe e ver os detalhes:
bin/kafka-topics.sh --describe --topic test-topic --bootstrap-server localhost:9092