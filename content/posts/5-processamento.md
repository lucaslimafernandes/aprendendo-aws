---
title: "Realizando o processamento de vídeo"
date: 2025-07-09T22:01:42-03:00
draft: true
---

Lambda com LocalStack não deu bom, pelo menos por aqui 😅

Na versão community ele não permite criar lambdas a partir de um container.

Seguindo o que já construí até [aqui](https://lucaslimafernandes.github.io/aprendendo-aws/posts/3-criando-uma-api-upload-s3-sqs/)

Agora o [repositório](https://github.com/lucaslimafernandes/aws-lane-detection) possui a seguinte estrutura:

```plaintext
.
├── docker-compose.yaml
├── LICENSE
├── README.md
└── services
    ├── api
    │   ├── aws_utils.py
    │   ├── Dockerfile
    │   ├── docs
    │   │   └── algumas imagens aqui
    │   ├── example.env
    │   ├── main.py
    │   ├── README.md
    │   ├── requirements.txt
    │   └── settings.py
    └── video_processor
        ├── Dockerfile
        ├── example.env
        ├── main.py
        └── requirements.txt
```

Onde foi adicionado a pasta `video_processor` 

## Processamento

Aqui está o novo trecho do projeto. Ele consome mensagens da fila SQS, faz o download de vídeos do S3, realiza o processamento com OpenCV para detecção de faixas e envia os resultados processados de volta ao S3.

Sendo assim:

1. O serviço entra em loop contínuo (daemon-like).
2. A cada 10 segundos, verifica se há novas mensagens na fila `future-processing`.
3. Quando há mensagem:
   - Faz download do vídeo do bucket `lane-bucket`.
   - Processa os frames aplicando o algoritmo de detecção de faixas.
   - Salva os frames e resultados processados no S3, em uma pasta única para cada vídeo.

Para criar a conexão com LocalStack/AWS foi utilizada a biblioteca boto3

```python3
ENDPOINT_URL = os.getenv("AWS_ENDPOINT_URL", None)
REGION_NAME  = os.getenv("AWS_REGION", "ue-east-1")

sqs = boto3.client("sqs", endpoint_url=ENDPOINT_URL, region_name=REGION_NAME)
s3  = boto3.client("s3", endpoint_url=ENDPOINT_URL, region_name=REGION_NAME)
```

## Executando

Bom, vamos iniciar os serviços com o Docker Compose

Seguindo o passo anterior, vamos garantir que o container localstack já exista

```bash
# Criar a rede
docker network create localstack-net

# Iniciar o container
docker run -d \
  --name localstack-main \
  --network localstack-net \
  -p 127.0.0.1:4566:4566 -p 127.0.0.1:4510-4559:4510-4559 \
  -v /var/run/docker.sock:/var/run/docker.sock localstack/localstack \
  localstack/localstack
```

Caso já exista o container, basta iniciá-lo 

```bash
docker start localstack-main
```

Agora, vamos garantir que o bucket e a fila existam, pode-se recorrer ao [post 3](https://lucaslimafernandes.github.io/aprendendo-aws/posts/3-criando-uma-api-upload-s3-sqs/), mas vou deixar os comandos CLI aqui, pois vou utilazos também:

```bash
# Bucket S3
awslocal s3 mb s3://lane-bucket

# Fila SQS
awslocal sqs create-queue --queue-name future-processing
```


Então, dentro da pasta do repositório vamos iniciar o compose

```bash
docker compose up --build
```

E então teremos o output:

```plaintext
 ✔ api                         Built                                       0.0s 
 ✔ processor                   Built                                       0.0s 
 ✔ Container processor         Recreated                                   0.1s 
 ✔ Container api-ec2-simulate  Recreated                                   0.1s 
Attaching to api-ec2-simulate, processor
api-ec2-simulate  | INFO:     Started server process [1]
api-ec2-simulate  | INFO:     Waiting for application startup.
api-ec2-simulate  | INFO:     Application startup complete.
api-ec2-simulate  | INFO:     Uvicorn running on http://0.0.0.0:8080 (Press CTRL+C to quit)
```


Abrindo a url `localhost:8080` veremos a mensagem: `"status": "healthy"`

Sugiro seguir para `localhost:8080/docs`, vamos usar o Swagger UI

Vamos seguir para o endpoint `upload` e clicar em `Try it out`

![Try it ou upload](https://lucaslimafernandes.github.io/aprendendo-aws/images/5/image.png)

Selecione o arquivo e clique em execute:

![upload](https://lucaslimafernandes.github.io/aprendendo-aws/images/5/image-1.png)

Receberemos abaixo a resposta:

```plaintext
{
  "message": "mov_0288_b.mp4 uploaded successfully.",
  "s3_url": "http://localstack-main:4566/lane-bucket/mov_0288_b.mp4"
}
```

Caso queira fazer via `curl`

```bash
curl -X 'POST' \
  'http://localhost:8080/upload/' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'file=@mov_0288_b.mp4;type=video/mp4'
```


## Resultados


Podemos acessar o Bucket S3 via [LocalStack](https://app.localstack.cloud/inst/default/resources)

E conferir o vídeo original e a pasta resultados, onde foram salvos os resultados dos processamentos

![bucket S3](https://lucaslimafernandes.github.io/aprendendo-aws/images/5/image-2.png)


Arquivo final salvo

![Arquivo final](https://lucaslimafernandes.github.io/aprendendo-aws/images/5/image-3.png)

Detalhe: Só salvou um frame em cada pasta (Bug que deverei corrigir lá no repositório, mas não agora, por enquanto o teste foi muito bom!)


Vamos ver a fila SQS

1. Ao recém realizar o upload do vídeo

![SQS 1](https://lucaslimafernandes.github.io/aprendendo-aws/images/5/image-4.png)

2. Após o processamento

![SQS 2](https://lucaslimafernandes.github.io/aprendendo-aws/images/5/image-5.png)


Obs.: Tive de relizar 3x o upload, para conseguir pegar o delay do processamento 😅

Por hoje fico aqui. No próximo post, irei verificar a correção se salvar apenas uma imagem em cada pasta, irei ver sobre a notificação de conclusão e salvar em uma tabela do DynamoDB.

