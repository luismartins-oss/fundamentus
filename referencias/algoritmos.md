# Algoritmos e estruturas de dados
> Destilado de: cap-027, cap-028, cap-029 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).

## Ideia central
Para o autor, memória e arquivos são uma fita de bits, e só a estrutura escolhida pelo programador dá sentido a eles: a mesma informação pode virar array, lista ligada, hashtable, árvore ou grafo, e cada forma troca custo de inserção, busca, memória e manutenção de um jeito diferente. Árvores estão por baixo de índices, filesystems, compiladores e compressão; grafos, por baixo de redes, recomendação e Git. Um agente acerta os fundamentos quando escolhe a estrutura pelo padrão de acesso real (mais escrita ou leitura? precisa de ordem? cabe em memória? que volume?) e aceita que tudo é troca entre CPU, memória, espaço e precisão.

## Conceitos que todo sistema depende

### Representação e custo
- **Bits só ganham sentido pela estrutura** — Bits sozinhos não significam nada; é o programador que decide que um trecho é inteiro, string ou um nó com ponteiro para outro endereço. Navegar entre nós custa trabalho, e algoritmos de ordenação e procura existem para minimizar esse trabalho. [cap-027]
- **Tipo abstrato vs estrutura concreta** — Pilha (push/pop), fila (enqueue/dequeue) e fila de prioridade são conceitos que podem ser implementados com array, lista ligada ou árvore (heap binária). Separar o "o quê" do "como" permite trocar a implementação. [cap-027]
- **Complexidade como bússola** — Particionar o problema ao meio repetidamente é a assinatura do logarítmico; ordenar por comparação (Quicksort, Mergesort) é loglinear; caixeiro viajante por força bruta é fatorial. Casos excepcionais podem derrubar um algoritmo O(log n) para O(n). [cap-027]
- **Espaço também é custo** — Mergesort gasta memória temporária extra para ordenar; a árvore quase não gasta espaço adicional para manter ordem. Avaliar só velocidade é metade da conta. [cap-027]
- **Inserir barato vs deixar pronto para buscar** — Inserir no fim de um array é O(1) (ou O(n) se precisar realocar e copiar), mas a lista fica desordenada e ordenar custa O(n log n). Uma árvore de busca binária insere em O(log n) já ordenado. [cap-027]
- **Busca binária** — Só funciona em dado ordenado: compara com o meio e descarta metade a cada passo. No exemplo do autor, 4 operações contra 10 da varredura ingênua, e a economia cresce com o tamanho. [cap-027]
- **Função de hash para hashtable** — O ideal é saída com distribuição uniforme (não gaussiana), o mais próxima de aleatória possível, com poucas colisões; djb2 é boa padrão e CRC32 é bom mas exige tabela de 1 KB. Hashes criptográficos (SHA-512) custam mais e não são a escolha eficiente sem requisito de segurança. [cap-027]

### Árvores
- **Árvore de busca binária (BST)** — Cada nó aponta para esquerda (menores) e direita (maiores ou iguais); o percurso in-order devolve a lista ordenada. Busca e inserção são proporcionais à altura da árvore, não ao total de elementos. [cap-027]
- **Desbalanceamento** — Inserir dados já em sequência (100, 101, 102...) numa BST ingênua gera um "galho seco": na prática uma lista ligada, com busca O(n). [cap-027]
- **Red-Black** — Um bit de cor por nó; raiz preta, nó vermelho não tem pai nem filho vermelho, todo caminho até o nó nulo tem o mesmo número de pretos. Recolorir e rotacionar (rotação custa O(1)) mantém a árvore balanceada e as operações em O(log n). [cap-027]
- **AVL** — Cada nó guarda um fator de balanço −1, 0 ou 1 e a árvore é mantida mais estritamente balanceada que a Red-Black. Busca um pouco mais rápida; inserção mais cara, com rotações mais complicadas e tempo mais variável. [cap-027]
- **Onde Red-Black aparece** — `std::map` do C++, `TreeMap`/`TreeSet` do Java e o kernel Linux (escalonadores de I/O, timers de alta resolução, áreas de memória virtual, descritores do epoll). [cap-027]
- **B-Tree e B+Tree** — Árvores auto-balanceadas com muitos filhos por nó; na B+Tree os nós internos guardam só chaves e as folhas guardam os valores, ideal para armazenamento orientado a blocos. Segundo o autor, filesystems (EXT4, NTFS, APFS) e bancos relacionais (DB2, SQL Server, Oracle, SQLite) usam B+Tree ou derivados. [cap-027]
- **Árvore é uma lista ordenada não linear** — A imagem de hierarquia engana: o objetivo é uma lista ordenada mais barata de buscar, inserir e apagar do que array ou lista ligada. [cap-027]
- **Árvores em todo lugar** — Filesystem (diretórios), DOM do HTML, hierarquia de classes, a AST que compiladores e interpretadores geram do código e a codificação de Huffman. [cap-027, cap-029]
- **Coloração fora das árvores** — O tri-color marking de garbage collectors usa a mesma ideia de cores para categorizar o que pode ser descartado. [cap-027]

