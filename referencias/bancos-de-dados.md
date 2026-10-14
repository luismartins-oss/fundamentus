# Bancos de dados
> Destilado de: cap-054, cap-055, cap-056 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).

## Ideia central
Um banco de dados não é mágica nem "uma tabela num JSON": é um conjunto de estruturas de dados em disco (árvores B/B+ organizadas em páginas), um interpretador de linguagem (SQL é só a interface), índices, um log de transações para sobreviver a panes e um servidor que arbitra acesso concorrente. Quem não entende esse mecanismo modela por superstição: cria índice demais ou de menos, traz a tabela inteira pra memória, faz loop de UPDATE, confia no ORM e escolhe NoSQL por hype. Para a imensa maioria dos sistemas, o caminho é um relacional maduro e gerenciado (Postgres/MariaDB) com cache, réplicas de leitura e jobs assíncronos. NoSQL distribuído e Big Data resolvem problemas de volume e velocidade de escrita que quase ninguém tem, e cobram em consistência, planejamento e dinheiro.

## Conceitos que todo sistema depende
- **Tabela = coleção de registros; SELECT = filtro** — O "banco estúpido" do autor é um array de objetos JavaScript: `insert` é `push`, `select ... where` é `filter`, `select col` é `map`, `order by` é `sort`. Funcionalmente imita SQL, mas não tem nada do que faz um banco real (persistência, índice, concorrência, integridade). [cap-054]
- **Registro de tamanho fixo e offset** — Num array, todo elemento tem o mesmo tamanho, então achar a posição N é aritmética (tempo constante). O DBF do dBase era isso em disco: cabeçalho de metadados e registros fixos (ex.: 100+13+100 = 213 bytes), navegados pulando de 213 em 213 bytes. [cap-056]
- **Table scan** — Sem índice, achar algo é olhar linha por linha até o fim (mesmo depois de achar, porque pode haver outro). No benchmark do autor com ~10 milhões de objetos em RAM, varrer a tabela levou ~150 ms; em disco seria 100–200 vezes mais lento. [cap-054]
- **Índice = cópia ordenada da coluna numa árvore** — Ordenado, dá pra fazer busca binária: começar no meio e descartar metade a cada comparação. Nas 37 linhas do exemplo, o pior caso caiu de 37 comparações para 6. No benchmark de ~10M linhas, consultar o índice custou ~10 ms e buscar os registros pelos ids mais ~35 ms (total ~40 ms, ao menos 3x melhor que o scan; o autor estima ~10x em C). [cap-054]
- **Complexidade logarítmica na prática** — Com 1 ms por comparação, 10 mil registros em scan custam até 10 s; com índice B-Tree, pouco mais de 10 ms. Dobrar os dados quase não mexe no tempo da busca indexada; ganho de 100x ou mais é comum. [cap-056]
- **Árvores balanceadas (AVL, Red-Black)** — Inserir custa um pouco mais (rotações para rebalancear), mas buscar exige menos comparações. O autor usou AVL para os índices em memória do protótipo. [cap-054]
- **Índice tem custo de escrita e de espaço** — Cada índice é mais uma estrutura a atualizar em todo INSERT, UPDATE e DELETE: indexar as 4 colunas transformou 1 insert em 5; numa tabela de 50 colunas todas indexadas seriam 51. No DBF de exemplo os índices ocupavam quase o dobro da tabela. [cap-054, cap-056]
- **Índice composto** — Para buscar por idade E cidade ao mesmo tempo, o índice deve conter os dois valores na mesma árvore; buscas separadas por cada coluna pedem índices separados. [cap-054]
- **Planejador de consultas e EXPLAIN** — Uma query real combina busca em índices com table scan no que não é indexado; o banco escolhe a estratégia e o EXPLAIN mostra qual. O Postgres pode até dividir um SELECT entre vários núcleos. [cap-054]
- **Disco lê em blocos, não em bytes** — HD lê no mínimo um setor/bloco (512 bytes a 4 KB) por vez, e acesso aleatório é caro (cabeçote, trilhas, disco girando). Ler 1 byte ou 4 KB custa quase o mesmo; por isso vale trazer mais dados contíguos e descartar o que não usa. [cap-054]
- **B-Tree e B+Tree** — Árvores balanceadas ("B" não é de binário) com alto fan-out: cada nó guarda dezenas ou centenas de chaves contíguas numa página, então poucos seeks cobrem muitos registros, e todas as folhas ficam na mesma profundidade. Na B+Tree só as folhas têm dados; os nós de cima guardam ponteiros. [cap-054, cap-056]
- **Páginas** — O SQLite não aloca registro a registro como o DBF: aloca páginas (intervalos de endereços) organizadas em B-Tree, com os registros como folhas, tanto para a tabela quanto para os índices, num único arquivo (até ~280 TB teóricos). Listagens ordenadas e SELECTs de faixa ficam baratos porque os vizinhos estão na mesma página. [cap-056]
- **Árvores B estão em todo lugar** — SQLite, MySQL, Postgres, Oracle, SQL Server, MongoDB e sistemas de arquivos como ZFS e BTRFS usam variantes de B-Tree/B+Tree. Bancos voltados a escrita massiva como Cassandra usam LSM-trees, não B-Tree, no armazenamento (complemento, não está no compêndio: o autor generaliza "todos usam árvore B"). [cap-054, cap-056]
- **Deleção marca, não remove** — No DBF, apagar só liga um bit de "apagado" e o arquivo não encolhe; o PACK recria o arquivo copiando o que é válido (precisa de espaço livre do tamanho do original). A mesma ideia sobrevive no VACUUM/AUTO VACUUM do Postgres e do SQLite. [cap-056]
- **Transaction log / journal (WAL)** — Arquivo append-only onde a operação é registrada antes de ser executada; após uma pane, o banco refaz (replay) só o que não concluiu. O WAL do SQLite escreve as mudanças num arquivo separado e depois promove; se a luz cair, o original está intacto, e leitores podem consultar o original enquanto a transação corre (um tipo de copy-on-write). [cap-054, cap-056]
- **Escrita in-place corrompe** — O REPLACE do dBase escrevia por cima do registro; uma queda de luz no meio de uma atualização em massa (ex.: o 9º dígito em SP, 2012) deixava o arquivo corrompido. Por isso backup diário em disquete era rotina nos anos 80/90. [cap-056]
- **ACID** — Atomicidade (todas as operações da transação ou nenhuma), Consistência (unicidade, chaves estrangeiras e demais regras sempre valem), Isolamento (transações em paralelo dão o mesmo resultado que em sequência) e Durabilidade (se respondeu "gravei", está fisicamente no disco). Após falha, o banco volta ao último estado consistente; nunca segue inconsistente. [cap-054, cap-056]
- **Por que escrever custa mais que ler no relacional** — Cada escrita precisa garantir durabilidade em disco, não pisar em outras conexões (controle de concorrência como MVCC, usado pela maioria) e manter log para rollback de transação que falha no meio. [cap-055]
- **Concorrência no acesso aos dados** — O jeito mais tosco de isolamento é travar o arquivo inteiro por transação. Era assim com DBF compartilhado em rede (Novell, IPX/SPX): cada máquina com seu programa, locks que nem sempre funcionavam e corrupção rotineira. [cap-056]
- **Modelo cliente-servidor** — Com 32 bits e mais RAM, um único processo servidor passou a ser o dono dos arquivos e os clientes só pedem "grava"/"pesquisa" via SQL; o servidor garante integridade e pode recusar operações inválidas. O custo é a rede: empacotar, rotear, bufferizar. O "servidor" do protótipo do autor é um TCP simples feito em ~15 minutos; o difícil é o resto. [cap-054, cap-056]
- **Latência e volume pela rede** — Rodar local é sempre mais rápido; via rede, o resultado é quebrado em pacotes, roteado e bufferizado antes de chegar. Um `SELECT *` numa tabela de 10 GB sem streaming ocupa toda a memória do cliente e ainda o swap. [cap-056]
- **SQL é linguagem, não formato de armazenamento** — Dizer que um banco "é SQL" não descreve como ele guarda os dados: o protótipo do autor fala SQL sobre um array JavaScript, e o Postgres devolve JSON se você pedir. [cap-054, cap-056]
- **Pipeline de um interpretador de SQL** — O texto passa pelo lexer (tokens) e pelo parser (parse tree); um listener percorre a árvore nó a nó e gera as operações internas. O autor reaproveitou a gramática do SQLite no ANTLR4 e transpilou SQL para chamadas JavaScript; expressões aninhadas no WHERE exigem tratamento recursivo (BETWEEN, IN, NOT que inverte operadores). [cap-054]
- **Interpretador embutido e stored procedures** — Todo banco carrega um interpretador. Stored procedures (PL/pgSQL no Postgres, PL/SQL no Oracle, T-SQL no SQL Server) são programas com IF e variáveis, e o Postgres as compila com JIT via LLVM. [cap-054]
- **Serialização (marshalling) tem preço** — Converter estrutura binária em texto duplica os dados em memória e incha: o JSON do array de usuários tinha 4,3 KB contra ~3 KB em memória (>40% maior; 1 GB viraria ~1,4 GB). E `JSON.stringify` regrava tudo de uma vez: não dá para inserir ou alterar uma linha incrementalmente. [cap-054]
- **Formato de consumo ≠ formato de armazenamento** — O MongoDB recebe e devolve JSON, mas guarda BSON (binário) numa B+Tree; o Postgres pode devolver JSON via SQL. Pedir em JSON exige memória extra para traduzir. Entre serviços, prefira protocolos binários (ex.: Protobuf); JSON se justifica em APIs públicas de pouco tráfego. [cap-054, cap-056]
- **Streaming e teto de memória** — Os dados pedidos vão parar em algum lugar. Puxar tudo para um array e depois montar um PDF em memória faz o consumo crescer com o volume; ler linha a linha e gravar a página no disco incrementalmente (ex.: PDFBox) mantém um teto fixo, seja o relatório de 100 ou de 1000 pedidos. [cap-056]
- **Operar no servidor, não no cliente** — Um único `UPDATE ... SET telefone = '9' || telefone WHERE ...` faz uma conexão, um commit e uma espera; o jeito ingênuo (SELECT * para a memória e um UPDATE por linha) faz 100 mil commits, 100 mil atualizações de índice e 100 mil idas e voltas, ordens de grandeza mais lento. [cap-056]
- **Constraints** — NOT NULL, tipos, tamanho, unicidade e chaves estrangeiras garantem no banco que só existem dados válidos. No DBF isso era um monte de IFs no código de cada cliente. [cap-056]
- **Validação em camadas** — A mesma regra no schema, no model do ORM e no componente do front-end, com testes unitários para cada uma e um teste de integração que cubra todas. A redundância é proposital, como lavar as mãos de novo antes da cirurgia. [cap-056]
- **Normalização × desempenho** — Normalizar reduz redundância, mas joins custam caro; relacionais e NoSQL privilegiam leitura de tabelas denormalizadas. Views materializadas do Postgres funcionam como uma tabela denormalizada real. Ao denormalizar, parte da consistência migra para a aplicação (enums, validações, testes). [cap-055]
- **Cache-aside** — A aplicação procura primeiro no Redis/Memcached; no miss, vai ao banco, grava no cache com expiração e responde. O cache pode ser reconstruído do zero após pane, e é o lugar para agregações caras (contadores, médias, métricas). [cap-055, cap-056]
- **Réplicas de leitura** — Escritas (INSERT/UPDATE/DELETE) vão a um único primário; réplicas somente leitura atendem pesquisas, com um pequeno atraso de replicação que costuma valer a pena. Relacionais nasceram para um servidor parrudo e escalam verticalmente; replicação bidirecional "é sempre complicado". [cap-056]
- **Fila na frente da escrita** — Sob pico de escrita (Black Friday), cada compra entra numa fila e um worker grava no banco no ritmo que ele aguenta: o total fica mais lento, mas nada se perde por timeout. [cap-055]
- **Busca textual é outro tipo de índice** — Busca parcial, por som ou por relevância é full-text search (Elasticsearch, Solr ou FTS do próprio Postgres/MySQL); para tirar carga do banco, o autor mantém um Elasticsearch separado e escreve nos dois. [cap-055, cap-056]
- **Arquivamento** — Tabelas de centenas de GB ou TB deixam índices lentos. Dados mortos (pedidos antigos, ex-funcionários) devem ser movidos periodicamente para outro banco ou backup frio (fita, Glacier); mover, não apagar, e manter backup por no mínimo 5 anos por causa da lei. [cap-056]
- **Big Data é outra categoria** — Empresas que geram gigabytes por dia (delivery, transporte, analytics, relógios, jogos online, exchanges) nunca têm um banco só: usam vários, de tipos diferentes por caso de uso, além de Data Warehouses, Data Lakes, Spark/Hadoop e uma equipe de infraestrutura dedicada. Não é território de micro empresa nem de freelancer. [cap-056]
- **Big Data = volume + velocidade de escrita** — Analytics, batimentos de relógio, telemetria de jogos e exchanges geram gigabytes por minuto e pedem agregações (média por hora, por dia). Ali a durabilidade de cada linha vira gargalo, e perder ou duplicar uma linha não importa; num extrato bancário, importa sempre. [cap-056]
- **NoSQL é quase sempre key-value** — Hash/dicionário distribuído: Mongo é key-value cujo valor é um documento (document store); Cassandra é wide column (key-value multidimensional com column families), feito para cluster multi-região com muita escrita. A maioria não é transacional nem analítica: as consultas precisam ser planejadas e indexadas antes. [cap-055]
- **Teorema CAP (Brewer, 2000)** — Um banco distribuído satisfaz no máximo duas de três: consistência, disponibilidade e tolerância a partição. Não é binário: com disponibilidade + tolerância ainda há consistência, só que eventual (réplicas divergem por segundos). Cassandra e Dynamo permitem ajustar o nível de consistência. [cap-056]
- **Escala vertical × distribuição** — Relacionais nasceram para um servidor parrudo e escalam com mais RAM e CPU. Distribuir a escrita entre vários nós exige abrir mão da consistência forte, e é aí que o NoSQL entra. [cap-056]
- **Replicação × particionamento (sharding)** — Replicar copia os mesmos dados para redundância e para dividir leitura; particionar divide dados diferentes entre servidores para dividir processamento e escrita. Mongo chama de replica sets e sharding. [cap-056]
- **Consistent hashing** — Distribuir por `id mod N` balanceia, mas mudar N reembaralha quase tudo. No anel de hashes (0 a 2^64; Cassandra usa Murmur3), cada servidor ocupa um ponto e cada chave vai para o próximo servidor no sentido horário; adicionar ou remover um nó só move os dados vizinhos, e múltiplos pontos por nó evitam desequilíbrio. [cap-056]
- **Espaço de números para ids** — Um inteiro de 32 bits vai de 0 a ~4 bilhões; um de 64 bits, a ~18 quintilhões. Um hash não criptográfico de 64 bits (o Murmur3 do Cassandra) basta para ids; um SHA-512 é exagero. [cap-056]
- **Chave de partição** — No DynamoDB, dados com a mesma chave de partição ficam juntos (com chave de ordenação opcional); consultar várias partições exige Scan, muito mais caro. A estratégia de partição tem de ser definida no início. [cap-056]
- **Custo de NoSQL gerenciado** — No exemplo do autor (calculadora AWS, 2023), o DynamoDB anuncia US$ 0,25 por GB/mês, mas um app pequeno com 100–400 escritas/s e capacidade reservada sai por ~US$ 26/mês mais ~US$ 180 adiantados; a cobrança é por escrita e leitura por segundo, pico, nível de consistência e reserva. O Firebase é gratuito até ~1 GB; o MongoDB Atlas, ~US$ 0,10 por milhão de leituras ou ~US$ 57/mês dedicado. Os números são datados; o princípio é que o preço sobe junto com o sucesso. [cap-056]
- **Firebase × MongoDB Atlas** — Os dois aceitam documentos JSON sem schema rígido e escalam e replicam sozinhos em várias regiões. O Firebase tem atualização em tempo real para mobile, mas modelo, queries e transações limitados; o Mongo tem documentos aninhados, queries mais ricas (joins, índices secundários) e mais suporte a transações. [cap-056]
- **Relacional gerenciado** — RDS (Postgres, MariaDB, MySQL), Google Cloud SQL, Azure SQL. Segundo o autor (2023), o Aurora tem mais disponibilidade e o dobro do desempenho no mesmo hardware, por um preço maior. Operar o próprio banco só compensa com terabytes diários de dados críticos. [cap-055, cap-056]
- **IDs não sequenciais em público** — Id auto-incremental na URL convida a enumerar outros usuários se algum controle de acesso falhar. Para ids expostos, use números aleatórios grandes (ex.: derivados de UUID) com constraint de unicidade; em tabelas internas tanto faz. [cap-056]
- **SQLite é embarcado** — Sem servidor: o programa abre o arquivo direto, roda com menos de 1 MB de RAM e está em celulares, roteadores, TVs e relógios. A amalgamation reúne o código num único `sqlite3.c` de ~240 mil linhas (~145 mil sem linhas em branco, 8,4 MB): é o banco SQL completo mais estudável. [cap-054, cap-056]
- **FaaS (Lambda, Cloud Functions)** — São containers pequenos e efêmeros, disparados por evento (HTTP, fila), que desligam quando ociosos; a implementação varia por fornecedor, não é padrão. Servem para trabalho pesado ou cargas com picos imprevisíveis e longos períodos parados (fila SQS → Lambda → banco; upload no S3 → processamento). Servir uma aplicação web inteira assim é, para o autor, o uso errado: quanto maior o app, mais demora para subir o container. [cap-055]
- **Gestão de risco** — Mais segurança, disponibilidade, durabilidade e integridade custam desempenho e dinheiro. A pergunta certa é quanto custa ficar dois dias fora numa Black Friday ou perder meio dia de dados financeiros. [cap-055]
- **Lock-in de nuvem** — Serviços como o Aurora (MySQL modificado, com as modificações fechadas) e o Lambda prendem a aplicação no "jardim murado" do fornecedor justamente quando o negócio dá certo; software como serviço troca propriedade por conveniência. [cap-055]
- **Monitoramento em produção** — Um APM com agente dentro da aplicação (ex.: New Relic) mostra, depois de cada deploy, se a memória estabiliza ou sobe devagar até o crash, se o consumo cresce na proporção do tráfego e quais trechos gastam mais. Depois que a aplicação está em produção, a manutenção se guia por métricas, medindo antes e depois. [cap-056]
- **Integridade e hardware** — Journaling (NTFS, ext4), WAL e copy-on-write (BTRFS) tornaram rara a perda de dados; hoje ela vem sobretudo de hardware ruim (cartão SD, SSD falsificado) e da falta de backup. [cap-056]

