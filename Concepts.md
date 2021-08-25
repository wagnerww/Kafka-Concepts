# CONCEITOS

### O Básico sobre kakfa
- Altíssimo THROUGHPUT = tem uma alta capacidade de receber e processar
as requisições
- Latência extremamente baixa(2ms)
- Escalável
- Aramazenamento = Aramazena em uma area de dados, com resiliencia todas as
informações que são armazenada nele
- Alta disponibilidade
- Se conecta com quase tudo - Desenvolvido em JAVA, mas tem driver para quase
todas as linguagens e muitas bibliotecas com muitas e diversas coisas prontas.
- Muitas ferramentas open-source, porque ele possui essa licenca
- Quem usa: Linkedin, Netflix, Uber, Twiter, Spotify e inumeros bancos
- Quando usar? Quando se tem alto volume de dados e quando não se tem tanto volume mas se tem muitos sistemas que precisam se conectar. Esse dois casos são os principais.

### Diferença básica do RabbitMQ
  - O Rabbit tem um apenas um consumer, para ler a fila, após o ack(confirmação da leitura), a mensagem já eras.
  - No Kafka, pode se ter vários sistemas lendo a mesmas mensagem e ela vai esta disponivel para todos, porque ela ganha um ID sequencial e é possível retroceder ou ler quantas vezes eu quero ela, como se fosse um sistema de log, vai jogando ela e ela será aramazenada sequencialmente

### Dinâmcia básica de funcionamento
 Um producer produz uma mensagem envia ao Kafka que possui seus Nodes, chamados de brokers. Broker é uma máquina e cada broker tem o seu próprio banco de dados e aí ela pode ficar ficar no Broker A, B, C, etc. O consumidor vai até o broker e consome a mensagem. O Kafka por sí só, não envia mensagem para ninguém!


Producer ->  (Kafka -> Brokers A,B,C(clusters do kafa)) <- Consumer

- O Básico para ir para a produção: 3 Brokers, ou seja, 3 máquinas.

- Os brokers se comunicão o tempo inteiro, e isto é feito por baixo dos panos usando o zookeeper, que nada mais é um service discovery para os clusters do kafka. mas o kafka está tirando o zookeeper para ficar independente neste ponto.

### Topicos(Topic)
 - É o canal de comunicação responsável por receber e disponibilizar os dados enviados para o kafka, ou seja, é onde envia e faz a leitura das mensagens.
- O topico é como se fosse um log, ele vai armazeando mensagem uma atrás da outra, e cada mensagem ganha um ID, que é também chamado de offset(0,1,2,3....). Então é possível ler ela varias vezes.

### Anatomia de um registro(offset)
  - Headers
  - keys(Tipo da mensagem/agrupamento)
  - value = payload, ou seja, a mensagem.
  - timestamp

### Partições
  - Cada topico pode ter uma ou mais partições e garantir a distribuição e resiliencia dos seus dados.
  - Com mais partições é possível ter mais consumidores e com isso, a velocidade d eleitura aumenta, por que as mensagem são distribuidas em partições. Ao invés de se ter 1 milhão de mensagem em uma unica partição, é melhor se ter 500 mil em 2 partoções e 2 consumidores, ou seja, quanto mais partições mais consumidores eu poderei ter.
  - Se tiver 3 partições, cada hora a mensagem pode cair em alguma partição, ou seja, ela não é replicada para as demais, fazendo assim um round-robbin.
  - Um consumidor pode ler mais de uma partição, no caso quando se tem mais partições que consumidores.

  ### Efeito colateral de trabalhar com partições
  Imagina que você precisa lançar uma transferencia e após um estorno. Pode haver um consumidor de estorno mais rápido que o que processa a transferencia, ou seja, não será processada em ordem, o estorno será executado primeiro do que a transferencia. Até por que o estorno pode parar em partições diferente da transferencia. Só da para garantir a ordem das mensagem dentro de uma unica partição.

  - Como é possível garantir a ordem das mensagem? Pra isso as keys ajudam
  - Sobre Keys: O Kafka agrupa todas as mensagens que são mandadas com uma mesma key sempre para a mesma partição. Então, se existem 3 partições e foi enviada uma mensagem com a key="Movimentacao" e está parou na partição 1, a próxima mensagem que tiver essa key o kafka identifica e manda essa para a partição 1 também. Garantindo assim a ordem de entrega.
  - Quando não se tem ordem dependente, não precisa usar keys.

### Partições Distruibuídas
  - Replicator Factor= Quando é criado um tópico é especificado quantas partições ele terá, e assim será criado 2 partições em cada broker, garantindo assim a resiliencia, ou seja, um bkp/cópia. Quanto mais critica é a aplicação, mais deve ser aumentada o replicator factor, mas aí é necessário mais espaço de armazenamento.
  - Normalmente se usa 2 ou 3, 3 quando é algo muito critico. O 2 é o minimo.

#### Partion leadership
 - O Kafka designa sempre uma partição líder e o consumidor sempre irá ler desta. As demais são reserva, tmabém chamadas de followers.

### Producer - Garantia de entrega
 O producer sempre vai gravar na partição líder e poderá setar um parâmetro:
  - "Ack 0" = Acknologe none - Mando a mensagem e não precisa retornar que a mensagem deu certo, normalmente chamado de FF(Fire and Forget, dispara e esquece). Método um pouco arriscado, pq não tem retorno de gravação. Por outro lado, o kafka processa mais rápido. Deve estar ciente que pode perder mensagem.
  - "ack 1" = Acknologe Leader - Mando a mensagem, ela vai para o lider, o lider salva a mensagem e este envia uma resposta pro producer dizendo qeu salvou a mensagem e tals. O único problema é que o kafka pode demorar para sincronizar com os outros brokers e se o broker leader cair antes de sincronizar a mensagem é perdida.
  - "ack -1" = Acknologe ALL - Mando a mensagem, bate no lider, o lider vai salvar e vai replicar nos followers e estes vão avisar o lider que as mensagens foram salvas. Depois, o líder vai avisar o producer que a mensagem foi salva. O caso é, nunca tu vai perder a mensagem! Garantia total que foi salva e replicada! O problema é que é mais lento.

### Producer - Indepotência
É quando o producer manda a mensagme e pro algum motivo, por exemplo, falha de rede e não conseguiu enviar. Este por sua vez vai fazer um "retry", tendo novamente, mas a menagem já foi enviada por outro produtor. O que vai acontecer é que vai duplicar.

Quando  o produtor é indepotente, o kafka faz o descarte de uma, e ainda coloca na ordem certa, porém o processo é mais lento, mas evita que o consumer leia 2x a mesma mensagem.

### Consumers

### Consumers Groups
É quando vários consumidores são agrupados para ler um tópico com várias partições. Quando é um grupo, o kafaka organiza para que os consumers se distruibem e façam a leitura de partições diferentes. Por exemplo, quando tenho um tópico com 3 partições e 3 consumers no mesmo grupo, cada consumer lê uma partição. Se fosse 2 consumers, um deles iria ler 2 partições. Porém!!! Quando, tiver mais consumers do que partições, ele vai ficar "idle" parado. Ou seja, é um consumer de grupo por partição. Se for de outro grupo, aí sim pode, por que estão em grupos diferentes. Quando não for um grupo, ele trata o consumer como seno único.