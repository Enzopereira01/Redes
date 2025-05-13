#  Como Realizar um load Balance simples com Docker

1. **Instalar o Docker e Docker Compose**:
```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

# Adiciona chave GPG oficial do Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Adiciona repositório Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

# Instala Docker e Docker Compose
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verifica se Docker está funcionando
sudo docker --version
sudo docker compose version
```

2. Estrutura do Projeto
```bash
# Crie uma pasta para seu projeto:
mkdir ~/docker-loadbalance && cd ~/docker-loadbalance

# Crie uma pasta web para cada instancia como web1, web2 e web3 dentro da pasta docker-loadbalance:
mkdir -p web1 web2 web3

# Crie um arquivo index.html em cada pasta web. Exemplo na pasta web1:
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Container 1</title>
</head>
<body>
  <h1>Sou o container 1</h1>
</body>
</html>


# Crie um arquivo Dockerfile em cada pasta web sem nenhuma alteração

FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html

```

3. docker-compose.yml com Load Balancer (Nginx)
```bash
# Crie um arquivo docker-compose.yml na pasta docker-loadbalance
version: '3'

services:
  web1:
    build: ./web1
    container_name: web1
    expose:
      - "80"

  web2:
    build: ./web2
    container_name: web2
    expose:
      - "80"

  web3:
    build: ./web3
    container_name: web3
    expose:
      - "80"

  nginx:
    image: nginx:alpine
    container_name: nginx-loadbalancer
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - web1
      - web2
      - web3

# Crie um nginx.conf (arquivo no mesmo diretório do docker-compose.yml):
events {}

http {
    upstream backend {
        server web1:80;
        server web2:80;
        server web3:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
```
4. Rodar tudo
```bash
sudo docker compose up -d
```
(Opicional) - Caso queira parar o docker compose utilize:
```bash
sudo docker compose down
```

Agora é só entrar no site com o seu IPv4 Publico da sua instância.
