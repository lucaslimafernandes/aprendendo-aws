---
title: "[TUTORIAL] Usando Lambda com Localstack"
date: 2025-07-06T09:28:13-03:00
draft: false
---

Bom, para prossegui com o projeto, devo realizar o processamento dos vídeos, isso incluíra:

1. Pré-processamento
2. Extração de características
3. Detecção de faixas
4. Rastreamento de faixas

Para isso, pensei em usar Lambda para o processamento dos vídeos.

O LocalStack possui algumas limitações quanto a isso, por exemplo, não consegui criar uma Lambda com LocalStack usando uma imagem de container. Precisarei fazer com o arquivo zipado.

Vou mostrar o passo a passo, com duas formas, uma usando a interface da LocalStack e a outra via AWS-CLI com awslocal.

## Passo 1 - Criando um scrip python

Vamos salvar um arquivo chamado `app.py`
```python
def lambda_handler(event, context):

    return {
        'statusCode': 200,
        'body': f"test ok"
    }

```

Vamos zipar ele, usei o terminal para isso:

```bash
zip app.zip app.py
```

## Passo 2 - Entender alguns itens do Lambda

### Function name

Function name é o nome da função Lambda, aqui usei test1 como nome

Vamos usar ele sempre que quisermos:

invocar a função, ver logs, atualizar código, configurar eventos, etc.

### Runtime

Runtime indica qual versão da linguagem será usada para rodar o código.

Usei aqui o python3.11 (poderia ser o 3.10, 3.12 ...)

Outros exemplos de runtimes disponíveis:
- nodejs
- java
- go (Golang é uma ótima linguagem, fica a dica)

### Role

Esse é o ARN da role IAM (permissões) que a Lambda usaria na AWS real.
No LocalStack, não é validado de verdade, mas precisa seguir o formato certo.

Usamo aqui: arn:aws:iam::000000000000:role/lambda-role

Na AWS, essa role define o que a Lambda pode acessar, como:

- buckets S3
- bancos de dados
- logs do CloudWatch

Formato obrigatório:

`arn:aws:iam::<ID>:role/<nome-da-role>`

### Handler: 

Esse campo diz para o AWS Lambda qual função chamar dentro do seu código.

No teste usado foi `app.lambda_handler`, sendo 
- app: o nome do arquivo.
- lambda_handler: a função dentro do arquivo que quero executar.

Formato geral: `<nome-do-arquivo>.<nome-da-função>`


## Passo 3a - Criar o Lambda - usando interface

Vamos acessar o painel da LocalStack: https://app.localstack.cloud/inst/default/resources, já tinha falado dele no post [Criando a primeira API para upload de vídeos - S3 e SQS](https://lucaslimafernandes.github.io/aprendendo-aws/posts/3-criando-uma-api-upload-s3-sqs/)

E buscar pelo recurso "lambda"

![Buscar o lambda](https://lucaslimafernandes.github.io/aprendendo-aws/images/4/image.png)

Ao acessar, clicar em criar

![Criar nova](https://lucaslimafernandes.github.io/aprendendo-aws/images/4/image-1.png)

E preencher as insformações:

![Preencher dados](https://lucaslimafernandes.github.io/aprendendo-aws/images/4/image-2.png)

- Function name: test1
- Runtime: python3.11
- Role: arn:aws:iam::000000000000:role/lambda-role
- Handler: app.lambda_handler
- Upload zip file

e pronto, só salvar

Será exibido os dados da Lambda

![Lambda](https://lucaslimafernandes.github.io/aprendendo-aws/images/4/image-3.png)

E via painel, podemos invocar a chamada dela

![Invoke](https://lucaslimafernandes.github.io/aprendendo-aws/images/4/image-4.png)

Segue o resultado, deu tudo certo!!!

![Teste](https://lucaslimafernandes.github.io/aprendendo-aws/images/4/image-5.png)

## Passo 3b - Criar o Lambda - usando AWS-CLI

No terminal, no diretório onde salvamos o arquivo app.zip

```bash
awslocal lambda create-function \
  --function-name test1 \
  --runtime python3.11 \
  --handler app.lambda_handler \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --zip-file fileb://app.zip
```

Teremos o seguinte output

```plaintext
# output
{
    "FunctionName": "test1",
    "FunctionArn": "arn:aws:lambda:us-east-1:000000000000:function:test1",
    "Runtime": "python3.11",
    "Role": "arn:aws:iam::000000000000:role/lambda-role",
    "Handler": "app.lambda_handler",
    "CodeSize": 302,
    "Description": "",
    "Timeout": 3,
    "MemorySize": 128,
    "LastModified": "2025-07-06T09:38:23-03:00",
    "CodeSha256": "a6W95W7SVJfZwMi1N4uipIaTroHwDsOwp3NMZlXyMMM=",
    "Version": "$LATEST",
    "TracingConfig": {
        "Mode": "PassThrough"
    },
    "RevisionId": "3a016d90-dbbe-4c0c-a406-1ef34d25d42f",
    "State": "Pending",
    "StateReason": "The function is being created.",
    "StateReasonCode": "Creating",
    "PackageType": "Zip",
    "Architectures": [
        "x86_64"
    ],
    "EphemeralStorage": {
        "Size": 512
    },
    "SnapStart": {
        "ApplyOn": "None",
        "OptimizationStatus": "Off"
    },
    "RuntimeVersionConfig": {
        "RuntimeVersionArn": "arn:aws:lambda:us-east-1::runtime:8eeff65f6809a3ce81507fe733fe09b835899b99481ba22fd75b5a7338290ec1"
    },
    "LoggingConfig": {
        "LogFormat": "Text",
        "LogGroup": "/aws/lambda/test1"
    }
}
```

Para invocar via AWS-CLI

```bash
awslocal lambda invoke \
  --function-name test1 \
  /dev/stdout
```

```plaitext
# output
{"statusCode": 200, "body": "test ok"}{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
```

`/dev/stdout` faz a impressão no terminal.

Para salvar em um arquivo:

```bash
awslocal lambda invoke \
  --function-name test1 \
  /dev/stdout >> arquivo.txt
```
