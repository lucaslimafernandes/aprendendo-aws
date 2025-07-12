---
title: "Amazon S3: Fundamentos do armazenamento na nuvem AWS"
date: 2025-07-12T22:32:10-03:00
draft: true
---

Bom, vou falar um pouco sobre Amazon S3 - Simple Storage Service - Irei seguir o conteúdo aprendido no próprio site da AWS Educate.

S3 é um serviço de armazenamento, que oferece escalabilidade, disponibilidade e segurança.

Os casos de uso incluem sites, aplicativos móveis, backup e restauração, arquivos, aplicativos empresariais, dispositivos de Internet das Coisas (IoT) e análise de big data.

## Conceitos básicos

O S3 armazena objetos, objetos armazenam:

- Dados
- Metadados: tipo de conteúdo, data da última notificação e outras informações.
- Chave (object key): Identificador único

Os objetos são armazenados dentro de um Bucket, que é semelhante a um diretório (ou pasta)

Buckets são criados dentro de uma região.

A combinação do bucket name, chave e Identificador de versão(caso se aplique) identificam exclusivamente o objeto.

Armazenamentos podem ser "ativo" ou de "arquivo"

- Ativo: Dados que você usa o tempo todo, ou com menos frequência mas de acesso rápido
- Arquivo: Dados que você raramente usa, mas deve ser mantido (por exemplo contratos, itens de auditorias, etc).


## Classes

S3 possue 6 classes, estas classes representam a velocidade de acesso aos arquivos, sendo que isso influência em custos, sendo um fator importante de escolha.

- S3 Standard: Sites estáticos; arquivos frequentemente acessados.
- S3 Standard-IA (infrequent acess): Backups de sistemas; arquivos raramente acessados, mas se necessário, de rápido acesso.
- S3 One Zone-IA: Armazenamento para backups de replicação entre regiões de outros buckets; cópias de backups locais; arquivos facilmente substituíveis.
- S3 Glacier: Backups de longo prazo para fins de conformidade; substituição de fitas magnéticas on-premises; ativos de mídia digital para grandes arquivos de mídia. Para recuperação demora horas no modo padrão, e de 5-10 min no modo de recuperação acelerada.
- S3 Glacier Deep Archive: Arquivos que devem ser retidos para fins de conformidade; longo prazo. Acima de 12 horas para recuperação.
- S3 Intelligent-Tiering: Cargas de trabalho imprevisíveis; cargas de trabalho desconhecidas; que mudam rapidamente. Classe inteligente, busca a melhor solução entre as classes.

## Custos

## Segurança

## Transferência e Uploads

## 

