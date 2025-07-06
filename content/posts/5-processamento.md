---
title: "5 Processamento"
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


## Resultados