### Índices de banco de dados
- **Índice é outra árvore** — Sem índice a consulta cai em table scan, caçando registro a registro no disco. Com índice (B-Tree), ela percorre a árvore compacta dos campos indexados e busca só os registros necessários. [cap-027]
- **Cada índice tem preço** — Cada índice é mais uma árvore a rebalancear em todo insert, update e delete. Acelera leitura e encarece escrita. [cap-027]

### Grafos
- **Grafo** — Nós ligados por arestas (edges), com ou sem direção, representáveis como matriz ou lista de adjacência. Redes de computadores, redes sociais, a Web, logística e o Git (um DAG, grafo acíclico dirigido) são grafos; a árvore é um caso particular. [cap-027, cap-028]
- **Navegação e estrutura** — Menor caminho (Dijkstra) é o "hello world" de grafos; nós muito conectados são hubs ("influencers"). Modelos estatísticos de redes (Erdős-Rényi, Watts-Strogatz, Barabási-Albert, mundos pequenos) explicam como as conexões reais se distribuem. [cap-027, cap-028]
- **PageRank** — A nota é a probabilidade de um navegador aleatório cair na página, contando quem linka para ela. É um ranking global, vulnerável a fazendas de links (daí o TrustRank) e que confunde popularidade com autoridade. [cap-028]
- **HITS** — Dá duas notas por página, de hub (linka para muitos) e de autoridade (recebe links de muitos hubs), por tópico. Mais justo que o PageRank global e mais caro de computar. [cap-028]
- **SALSA e random walk** — Em vez de percorrer e pontuar o grafo inteiro, amostra com caminhadas aleatórias sobre um grafo bipartido (hubs de um lado, autoridades do outro), como uma pesquisa de opinião. Foi a base do "Who To Follow" do Twitter. [cap-028]
- **Algoritmos de grafo supõem memória e iteração** — A maioria assume o grafo inteiro em RAM e é iterativa; distribuí-la em Hadoop/MapReduce exige reembaralhar dados entre servidores (shuffling). O Twitter contornou com PageRank personalizado de uma passada, particionamento por ID, cache de intermediários e compressão do grafo. [cap-028]
- **Quanto cabe em memória** — O paper citado estima que 10 bilhões de arestas, numa representação ingênua, ocupam uns 80 GB, o que um único servidor aguenta. A pergunta de projeto do Twitter era se os dados cresceriam mais devagar que o barateamento da memória. [cap-028]
- **Especializar com janela de tempo** — O GraphJet mantém em memória só as interações das últimas n horas, num grafo bipartido com API mínima: expira em vez de apagar, timestamp grosseiro e nada de banco de grafos genérico. Chegou a recomendações abaixo de 10 ms, com o gargalo nas filas (Kafka). [cap-028]
- **Relevância por álgebra linear** — Documentos e consultas viram vetores de contagem de termos (Vector Space Model) e o produto escalar ou a similaridade de cosseno mede relevância. É a base do Lucene/Elastic e dos embeddings. [cap-028]
- **Vizinho mais próximo aproximado (ANN)** — Em espaços de muitas dimensões a solução exata é inviável; LSH e KD-trees trocam um pouco de precisão por muita performance. É a mesma lógica da amostragem do SALSA. [cap-028]
- **Pipeline de recomendação** — Seleção de ~1500 candidatos (metade de quem você segue, metade de fora), ranker leve, ranker pesado (rede neural de ~48 milhões de parâmetros) e por fim heurísticas e filtros. Recomendação é probabilidade, e por isso exige estatística, álgebra linear e cálculo. [cap-028]

