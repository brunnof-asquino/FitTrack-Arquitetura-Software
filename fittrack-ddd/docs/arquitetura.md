# Arquitetura proposta

## 1. Estilo arquitetural

Propõe-se um **monólito modular baseado em DDD**, em vez de microsserviços. O projeto atual já é uma API organizada por módulos funcionais e um frontend separado. A proposta melhora os limites internos e as dependências sem introduzir, nesta etapa, a complexidade operacional de múltiplos serviços, deploys independentes e comunicação distribuída.

## 2. Camadas

### Domain

Contém entidades, objetos de valor, regras de negócio e contratos de domínio. Não deve importar Express, Sequelize, MySQL, Multer, JWT ou APIs do sistema de arquivos.

Submódulos propostos:

- `users/`
- `workouts/`
- `interactions/`

### Application

Contém casos de uso e coordena o domínio com dependências externas por meio de contratos. Exemplos: `RegisterUser`, `PublishWorkout`, `DeleteWorkout`, `AddComment` e `ToggleLike` (nomes ilustrativos, não classes já implementadas).

### Infrastructure

Implementa detalhes técnicos: repositórios Sequelize/MySQL, armazenamento de arquivos, hash de senha, emissão/verificação de tokens e outros adaptadores técnicos. Os serviços existentes usam diretamente modelos Sequelize e `fs`; na arquitetura-alvo essas dependências seriam isoladas aqui.

### Interfaces

Adapta entradas e saídas externas. Neste projeto, inclui rotas Express, controladores HTTP, validação de requisições e formatação de respostas. A camada HTTP invoca casos de uso em vez de conter regras de negócio.

## 3. Regra de dependência

A direção desejada é de fora para dentro: interfaces e infraestrutura podem depender de contratos definidos pelas camadas internas; o domínio não depende de infraestrutura. A aplicação coordena os casos de uso e deve evitar acoplamento direto a frameworks.

```text
Interfaces HTTP ───────┐
                       v
                  Application ─────> Domain
                       ^
                       |
Infrastructure ── implementa contratos usados pela aplicação/domínio
```

O diagrama é conceitual. Contratos de repositório podem ser definidos na aplicação ou no domínio conforme a responsabilidade do contrato; a implementação concreta fica na infraestrutura.

## 4. Organização de diretórios proposta

```text
src/
├── domain/
│   ├── users/
│   ├── workouts/
│   └── interactions/
├── application/
│   ├── users/
│   ├── workouts/
│   └── interactions/
├── infrastructure/
│   ├── persistence/
│   ├── media/
│   └── security/
└── interfaces/
    └── http/
```

Os diretórios são uma organização-alvo para a proposta acadêmica. Não contêm implementação migrada do backend original.

## 5. Relação com a arquitetura existente

| Arquitetura atual | Direção proposta |
|---|---|
| `modules/*/*Routes.js` | `interfaces/http/routes/` |
| `modules/*/*Controller.js` | `interfaces/http/controllers/` |
| `modules/*/*Service.js` | Dividir entre `application/` e regras de `domain/`, conforme responsabilidade |
| `modules/*/*Model.js` (Sequelize) | `infrastructure/persistence/` |
| `config/database.js` e associações Sequelize | `infrastructure/persistence/` |
| `middlewares/auth.js`, JWT e bcrypt | Adaptadores técnicos em `infrastructure/security/` e integração na interface HTTP |
| `middlewares/*Multer.js` e operações `fs` | `infrastructure/media/` |

A tabela sugere uma migração conceitual; não significa que cada arquivo deva ser movido mecanicamente. A separação deve seguir a responsabilidade real do código.

## 6. Justificativa

- **Coesão:** código relacionado a uma mesma capacidade de negócio fica agrupado.
- **Baixo acoplamento:** regras do domínio não dependem diretamente do framework web ou do ORM.
- **Testabilidade:** casos de uso podem ser testados com dependências simuladas, sem exigir banco real em todos os testes.
- **Evolução:** detalhes técnicos podem ser substituídos com menor impacto nas regras de negócio.
- **Proporcionalidade:** o monólito modular aproveita a organização existente sem exigir microsserviços prematuramente.

## 7. Limites e próximos refinamentos

Esta proposta é documental. Antes de uma implementação real, devem ser confirmadas as regras de negócio, os limites de cada contexto, a necessidade de agregados e objetos de valor, e os contratos de persistência. A arquitetura DDD não é alcançada apenas pela estrutura de pastas; depende de o código expressar o modelo de domínio e suas regras.
