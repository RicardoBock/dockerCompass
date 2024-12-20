# DOCKER E AWS - COMPASS UOL
## Prática para executar os conhecimentos obtidos em Docker e AWS durante o programa de bolsas da Compass UOL
## REQUISITOS:
* Conta AWS
* VPC
* Instância EC2 Amazon Linux 2023
* Security Groups
* RDS - Database MYSQL
* EFS - File System
* ELB - Elastic Load Balancer

## Criação de uma VPC de três camadas:
A criação de uma VPC (<i>Virtual Private Cloud</i>) é necessária, no qual será constituída por 6 subnets, 2 públicas e 4 privadas, 1 internet gateway para acesso a rede da subnet pública e 1 NAT gateway para acesso da rede das subnets privadas, as sub-redes vão ser utilizadas da seguinte forma:
 * Duas públicas para o acesso do Load Balancer e do bastion host as instâncias na sub-redes privadas;
 * Duas privadas para utilização das instâncias privadas;
 * Duas privadas para a utilização dos bancos de dados.
<imagem do esquema do VPC>

## Criação dos grupos de segurança:
Os grupos de segurança servem para configurar os acessos dos diferentes tipos de protocolo que permite apenas o tráfego autorizado e bloqueia tentativas de acesso não autorizadas, é necessário configurar os grupos de segurança para o EC2, RDS, EFS e ELB. As configurações necessárias se encontram a seguir nas tabelas:</br>
<b>CONFIGURAÇÃO DO SECURITY GROUP EC2 </b> 


INBOUND RULES:
<table>
  
  <thead>
    <tr>
      <th>TIPO
      </th>
      <th>
        INTERVALO DE PORTAS
      </th>
      <th>
        ORIGEM
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>SHH</td>
      <td>22</td>
      <td>Meu IP</td>
    </tr>
    <tr>
      <td>NFS</td>
      <td>2049</td>
      <td>Security Group do EFS</td>
    </tr>
    <tr>
      <td>HTTP</td>
      <td>80</td>
      <td>Security Group do ELB</td>
    </tr>
    <tr>
      <td>MYSQL/Aurora</td>
      <td>3306</td>
      <td>Security Group do RDS</td>
    </tr>
  </tbody>
</table>


OUTBOUND RULES:


<table>
  
  <thead>
    <tr>
      <th>TIPO
      </th>
      <th>
        INTERVALO DE PORTAS
      </th>
      <th>
        ORIGEM
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>HTTP</td>
      <td>80</td>
      <td>0.0.0.0/0</td>
    </tr>
    <tr>
      <td>MYSQL/Aurora</td>
      <td>3306</td>
      <td>Security Group do RDS</td>
    </tr>
    <tr>
      <td>NFS</td>
      <td>2049</td>
      <td>Security Group do EFS</td>
    </tr>
    <tr>
      <td>Todo o tráfego</td>
      <td>Tudo</td>
      <td>0.0.0.0/0</td>
    </tr>
  </tbody>
</table>
</br>
<hr>
<b>CONFIGURAÇÃO DO SECURITY GROUP DA RDS: </b>
<br>

INBOUND RULES:


<table>
  <thead>
    <tr>
      <th>TIPO
      </th>
      <th>
        INTERVALO DE PORTAS
      </th>
      <th>
        ORIGEM
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>MYSQL/Aurora</td>
      <td>3306</td>
      <td>Security Group da EC2</td>
    </tr>
  </tbody>
</table>


OUTBOUND RULES:


<table>
  <thead>
    <tr>
      <th>TIPO
      </th>
      <th>
        INTERVALO DE PORTAS
      </th>
      <th>
        ORIGEM
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Todo o tráfego</td>
      <td>Todas</td>
      <td>0.0.0.0/0</td>
    </tr>
  </tbody>
</table>
</br>
<hr>
<b>CONFIGURAÇÃO DO SECURITY GROUP DA EFS: </b></br>


