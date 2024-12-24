<img src=https://github.com/RicardoBock/dockerCompass/blob/main/imgs/compass.png>
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

## Criação de uma VPC:
A criação de uma VPC (<i>Virtual Private Cloud</i>) é necessária, no qual será constituída por 4 subnets, 2 públicas e 2 privadas, 1 internet gateway para acesso a rede da subnet pública e 1 NAT gateway para acesso da rede das subnets privadas, as sub-redes vão ser utilizadas da seguinte forma:
 * Duas públicas para o acesso do Load Balancer e do bastion host as instâncias na sub-redes privadas;
 * Duas privadas para utilização das instâncias privadas e do banco de dados;
   
<img src=https://github.com/RicardoBock/dockerCompass/blob/main/imgs/vpc.png>

<img src=https://github.com/RicardoBock/dockerCompass/blob/main/imgs/subnets.png>

## Criação dos grupos de segurança:
Os grupos de segurança servem para configurar os acessos dos diferentes tipos de protocolo que permite apenas o tráfego autorizado e bloqueia tentativas de acesso não autorizadas, é necessário configurar os grupos de segurança para o EC2, RDS, EFS e ELB. As configurações necessárias se encontram a seguir nas tabelas:</br>

<img src=https://github.com/RicardoBock/dockerCompass/blob/main/imgs/Ssgsss.png>
<hr>

**CONFIGURAÇÃO DO SECURITY GROUP EC2 PÚBLICO**

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
      <td>HTTP</td>
      <td>80</td>
      <td>0.0.0.0/0</td>
    </tr>
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
      <td>Tudo</td>
      <td>0.0.0.0/0</td>
    </tr>
  </tbody>
</table>
  
 <hr>
      
**CONFIGURAÇÃO DO SECURITY GROUP EC2 PRIVADO**  


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
      <td>Security Group da EC2 Pública</td>
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

A criação do banco de dados foi realizada utilizando o RDS (<i>Relational Database Services</i>), o banco de dados alocado foi o MySQL v8.0.39, o grupo de segurança criado para o RDS anteriormente foi utilizado, o nome do banco de dados, o usuário e a senha foram definidas e suas sub-redes privadas em duas zonas de disponibilidade:

* us-east-1a (PROJETO-WORDPRESS-PRIVADA-1A)
* us-east-1b (PROJETO-WORDPRESS-PRIVADA-1B)


## CRIAÇÃO DO ELASTIC FYLE SYSTEM

O _Amazon Elastic File System_ é utilizado para armazenamento de arquivos elástico, isto é, aumenta ou diminui o seu tamanho de acordo com a necessidade, permitindo assim escalabilidade, para a sua criação é realizado o seguinte processo:

1º Criação da EFS <br>
2º Configuração das subredes e o grupo de segurança que será utilizado o EFS <br>
3º Anexar via DNS com a linha no terminal <br>

```
sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport file-system-id.efs.aws-region.amazonaws.com:/ /efs-mount-point
```
Quando criado o EFS e atrelado a instância é possível verificar seu funcionamento por meio do comando

```
df -h
```

<img src=https://github.com/RicardoBock/dockerCompass/blob/main/imgs/efs%20funcional.png>


## Instalação e configuração do Docker na inicialização da EC2
A criação da de um <i>NAT GATEWAY</i> para a instalação dos pacotes na instância privada é necessário, para a realização desta etapa é necessária, possibilitando a conexão das instâncias privadas para fora da VPC sem a possibilidade de acesso externo, com a criação do NAT GATEWAY e um ip elástico atribuido a ele, na tabela de rotas das rotas da subnet privadas é necessário anexar a _NAT GATEWAY_ as rotas da subnet específica


<img src=https://github.com/RicardoBock/dockerCompass/blob/main/imgs/natgateway.png>



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

* us-east-1a (PROJETO-WORDPRESS-PRIVADA-1A)
* us-east-1b (PROJETO-WORDPRESS-PRIVADA-1B)

<img instancia>

A criação de uma instância em uma subrede pública foi realizada para a utilização dela como um bastion-host, no qual é possível o acesso por meio da cópia da chave .pem da maquina particular para esta instância e acessando esta instância por meio do código:

```
cd ~.ssh
vi "nome-da-chave.pem" (aqui é realizada a cópia da instância da máquina local)
ssh -i "nome-da-chave.pem" ec2-user@ip-privado-da-instância
```


## CRIAÇÃO DO LOAD BALANCER


Para esta aplicação foi utilizado o <i>Classic Load Balancer</i>, sua função é distribuir o tráfego de aplicações de entrada entre vários destinos e dispositivos virtuais em uma ou mais zonas de disponibilide
Na criação foi necessário modificar o PATH de verificação de integridade para _"/wp-admin/install.php"_ devido ao código de retorno de sucesso, que no caso do wordpress é 301 ou 302 e do classic load balancer é 200, caso não seja feita esta modificação, as instâncias não vão passar no _HEALTH CHECK_ e não sera possível acessar o site por meio do DNS do load balancer.
Após a criação e atríbuidas as instâncias privadas, será realizado o teste de integridade, caso teste de integridade concluído com sucesso será possível acessar o site via DNS

<img src=https://github.com/RicardoBock/dockerCompass/blob/main/imgs/Instancias_health_check.png>

Ao acessar o DNS em que o AWS disponibiliza, e a página de configuração do Wordpress aparecer, o load balancer e as instâncias estão funcionais 

<img src=https://github.com/RicardoBock/dockerCompass/blob/main/imgs/site_funcional.png>

<img src=https://github.com/RicardoBock/dockerCompass/blob/main/imgs/Site_funcional_2.png>


## CRIAÇÃO DO AUTO SCALLING


O Auto Scalling é uma função da AWS que permite a alocação de mais instâncias dependendo do número de acessos e requests nos servidores, permitindo uma escalabilidade nos serviços e disponibilidade instantânea, sua configuração é feita com a quantidade mínima e máxima de instâncias que é desejado obter caso ocorra algum problema, como foi criado o auto scalling para meios de educação o máximo de instâncias configurados foram 2, o teste é realizado derrubando a instância original e percebendo a reposição quase instantânea por outra reposição.

<img src=https://github.com/RicardoBock/dockerCompass/blob/main/imgs/auto-scalling-funcionando.png>

## CONCLUSÃO

A utilização da AWS e seus serviços permite com que aplicações WEB estejam disponíveis e integros, com a possibilidade de escalabilidade de acordo com as necessidades dos clientes, conceitos como docker, banco de dados, requisições e seus códigos e arquitetura de sistemas foi reforçado com a prática deste projeto.