## Heurísticas de decisão
| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Começando um sistema web comum | Postgres ou MariaDB gerenciado (RDS/Cloud SQL) + Redis/Memcached + jobs assíncronos (Kafka/SQS) | Resolve "90% dos seus problemas" e é o que os frameworks esperam | cap-056 |
| Precisa de documentos JSON sem schema | Use o suporte a JSON do Postgres antes de adotar outro banco | Evita um segundo banco só por flexibilidade de schema | cap-056 |
| Só um protótipo descartável | Firebase serve | Barato e rápido; os limites aparecem com volume | cap-056 |
| Cogitando instalar e operar o banco por conta própria | Use banco gerenciado, a menos que tenha terabytes diários de dados críticos | O banco é a pior parte para manter; um sysadmin custa 5–10x a conta do serviço | cap-055 |
| Consulta lenta | Rode EXPLAIN e crie índice para as expressões realmente filtradas (composto se filtra junto) | A query mistura índice e table scan; o plano mostra qual | cap-054 |
| Tentação de indexar tudo | Crie o mínimo de índices que cobre a maioria das buscas | Cada índice multiplica o custo de INSERT/UPDATE/DELETE e ocupa espaço | cap-054, cap-056 |
| Precisa alterar muitas linhas | Um único UPDATE/DELETE com WHERE, executado no servidor | Uma conexão, um commit; o loop é ordens de grandeza mais lento | cap-056 |
| Código cliente com loop sobre linhas vindas do banco | Trate como red flag e reescreva como operação de conjunto | A melhor operação não traz dado nenhum para o cliente e resolve tudo no servidor | cap-056 |
| Relatório ou export grande | Selecione só as colunas necessárias, faça streaming e grave a saída incrementalmente | O uso de memória deve ter teto fixo, não crescer com o volume | cap-056 |
| Regras de integridade de dados | Constraints no schema + as mesmas validações no model/ORM e no front, com testes | Dado inconsistente é o pior problema; checar duas ou três vezes compensa | cap-056 |
| Join recorrente com dado que nunca muda (ex.: nome de estado) | Denormalize e garanta a consistência na aplicação (enums, validações) | Join custa; a chance de "São Paulo" mudar de nome é praticamente zero | cap-055 |
| Join caro e frequente em leitura | View materializada ou tabela denormalizada | Troca disco e redundância por desempenho | cap-055 |
| Leitura pesada e repetida | Cache-aside com expiração; agregações pré-calculadas no cache | Tira a carga de pesquisa do banco; o cache é reconstruível | cap-055, cap-056 |
| Leitura pesada que não cabe em cache | Réplicas de leitura; consultas pesadas vão para a réplica | Escrita fica num primário; réplicas escalam a leitura | cap-055, cap-056 |
| Pico de escrita (Black Friday) | Fila + worker escrevendo no ritmo do banco | Mais servidores de app no mesmo banco só aumentam a contenção | cap-055 |
| Achar que async/await resolve o pico | Não resolve: use filas | async ajuda, mas não elimina a contenção no banco | cap-055 |
| Busca por relevância ou similaridade | Elasticsearch (ou o FTS do banco) e escrita nos dois | É outro tipo de índice; tira processamento do banco | cap-055, cap-056 |
| Tabelas de centenas de GB / TB | Arquive dados mortos em outro banco ou backup frio, periodicamente | Índices crescem com a tabela até ficarem lentos | cap-056 |
| Dados onde perder ou duplicar uma linha é inaceitável (financeiro, fiscal, e-commerce) | Relacional com ACID | Consistência forte é requisito, não opção | cap-056 |
| Telemetria/analytics de gigabytes por minuto, tolerante a perda | NoSQL distribuído ou soluções de Big Data com consistência eventual | Velocidade de escrita importa mais que durabilidade de cada linha | cap-056 |
| Vai distribuir dados em vários nós | Defina a chave de partição/shard antes de ter dados | Mudar depois exige refazer a infraestrutura e mover tudo | cap-055, cap-056 |
| Escolhendo NoSQL gerenciado (Dynamo, Atlas, Firestore) | Simule o custo no pico e daqui a 6 meses, não só no começo | Cobra por operação/capacidade; fica mais caro justo quando faz sucesso | cap-056 |
| Id exposto em URL/API | Use id aleatório grande com constraint de unicidade | Id sequencial facilita enumerar outros registros | cap-056 |
| Dados locais num dispositivo ou app desktop | SQLite | Embarcado, minúsculo e confiável; não foi feito para ser servidor | cap-056 |
| Comunicação entre serviços seus | Protocolo binário (ex.: Protobuf) | Texto gasta espaço, banda e CPU com serialização | cap-054, cap-056 |
| Leu um post-mortem de big tech ("migramos de Postgres para MySQL") | Verifique se o problema existe na sua escala antes de agir | Limites citados pela Uber eram de casos extremos, na escala da Uber | cap-055 |
| Usar FaaS (Lambda) com banco | Como worker de fila (SQS → Lambda → banco), não para servir a aplicação web | Containers efêmeros têm cold start; é o equivalente a um worker | cap-055 |
| App mobile que precisa receber mudanças em tempo real | Firebase | Tem push em tempo real nativo; no Mongo isso custa desenvolvimento | cap-056 |
| Documentos com queries complexas, índices secundários ou transações | MongoDB Atlas antes do Firebase; se transação é central, relacional (RDS) | O Firebase limita queries e transações | cap-056 |
| Escrita em várias regiões geográficas ao mesmo tempo | Cassandra ou Dynamo; para empresa média, DynamoDB gerenciado | Foram feitos para multi-datacenter com consistência ajustável | cap-056 |
| Site público em produção | CDN/WAF na frente (o autor recomenda Cloudflare) | Cacheia assets e é a primeira linha de defesa contra ataques e DDoS | cap-055 |
| Decidir quanto investir em redundância e backup | Estime o custo de ficar fora e de perder dados | Durabilidade e disponibilidade são gestão de risco | cap-055 |
| Depois de cada deploy | Compare memória e latência antes e depois no APM | Memória subindo devagar até o crash denuncia problema antes do usuário | cap-056 |
| Projeto novo, sem usuários nem dados | Qualquer banco razoável serve; lance a 1.0 e decida com usuários, dados e receita reais | Só com uso real se descobre o que é preciso | cap-056 |
| Debate de bolha dev ou recomendação de influenciador | Vá à documentação oficial e aos fundamentos do problema | Nenhum influenciador conhece o seu cenário | cap-056 |