### Dados distribuídos e serialização
- **ID com propósito** — Autoincremento tem problema de segurança e não particiona; UUID aleatório evita colisão mas ocupa espaço sem servir a nada. Snowflake (timestamp + servidor + sequência), TimeUUID (Cassandra) e chave de partição + chave de ordenação (DynamoDB) embutem tempo e localização no ID. [cap-028]
- **Consistent hashing** — Mapeia chaves de partição em nós virtuais num anel. Quando servidores entram ou saem, só os ranges afetados são redistribuídos, mantendo a distribuição uniforme. [cap-028]
- **MapReduce** — Roda o algoritmo em cada partição dos dados e depois reduz os resultados parciais, o que acrescenta uma etapa extra de redução. Para o autor, gigabytes por dia ou terabytes por mês já são território de Big Data, em que particionar é a única saída. [cap-028]
- **Serialização com schema** — JSON é texto e leva o schema em cada registro; Protobuf, Thrift e Avro definem os tipos fora e trafegam binário compacto, mais barato de parsear (no exemplo, quase 200 bytes contra menos de 80). A comunicação entre serviços costuma ir por RPC (gRPC; Finagle no Twitter). [cap-028]

### Compressão
- **Lossless vs lossy** — Lossless reconstrói o original exato (RLE, Huffman, LZ77, Deflate, PNG, FLAC, ProRes/DNxHR). Lossy joga fora o que os sentidos não percebem (chroma subsampling, quantização do JPEG, H.264/H.265, MP3/AAC) e não volta. [cap-029]
- **Run-Length Encoding** — Substitui sequências repetidas por símbolo + contagem ("FFFFF" vira "F5"). O pior caso é entrada de alta entropia, quase sem repetição. [cap-029]
- **Huffman** — Árvore binária de frequências: símbolos mais frequentes ficam perto da raiz e ganham códigos mais curtos. Como a árvore precisa ir junto, não compensa em textos curtos; em textos longos o autor fala em reduzir até umas 5 vezes. [cap-029]
- **LZ77 e janela deslizante** — Repetições viram ponteiros (quantos caracteres voltar, quantos ler) dentro de uma janela em memória. Janela maior comprime mais, mas gasta mais RAM e exige ponteiros maiores; Deflate (gzip, zip, PNG) é LZ77 + Huffman. [cap-029]
- **Formato não é algoritmo** — zip, rar e 7z são formatos de arquivo e metadados. Por dentro usam variações de Lempel-Ziv (LZSS, LZMA) e Huffman, ajustadas de jeitos diferentes. [cap-029]
- **Mudar a representação** — RGB para YUV é conversão linear sem perda, mas separa luma (que o olho percebe mais) de chroma. Isso permite reduzir chroma (4:2:0) com ~40% menos dados sem diferença perceptível, exceto em detalhes finos como texto pequeno. [cap-029]
- **Pipeline do JPEG** — Blocos 8×8, DCT, quantização (o slider de qualidade escolhe a tabela que divide os coeficientes e zera as altas frequências), zigue-zague, RLE e Huffman. Resultado na faixa de 20:1. [cap-029]
- **Vídeo é quadro completo + diferenças** — Codecs como H.264 gravam um quadro inteiro e depois só os deltas, como commits de Git. Cenário parado comprime muito, câmera tremida comprime pouco, por isso o tamanho do arquivo não é linear com a duração. [cap-029]
- **Transformadas** — DFT/FFT convertem sinal contínuo em frequências discretas; a FFT derrubou o custo de quadrático para n log n. A DCT (só números reais) é a variante usada em imagem. [cap-029]
- **Amostragem** — O mundo analógico não cabe inteiro em bits: digitalizar é escolher samples (pixels, quadros por segundo, samples de áudio como 44,1 kHz/16 bits do CD). Resolução e frequência definem fidelidade e tamanho antes de qualquer compressão. [cap-029]
- **Formato de consumo vs formato de edição** — H.264/H.265, JPEG e MP3/AAC são feitos para baixar e assistir, com chroma reduzido e deltas entre quadros. ProRes, DNxHR, RAW e FLAC preservam informação e quadros completos para editar, ao custo de arquivos ordens de grandeza maiores (DNxHR na faixa de 1 GB por minuto). [cap-029]

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Poucos elementos (ex.: campos de um formulário) | Qualquer lista serve; não otimize | Com pouco dado, até força bruta tem diferença irrelevante | cap-027 |
| Precisa de dados ordenados com inserções frequentes | Árvore balanceada em vez de array + reordenar | Insere já ordenado em O(log n); reordenar array custa O(n log n) | cap-027 |
| Mais inserções que buscas | Red-Black | Inserção mais barata, rotações simples | cap-027 |
| Busca crítica, inserção rara | AVL | Mais estritamente balanceada, busca um pouco mais rápida | cap-027 |
| Na dúvida entre Red-Black e AVL | Red-Black como padrão | AVL só quando você sabe que precisa das propriedades dela | cap-027 |
| BST alimentada com dados quase em ordem | Use árvore auto-balanceada | BST ingênua degenera em lista ligada, O(n) | cap-027 |
| Consulta lenta com table scan | Indexe o campo filtrado | Índice é árvore que evita varrer registro a registro | cap-027 |
| Tabela com muita escrita | Não crie índice para tudo; só os que consultas reais usam | Cada índice é rebalanceado em todo insert/update/delete | cap-027 |
| Chave de hashtable sem requisito de segurança | Hash rápido e uniforme (djb2, CRC32) | Hash criptográfico gasta CPU sem benefício | cap-027 |
| Ordenar dados que não cabem na RAM | Abordagem do Mergesort (ler e gravar sequencialmente) | Ele nasceu para listas em fita, sem estourar a RAM | cap-028 |
| Relacionamentos massivos navegados em tempo real | Estrutura especializada em memória, não SQL relacional | O autor diz que é um dos poucos cenários onde relacional não é adequado | cap-028 |
| Dados que perdem valor rápido (feed, cache) | Janela de tempo + expiração em vez de deleção | Sub-grafo menor, código mais simples | cap-028 |
| Primeira versão de um produto | Comece simples (MySQL, cálculo em batch) e especialize só com experiência | O Twitter teve 3 gerações de recomendação; o GraphJet só veio quando sabiam o que precisavam | cap-028 |
| Dados demais para uma máquina | Particione e use map/reduce, aceitando a etapa extra de redução | Dividir para conquistar | cap-028 |
| ID em sistema distribuído | ID semi-aleatório com tempo e origem (Snowflake, TimeUUID) | Particiona, ordena por tempo, colisão perto de zero | cap-028 |
| Servidores entrando e saindo do cluster | Consistent hashing com nós virtuais | Redistribui só os ranges afetados | cap-028 |
| API para humanos ou integração pequena | JSON | Legível; "Pro ecommerce da esquina, de fato, tanto faz" | cap-028 |
| Tráfego interno entre serviços em alto volume | Protobuf/gRPC (ou Avro em streaming) | Binário menor e parsing mais barato | cap-028 |
| Busca exata inviável (muitas dimensões, grafo enorme) | ANN (LSH, KD-tree) ou amostragem (random walk) | Troca um pouco de precisão por ordens de grandeza de performance | cap-028 |
| Escolher modelo estatístico ou de ML | Entenda a matemática e a distribuição dos dados antes | Regressão linear onde não cabe dá resultado errado | cap-028 |
| Servir páginas web | Ative gzip/deflate | "Se não vem, deveria." | cap-029 |
| Arquivo para editar ou guardar como master | Formato lossless (RAW, PNG, ProRes/DNxHR) | Lossy perde a cada salvamento e tira informação de cor | cap-029 |
| Arquivo para distribuir e consumir | Lossy (JPEG, H.264/H.265) | 20:1 ou mais sem perda perceptível | cap-029 |
| Imagem com traço nítido, texto ou transparência | PNG | JPEG borra bordas e cria blocos | cap-029 |
| Editar vídeo | Codec só de quadros completos, mesmo com arquivo maior | Reconstruir frames a partir de deltas custa CPU e RAM; disco é barato | cap-029 |
| Quer arquivo menor | Aceite mais CPU e RAM (HEVC ~50% menor que H.264, com bem mais processamento) | Compressão troca espaço por processamento | cap-029 |
| Streaming | Bitrate abaixo da banda: baixar tem de levar menos tempo que assistir | Pré-requisito para tocar enquanto baixa | cap-029 |
| Compressão transparente do disco (ex.: NTFS) | O autor não recomenda | Overhead em toda leitura e escrita para economizar pouco; disco é barato | cap-029 |
| Antes de escolher a técnica | Faça a conta de padeiro em ordens de grandeza | 4K cru daria ~5 TB em 2 h; a conta mostra que compressão é obrigatória | cap-029 |

