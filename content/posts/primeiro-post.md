---
title: "O que é AWS e o que é LocalStack?"
date: 2025-06-27
draft: false
---

Para continuar minha evolução profissional, percebi que entender a AWS era fundamental. Procurando conteúdos sobre o assunto, descobri o LocalStack, uma forma de simular o ambiente AWS localmente.

## O que é AWS?

A **Amazon Web Services (AWS)** é a maior plataforma de computação em nuvem do mundo. Ela oferece **serviços sob demanda** como:

- Armazenamento de arquivos (S3)
- Execução de funções sem servidor (Lambda)
- Bancos de dados gerenciados (DynamoDB, RDS)
- Criação de APIs (API Gateway)
- Computação (EC2)

A AWS oferece 12 meses de uso gratuitos, mas se posso testar localmente antes... por que não?

## O que é LocalStack?

O **LocalStack** simula os principais serviços da AWS **localmente na sua máquina**, sem custo. Ou seja:

> Você pode desenvolver, testar e até automatizar recursos como Lambda, S3, DynamoDB e API Gateway **sem sair do seu PC**.

Tudo funciona com Docker. Basta instalar o LocalStack e ele vai subir containers com endpoints compatíveis com a AWS. Você usa a mesma AWS CLI, SDKs e ferramentas como se estivesse usando a nuvem real.

### Exemplo de uso:
- Crio uma função Lambda local
- Invoco com a AWS CLI
- Persisto dados simulados no DynamoDB local

No final, quando tudo estiver pronto, posso migrar facilmente para a AWS real.

### Instalação

Para instalar, basta seguir o manual: https://docs.localstack.cloud/getting-started/installation/

Seguindo a instalação do LocalStack CLI, ou via Docker.

Após, basta instalar as ferramentas CLI AWS:

```bash
pip install awscli
pip install awscli-local
```

**Observação**
Meu ambiente é o Linux, e escreverei pensando nele ;)

## O objetivo deste blog

Quero compartilhar minha jornada aprendendo AWS **com apoio do LocalStack**.

---


