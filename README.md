# Configuração do Ambiente no EC2

Este guia explica como configurar o ambiente de produção para a aplicação Flask em uma instância EC2 da AWS.

## 1. Conectar ao EC2

```bash
ssh -i "sua-chave.pem" ec2-user@seu-ip-ec2
```

## 2. Atualizar o Sistema

```bash
sudo yum update -y
```

## 3. Instalar Python e Dependências

```bash
# Instalar Python 3.11
sudo yum install -y python3.11
sudo alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1
sudo alternatives --set python3 /usr/bin/python3.11

# Instalar pip
curl -O https://bootstrap.pypa.io/get-pip.py
python3 get-pip.py --user

# Instalar virtualenv
pip3 install --user virtualenv
```

## 4. Configurar o Projeto

```bash
# Criar diretório do projeto
mkdir -p /home/ec2-user/sompo_app
cd /home/ec2-user/sompo_app

# Clonar o repositório (se estiver usando git)
git clone seu-repositorio.git .

# Criar ambiente virtual
python3 -m venv venv
source venv/bin/activate

# Instalar dependências
pip install -r requirements.txt
```

## 5. Configurar Gunicorn

```bash
# Instalar Gunicorn (já está no requirements.txt)
pip install gunicorn

# Testar Gunicorn
gunicorn --bind 0.0.0.0:8000 main:app
gunicorn -w 4 -b 0.0.0.0:8000 main:app
```

## 6. Configurar Supervisor (gerenciador de processos)

```bash
# Instalar Supervisor
sudo yum install -y supervisor
sudo service supervisord start
sudo chkconfig supervisord on

# Criar arquivo de configuração
sudo nano /etc/supervisord.d/sompo_app.ini
```

Conteúdo do arquivo `sompo_app.ini`:

```ini
[program:sompo_app]
directory=/home/ec2-user/sompo_app
command=/home/ec2-user/sompo_app/venv/bin/gunicorn --workers 3 --bind 0.0.0.0:8000 main:app
autostart=true
autorestart=true
stderr_logfile=/var/log/sompo_app/sompo_app.err.log
stdout_logfile=/var/log/sompo_app/sompo_app.out.log
user=ec2-user

[supervisord]
environment=
    FLASK_ENV="production",
    FLASK_APP="main.py"
```

```bash
# Criar diretório para logs
sudo mkdir -p /var/log/sompo_app
sudo chown -R ec2-user:ec2-user /var/log/sompo_app

# Recarregar supervisor
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start sompo_app
```

## 7. Configurar Nginx como Proxy Reverso

```bash
# Instalar Nginx
sudo amazon-linux-extras install nginx1

# Configurar Nginx
sudo nano /etc/nginx/conf.d/sompo_app.conf
```

Conteúdo do arquivo `sompo_app.conf`:

```nginx
server {
    listen 80;
    server_name _;  # Substitua pelo seu domínio se houver

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

```bash
# Testar configuração do Nginx
sudo nginx -t

# Iniciar e habilitar Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

## 8. Configurar o Firewall (Security Group)

Na AWS Console:
1. Abrir porta 80 (HTTP)
2. Abrir porta 443 (HTTPS) se usar SSL
3. Manter porta 22 (SSH) aberta apenas para IPs confiáveis

## 9. Verificar Logs

```bash
# Logs do Supervisor
sudo tail -f /var/log/sompo_app/sompo_app.out.log
sudo tail -f /var/log/sompo_app/sompo_app.err.log

# Logs do Nginx
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

## 10. Comandos Úteis

```bash
# Reiniciar a aplicação
sudo supervisorctl restart sompo_app

# Ver status da aplicação
sudo supervisorctl status

# Reiniciar Nginx
sudo systemctl restart nginx

# Ver logs em tempo real
sudo tail -f /var/log/sompo_app/sompo_app.out.log
```

## 11. SSL/HTTPS (Opcional)

Para configurar HTTPS com Certbot:

```bash
# Instalar Certbot
sudo yum install -y certbot python3-certbot-nginx

# Obter certificado e configurar Nginx
sudo certbot --nginx -d seu-dominio.com
```

## 12. Monitoramento

Recomendado configurar:
- AWS CloudWatch para métricas do EC2
- AWS CloudWatch Logs para centralizar logs
- Configurar alarmes para CPU, memória e disco

## Troubleshooting

1. Se a aplicação não iniciar:
   - Verificar logs: `sudo tail -f /var/log/sompo_app/sompo_app.err.log`
   - Verificar permissões: `ls -la /home/ec2-user/sompo_app`
   - Verificar variáveis de ambiente

2. Se Nginx retornar 502:
   - Verificar se Gunicorn está rodando: `ps aux | grep gunicorn`
   - Verificar logs do Nginx: `sudo tail -f /var/log/nginx/error.log`

3. Problemas de permissão:
   - Verificar owner dos arquivos: `ls -la /home/ec2-user/sompo_app`
   - Ajustar se necessário: `sudo chown -R ec2-user:ec2-user /home/ec2-user/sompo_app`


Conexão ao EC2
Atualização do sistema
Instalação do Python e dependências
Configuração do projeto
Configuração do Gunicorn
Configuração do Supervisor
Configuração do Nginx como proxy reverso
Configuração do Firewall (Security Groups)
Monitoramento de logs
Comandos úteis
Configuração opcional de SSL/HTTPS
Monitoramento com CloudWatch
Seção de troubleshooting
A configuração usa as melhores práticas:

Gunicorn como servidor WSGI
Supervisor para gerenciar processos
Nginx como proxy reverso
Logs centralizados
Sistema de processos resiliente
Opção para SSL/HTTPS
Siga as instruções em ordem para ter um ambiente de produção robusto e seguro. Lembre-se de ajustar as configurações conforme suas necessidades específicas, especialmente:

Domínios
Portas
Variáveis de ambiente
Security Groups
Configurações de SSL se necessário