## Armadilhas e mitos
- "Persistir é só dar JSON.stringify no array e salvar." → Duplica a memória, incha mais de 40% e obriga a regravar tudo a cada mudança; um banco grava binário em páginas e altera só o que mudou. [cap-054]
- "Mongo guarda JSON, então JSON deve ser eficiente." → JSON é a interface; o armazenamento é BSON binário em B+Tree. A linguagem ou o formato que você usa por cima não diz nada sobre a arquitetura por baixo. [cap-056]
- "Existe banco SQL e banco NoSQL como categorias de armazenamento." → SQL é só uma linguagem que o banco entende; o protótipo do autor fala SQL sobre um array em memória. [cap-054, cap-056]
- "Índice deixa tudo mais rápido, então o banco deveria indexar todas as colunas." → Índice acelera leitura e desacelera toda escrita, além de ocupar espaço (às vezes o dobro da tabela). [cap-054, cap-056]
- "No meu ambiente de dev é rápido, então está bom." → Com 10, 100 ou mil linhas nada faz diferença; o impacto só aparece com centenas de milhares ou milhões, em produção. [cap-054, cap-056]
- "O ORM vai gerar queries eficientes." → Nenhum ORM faz isso por você; queries ineficientes estão entre os maiores desperdícios de memória e desempenho, e trocar de banco não conserta código ruim. [cap-055, cap-056]
- "Validar no front-end basta." → Banco que aceita tudo vive inconsistente; constraints no banco e validação repetida nas camadas de cima. [cap-056]
- "NoSQL é mais fácil de configurar e manter." → Uma infraestrutura NoSQL de verdade (sem ponto único de falha, recuperação sem downtime) dá tanto ou mais trabalho que um relacional; nenhum banco é "seguir o tutorial". [cap-055]
- "MongoDB é bala de prata de facilidade e desempenho." → O ponto ótimo é o dataset caber em RAM; em 2019 o "ok" padrão significava "recebi, gravo quando der", e exigir confirmação de réplica custa o desempenho que motivou a escolha. Desde o MongoDB 5.0 o write concern padrão passou a ser `majority` (complemento, não está no compêndio). Sharding precisa ser planejado do início. [cap-055]
- "Cassandra serve para o banco do meu blog." → Foi feito para cluster com muitos nós, de preferência em várias regiões, e muita escrita; numa instância só, não faz sentido. [cap-055]
- "NoSQL substitui joins e relacionamentos." → NoSQL não foi feito para joins; granular demais e emular joins é o jeito errado de usá-lo. Em shards, joins entre servidores são lentos. [cap-055]
- "Normalizar ao máximo é sempre o correto." → É mais bonito na modelagem, mas pode piorar o desempenho; denormalizar dados que não mudam é um trade-off legítimo. [cap-055]
- "CAP: escolha exatamente duas e perca a terceira por inteiro." → Não é binário (há consistência eventual), os termos são ambíguos e partições de rede são raras hoje: otimize para o caso comum e degrade com elegância quando acontecer. [cap-056]
- "Distribuir por `id mod N` resolve o balanceamento." → Funciona até precisar mudar N; aí quase tudo muda de servidor. Por isso existe consistent hashing. [cap-056]
- "Serviço gerenciado é barato." → O preço por GB engana: paga-se por escrita e leitura por segundo, pico, reserva e transferência; quanto mais sucesso, maior a conta, e há lock-in (Aurora e Lambda prendem você na AWS). [cap-055, cap-056]
- "Ids auto-incrementais em URL não têm problema." → Basta um controle de acesso esquecido para alguém enumerar ids e abrir contas alheias; não dê essa informação de graça. [cap-056]
- "Para usar um banco é preciso saber em detalhes como ele foi feito." → Não; basta um modelo mental mínimo (tabela, índice em árvore, log, servidor) para ele deixar de ser caixa-preta mágica. [cap-054]
- "Dá para fazer um banco de verdade no fim de semana." → O protótipo levou 2–3 dias e não tem persistência em árvore, cache, locks nem limite de conexões; bancos reais acumulam mais de 60 anos de pesquisa e milhares de horas de trabalho. [cap-054]
- "Async/await resolve picos no banco." → Ajuda, mas o que segura a Black Friday são filas controláveis. [cap-055]
- "FaaS é uma revolução para hospedar aplicações web." → São containers efêmeros, algo como um worker dyno do Heroku; para app web grande, a latência de subir o container é o preço. [cap-055]
- "Postgres ou MySQL gerenciado na nuvem é software livre." → As modificações do Aurora são fechadas, e serviços como o Lambda prendem você ao fornecedor. [cap-055]
- "Banco moderno não perde dados, então backup é detalhe." → Journaling e WAL protegem contra queda de luz, não contra hardware ruim, erro humano ou exigência legal; para empresa pequena, ter um backup que funciona é o primeiro passo. [cap-056]
- "Banco exótico resolve meu problema." → Exótico também significa profissionais mais raros e caros de contratar ou treinar; muitas vezes o hype encarece o desenvolvimento. [cap-056]

