Produtor e Consumidor

Criei um script que envia 100 mensagens usando o comando para produzir mensagens:
 for x in {1..100}; do echo $x; sleep 2; done | bin/kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092

Para consumir as mensagens execute o comando abaixo:
bin/kafka-console-consumer.sh --topic test-topic --from-beginning --bootstrap-server localhost:9092