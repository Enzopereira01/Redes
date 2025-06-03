
# 🐳 Flask + Docker + NGINX Load Balancer + AWS RDS

Este projeto é uma aplicação web em Flask containerizada com Docker, utilizando o NGINX como balanceador de carga interno entre múltiplos containers Flask, todos acessando um banco de dados MySQL hospedado em uma instância RDS da AWS.

---

## 📌 Funcionalidades

- Cadastro de usuários (nome e email)
- Listagem de usuários
- Exclusão de usuários
- Load balancing entre 3 instâncias Flask com NGINX reverso
- Banco de dados externo (AWS RDS)

---

## 🗂️ Estrutura do Projeto

```
meu-projeto/
├── app/
│   ├── app.py
│   ├── requirements.txt
│   └── templates/
│       ├── form.html
│       ├── usuarios.html
│       └── deletar.html
├── nginx/
│   └── default.conf
├── Dockerfile
└── docker-compose.yml
```

---

## ⚙️ Pré-requisitos

- Instância EC2 Ubuntu configurada com:
  - Docker
  - Docker Compose
- Banco de dados MySQL configurado na AWS RDS

---

### ⚠️ Permissões da RDS

Garanta que sua RDS esteja acessível a partir do IP público da instância EC2 (verifique o grupo de segurança).

---

## 🚀 Como subir a aplicação

1. O que vai possuir em cada arquivo:

 docker-compose.yml :
```bash

version: '3.8'

services:
  app1:
    build: .
    environment:
      - INSTANCE_NAME=app1
      - DB_HOST=<colocar o seu endpoint>
      - DB_USER=<colocar o seu usuário do banco de dados>
      - DB_PASS=<colocar a sua senha do banco de dados>
      - DB_NAME=<>

  app2:
    build: .
    environment:
      - INSTANCE_NAME=app2
      - DB_HOST=<colocar o seu endpoint>
      - DB_USER=<colocar o seu usuário do banco de dados>
      - DB_PASS=<colocar a sua senha do banco de dados>
      - DB_NAME=<o nome do seu banco>

  app3:
    build: .
    environment:
      - INSTANCE_NAME=app3
      - DB_HOST=<colocar o seu endpoint>
      - DB_USER=<colocar o seu usuário do banco de dados>
      - DB_PASS=<colocar a sua senha do banco de dados>
      - DB_NAME=<o nome do seu banco>

  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app1
      - app2
      - app3

```

Dockerfile :
```bash
FROM python:3.10-slim

WORKDIR /app

COPY ./app /app

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 5000

CMD ["python", "app.py"]

```

default.conf :
```bash
upstream flaskapp {
    server app1:5000;
    server app2:5000;
    server app3:5000;
}

server {
    listen 80;

    location / {
        proxy_pass http://flaskapp;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

app.py
```bash
from flask import Flask, request, render_template, redirect, url_for
import mysql.connector
import os

app = Flask(__name__)

db_config = {
    'host': os.environ['DB_HOST'],
    'user': os.environ['DB_USER'],
    'password': os.environ['DB_PASS'],
    'database': os.environ.get('DB_NAME', 'meubanco')
}