## Checklist para aplicar num sistema real
- [ ] O banco escolhido corresponde ao problema (consistência forte? volume e velocidade de escrita? dados locais?) e não ao hype ou a um post de big tech?
- [ ] Existe um motivo concreto para não usar um relacional gerenciado com cache e fila? Se a resposta for "escala", ela está medida?
- [ ] O banco é gerenciado ou há gente qualificada, backup testado e plano de recuperação para operá-lo?
- [ ] Existe backup que funciona (restauração testada)? Há política de retenção e arquivamento de dados antigos?
- [ ] Cada consulta frequente tem índice adequado (composto quando filtra várias colunas juntas)? O EXPLAIN foi conferido nas consultas críticas?
- [ ] Há índices sem uso pesando nas escritas? Quantos índices tem cada tabela de escrita intensa?
- [ ] As consultas trazem só as colunas e linhas necessárias (nada de `SELECT *` sem motivo)?
- [ ] As consultas foram testadas com volume parecido com o de produção (centenas de milhares ou milhões de linhas), e não só com dados de desenvolvimento?
- [ ] Algum código faz loop sobre linhas do banco emitindo um comando por linha (N+1, UPDATE em loop)? Dá para trocar por uma operação de conjunto no servidor?
- [ ] Relatórios e exports usam streaming e escrita incremental, com teto de memória independente do volume?
- [ ] As regras de integridade estão no schema (NOT NULL, unicidade, FK, tipos, tamanhos) e repetidas no model e no front, com testes?
- [ ] Onde houve denormalização, quem garante a consistência na aplicação (enums, validações)?
- [ ] O ORM está gerando SQL razoável? Alguém olhou o SQL gerado das rotas mais usadas?
- [ ] As escritas críticas confirmam durabilidade de verdade (transação, write concern/replicação) antes de dizer "ok" ao usuário?
- [ ] Operações que precisam ser tudo-ou-nada estão dentro de uma transação?
- [ ] Leituras repetidas passam por cache com expiração? Dá para reconstruir o cache do zero?
- [ ] Consultas pesadas ou de relatório vão para uma réplica de leitura, e o atraso de replicação é aceitável para elas?
- [ ] Picos de escrita passam por fila, em vez de abrir mais conexões no mesmo banco?
- [ ] Busca textual por relevância está num índice próprio (FTS/Elasticsearch), não em `LIKE` sobre a tabela principal?
- [ ] Se há distribuição/sharding, a chave de partição foi definida antes e as consultas principais não exigem varrer todas as partições?
- [ ] O nível de consistência (forte ou eventual) de cada dado foi decidido conscientemente (saldo ≠ métrica de clique)?
- [ ] Ids expostos publicamente não são sequenciais e o acesso é sempre checado contra a sessão?
- [ ] Há monitoramento (APM) para medir consumo e latência antes e depois de cada mudança?
- [ ] O custo do banco gerenciado foi projetado para o pico e para o cenário de sucesso, e o lock-in foi considerado?
- [ ] A comunicação entre serviços internos evita serializar texto à toa?
- [ ] Alguém estimou quanto custa ficar fora do ar e perder um dia de dados, e a redundância é proporcional a isso?
- [ ] Há CDN/WAF na frente do site público?
- [ ] Se há FaaS, ele consome eventos e filas, em vez de servir a aplicação web?
- [ ] O lock-in em serviços proprietários do fornecedor foi uma decisão consciente?
- [ ] A escolha de um banco menos comum considerou o custo de contratar ou treinar quem saiba operá-lo?

