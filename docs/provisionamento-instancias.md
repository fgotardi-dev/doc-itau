Terraform

provider "aws" {
  access_key = var.access_key
  secret_key = var.secret_key
  region     = "us-east-1"
}

resource "aws_security_group" "broker_sg1" {
  name = "broker1"
  # description = "Libera portas na instancia EC2"

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

  #egress {
  #  description = "BROKER to EC2"
  #  from_port   = 9092
  #  to_port     = 9092
  #  protocol    = "tcp"
  #  cidr_blocks = ["0.0.0.0/0"]
  #}

  ingress {
    description = "ICMP to EC2"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "ICMP to EC2"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "ec2_instance1" {
  ami                    = var.ami_id
  subnet_id              = var.subnet_id_a
  instance_type          = var.instance_type
  key_name               = var.ami_key_pair_name
  vpc_security_group_ids = [aws_security_group.broker_sg1.id]
  availability_zone      = "us-east-1a"
  user_data              = <<EOF
#! /bin/bash
sudo apt update
sudo apt upgrade -y
sudo apt install openjdk-11-jre-headless -y
sudo mkdir /usr/local/kafka-server
sudo curl "https://archive.apache.org/dist/kafka/2.7.0/kafka_2.12-2.7.0.tgz" -o /usr/local/kafka-server/kafka.tgz
sudo tar -xvzf /usr/local/kafka-server/kafka.tgz -C /usr/local/kafka-server/ --strip 1
sudo mv /usr/local/kafka-server/kafka/* /usr/local/kafka-server/
echo "delete.topic.enable = true" >> /usr/local/kafka-server/config/server.properties
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
sudo systemctl daemon-reload
sudo systemctl enable --now kafka.service
EOF

  tags = {
    Name = "broker1"
  }
}

resource "aws_security_group" "broker_sg2" {
  name        = "broker2"
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

  #egress {
  #  description = "BROKER to EC2"
  #  from_port   = 9092
  #  to_port     = 9092
  #  protocol    = "tcp"
  #  cidr_blocks = ["0.0.0.0/0"]
  #}

  ingress {
    description = "ICMP to EC2"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "ICMP to EC2"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "ec2_instance2" {
  ami                    = var.ami_id
  subnet_id              = var.subnet_id_b
  instance_type          = var.instance_type
  key_name               = var.ami_key_pair_name
  vpc_security_group_ids = [aws_security_group.broker_sg2.id]
  user_data              = <<EOF
#! /bin/bash
sudo apt update
sudo apt upgrade -y
sudo apt install openjdk-11-jre-headless -y
sudo mkdir /usr/local/kafka-server
sudo curl "https://archive.apache.org/dist/kafka/2.7.0/kafka_2.12-2.7.0.tgz" -o /usr/local/kafka-server/kafka.tgz
sudo tar -xvzf /usr/local/kafka-server/kafka.tgz -C /usr/local/kafka-server/ --strip 1
sudo mv /usr/local/kafka-server/kafka/* /usr/local/kafka-server/
echo "delete.topic.enable = true" >> /usr/local/kafka-server/config/server.properties
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
sudo systemctl daemon-reload
sudo systemctl enable --now kafka.service
EOF

  availability_zone = "us-east-1b"
  tags = {
    Name = "broker2"
  }
}

resource "aws_security_group" "broker_sg3" {
  name        = "broker3"
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

  #egress {
  #  description = "BROKER to EC2"
  #  from_port   = 9092
  #  to_port     = 9092
  #  protocol    = "tcp"
  #  cidr_blocks = ["0.0.0.0/0"]
  #}

  ingress {
    description = "ICMP to EC2"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "ICMP to EC2"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "ec2_instance3" {
  ami                    = var.ami_id
  subnet_id              = var.subnet_id_c
  instance_type          = var.instance_type
  key_name               = var.ami_key_pair_name
  vpc_security_group_ids = [aws_security_group.broker_sg3.id]
  user_data              = <<EOF
#! /bin/bash
sudo apt update
sudo apt upgrade -y
sudo apt install openjdk-11-jre-headless -y
sudo mkdir /usr/local/kafka-server
sudo curl "https://archive.apache.org/dist/kafka/2.7.0/kafka_2.12-2.7.0.tgz" -o /usr/local/kafka-server/kafka.tgz
sudo tar -xvzf /usr/local/kafka-server/kafka.tgz -C /usr/local/kafka-server/ --strip 1
sudo mv /usr/local/kafka-server/kafka/* /usr/local/kafka-server/
echo "delete.topic.enable = true" >> /usr/local/kafka-server/config/server.properties
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
sudo systemctl daemon-reload
sudo systemctl enable --now kafka.service
EOF

  availability_zone = "us-east-1c"
  tags = {
    Name = "broker3"
  }
}

resource "aws_security_group" "zook_sg" {
  name        = "zookeeper"
  description = "Libera 22 e 2181  na instancia EC2"

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

  #  egress {
  #    description = "ZOOKEEPER to EC2"
  #    from_port   = 2181
  #    to_port     = 2181
  #    protocol    = "tcp"
  #    cidr_blocks = ["0.0.0.0/0"]
  #  }

  ingress {
    description = "ICMP to EC2"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "ICMP to EC2"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "ec2_instance4" {
  ami                    = var.ami_id
  subnet_id              = var.subnet_id_d
  instance_type          = var.instance_type
  key_name               = var.ami_key_pair_name
  vpc_security_group_ids = [aws_security_group.zook_sg.id]
  user_data              = <<EOF
#!/bin/bash
sudo apt update
sudo apt upgrade -y
sudo apt install openjdk-11-jre-headless -y
sudo mkdir /usr/local/kafka-server
sudo curl "https://archive.apache.org/dist/kafka/2.7.0/kafka_2.12-2.7.0.tgz" -o /usr/local/kafka-server/kafka.tgz
sudo tar -xvzf /usr/local/kafka-server/kafka.tgz -C /usr/local/kafka-server/ --strip 1
sudo mv /usr/local/kafka-server/kafka/* /usr/local/kafka-server/
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
sudo systemctl daemon-reload
sudo systemctl enable --now zookeeper.service
EOF
  availability_zone      = "us-east-1d"
  tags = {
    Name = "zookeeper"
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

  # egress {
  #   description = "PROMETHEUS to EC2"
  #   from_port   = 9090
  #   to_port     = 9090
  #   protocol    = "tcp"
  #   cidr_blocks = ["0.0.0.0/0"]
  # }

  #  egress {
  #    description = "GRAFANA to EC2"
  #    from_port   = 3000
  #    to_port     = 3000
  #    protocol    = "tcp"
  #    cidr_blocks = ["0.0.0.0/0"]
  #  }

  ingress {
    description = "ICMP to EC2"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "ICMP to EC2"
    from_port   = -1
    to_port     = -1
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "ec2_instance5" {
  ami                    = var.ami_id
  subnet_id              = var.subnet_id_f
  instance_type          = var.instance_type
  key_name               = var.ami_key_pair_name
  vpc_security_group_ids = [aws_security_group.prom_sg.id]
  availability_zone      = "us-east-1f"
  tags = {
    Name = "prometheus"
  }
}