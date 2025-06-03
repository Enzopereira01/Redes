# VPN utilizando a AWS e o OpenVPN

Este projeto documenta a configuração de uma VPN utilizando o **OpenVPN** em uma instância **EC2 na AWS**, permitindo acesso seguro a serviços HTTP internos por meio de um túnel VPN.

---

## 📌 Objetivo

Permitir que clientes se conectem à rede privada de uma instância EC2 via VPN, com acesso ao servidor web hospedado na mesma.

---

## ⚙️ Configuração do Servidor

### ✅ Sistema Operacional
- Ubuntu Server

### ✅ Software instalado
- OpenVPN
- Easy-RSA

### 📁 Arquivo `server.conf`

```conf
dev tun
server 192.168.0.0 255.255.255.0
ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key
dh /etc/openvpn/easy-rsa/pki/dh.pem
port 5001
proto udp
verb 4
keepalive 10 120
persist-tun
float
data-ciphers AES-256-CBC
auth SHA256
push "route X.X.X.X 255.255.0.0" <para forçar a ir no ip privato da sua intancia ec2>
```

---

## 👤 Configuração do Cliente

### 📄 Arquivo `client.ovpn`

```conf
client
dev tun
proto udp
remote <IP-PÚBLICO-EC2> 5001
<casso o seu cliente seja winwdows e esteja usando o OpenVPN, considere utilizar o direcionamento para localizar ca, cert e key para não gerar qualquer problema>
ca ca.crt
cert client1.crt
key client1.key
remote-cert-tls server
keepalive 10 120
persist-tun
float
data-ciphers AES-256-CBC
data-ciphers-fallback AES-256-CBC
auth SHA256
verb 4
nobind
disable-dco
```

> Substitua `<IP-PÚBLICO-EC2>` pelo IP atual da instância EC2. Se não for fixo, será necessário atualizar o arquivo `.ovpn` após cada reinicialização da instância.

---

## 🔐 Security Groups na AWS

Configure o **Security Group** da EC2 com as seguintes regras de entrada:

| Tipo     | Protocolo | Porta | Origem         |
|----------|-----------|-------|----------------|
| HTTP     | TCP       | 80    | 192.168.0.0/24 |
| HTTPS    | TCP       | 443   | 192.168.0.0/24 |
| OpenVPN  | UDP       | 5001  | 0.0.0.0/0      |

---

## 🧪 Teste de Conexão

Após conectar à VPN:

1. Verifique o IP local atribuído (deverá ser algo como `192.168.0.x`).
2. Acesse o serviço HTTP da EC2 com:
   ```
   http://x.x.x.x
   ```
   Substitua pelo IP privado da EC2.

---

## 🔄 Comandos úteis no servidor

```bash
sudo systemctl start openvpn-server@server
sudo systemctl stop openvpn-server@server
sudo systemctl restart openvpn-server@server
sudo systemctl status openvpn-server@server
```

---

## 📝 Observações

- O IP privado da EC2 é necessário para acessar serviços HTTP por meio da VPN.
- Para evitar edições manuais frequentes no `.ovpn`, considere usar **Elastic IP** na AWS.
- Verifique sempre o `iptables` e a configuração de IP forwarding se houver problemas de roteamento.
- O projeto web que foi realizado anteriormente pode ser integrado na mesma instancia que está o sevidor da vpn

---


