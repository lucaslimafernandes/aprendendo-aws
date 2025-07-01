---
title: "Criando a primeira API para upload de vídeos - S3 e SQS"
date: 2025-07-05T14:38:07-03:00
draft: false
---

Vamos começar com uma API com FastAPI para realizar upload de vídeos a um bucket S3 e inserir um item na fila para processamento com o SQS.

Primeiramente, tentei realizar com Django, pois conseguiria de forma simples e fácil, um sistema de autenticação e salvar os dados em banco de dados, não consegui, então vamos simplificar e fazer funcionar!

O segundo perrengue foi em decidir como executar a API, tentei usar EC2, mas na documentação da LocalStack diz que não está disponível na versão community, então, vamos usar docker para rodar!

Criei um novo repositório, será um monorepo, com o sistema

[aws-lane-detection](https://github.com/lucaslimafernandes/aws-lane-detection)

Dentro da pasta `services/api`, o código está disponível lá.

Foi criada 3 rotas, mas a rota interessante aqui é a `/upload/`

| Método | Rota       | Descrição                       |
| ------ | ---------- | ------------------------------- |
| GET    | `/`        | Health check                    |
| GET    | `/buckets` | Lista buckets do S3             |
| POST   | `/upload/` | Faz upload de vídeo e envia SQS |


Abaixo, segue como executar, e por fim, o resultado!

Qualquer dúvida, podem entrar em contato =D, ficarei feliz em conversar!

## Tecnologias

- FastAPI
- LocalStack (S3, SQS)
- Docker & Docker Compose
- boto3

## Executando

### 1. Suba o LocalStack manualmente

```bash
docker network create localstack-net

docker run -d \
  --name localstack-main \
  --network localstack-net \
  -p 4566:4566 \
  -e SERVICES=s3,sqs \
  -e DEBUG=1 \
  localstack/localstack
```

### 2. Criar a queue e o bucket

Necessário possuir a cli do AWS instalada

```bash
pip install awscli
pip install awscli-local
```

#### Para cria manualmente:

```bash
awslocal s3 mb s3://lane-bucket
awslocal sqs create-queue --queue-name future-processing
```

#### Ou usando a página da LocalStack: https://app.localstack.cloud/inst/default/resources

Necessário criar conta, possuem opção **free**

1. Para o S3:

- Ao acessar a página, buscar por `S3`

![LocalStack](https://lucaslimafernandes.github.io/aprendendo-aws/images/3/image.png)

- Acessar `create`

![create bucket](https://lucaslimafernandes.github.io/aprendendo-aws/images/3/image1.png)

- Verificar a região, preencher o nome e criar

![Preencher infos bucket](https://lucaslimafernandes.github.io/aprendendo-aws/images/3/image2.png)

- Após criar, nesta página, conseguirá ver os arquivos que forem enviados.

2. Para o SQS:

- Buscar por `SQS`

![Buscar SQS](https://lucaslimafernandes.github.io/aprendendo-aws/images/3/image3.png)

- Clicar em `create`

![create SQS](https://lucaslimafernandes.github.io/aprendendo-aws/images/3/image4.png)

- Verificar a região, preencher o nome e criar

![preencher SQS](https://lucaslimafernandes.github.io/aprendendo-aws/images/3/image5.png)


### 3. Subir a api

Primeiramente, copiar o arquivo `example.env` para `.env` e preencher com os dados adequados.

```bash
docker-compose up --build
```

A API estará disponível em: `http://localhost:8080`

**Sugestão:** Usar `http://localhost:8080/docs` para acessar as routas via Swagger UI

| Método | Rota       | Descrição                       |
| ------ | ---------- | ------------------------------- |
| GET    | `/`        | Health check                    |
| GET    | `/buckets` | Lista buckets do S3             |
| POST   | `/upload/` | Faz upload de vídeo e envia SQS |


## Funcionou !!!

Ao subir o vídeo, eis o retorno em `json`

```json
{
  "message": "cctv052x2004080516x01642.avi uploaded successfully.",
  "s3_url": "http://localstack-main:4566/lane-bucket/cctv052x2004080516x01642.avi"
}
```

![bucket file](image6.png)

E eis a mensagem na queue

```json
{"bucket": "lane-bucket", "filename": "cctv052x2004080516x01642.avi", "url": "http://localstack-main:4566/lane-bucket/cctv052x2004080516x01642.avi", "event": "UPLOAD_VIDEO"}
```

![Message Queue](image7.png)

