
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
├── docker-compose.yml
└── README.md
```

---

## ⚙️ Pré-requisitos

- Instância EC2 Ubuntu configurada com:
  - Docker
  - Docker Compose
- Banco de dados MySQL configurado na AWS RDS

---

## 🧱 Configurações da RDS

- **Endpoint:** `database-2.c29urnittqn0.us-east-1.rds.amazonaws.com`
- **Usuário:** `admin`
- **Senha:** `fatec1234`
- **Banco:** `meubanco`

### ⚠️ Permissões da RDS

Garanta que sua RDS esteja acessível a partir do IP público da instância EC2 (verifique o grupo de segurança).

---

## 🚀 Como subir a aplicação

1. Clone o repositório:

```bash
git clone https://github.com/seu-usuario/seu-repo.git
cd seu-repo
```

2. Suba os containers:

```bash
sudo docker-compose up -d --build
```

3. Acesse no navegador:

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

## 📦 Exemplo de Build manual

Se desejar buildar e testar o container separadamente:

```bash
docker build -t flask-app .
docker run -p 5000:5000 flask-app
```

---

## 🛠️ Tecnologias utilizadas

- Python 3.10
- Flask
- Docker / Docker Compose
- NGINX
- MySQL (AWS RDS)

---

## 📄 Licença

Este projeto está licenciado sob a [MIT License](LICENSE).

---

