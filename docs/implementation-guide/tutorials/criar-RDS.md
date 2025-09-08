# Como criar uma instância RDS PostgreSQL com pgvector

> Este tutorial mostra como criar uma instância RDS PostgreSQL configurada para usar a extensão pgvector, ideal para aplicações de IA que precisam armazenar e consultar vetores.

## 1. Acesse o Console AWS
- Entre em https://aws.amazon.com/ e faça login na sua conta.

## 2. Acesse o serviço RDS
- No menu "Services", procure por **RDS** e clique para abrir o painel.

## 3. Clique em "Create Database"
- Clique no botão **Create database**.

## 4. Configure o banco de dados

### Opções básicas:
- **Database creation method**: Standard create
- **Engine type**: PostgreSQL
- **Version**: PostgreSQL 15.4-R2 ou superior (para suporte ao pgvector)

### Templates:
- **Templates**: Free tier (se elegível) ou Dev/Test

### Configurações:
- **DB instance identifier**: `postgres-pgvector` (ou nome de sua escolha)
- **Master username**: `postgres`
- **Master password**: Crie uma senha segura e anote

### Configuração da instância:
- **DB instance class**: 
  - Free tier: `db.t3.micro`
  - Produção: `db.t3.small` ou superior
- **Storage type**: General Purpose SSD (gp2)
- **Allocated storage**: 20 GB (mínimo)

### Conectividade:
- **VPC**: Default VPC
- **Public access**: Yes (para acesso externo durante desenvolvimento)
- **VPC security group**: Create new
- **Database port**: 5432

## 5. Configurações adicionais

### Opções do banco:
- **Initial database name**: `vectordb`
- **DB parameter group**: default.postgres15 (ou criar um customizado)

### Backup:
- **Backup retention period**: 7 days
- **Backup window**: Escolha um horário conveniente

### Monitoramento:
- **Enable Performance Insights**: Recomendado para produção

## 6. Clique em "Create Database"
- Revise todas as configurações e clique em **Create database**.
- A criação pode levar de 10-20 minutos.

## 7. Configure a extensão pgvector

Após a criação do RDS, conecte-se ao banco e instale a extensão:

```sql
-- Conecte-se ao banco usando psql ou outro cliente
-- psql -h <RDS_ENDPOINT> -U postgres -d vectordb

-- Instale a extensão pgvector
CREATE EXTENSION IF NOT EXISTS vector;

-- Verifique se foi instalada
\dx vector
```

## 8. Teste a funcionalidade

```sql
-- Crie uma tabela de exemplo com vetores
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    filename TEXT,
    content TEXT,
    embedding vector(384)  -- Para embeddings de 384 dimensões
);

-- Insira um documento de teste
INSERT INTO documents (filename, content, embedding) 
VALUES ('teste.txt', 'Este é um documento de teste', '[0.1, 0.2, 0.3]'::vector);

-- Consulte documentos similares
SELECT filename, content, 
       1 - (embedding <=> '[0.1, 0.2, 0.4]'::vector) as similarity
FROM documents 
ORDER BY embedding <=> '[0.1, 0.2, 0.4]'::vector
LIMIT 5;
```

## 9. Configurações de segurança

### Para produção:
- Configure o security group para permitir acesso apenas de IPs específicos
- Use SSL/TLS para conexões
- Configure backup automático
- Monitore performance e logs

### String de conexão:
```python
# Exemplo para Python
import psycopg

conn = psycopg.connect(
    host="<RDS_ENDPOINT>",
    port=5432,
    dbname="vectordb",
    user="postgres",
    password="<SUA_SENHA>",
    sslmode="require"
)
```

Pronto! Seu RDS PostgreSQL está configurado com pgvector para aplicações de IA e busca vetorial.