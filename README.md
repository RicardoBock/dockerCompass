# DOCKER E AWS - COMPASS UOL
## Prática para executar os conhecimentos obtidos em Docker e AWS durante o programa de bolsas da Compass UOL
## REQUISITOS:
* Conta AWS
* VPC
* Instância EC2 Amazon Linux 2023
* Security Groups
* RDS - Database MYSQL
* EFS - File System

## Instalação e configuração do Docker na inicialização da EC2
A instalação e configuração da máquina foi realizada via Script de <b>Start Instance</b> (user_data.sh). Utilizando os dados de Usuário antes de inicializar a instância em sua configuração é possível 
configurar tarefas comuns automatizadas e executar scripts, para a instalação do docker em um AMI foi utilizado o seguinte script:

```
#!/bin/bash

sudo yum update -y
sudo yum install git -y
sudo yum install docker -y
sudo usermod -a -G docker ec2-user
id ec2-user
sudo newgrp docker

#Ativar docker
sudo systemctl enable docker.service
sudo systemctl start docker.service

#Instalar o docker compose
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose


```


