# Docker e AWS - Compass UOL
Prática para executar os conhecimentos obtidos em Docker e AWS durante o programa de bolsas da Compass UOL
## Instalação e configuração do Docker no Host EC2
A instalação e configuração da máquina foi realizada via Script de Start Instance (user_data.sh). Antes de inicializar a instância em sua configuração é possível 
configurar tarefas comuns automatizadas e executar scripts, para a instalação do docker em um AMI foi utilizado o seguinte script:

```
#!/bin/sh 
# Atualiza os pacotes do SO e instala o serviço docker
  $ sudo yum update -y 
  $ sudo yum install -y docker 
# Inicia o serviço
  $ Docker sudo service docker start
# Adiciona o usuário ec2-user ao grupo docker (para evitar uso do sudo com Docker) 
  $ sudo usermod -aG docker ec2-user 
# Configura o Docker para iniciar automaticamente ao boot
  $ sudo service docker start
# (Opcional) Baixa uma imagem de teste para verificar a instalação 
  $ sudo docker pull hello-world
```