INBOUND RULES:


<table>
  <thead>
    <tr>
      <th>TIPO
      </th>
      <th>
        INTERVALO DE PORTAS
      </th>
      <th>
        ORIGEM
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>NFS</td>
      <td>2049</td>
      <td>Security Group da EC2</td>
    </tr>
  </tbody>
</table>


OUTBOUND RULES:


<table>
  <thead>
    <tr>
      <th>TIPO
      </th>
      <th>
        INTERVALO DE PORTAS
      </th>
      <th>
        ORIGEM
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Todo o tráfegos</td>
      <td>Todas</td>
      <td>0.0.0.0/0</td>
    </tr>
  </tbody>
</table>
<hr>
<b>CONFIGURAÇÃO DO SECURITY GROUP DA ELB: </b></br>


INBOUND RULES:


<table>
  <thead>
    <tr>
      <th>TIPO
      </th>
      <th>
        INTERVALO DE PORTAS
      </th>
      <th>
        ORIGEM
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>HTTP</td>
      <td>80</td>
      <td>0.0.0.0/0</td>
    </tr>
  </tbody>
</table>


OUTBOUND RULES:


<table>
  <thead>
    <tr>
      <th>TIPO
      </th>
      <th>
        INTERVALO DE PORTAS
      </th>
      <th>
        ORIGEM
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>HTTP</td>
      <td>80</td>
      <td>0.0.0.0/0</td>
    </tr>
  </tbody>
</table>
<hr>

## CRIAÇÃO DO BANCO DE DADOS RDS

A criação do banco de dados foi realizada utilizando o RDS (<i>Relational Database Services</i>), o banco de dados alocado foi o MySQL v8.0.39, o grupo de segurança criado para o RDS anteriormente foi utilizado, o nome do banco de dados, o usuário e a senha foram definidas e suas sub-redes privadas próprias em duas zonas de disponibilidade:

* us-east-1a (rds-subnet-privada-1a)
* us-east-1b (rds-subnet-privada-1b)

Ao criar o banco de dados é gerado um endpoint necessário para utilizar no documento compose.yml.

## CRIAÇÃO DO ELASTIC FYLE SYSTEM

O <i>Amazon Elastic File System</i> é utilizado para armazenamento de arquivos elástico, isto é, aumenta ou diminui o seu tamanho de acordo com a necessidade, permitindo assim escalabilidade 

## Instalação e configuração do Docker na inicialização da EC2
A máquina alocada na EC2 foi a AMI 2023 (<i>Amazon Machine Image</i>), o grupo de segurança para EC2 criado anteriormente foi utilizado e a instalação e configuração da máquina foi realizada via Script de <b>Start Instance</b> (user_data.sh). Utilizando os dados de Usuário antes de inicializar a instância em sua configuração é possível configurar tarefas comuns automatizadas e executar scripts, para a instalação do docker foi utilizado o seguinte script:

```
#!/bin/bash
 
sudo yum update -y
sudo yum install -y docker
 
sudo systemctl start docker
sudo systemctl enable docker
 
sudo usermod -aG docker ec2-user
newgrp docker
 
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
 
sudo chmod +x /usr/local/bin/docker-compose
 
sudo mkdir /app
 
cat <<EOF > /app/compose.yml
services:
 
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: <RDS_ENDPOINT>
      WORDPRESS_DB_USER: <RDS_USER>
      WORDPRESS_DB_PASSWORD: <RDS_PASSWORD>
      WORDPRESS_DB_NAME: <RDS_DATABASE>
    volumes:
      - /mnt/efs:/var/www/html
EOF
 
docker-compose -f /app/compose.yml up -d


```

Duas instâncias criadas em diferentes sub-redes privadas em diferentes zonas de disponibilidade diferentes:

* us-east-1a (ec2-subnet-privada-1a)
* us-east-1b (ec2-subnet-privada-1b)





