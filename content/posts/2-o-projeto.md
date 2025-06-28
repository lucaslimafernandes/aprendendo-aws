---
title: "O Projeto!"
date: 2025-06-28T13:28:26-03:00
draft: true
---

Aqui vou explicar o que será feito com AWS (simulada pelo LocalStack) e como vou estruturar esse projeto para aprender de forma prática.

## Objetivo

Aprender de forma prática, enquanto estudo AWS para a certificação. Primeiramente usando **LocalStack**, e após migrando de fato para a AWS.

O projeto baseia-se em um trabalho do meu Mestrado em Ciência da Computação, que implementa o artigo:

> **Vision-Based Robust Lane Detection and Tracking in Challenging Conditions**

Vou precisar reescrever algumas partes, mas tudo bem :)

## Detecção de faixa

![Detecção de faixa](/images/projeto.png)

A ideia é:

1. Receber um vídeo (via API ou upload).
2. Dividir em frames e salvar no **S3**.
3. Processar os frames em serviços separados.
4. Classificar cada frame e registrar os dados no banco (**DynamoDB ou RDS**).
5. Enviar notificações sobre o processo (via **SNS, SQS ou SES**).


## Possíveis candidatas de uso

- **S3**: Simple Storage Service
- **Lambda**: Computação sem servidor
- **EC2**: Elastic Compute Cloud
- **ECS**: Elastic Container Service
- **Fargate**: Computação sem servidor para contêineres
- **SQS**: Simple Queue Service
- **SES**: Simple Email Service
- **SNS**: Simple Notification Service
- **DynamoDB**: noSQL
- **RDS**: Relational Database Service



