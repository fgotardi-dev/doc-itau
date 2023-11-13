Instalação Prometheus e Grafana

No servidor do kafka baixe os arquivos jmx e de configuração:
wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.19.0/jmx_prometheus_javaagent-0.19.0.jar
https://github.com/prometheus/jmx_exporter/blob/main/example_configs/kafka-0-8-2.yml

No servidor do kafka adicione as linhas no systemd do kafka /etc/systemd/system/kafka.service:
[Unit]
Requires=zookeeper.service
After=zookeeper.service
[Service]
Type=simple
Environment="KAFKA_OPTS=-javaagent:/home/ubuntu/prometheus/jmx_prometheus_javaagent-0.19.0.jar=8080:/home/ubuntu/prometheus/kafka-0-8-2.yml"
ExecStart=/usr/local/kafka-server/bin/kafka-server-start.sh /usr/local/kafka-server/config/server.properties
ExecStop=/usr/local/kafka-server/bin/kafka-server-stop.sh
StandardOutput=append:/usr/local/kafka-server/kafka.log
StandardError=append:/usr/local/kafka-server/kafka.log
Restart=on-abnormal
[Install]
WantedBy=multi-user.target

No servidor do prometheus baixe o arquivo:
https://github.com/prometheus/prometheus/releases/download/v2.47.2/prometheus-2.47.2.linux-amd64.tar.gz

No mesmo servidor edite o arquivo:
/etc/systemctl/system/prometheus.service

E adicione as linhas para adicionar o serviço no systemd:
# /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus Server
after=network-online.target

[Service]
ExecStart=/opt/prometheus/prometheus --config.file=/opt/prometheus/prometheus.yml --storage.tsdb.path=/opt/prometheus/data

[Install]
WantedBy=multi-user.target

Baixe o grafana e descompacte:
https://dl.grafana.com/oss/release/grafana-10.2.0.linux-amd64.tar.gz

tar -xzvf grafana-10.2.0.linux-amd64.tar.gz

Edite o arquivo defaults.ini para acessar como anonymous de acordo com as informações abaixo:
(anonymous) 
enable=true
org_role= Admin

Para teste execute o comando para ver se roda o grafana:

./grafana-server

Pare o serviço e crie o arquivo grafana.service no diretório /etc/systemd/system/
 # /etc/systemd/system/grafana.service
[Unit]
Description=Grafana Server
After=network-online.target

[Service]
WorkingDirectory=/opt/grafana
ExecStart=/opt/grafana/bin/grafana-server

[Install]
WantedBy=multi-user.target



