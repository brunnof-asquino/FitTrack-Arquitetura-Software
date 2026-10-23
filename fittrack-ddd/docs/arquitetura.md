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

## 7. Contratos e casos de uso

Nesta proposta de arquitetura, a camada de aplicação coordena os casos de uso do FitTrack e utiliza contratos para acessar recursos externos. A infraestrutura fornece as implementações concretas desses contratos.

Essa separação reduz o acoplamento entre as regras de negócio e os detalhes técnicos de persistência, armazenamento de arquivos e segurança.

### Contratos propostos

Os contratos a seguir representam interfaces conceituais da arquitetura-alvo. Eles não indicam que essas interfaces já estejam implementadas no código atual.

| Contrato | Responsabilidade |
|---|---|
| `UserRepository` | Definir operações para consultar e persistir usuários. |
| `WorkoutRepository` | Definir operações para criar, consultar, atualizar e remover treinos. |
| `InteractionRepository` | Definir operações de persistência para curtidas e comentários. |
| `MediaStorage` | Definir operações para armazenar, consultar e remover arquivos de mídia. |
| `PasswordHasher` | Definir operações para gerar e verificar hashes de senhas. |
| `TokenService` | Definir operações para emitir e validar tokens de autenticação. |

Os contratos devem ser definidos de acordo com as necessidades dos casos de uso. A infraestrutura poderá implementá-los utilizando as tecnologias adequadas, como Sequelize e MySQL para persistência e adaptadores específicos para mídia e autenticação.

### Casos de uso propostos

| Caso de uso | Responsabilidade |
|---|---|
| `RegisterUser` | Coordenar o cadastro de um usuário e o tratamento seguro de sua senha. |
| `PublishWorkout` | Coordenar a publicação de um treino e o armazenamento dos dados e arquivos necessários. |
| `UpdateWorkout` | Coordenar a atualização de um treino, respeitando as regras de autorização e negócio. |
| `DeleteWorkout` | Coordenar a remoção de um treino e o tratamento dos recursos associados. |
| `LikeWorkout` | Registrar uma curtida conforme as regras de negócio definidas. |
| `UnlikeWorkout` | Remover uma curtida existente. |
| `AddComment` | Validar e registrar um comentário em um treino. |
| `SearchWorkouts` | Coordenar a busca de treinos conforme os critérios disponíveis. |

Esses casos de uso são uma proposta de organização da aplicação. Suas regras detalhadas devem ser confirmadas a partir do comportamento real do sistema e dos requisitos do domínio.

### Fluxo de dependências

1. Um controller da interface HTTP recebe a requisição e encaminha os dados para o caso de uso correspondente.
2. O caso de uso coordena a operação e aplica as regras pertinentes ao domínio.
3. Quando precisa acessar dados ou serviços externos, a aplicação utiliza os contratos definidos para essa finalidade.
4. A infraestrutura implementa os contratos e realiza as operações técnicas, como persistência no banco de dados, armazenamento de mídia e segurança.
5. O resultado retorna pela camada de aplicação até a interface HTTP, que produz a resposta apropriada.

### Benefícios arquiteturais

- **Separação de responsabilidades:** cada camada possui uma finalidade clara.
- **Baixo acoplamento:** os casos de uso não precisam conhecer diretamente os detalhes dos adaptadores concretos.
- **Testabilidade:** os contratos permitem substituir dependências externas por implementações de teste.
- **Manutenibilidade:** mudanças na persistência ou no armazenamento podem ser isoladas da lógica de aplicação, desde que os contratos sejam preservados.

A adoção efetiva dessa estrutura exigiria uma refatoração gradual do código existente. Esta documentação descreve a arquitetura proposta e não afirma que essa refatoração já tenha sido realizada.

## 8. Limites e próximos refinamentos

Esta proposta é documental. Antes de uma implementação real, devem ser confirmadas as regras de negócio, os limites de cada contexto, a necessidade de agregados e objetos de valor, e os contratos de persistência. A arquitetura DDD não é alcançada apenas pela estrutura de pastas; depende de o código expressar o modelo de domínio e suas regras.