instance_name = os.environ.get('INSTANCE_NAME', 'instancia-desconhecida')

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        nome = request.form['nome']
        email = request.form['email']

        conn = mysql.connector.connect(**db_config)
        cursor = conn.cursor()

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS usuarios (
                id INT AUTO_INCREMENT PRIMARY KEY,
                nome VARCHAR(100),
                email VARCHAR(100)
            )
        """)
        cursor.execute("INSERT INTO usuarios (nome, email) VALUES (%s, %s)", (nome, email))
        conn.commit()
        cursor.close()
        conn.close()

        return redirect(url_for('usuarios'))

    return render_template('form.html', instancia=instance_name)

@app.route('/usuarios')
def usuarios():
    conn = mysql.connector.connect(**db_config)
    cursor = conn.cursor(dictionary=True)
    cursor.execute("SELECT id, nome, email FROM usuarios")
    usuarios = cursor.fetchall()
    cursor.close()


    return render_template('usuarios.html', usuarios=usuarios, instancia=instance_name)

@app.route('/deletar', methods=['GET'])
def deletar_usuario_page():
    conn = mysql.connector.connect(**db_config)
    cursor = conn.cursor(dictionary=True)
    cursor.execute("SELECT id, nome, email FROM usuarios")
    usuarios = cursor.fetchall()
    cursor.close()
    conn.close()
    return render_template('deletar.html', usuarios=usuarios, instancia=instance_name)

@app.route('/deletar/<int:id>', methods=['POST'])
def deletar_usuario(id):
    conn = mysql.connector.connect(**db_config)
    cursor = conn.cursor()
    cursor.execute("DELETE FROM usuarios WHERE id = %s", (id,))
    conn.commit()
    cursor.close()
    conn.close()
    return redirect(url_for('usuarios'))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)

```

requirements.txt :
```bash
  Flask
mysql-connector-python
```

deletar.html :
```bash
<!DOCTYPE html>
<html>
<head>
    <title>Excluir Usuários</title>
</head>
<body>
    <nav>
        <a href="/">Cadastro</a> |
        <a href="/usuarios">Lista de Usuários</a> |
        <a href="/deletar">Excluir Usuários</a>
    </nav>
    <h1>Excluir Usuários</h1>
    <p><strong>Instância ativa:</strong> {{ instancia }}</p>

    <table border="1">
        <tr>
            <th>Nome</th>
            <th>Email</th>
            <th>Ação</th>
        </tr>
        {% for u in usuarios %}
        <tr>
            <td>{{ u.nome }}</td>
            <td>{{ u.email }}</td>
            <td>
                <form method="POST" action="/deletar/{{ u.id }}">
                    <input type="submit" value="Excluir" onclick="return confirm('Tem certeza que deseja excluir este usuário?')">
                </form>
            </td>
        </tr>
        {% endfor %}
    </table>
</body>
</html>

```

form.html :
```bash
<!DOCTYPE html>
<html>
<head>
    <title>Cadastro de Usuário</title>
</head>
<body>
    <nav>
        <a href="/">Cadastro</a> |
        <a href="/usuarios">Lista de Usuários</a> |
        <a href="/deletar">Excluir Usuários</a>
    </nav>
    <h1>Cadastro de Usuário</h1>
    <p><strong>Intância ativa:</strong> {{instancia}}</p>
    <form method="POST">
        <label>Nome:</label><br>
        <input type="text" name="nome"><br>
        <label>Email:</label><br>
        <input type="email" name="email"><br><br>
        <input type="submit" value="Cadastrar">
    </form>
</body>
</html>

```

usuarios.html :
```bash
<!DOCTYPE html>
<html>
<head>
    <title>Usuários Cadastrados</title>
</head>
<body>
    <nav>
        <a href="/">Cadastro</a> |
        <a href="/usuarios">Lista de Usuários</a> |
        <a href="/deletar">Excluir Usuários</a>
    </nav>
    <h1>Lista de Usuários</h1>
    <p><strong>Intância ativa:</strong> {{instancia}}</p>
    <table border="1">
        <tr>
            <th>Nome</th>
            <th>Email</th>
        </tr>
        {% for u in usuarios %}
        <tr>
            <td>{{ u.nome }}</td>
            <td>{{ u.email }}</td>
        </tr>
        {% endfor %}
    </table>
</body>
</html>

```

2. O que tem que possuir no banco:
```bash
CREATE DATABASE IF NOT EXISTS meubanco;

USE meubanco;

CREATE TABLE IF NOT EXISTS usuarios (
id INT AUTO_INCREMENT PRIMARY KEY,
nome VARCHAR(100) NOT NULL,
email VARCHAR(100) NOT NULL
);

SELECT * FROM usuarios;
```
obs: o banco que está sendo utilizado no rds é o mysql


3. Suba os containers:

```bash
sudo docker-compose up -d
```

4. Acesse no navegador:

```bash
http://<IP-PUBLICO-DA-EC2>/
```

---

## 🔀 Rotas disponíveis

| Rota            | Função                           |
|-----------------|----------------------------------|
| `/`             | Formulário de cadastro           |
| `/usuarios`     | Listagem dos usuários cadastrados|
| `/deletar`      | Página para deletar usuários     |

---

## 🐳 Containers em execução

- `app1`, `app2`, `app3`: Instâncias Flask (porta 5000 interna)
- `nginx`: Balanceador de carga (porta 80)

---

## 🛠️ Tecnologias utilizadas

- Python 3.10
- Flask
- Docker / Docker Compose
- NGINX
- MySQL (AWS RDS)

---

## Situacional

 Casso ocorra algum problema quando for renicinar o compose com por conta de alguma alteração realize os seguintes comandos:

```bash
sudo docker-compose down --volumes --remove-orphans
sudo docker system prune -a -f
```


