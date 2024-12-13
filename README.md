# Deploy de Aplicação WordPress no AWS EC2 com Docker e EFS

Este projeto descreve o processo de instalação e configuração de uma aplicação WordPress utilizando Docker, EFS e Load Balancer na AWS.

## Pré-requisitos

- Conta na AWS
- Instância EC2 com Amazon Linux 2
- EFS configurado
- Git instalado na sua máquina local

## Passo a Passo

### 1. Criando Instância EC2

1. Faça login no [AWS Management Console](https://aws.amazon.com/console/).
2. Navegue até **Services** > **EC2**.
3. Clique em **Launch Instance**.
4. Selecione **Amazon Linux 2 AMI**.
5. Escolha o tipo de instância **t2.micro**.
6. Na seção **Advanced Details**, adicione o seguinte script em **User data**:

    ```bash
    #!/bin/bash
    sudo yum update -y
    sudo yum install docker -y
    sudo systemctl start docker
    sudo systemctl enable docker
    sudo usermod -aG docker ec2-user
    sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    sudo mv /usr/local/bin/docker-compose /bin/docker-compose
    sudo yum install nfs-utils -y
    sudo mkdir /mnt/efs/
    sudo chmod +rwx /mnt/efs/
    ```

### 2. Configurando o EFS

1. Crie um sistema de arquivos EFS no console da AWS.
2. Monte o EFS na sua instância EC2:
    ```bash
    sudo mount -t nfs4 -o nfsvers=4.1 your_efs_dns:/ /mnt/efs
    ```

### 3. Criando o Docker Compose para WordPress

1. Conecte-se à sua instância EC2 via SSH.
2. Crie um arquivo `docker-compose.yml`:
    ```bash
    nano docker-compose.yml
    ```

3. Adicione o seguinte conteúdo ao arquivo:
    ```yaml
    version: '3.7'
    services:
      wordpress:
        image: wordpress:latest
        ports:
          - "80:80"
        environment:
          WORDPRESS_DB_HOST: db
          WORDPRESS_DB_USER: root
          WORDPRESS_DB_PASSWORD: example
        volumes:
          - wordpress_data:/var/www/html
          - efs_data:/var/www/html/wp-content

      db:
        image: mysql:5.7
        environment:
          MYSQL_ROOT_PASSWORD: example
        volumes:
          - db_data:/var/lib/mysql

    volumes:
      wordpress_data: {}
      db_data: {}
      efs_data:
        driver_opts:
          type: nfs
          o: addr=your_efs_dns,nolock
          device: :/
    ```

4. Inicie os contêineres:
    ```bash
    docker-compose up -d
    ```

### 4. Configurando o Load Balancer

1. No console da AWS, vá para **Services** > **EC2** > **Load Balancers**.
2. Clique em **Create Load Balancer** e selecione **Classic Load Balancer**.
3. Configure o Load Balancer para balancear o tráfego na porta 80.
4. Adicione suas instâncias EC2 ao Load Balancer.
5. Configure as regras de segurança para permitir o tráfego HTTP (porta 80).

### 5. Testando a Aplicação WordPress

1. Acesse o DNS do Load Balancer no seu navegador para verificar se a aplicação WordPress está funcionando.
2. A aplicação deve estar acessível na porta 80.

### 6. Versionamento com Git

1. Inicialize o repositório Git:
    ```bash
    git init
    ```

2. Adicione os arquivos ao repositório:
    ```bash
    git add .
    ```

3. Faça o commit das alterações:
    ```bash
    git commit -m "Initial commit"
    ```

4. Conecte-se ao repositório remoto e envie os arquivos:
    ```bash
    git remote add origin your_repository_url
    git push -u origin master
    ```

### 7. Documentação

Este documento serve como um guia passo a passo para configurar e executar uma aplicação WordPress utilizando Docker e serviços AWS. Para mais detalhes, consulte a [documentação oficial do Docker](https://docs.docker.com/), [WordPress](https://wordpress.org/support/), e [AWS](https://aws.amazon.com/documentation/).

---
