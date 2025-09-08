# HConectar: Conectar EC2 a RDS PostgreSQL
> Este tutorial mostra como conectar uma instância EC2 Ubuntu a um banco de dados PostgreSQL hospedado no Amazon RDS.  
## 1. Acesse o Console AWS
- Entre em https://aws.amazon.com/ e faça login na sua conta.
## 2. Acesse o serviço EC2
- No menu "Services", procure por **EC2** e clique para abrir o painel.
## 3. Acesse o serviço RDS
- No menu "Services", procure por **RDS** e clique para abrir o painel. 
## 4. Obtenha o endpoint do RDS
- No painel do RDS, selecione sua instância PostgreSQL.
- Na aba **Connectivity & security**, copie o **Endpoint** (ex: `mydbinstance.123456789012.us-east-1.rds.amazonaws.com`).       
- Anote também a **Port** (geralmente `5432`).
## 5. Configure o Security Group do RDS
- No painel do RDS, clique em **DB Security Groups**.
- Selecione o grupo de segurança associado à sua instância RDS.
- Clique em **Edit inbound rules**.
- Adicione uma nova regra:
  - **Type**: PostgreSQL
  - **Source**: Custom e insira o IP público da sua instância EC2 (ou um intervalo de IPs, se necessário).
- Clique em **Save rules**.
## 6. Acesse sua instância EC2 via SSH
- No painel do EC2, selecione sua instância e clique em **Connect**.
- Siga as instruções para se conectar via SSH, usando o arquivo `.pem` que você baixou ao criar a instância. Exemplo de comando:

```bash
ssh -i /path/to/your-key.pem ubuntu@<EC2_PUBLIC_IP>
``` 
> Dica: No Windows, use o Git Bash ou WSL para rodar o comando SSH.
## 7. Instale o cliente PostgreSQL na EC2   
Após conectar, atualize o sistema e instale o cliente PostgreSQL:

```bash
sudo apt update
sudo apt install postgresql-client -y
```
## 8. Conecte ao banco de dados RDS PostgreSQL
Use o comando `psql` para conectar ao banco de dados RDS, substituindo `<RDS_ENDPOINT>`, `<PORT>`, `<DB_NAME>`, `<USERNAME>` e `<PASSWORD>` pelos valores corretos:

```bash
psql -h <RDS_ENDPOINT> -p <PORT> -d <DB_NAME> -U <USERNAME>
``` 
Exemplo:

```bash
psql -h mydbinstance.123456789012.us-east-1.rds.amazonaws.com -p 5432 -d mydatabase -U myuser
``` 
- Quando solicitado, insira a senha do usuário do banco de dados.
## 9. Teste a conexão   
Se a conexão for bem-sucedida, você verá o prompt do PostgreSQL. Você pode executar comandos SQL para testar:

```sql
SELECT version();
``` 
- Para sair do psql, digite `\q` e pressione Enter.
Pronto! Sua instância EC2 está conectada ao banco de dados PostgreSQL hospedado no Amazon RDS. Agora você pode desenvolver e implantar suas aplicações que interagem com o banco de dados.