## Armadilhas e mitos
- "Meu programa tá lento, deve ser o CPU velho" → Quase sempre é estrutura ou algoritmo errado; programas antigos rodavam mais rápido em hardware pior. [cap-027]
- "Inserir em array é mais rápido que em árvore, logo array é melhor" → Comparação injusta: a árvore entrega ordenado, o array ainda teria de ser ordenado depois. [cap-027]
- "Hash bom tem distribuição gaussiana" → Tem de ser uniforme; gaussiana concentra resultados em poucos valores e enche alguns buckets. [cap-027]
- "SHA-512 é melhor para hashtable porque é mais seguro" → É desenhado para segurança e custa mais; sem requisito de segurança, não é o mais eficiente. [cap-027]
- "Fatorial recursivo tem complexidade fatorial" → É linear; o exemplo de complexidade fatorial é o caixeiro viajante por força bruta. [cap-027]
- "B-Tree é árvore binária" → Não; cada nó tem vários filhos. [cap-027]
- "Mais índices deixam o banco mais rápido" → Deixam a leitura mais rápida e a escrita mais lenta. [cap-027]
- "O heap da memória é a estrutura de dados heap" → cap-027 aproxima os dois, mas o segmento de alocação dinâmica e a fila de prioridade só compartilham o nome; alocadores organizam a memória em arenas, bins e chunks (ver linguagens.md, cap-037). (complemento, não está no compêndio)
- "Árvore sempre busca melhor que hashtable" → Para chave exata, hashtable bem distribuída tem busca O(1) em média; a árvore ganha quando é preciso ordem, intervalo ou pior caso garantido. (complemento, não está no compêndio)
- "Li o trecho de código (ou o README) e sei como o sistema se comporta" → Sem build, configs de produção, dados de treino e testes, a conclusão é inconclusiva. No código do Twitter, pesos citados em posts só aparecem no README, e uma chamada com `applyBoosts` falso pode anular os boosts; rastreie quem chama o quê. [cap-028]
- "A chave `author_is_elon` prova manipulação" → Era um contador de métricas mandado para log; concluir sem seguir o fluxo é bikeshedding. [cap-028]
- "Clone de tutorial equivale ao sistema real" → É caixa de sapato frente a um Burj Khalifa; a escala muda a solução e nenhum curso cobre isso. [cap-028]
- "Rails não escala" (caso Twitter, 2007–2008) → O problema era particionar dados e processar grafos sem as ferramentas que existem hoje; nada em Python ou PHP escalaria também. [cap-028]
- "Programar não exige matemática" → Coisa simples não exige; grafos, recomendação e ML exigem álgebra linear, cálculo, estatística e probabilidade. [cap-028]
- "JSON serve para tudo" → Para tráfego interno de alto volume, desperdiça banda e processamento. [cap-028]
- "Qualidade 80% do JPEG guarda 80% dos detalhes" → O número só escolhe a tabela de quantização. [cap-029]
- "Converter 4:2:0 de volta a RGB recupera as cores" → Só duplica as amostras que sobraram; o chroma perdido não volta. [cap-029]
- "Reeditar e salvar o mesmo JPEG não tem problema" → Cada salvamento perde mais definição; edite a partir de RAW ou lossless. [cap-029]
- "Trocar tudo para WebP compensa" → Opinião do autor em 2022: só ~25% melhor que PNG e adoção baixa; reavalie com dados atuais. [cap-029]
- Terminologia de quadros de vídeo → cap-029 chama o quadro completo de "interframe" e os deltas de "intraframes"; na nomenclatura padrão é o contrário (quadro completo = I-frame, intra; deltas = inter-frames P/B). (complemento, não está no compêndio)

