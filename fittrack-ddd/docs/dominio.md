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
