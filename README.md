# FitTrack — Projeto e Arquitetura de Software

## Sobre o projeto

O **FitTrack** é uma plataforma voltada ao compartilhamento de treinos físicos, na qual os usuários podem publicar vídeos de exercícios, visualizar conteúdos de outros usuários e interagir por meio de curtidas, comentários e funcionalidades sociais.

Este repositório reúne a documentação e as propostas de evolução arquitetural desenvolvidas para a **atividade avaliativa N2 da disciplina Projeto e Arquitetura de Software**.

## Objetivo da atividade N2

A atividade tem como objetivo analisar e propor melhorias na arquitetura de software do FitTrack, aplicando conceitos e práticas relacionados a:

- **Domain-Driven Design (DDD):** organização da arquitetura a partir do domínio de negócio.
- **Arquitetura Hexagonal:** separação do núcleo da aplicação de tecnologias e mecanismos externos, por meio de portas e adaptadores.
- **Event-Driven Architecture (EDA):** utilização de eventos e comunicação assíncrona para integrar componentes.
- **Apache Kafka:** tecnologia proposta para a camada de mensageria.
- **Princípios SOLID:** análise de problemas de design e propostas de refatoração para melhorar a organização, a manutenção e a evolução do código.

O trabalho é dividido em três questões, cada uma com um objetivo específico.

## Questões da atividade

### Questão 1 — DDD e Arquitetura Hexagonal

Propõe a reorganização arquitetural do FitTrack com foco no domínio de negócio, identificando entidades, agregados, objetos de valor, repositórios, serviços de domínio e casos de uso.

A proposta também descreve a separação entre domínio, aplicação, infraestrutura e interfaces, buscando reduzir o acoplamento entre as regras de negócio e as tecnologias utilizadas.

**Documentação:** [`fittrack-ddd/`](./fittrack-ddd/)

### Questão 2 — Arquitetura Orientada a Eventos

Propõe a inclusão de uma camada de mensageria baseada em Apache Kafka, permitindo que eventos de negócio sejam publicados e processados de forma assíncrona.

A documentação apresenta um cenário de referência, os componentes envolvidos, o fluxo de eventos, a representação arquitetural com o modelo C4 e as justificativas para as escolhas técnicas e de negócio.

O uso de consumidores para notificações e estatísticas é apresentado como uma possibilidade de evolução, e não como funcionalidades já implementadas no sistema.

**Documentação:** [`fittrack-event-driven/`](./fittrack-event-driven/)

### Questão 3 — Análise dos princípios SOLID

Analisa o código-fonte existente do FitTrack para identificar violações dos princípios SOLID. Para cada ocorrência, o trabalho deve apresentar um trecho de código, o princípio relacionado, a justificativa da análise e uma sugestão de refatoração.

A análise também descreve a metodologia utilizada para permitir que a inspeção seja reproduzida.

**Documentação:** [`fittrack-solid/`](./fittrack-solid/)

> **Observação:** os caminhos das Questões 2 e 3 devem corresponder às pastas efetivamente criadas no repositório. Os links poderão ser ajustados conforme a organização final dos documentos.

## Organização do repositório

A documentação é organizada por questão para facilitar a navegação e distinguir as diferentes propostas arquiteturais.

```text
FitTrack-Arquitetura-Software/
├── README.md
├── fittrack-ddd/
│   ├── README.md
│   ├── docs/
│   └── diagrams/
├── fittrack-event-driven/
│   └── arquitetura-event-driven.md
└── fittrack-solid/
    └── analise-solid.md
```

A estrutura acima representa a organização planejada para os materiais da atividade e poderá ser ajustada à medida que cada questão for concluída.

## Escopo e situação da documentação

As propostas arquiteturais documentadas neste repositório têm caráter acadêmico e descritivo. A apresentação de uma arquitetura-alvo não significa que todos os seus componentes tenham sido implementados ou integrados ao sistema original.

O objetivo é demonstrar a compreensão dos conceitos estudados, justificar as decisões de arquitetura e registrar propostas de melhoria de forma clara e verificável.

## Informações Adicionais

**Aluno:** Brunno Fischer Asquino
**Curso:** Engenharia de Software, Católica de SC
**Disciplina:** Projeto e Arquitetura de Software
**Fase:** 5º Fase
**Professor:** Luiz Carlos Camargo

## Repositório

[FitTrack-DDD no GitHub](https://github.com/brunnof-asquino/FitTrack-DDD)