## Checklist para aplicar num sistema real
- [ ] Qual o padrão de acesso dominante (inserção, busca por chave exata, busca por intervalo, percurso ordenado)? A estrutura escolhida combina com ele?
- [ ] Fiz a conta de padeiro do volume (itens, bytes, taxa por segundo)? Volume pequeno pede simplicidade.
- [ ] Existe caso degenerado (dados já ordenados numa BST, muitas colisões de hash, entrada de alta entropia num RLE)?
- [ ] Contei a memória extra (buffers do Mergesort, janela do LZ77, cópias intermediárias), e não só CPU?
- [ ] Os índices do banco cobrem as consultas reais? Cada índice extra se paga frente ao custo de escrita?
- [ ] Alguma consulta faz table scan em tabela grande?
- [ ] Os dados cabem na memória de uma máquina? Se não, como particionar e quanto shuffle o algoritmo exige?
- [ ] Dados com validade curta usam janela e expiração em vez de deleção?
- [ ] IDs servem a particionamento e ordenação temporal? Autoincremento está exposto ou limitando a distribuição?
- [ ] O formato de serialização entre serviços é proporcional ao volume (JSON para humanos, binário para tráfego interno pesado)?
- [ ] Respostas HTTP saem comprimidas?
- [ ] Mídia: master e edição em lossless, distribuição em lossy, nunca uma cadeia de recompressões lossy?
- [ ] Estou reimplementando o que a biblioteca padrão já tem (árvore ordenada, gzip, hash)?
- [ ] Antes de afirmar algo sobre código alheio, rastreei chamadores, flags e configuração? Tenho testes ou dados para confirmar?
- [ ] Em estatística ou ML, conheço as premissas do modelo e a distribuição dos dados?
- [ ] A primeira versão está simples o bastante, deixando a especialização para quando houver experiência com o problema?

