Plano de Trabalho

Terraform, escrevi esse código para provisionar as instâncias na AWS:

main.tf
provider "aws" {
  access_key = var.access_key
  secret_key = var.secret_key
  region     = "us-east-1"
}

resource "aws_instance" "ec2_instance1" {
  ami                    = var.ami_id
  subnet_id              = var.subnet_id
  instance_type          = var.instance_type
  key_name               = var.ami_key_pair_name
  vpc_security_group_ids = [aws_security_group.broker_sg.id]

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
StandardError=append:/usr/local/kafka-server/kafka.log
Restart=on-abnormal
[Install]
WantedBy=multi-user.target" >> /etc/systemd/system/kafka.service
systemctl daemon-reload
systemctl enable --now kafka.service
systemctl start kafka
EOF

  tags = {
    Name = "broker"
  }
}

resource "aws_security_group" "broker_sg" {
  name        = "broker"
  description = "Libera 22 e 9092  na instancia EC2"

  egress {
    description = "HTTP to EC2"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "HTTPS to EC2"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SSH to EC2"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    description = "BROKER to EC2"
    from_port   = 9092
    to_port     = 9092
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "ICMP to EC2"
    from_port   = 8
    to_port     = 0
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }

}

resource "aws_instance" "ec2_instance2" {
  ami                    = var.ami_id
  subnet_id              = var.subnet_id
  instance_type          = var.instance_type
  key_name               = var.ami_key_pair_name
  vpc_security_group_ids = [aws_security_group.prom_sg.id]
  tags = {
    Name = "prometheus"
  }
}

resource "aws_security_group" "prom_sg" {
  name        = "prometheus"
  description = "Libera 22,9090 e 3000  na instancia EC2"

  egress {
    description = "HTTP to EC2"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    description = "HTTPS to EC2"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SSH to EC2"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "PROMETHEUS to EC2"
    from_port   = 9090
    to_port     = 9090
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "GRAFANA to EC2"
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "ICMP to EC2"
    from_port   = 8
    to_port     = 0
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }

}

Arquivo variables.tf:

variable "access_key" {
  description = "Access key to AWS console"
}
variable "secret_key" {
  description = "Secret key to AWS console"
}

variable "instance_name1" {
  description = "Name of the instance to be created"
  default     = "broker"
}

variable "instance_name2" {
  description = "Name of the instance to be created"
  default     = "prometheus"
}

variable "instance_type" {
  default = "t3a.large"
}

variable "subnet_id" {
  description = "The VPC subnet the instance(s) will be created in"
  default     = "subnet-696c1624"
}

variable "ami_id" {
  description = "The AMI to use"
  default     = "ami-0fc5d935ebf8bc3bc"
}

variable "ami_key_pair_name" {
  default = "itauser"
}