## Frases-âncora
> "Índices é um trade-off de deixar operações de pesquisa muito mais rápidas ao custo de deixar operações de escrita muito mais lentos." — cap-054

> "JSON é um formato de consumo, não um formato de armazenamento." — cap-056

> "Não existe um banco de dados "SQL", existem bancos de dados que entendem a linguagem SQL, só isso." — cap-056

> "A melhor operação trás zero dados do servidor e opera tudo lá." — cap-056

> "Pelo menos lembre disso como regra: se tem um loop, está errado, pense de novo." — cap-056

> "Em SQL, se tem um monte de IFs, provavelmente deveriam ser constraints." — cap-056

> "O grande gargalo numa aplicação web é o banco de dados, porque mesmo com replicação e sharding, você sempre vai ter contenção por causa das garantias ACID." — cap-055

> "Tudo que você não sabe como realmente funciona vira superstição." — cap-056

## Onde aprofundar
| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-054 | [Akitando] #118 - Fiz um servidor de "SQL"?? \| Entendendo Banco de Dados (09/05/2022) | https://www.akitaonrails.com/2022/05/09/akitando-118-fiz-um-servidor-de-sql-entendendo-banco-de-dados/ | precisar explicar (ou entender) o que há dentro de um banco: parser de SQL, índice em árvore com benchmark, custo de índice na escrita, transaction log, por que não persistir em JSON, B-Tree e disco em blocos. |
| cap-055 | [Akitando] #49 - Devo usar NOSQL? \| "ENDGAME" para Iniciantes em Programação (24/04/2019) | https://www.akitaonrails.com/2019/04/24/akitando-49-devo-usar-nosql-endgame-para-iniciantes-em-programacao-serie-comecando-aos-40/ | estiver decidindo entre relacional e NoSQL, desenhando cache, fila de escrita, denormalização ou busca textual, ou avaliando banco gerenciado, FaaS e lock-in. Datado (2019): preços e defaults citados mudaram. |
| cap-056 | [Akitando] #143 - Discutindo sobre Banco de Dados - Dos primórdios a Big Data (22/07/2023) | https://www.akitaonrails.com/2023/07/22/akitando-143-discutindo-sobre-banco-de-dados-dos-primordios-a-big-data/ | precisar da visão histórica (DBF → SQLite → cliente-servidor → NoSQL), de WAL e ACID, de operar no servidor em vez de em loop, constraints, réplicas, arquivamento, CAP, consistent hashing, particionamento no Dynamo ou custo de bancos como serviço. |
| cap-027 | [Akitando] #95 - Árvores: O Começo de TUDO (06/04/2021) | https://www.akitaonrails.com/2021/04/06/akitando-95-arvores-o-comeco-de-tudo/ | quiser a base de árvore binária, balanceamento, AVL/Red-Black e B/B+Tree, que os três capítulos acima pressupõem. |