## Frases-âncora
> "Índices são bons pra acelerar as pesquisas, mas o custo é aumentar o preço de operações de inserts, updates e deletes." — cap-027

> "Se você fizer mais inserções do que procuras, um Red Black é melhor. Se o tempo de inserção não for importante, mas pra você o tempo de procura for muito importante, então talvez AVLs sejam melhores." — cap-027

> "Um ID é muito mais do que só uma etiqueta qualquer. Ela serve pra facilitar pesquisas e pra facilitar balanceamento de sistemas distribuídos." — cap-028

> "Não pense em como apagar dados velhos, estabeleça uma janela de tempo e deixa expirar sozinho, que é boa prática de qualquer tipo de caching." — cap-028

> "Nem tudo que parece uma reta, é uma reta." — cap-028

> "Código de produção de verdade, de um produto que tem uso, nunca acaba." — cap-028

> "Huffman é um bom exemplo de porque todo programador precisa saber o que é uma árvore." — cap-029

> "Mas em geral, quanto menor consegue fazer o arquivo ficar, mais processamento e memória vai gastar." — cap-029

## Onde aprofundar

| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-027 | [Akitando] #95 - Árvores: O Começo de TUDO \| Estruturas de Dados e Algoritmos (06/04/2021) | https://www.akitaonrails.com/2021/04/06/akitando-95-arvores-o-comeco-de-tudo/ | precisar escolher entre array, hashtable e árvore, ver BST/Red-Black em código C ou entender por que índice acelera leitura e encarece escrita |
| cap-028 | [Akitando #140] Desbloqueando o "Algoritmo" do Twitter - Introdução a Grafos (11/04/2023) | https://www.akitaonrails.com/2023/04/11/akitando-140-desbloqueando-o-algoritmo-do-twitter-introducao-a-grafos/ | projetar recomendação ou ranking, processar grafos grandes, escolher IDs distribuídos, consistent hashing ou serialização binária, ou ler com rigor código de produção grande e incompleto |
| cap-029 | [Akitando] #116 - De 5 Tera a 25 Giga \| Compressão de Dados e Multimídia (23/03/2022) | https://www.akitaonrails.com/2022/03/23/akitando-116-de-5-tera-a-25-giga-compressao-de-dados-e-multimidia/ | lidar com compressão (gzip/deflate), formatos de imagem e vídeo, streaming, escolha lossy vs lossless ou implementar RLE, Huffman ou LZ77 |
