# Domínio do FitTrack

## 1. Descrição do domínio

O FitTrack é uma aplicação Web voltada ao compartilhamento de treinos físicos, permitindo que usuários criem seus perfis, publiquem treinos acompanhados de vídeos, consultem conteúdos publicados por outros usuários e interajam por meio de curtidas e comentários. A plataforma também disponibiliza recursos de busca e visualização de perfis, favorecendo a descoberta de conteúdos e a interação entre os participantes. Seu domínio de aplicação envolve o gerenciamento de usuários, a publicação e a consulta de treinos, as interações sociais e o gerenciamento dos arquivos multimídia associados às publicações.

## 2. Evidências no projeto atual

A análise dos arquivos do backend identifica modelos Sequelize para `User`, `Workout`, `Like` e `Comment`, bem como serviços e controladores correspondentes. A API também contém módulo de busca. O frontend apresenta telas e serviços para autenticação, perfis, treinos, feed, curtidas, comentários e busca.

Esta proposta distingue fatos observados de conceitos sugeridos. Por exemplo, acompanhamento entre usuários aparece como comportamento da interface, mas a existência de um modelo de domínio dedicado deve ser confirmada no código completo antes de ser tratada como entidade já implementada. A página de notificações está em construção; notificações não são consideradas requisito já implementado.

## 3. Contextos delimitados (proposta)

### 3.1. Contexto de Usuários

Responsável por cadastro, autenticação e informações de perfil.

- Conceito central observado: `User`.
- Casos de uso candidatos: registrar usuário, autenticar, consultar perfil, consultar perfil público e atualizar perfil.
- Observação: credenciais e hash de senha são detalhes sensíveis; a representação de domínio não deve expor senha em consultas de perfil.

### 3.2. Contexto de Treinos

Responsável pela publicação e consulta de treinos.

- Conceito central observado: `Workout`.
- Casos de uso candidatos: publicar treino, consultar detalhes, listar feed, listar treinos de um usuário, editar e excluir treino.
- Os caminhos de vídeo e capa são referências a arquivos, enquanto o armazenamento físico é uma preocupação de infraestrutura.

### 3.3. Contexto de Interações

Responsável pelas interações dos usuários com treinos.

- Conceitos observados: `Like` e `Comment`.
- Casos de uso candidatos: curtir/descurtir treino, comentar e remover comentário (confirmar as operações exatas no comportamento atual).
- O modelo atual impõe unicidade para a combinação usuário/treino em `Like`, evidenciando a regra de não registrar duas curtidas iguais para o mesmo par.

## 4. Conceitos de domínio e persistência

Os modelos Sequelize existentes representam dados persistidos, mas não são automaticamente equivalentes a entidades de domínio bem delimitadas. Na arquitetura proposta, as regras de negócio devem ser expressas no domínio ou nos casos de uso, e o Sequelize deve ficar na infraestrutura.

| Conceito | Evidência atual | Papel proposto |
|---|---|---|
| `User` | Modelo `modules/user/userModel.js` | Entidade de usuário no contexto de Usuários |
| `Workout` | Modelo `modules/workout/workoutModel.js` | Entidade de treino no contexto de Treinos |
| `Like` | Modelo `modules/like/likeModel.js` | Registro de interação no contexto de Interações |
| `Comment` | Modelo `modules/comment/commentModel.js` | Comentário associado a usuário e treino |
| Mídia do treino | Campos `workoutPath` e `thumbnailPath` | Referências no domínio; armazenamento via porta/adaptador técnico |

## 5. Regras de negócio identificadas ou a confirmar

### Observadas diretamente na implementação

- Usuário possui `username` e `email` únicos no banco.
- Treino requer título, arquivo de vídeo, capa e usuário proprietário.
- Uma curtida é única por combinação de usuário e treino, conforme índice único do banco.
- Comentário requer conteúdo, usuário e treino.
- A operação de edição de treino verifica se o solicitante é proprietário.
- A exclusão de treino verifica propriedade ou privilégio administrativo e remove registros relacionados de curtidas e comentários em uma transação.

### A confirmar como regras explícitas de domínio

- Limites de tamanho e formatos aceitos para mídia.
- Se o proprietário pode desativar comentários ou curtidas.
- Regras de bloqueio de usuários e efeitos sobre publicação/interação.
- Regras exatas para seguir/deixar de seguir usuários.
- Política de retenção e exclusão dos arquivos físicos.

As regras acima devem ser verificadas antes de serem formalizadas como invariantes de domínio.

## 6. Elementos de modelagem DDD

Esta seção apresenta como os principais elementos de Domain-Driven Design (DDD) podem ser aplicados ao FitTrack. As definições descrevem a arquitetura proposta e distinguem os conceitos observados no projeto das abstrações sugeridas para uma futura implementação.

### 6.1. Entidades (Entities)

Entidades são objetos identificados por uma identidade própria, que permanece relevante mesmo quando seus atributos mudam.

No FitTrack, propõem-se as seguintes entidades de domínio:

| Entidade | Responsabilidade |
|---|---|
| `User` | Representar um usuário cadastrado, seu identificador e as informações pertinentes ao perfil. |
| `Workout` | Representar um treino publicado, seu proprietário, seus dados e suas referências de mídia. |
| `Like` | Representar a curtida de um usuário em um treino, identificada pela interação registrada. |
| `Comment` | Representar um comentário associado a um usuário e a um treino. |

