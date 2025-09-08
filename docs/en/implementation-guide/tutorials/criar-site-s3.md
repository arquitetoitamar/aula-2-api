# omo criar um site estático no S3

> Este tutorial mostra como criar um site estático usando um bucket S3 na AWS.

## 1. Acesse o Console AWS
- Entre em https://aws.amazon.com/ e faça login na sua conta.

## 2. Acesse o serviço S3
- No menu "Services", procure por **S3** e clique para abrir o painel.

## 3. Crie um bucket S3
- Siga as instruções do tutorial anterior para criar um bucket S3.

## 4. Carregue os arquivos do seu site
- Selecione seu bucket na lista e clique em **Upload**.
- Arraste e solte os arquivos do seu site (HTML, CSS, JS, imagens, etc.) ou clique em **Add files** para selecionar arquivos do seu computador.

## 5. Configure o bucket para hospedagem de site
- Com o bucket selecionado, vá para a aba **Properties**.
- Encontre a seção **Static website hosting** e clique em **Edit**.
- Selecione **Enable** e preencha os campos:
  - **Index document**: `index.html`
  - **Error document**: `error.html` (ou outro arquivo de erro, se necessário)
- Clique em **Save changes**.

## 6. Defina permissões de acesso
- Vá para a aba **Permissions** do seu bucket.
- Clique em **Bucket Policy** e adicione a seguinte política para permitir acesso público:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::meu-bucket-teste/*"
    }
  ]
}
```

- Substitua `meu-bucket-teste` pelo nome do seu bucket.
- Clique em **Save**.

## 7. Acesse seu site
- Após configurar o bucket, você pode acessar seu site estático pelo endpoint fornecido na seção **Static website hosting**.

Pronto! Seu site estático está hospedado no S3 e pronto para uso.
