---
title: "Notificando finalização via Telegram (c/ SNS)"
date: 2025-07-11T23:11:32-03:00
draft: true
---

Mais um passo criado, desta vez com os seguintes itens:
- Correção bug de processamento
- Uso do AWS SNS (LocalStack)
- Notificações no Telegram

Vamos ao [repositório](https://github.com/lucaslimafernandes/aws-lane-detection)

Sua estrutura agora está assim:

```plaintext
.
├── create_services.sh
├── docker-compose.yaml
├── LICENSE
├── Passos.txt
├── README.md
├── services
│   ├── api
│   │   └── ... (envio de vídeos para processamento)
│   ├── notifier
│   │   └── ... (notificação via Telegram)
│   └── video_processor
│       └── ... (processamento de vídeo com OpenCV)
└── testes
    └── vídeos de exemplo
```

## Correção do Bug

Vamos pelo mais fácil primeiro:

no arquivo `services/video_processor/main.py`

```python
def lambda_handler(event, context):
    ...

    for record in event.get("Records", []):
        msg = json.loads(record["body"])

        frame_count = 0
        ...
        while True:
            ...

            # Aqui foi o que foi feito
            frame_count += 1
```

Sim, estava sobreescrevendo os frames, pois eu esqueci de adicionar +1 a cada loop.


## Notificações com SNS + Telegram

Foi adicionado o serviço notifier, responsável por:

1. Criar uma assinatura HTTP com o tópico SNS alertas

2. Confirmar a inscrição automaticamente

3. Enviar o conteúdo da mensagem para um bot do Telegram


O notifier foi construído usando `FastAPI`, e atua como um `listener HTTP` que envia mensagens ao Telegram assim que o SNS publica notificações.


### O que foi adicionado ao video_processor

Para enviar as notificações ao SNS alertas, foi bem simples:

```python
ENDPOINT_URL = os.getenv("AWS_ENDPOINT_URL", None)
REGION_NAME  = os.getenv("AWS_REGION", "us-east-1")
TOPIC_ARN = os.getenv("SNS_TOPIC_ARN", "arn:aws:sns:us-east-1:000000000000:alertas")

sns = boto3.client("sns", endpoint_url=ENDPOINT_URL, region_name=REGION_NAME)

...

msg = f"✅ Mensagem aqui"
response = sns.publish(
    TopicArn=TOPIC_ARN,
    Message=msg,
)
```

### Telegram

Para configurar o telegram, preciso de 2 itens:

- TELEGRAM_TOKEN: Token do Bot do Telegram
- TELEGRAM_CHAT_ID: Id da conversa para onde será enviada a mensagem

1. Primeiro, necessário conversar com o [BotFather](https://t.me/BotFather)

Ao iniciar a conversa com `/start`, criar o seu bot com `/newbot`

Será necessário dar um novo e username, e será retornado o seu token:

```plaintext
Done! Congratulations on your new bot. You will find it at t.me/YourBotUsername. You can now add a description, about section and profile picture for your bot, see /help for a list of commands. By the way, when you've finished creating your cool bot, ping our Bot Support if you want a better username for it. Just make sure the bot is fully operational before you do this.

Use this token to access the HTTP API:
XXXXXX:YYYYYYYYYYYY
Keep your token secure and store it safely, it can be used by anyone to control your bot.

For a description of the Bot API, see this page: https://core.telegram.org/bots/api
```

2. Ter uma conversa para o Bot mandar mensagem

Aqui eu criei um grupo, e via navegador web, pela url, eu peguei o ID, exemplo abaixo

```plaintext
https://web.telegram.org/k/#-1234567890

ID: "-1234567890"
```

Ou então, usar o [userinfobot](https://t.me/userinfobot)

Ele retornará o seu ID, que poderá ser utilizado.


### Executando tudo isso

Bom, conforme os posts anteriores, assumindo que o ambiente LocalStack já está em execução

```bash
docker start localstack-main

# ou então
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

Criar o bucket S3 e a fila SQS

```bash
# Bucket S3
awslocal s3 mb s3://lane-bucket

# Fila SQS
awslocal sqs create-queue --queue-name future-processing
```

Nos posts anteriores, foi mostrado como fazer via interface web

Iniciares os serviços (api, video_processor e notifier) via docker compose

```bash
docker compose up --build
```

Agora vamos a novidade:

### Criar um tópico SNS

Bom, eu fiz via CLI

#### Via CLI - aws-cli - awslocal


1. Criei o tópico

```bash
awslocal sns create-topic --name alertas
```

Irá retornar o `TopicArn`
```json
{
    "TopicArn": "arn:aws:sns:us-east-1:000000000000:alertas"
}
```


2. Inscrição no tópico

```bash
awslocal sns subscribe \   
  --topic-arn arn:aws:sns:us-east-1:000000000000:alertas \
  --protocol http \
  --notification-endpoint http://notifier:8081/sns-telegram
```

Onde:
- topic-arn: O tópico criado,
- protocol: O protocolo do serviço
- notificiation-endpoint: Para onde o SNS irá mandar uma mensagem de confirmação de inscrição e as futuras mensagens.


3. Teste manual de envio de mensagem (Publish):

```bash
awslocal sns publish \
  --topic-arn arn:aws:sns:us-east-1:000000000000:alertas \
  --message "✅ Mensagem de teste!"
```

Essa etapa, é feita no `video_processor`, o envio de mensagem.

#### Via interface LocalStack

[Acessar a interface](https://app.localstack.cloud/inst/default/resources)

1. Criar o tópico

Vamos buscar o serviço

![Busca do SNS](https://lucaslimafernandes.github.io/aprendendo-aws/images/6/image.png)


Acessar Create Topic

![SNS Details](https://lucaslimafernandes.github.io/aprendendo-aws/images/6/image-1.png)


Preencher o nome e clicar em submit

![SNS Create](https://lucaslimafernandes.github.io/aprendendo-aws/images/6/image-2.png)


Teremos os detalhes do tópico criado

![SNS Topic](https://lucaslimafernandes.github.io/aprendendo-aws/images/6/image-3.png)

Note, que há a aba de Details e Subscriptions, iremos para ela


2. Inscrição no tópico

Acessar subscriptions

![Subscriptions](https://lucaslimafernandes.github.io/aprendendo-aws/images/6/image-4.png)

Acessar o botão Create

![Criar subscription](https://lucaslimafernandes.github.io/aprendendo-aws/images/6/image-5.png)

Preencher as informações:

![Preencher topico](https://lucaslimafernandes.github.io/aprendendo-aws/images/6/image-6.png)


3. Teste manual de envio de mensagem (Publish):

Dentro do tópico, acessar o botão `Publish Message`

![Publish Message](https://lucaslimafernandes.github.io/aprendendo-aws/images/6/image-7.png)

Então, verificamos o tópico, escrevemos a mensagem e por fim enviamos a mensagem.

![alt text](https://lucaslimafernandes.github.io/aprendendo-aws/images/6/image-8.png)


## Resultados

Resultado final: ao terminar o processamento de vídeo, a notificação é enviada automaticamente para o Telegram!

![Resultados](https://lucaslimafernandes.github.io/aprendendo-aws/images/6/image-9.png)