Esses conceitos possuem correspondência com modelos de persistência observados no projeto atual. Na arquitetura-alvo, as entidades de domínio não devem depender diretamente dos modelos Sequelize.

### 6.2. Agregados (Aggregates)

Agregados são conjuntos de objetos de domínio tratados como uma unidade de consistência, protegidos por uma raiz de agregado (*Aggregate Root*).

Como proposta inicial para o FitTrack:

- **Agregado `User`:** tem `User` como raiz e concentra as informações e regras que pertencem à identidade e ao estado do usuário.
- **Agregado `Workout`:** tem `Workout` como raiz e concentra os dados e as regras essenciais de um treino, incluindo a referência ao proprietário e à mídia associada.
- **Agregado de interação:** `Like` e `Comment` podem ser tratados como agregados independentes, pois representam interações que podem ser registradas e removidas separadamente.

A separação de curtidas e comentários em agregados próprios é uma proposta de modelagem, não uma característica já implementada no sistema atual. Os limites definitivos devem considerar as regras de consistência e as operações reais do produto.

### 6.3. Objetos de Valor (Value Objects)

Objetos de valor representam conceitos definidos pelos seus atributos, e não por uma identidade própria. Idealmente, são imutáveis e concentram validações relacionadas ao significado dos valores.

Possíveis objetos de valor para o FitTrack:

| Objeto de valor proposto | Finalidade |
|---|---|
| `Email` | Representar um endereço de e-mail e centralizar as validações pertinentes. |
| `Username` | Representar o nome de usuário e concentrar suas regras de formato. |
| `WorkoutTitle` | Representar o título de um treino e aplicar as restrições de conteúdo definidas pelo domínio. |
| `MediaReference` | Representar uma referência a um arquivo de mídia sem expor ao domínio os detalhes de armazenamento físico. |

Esses objetos são sugestões para a modelagem. As validações exatas devem respeitar os requisitos confirmados do sistema. Por exemplo, a unicidade de e-mail e nome de usuário também depende de garantias de persistência e tratamento de concorrência, não apenas de validações locais.

### 6.4. Repositórios (Repositories)

Repositórios definem contratos para recuperar e persistir objetos de domínio, isolando o núcleo da aplicação dos detalhes técnicos do banco de dados.

Na arquitetura proposta, são considerados os seguintes contratos:

- `UserRepository`: operações de consulta e persistência de usuários.
- `WorkoutRepository`: operações de consulta e persistência de treinos.
- `LikeRepository`: operações para registrar, consultar e remover curtidas.
- `CommentRepository`: operações para registrar, consultar e remover comentários.

Os contratos devem ser definidos de acordo com as necessidades dos casos de uso. Suas implementações concretas ficam na infraestrutura e podem utilizar Sequelize e MySQL.

O uso de repositórios não significa que os modelos Sequelize atuais já tenham sido convertidos em entidades de domínio ou que esses contratos já estejam implementados.

### 6.5. Serviços de Domínio (Domain Services)

Serviços de domínio encapsulam regras de negócio que não pertencem naturalmente a uma única entidade ou objeto de valor. Devem representar conceitos do domínio, e não apenas agrupar operações técnicas.

No FitTrack, um possível candidato seria um serviço de política de publicação de treinos, caso os requisitos futuros estabeleçam regras que dependam de vários objetos do domínio e não possam ser adequadamente atribuídas a uma entidade específica.

Entretanto, ainda não há evidência suficiente para afirmar que esse serviço seja necessário. Regras simples devem permanecer nas entidades ou nos objetos de valor apropriados; a coordenação de operações e dependências externas pertence à camada de aplicação.

### 6.6. Fábricas (Factories)

Fábricas encapsulam a criação de objetos ou agregados quando o processo de construção é complexo ou exige regras específicas para garantir um estado inicial válido.

No FitTrack, uma possível `WorkoutFactory` poderia centralizar a criação de um treino caso o processo venha a exigir a composição e a validação de vários elementos de domínio.

Para objetos com construção simples, uma fábrica separada pode não trazer benefícios. Portanto, seu uso é opcional e deve ser justificado pela complexidade real do processo de criação, evitando abstrações desnecessárias.

### 6.7. Síntese da proposta

| Elemento DDD | Aplicação proposta no FitTrack |
|---|---|
| Entidades | `User`, `Workout`, `Like` e `Comment`. |
| Agregados | `User` e `Workout` como agregados candidatos; interações avaliadas separadamente. |
| Objetos de valor | `Email`, `Username`, `WorkoutTitle` e `MediaReference`, sujeitos à validação dos requisitos. |
| Repositórios | Contratos para persistir e consultar agregados e interações. |
| Serviços de domínio | Apenas quando houver regras de negócio que não pertençam naturalmente a uma entidade ou objeto de valor. |
| Fábricas | Apenas quando a criação de objetos ou agregados exigir lógica relevante de construção e validação. |

A aplicação desses elementos deve evoluir a partir das regras reais do FitTrack. A presença de um padrão na documentação representa uma decisão de modelagem proposta, não uma implementação já existente no código.
