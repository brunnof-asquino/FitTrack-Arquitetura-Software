# Arquitetura Orientada a Eventos do FitTrack

## 1. Objetivo

Esta proposta adiciona uma camada de comunicação assíncrona baseada em eventos à arquitetura do FitTrack, mantendo o sistema principal organizado como um monólito modular baseado em DDD.

A mensageria permite que acontecimentos relevantes do domínio sejam comunicados a outros componentes sem que o módulo responsável pela operação precise conhecer todos os interessados nesse acontecimento.

## 2. Evento de negócio

O evento inicial proposto é `WorkoutPublished`, que representa a publicação bem-sucedida de um treino.

O evento deve ser publicado somente depois que o treino for persistido com sucesso. Sua finalidade é comunicar um fato ocorrido, e não solicitar diretamente a execução de uma função específica.

Exemplo de dados conceituais do evento:

- `eventId`: identificador único do evento.
- `eventType`: tipo do evento, como `WorkoutPublished`.
- `occurredAt`: data e hora em que o acontecimento ocorreu.
- `workoutId`: identificador do treino publicado.
- `userId`: identificador do usuário responsável pela publicação.
- `schemaVersion`: versão do formato do evento.

Os campos apresentados são uma proposta inicial de contrato. O evento deve conter apenas os dados necessários aos consumidores, evitando informações pessoais ou sensíveis desnecessárias.

## 3. Tecnologia de mensageria

A tecnologia escolhida para a proposta é o Apache Kafka.

Kafka funciona como uma plataforma distribuída de eventos, permitindo que produtores publiquem registros em tópicos e que consumidores leiam esses registros de forma independente.

O tópico inicial proposto é `fittrack.workout-published`.

A chave da mensagem pode ser o identificador do treino (`workoutId`), permitindo que eventos com a mesma chave sejam direcionados à mesma partição, preservando sua ordem relativa dentro dessa partição.

## 4. Produtor e consumidores

O produtor será um componente de publicação de eventos integrado ao fluxo de publicação de treinos do FitTrack.

São propostos dois consumidores:

- **Consumidor de notificações:** poderá utilizar o evento para iniciar notificações a usuários interessados, quando essa funcionalidade e suas regras estiverem definidas.
- **Consumidor de estatísticas:** poderá atualizar métricas relacionadas à publicação de treinos.

Esses consumidores representam extensões futuras da arquitetura. Não se afirma que já existam no sistema atual.

## 5. Benefícios para o negócio

- Permitir a evolução de funcionalidades sem acoplar o fluxo de publicação a todos os processos secundários.
- Reduzir o trabalho executado diretamente durante a requisição de publicação.
- Permitir o processamento independente por consumidores diferentes.
- Facilitar a recuperação e o reprocessamento de eventos retidos, quando necessário.

## 6. Consistência e confiabilidade

O banco de dados do FitTrack e o Kafka não participam automaticamente de uma única transação atômica. Se o treino for salvo e a publicação do evento falhar, poderá haver divergência entre o estado persistido e os eventos publicados.

Para uma implementação mais confiável, recomenda-se avaliar o padrão **Transactional Outbox**. Nesse padrão, o treino e o registro do evento a publicar são gravados na mesma transação do banco de dados. Um componente separado publica posteriormente os registros pendentes no Kafka.

Os consumidores também devem ser idempotentes, de modo que a reentrega ou o reprocessamento de um evento não provoque efeitos duplicados indevidos.

Falhas temporárias devem contar com política de tentativas, observabilidade e procedimento de tratamento de mensagens que não possam ser processadas. A estratégia concreta dependerá da implementação escolhida.

## 7. Limites da proposta

Esta documentação apresenta uma proposta arquitetural, não uma implementação funcional. A adoção de Kafka não exige transformar imediatamente o FitTrack em microsserviços: o produtor pode permanecer dentro do monólito modular, enquanto os consumidores podem ser implantados separadamente quando houver justificativa operacional.
