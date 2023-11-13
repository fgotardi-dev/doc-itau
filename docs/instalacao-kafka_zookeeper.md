Instalação Kafka e Zookeeper Standlone

Para instalação do Kafka e Zookeeper escolhi usar o Terraform para criação das instâncias e configuração dos softwares através de bootstrape.

Execute o script no Terraform para fazer a instalação do Kafka e Zookeeper.

user_data = <<EOF
#!/bin/bash
apt update
apt upgrade -y
apt install openjdk-11-jre-headless -y
mkdir /usr/local/kafka-server
curl "https://archive.apache.org/dist/kafka/2.7.0/kafka_2.12-2.7.0.tgz" -o /usr/local/kafka-server/kafka.tgz
tar -xvzf /usr/local/kafka-server/kafka.tgz -C /usr/local/kafka-server/ --strip 1
mv /usr/local/kafka-server/kafka/* /usr/local/kafka-server/
echo "delete.topic.enable = true" >> /usr/local/kafka-server/config/server.properties
echo "[Unit]
Description=Apache Zookeeper Server
Requires=network.target remote-fs.target
After=network.target remote-fs.target
[Service]
Type=simple
ExecStart=/usr/local/kafka-server/bin/zookeeper-server-start.sh /usr/local/kafka-server/config/zookeeper.properties
ExecStop=/usr/local/kafka-server/bin/zookeeper-server-stop.sh
Restart=on-abnormal
[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/zookeeper.service
echo "[Unit]
Requires=zookeeper.service
After=zookeeper.service
[Service]
Type=simple
ExecStart=/usr/local/kafka-server/bin/kafka-server-start.sh /usr/local/kafka-server/config/server.properties
ExecStop=/usr/local/kafka-server/bin/kafka-server-stop.sh
StandardOutput=append:/usr/local/kafka-server/kafka.log
StandardError=append:/usr/local/kafka-server/kafka.log
Restart=on-abnormal
[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/kafka.service
systemctl daemon-reload
systemctl enable --now kafka.service
systemctl start kafka
EOF

Depois de adicionar a linha do java agent execute o comando curl:
curl localhost:8080

Adicione as linhas no arquivo kafka.service e zookeeper.service;
kafka.service:
Environment="KAFKA_OPTS=-javaagent:/home/ubuntu/prometheus/jmx_prometheus_javaagent-0.19.0.jar=8080:/home/ubuntu/prometheus/kafka-0-8-2.yml"

zookeeper.service:
Environment="EXTRA_ARGS=-javaagent:/home/ubuntu/prometheus/jmx_prometheus_javaagent-0.19.0.jar=8080:/home/ubuntu/prometheus/zookeeper.yaml"