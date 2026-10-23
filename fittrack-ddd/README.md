# FitTrack — Proposta de Arquitetura baseada em DDD

Este repositório documenta uma proposta de reorganização arquitetural do FitTrack com base em Domain-Driven Design (DDD), como parte da Questão 1 da avaliação de Projeto e Arquitetura de Software.

## Objetivo

Separar o domínio e as regras de negócio dos detalhes técnicos de entrega HTTP, persistência em banco de dados, autenticação e armazenamento de arquivos, mantendo a aplicação como um **monólito modular**. Esta é uma proposta arquitetural: não representa uma reimplementação funcional do backend atual.

## Domínio da aplicação

O FitTrack é uma aplicação Web voltada ao compartilhamento de treinos físicos, permitindo que usuários criem seus perfis, publiquem treinos acompanhados de vídeos, consultem conteúdos publicados por outros usuários e interajam por meio de curtidas e comentários. A plataforma também disponibiliza recursos de busca e visualização de perfis, favorecendo a descoberta de conteúdos e a interação entre os participantes. Seu domínio de aplicação envolve o gerenciamento de usuários, a publicação e a consulta de treinos, as interações sociais e o gerenciamento dos arquivos multimídia associados às publicações.

## Conteúdo

- [`docs/dominio.md`](docs/dominio.md): domínio, conceitos e limites propostos.
- [`docs/arquitetura.md`](docs/arquitetura.md): camadas, responsabilidades e regras de dependência.
- [`docs/glossario.md`](docs/glossario.md): vocabulário comum inicial.
- [`diagrams/arquitetura-ddd.md`](diagrams/arquitetura-ddd.md): diagrama Mermaid da proposta.
- [`src/`](src/): estrutura de diretórios ilustrativa para uma possível implementação futura; não contém código de produção.

## Contexto técnico observado

A aplicação existente possui um frontend em Vue 3, com Vue Router, Pinia, Axios e Vite. O backend é uma API Node.js/Express, utiliza Sequelize com MySQL e possui módulos para usuários, treinos, curtidas, comentários e busca. O backend também utiliza JWT/bcrypt para autenticação e Multer/armazenamento local para arquivos multimídia.

## Decisões e limites

1. **Monólito modular:** os módulos permanecem dentro de uma aplicação implantável como unidade, sem exigir microsserviços.
2. **Domínio independente de tecnologia:** regras de negócio não devem depender diretamente de Express, Sequelize, MySQL ou do sistema de arquivos.
3. **Aplicação coordena casos de uso:** a camada de aplicação coordena operações e depende de contratos, quando necessário.
4. **Infraestrutura implementa detalhes técnicos:** persistência, armazenamento de mídia e mecanismos de segurança ficam fora do núcleo de domínio.
5. **Proposta, não migração executada:** os diretórios em `src/` mostram uma organização-alvo; os arquivos do backend original não foram alterados nem migrados.

## Como clonar o repositório

Para obter uma cópia local, siga estes passos:

1. Instale o [Git](https://git-scm.com/downloads), caso ainda não esteja instalado.
2. Abra um terminal na pasta em que deseja guardar o projeto.
3. Execute o comando abaixo, substituindo `brunnof-asquino` pelo nome de usuário ou organização que hospeda o repositório:

```bash
git clone https://github.com/brunnof-asquino/fittrack-ddd.git
cd fittrack-ddd
```

O comando `git clone` baixa os arquivos e cria a pasta local `fittrack-ddd`. 

## Status

Proposta inicial para fins acadêmicos. Os limites de contexto e as regras de domínio deverão ser refinados conforme os requisitos confirmados do produto.
