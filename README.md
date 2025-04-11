# Redes


#  Imagem da Topologia 
![Topologia](/image/topologia.jpg?raw=true)


#  Como Realizar um load Balance simples com Docker

1. **Atualizar pacotes**:
```bash
sudo apt update && sudo apt upgrade -y
```

2. **Instalar dependências**:
```bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

3. **Adicionar a chave GPG oficial do Docker**:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

4. **Adicionar repositório Docker**:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```

5. **Instalar Docker**:
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

6. **Habilitar e iniciar o Docker**:
```bash
sudo systemctl enable docker
sudo systemctl start docker
```


7. **Criar um diretório com o conteúdo HTML que será servido**:
```bash
mkdir ~/meu-site
cd ~/meu-site
```

8. **Criar um Html Simples dentro do meu-site**:
```bash
echo "<h1>Bem-vindo ao Container!</h1>" > index.html
```

9. **Posteriomente crie um Dockerfile com**:
```bash
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

10. **Para contruir a imagem**:
```bash
docker build -t meu-site .
```

11. **Para iniciar os containers**:
```bash
docker run -d --name site1 meu-site
docker run -d --name site2 meu-site
docker run -d --name site3 meu-site
```

Casso já tenho os mesmos criados utilize 
```bash
docker start <nome_ou_id_do_container ou os três primeiro digitos do ID>
```

12. **Pegar o IP dos containers  para utilizar no Load Balancer**:
```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' site1
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' site2
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' site3
```

13. **Subir container com NGINX como Load Balancer**:
realize um cd - ( para sair de do meu-site ou o nome de foi dado anteriormente)
```bash
mkdir ~/nginx-lb
cd ~/nginx-lb
```

14. **Crie o nginx.conf dentro do nginx-lb**:

events {}
```bash
http {
    upstream meusites {
        server 172.17.0.2;  # IP do site1
        server 172.17.0.3;  # IP do site2
        server 172.17.0.4;  # IP do site3
    }

    server {
        listen 80;

        location / {
            proxy_pass http://meusites;
        }
    }
}
```

15.  **Crie o container com esse NGINX**:
```bash
docker run -d --name loadbalancer \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  -p 80:80 \
  nginx
```

 **Test**
agora é só pegar o Ip plublico para entrar no site que foi criado

**Opicional**

Casso queria alterar o html dos containers para diferencia-los pode utilizar dos seguintes comando casso os containers estejam rodando ainda.

 1. **comando exemplo**:
```bash
docker exec -it site1 sh                 - comando para entrar no container  
cd /usr/share/nginx/html                 - comando para acessar diretório que está o html
echo '<h1>Bem-vindo ao Container 1 !</h1>' > index.html - a alteração do html
exit                                     - comando para sair do container
```


e repetir esse processo para os outros containers.