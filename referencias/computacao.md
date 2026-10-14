# Como o computador funciona

> Destilado de: cap-014, cap-015, cap-016, cap-017, cap-018, cap-019, cap-020, cap-021 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).

## Ideia central

Para o autor, todo sistema se reduz a uma fita de bits endereçada onde instruções e dados convivem sem nenhuma diferença física, lida e escrita por uma CPU que só move bytes entre registradores e endereços, um pulso de clock por vez. Linguagens, objetos, arrays que "crescem sozinhos", hashes e garbage collectors são camadas em cima disso, e cada camada cobra em ciclos, memória ou pausas: "mágica" é só um custo que você não está vendo. Acertar os fundamentos é saber qual representação, qual região de memória e qual algoritmo estão por baixo de cada linha, porque é ali que nascem os bugs de precisão, os estouros, os vazamentos e a lentidão que só aparecem em produção, com volume real.

## Conceitos que todo sistema depende

### Bits, representação e interpretação

- **Tudo é binário; o formato é só convenção de leitura** — texto, código-fonte, vídeo e ROM são sequências de bits. "Texto" é o subconjunto de bytes que uma tabela (ASCII/Unicode) mapeia para caracteres; o editor mostra lixo ao abrir uma ROM porque não sabe interpretá-la. O binário real é uma linha contínua de bits, sem colunas nem quebras: o que o `xxd` mostra é só uma representação. [cap-014]
- **Encoding define quantos bytes cada caractere ocupa** — "hello world" em ASCII/UTF-8 ocupa 11 bytes (12 com o nulo do C); 7 símbolos em katakana ocupam 21 bytes. Em C um `char` é 1 byte; em Java/C# é UTF-16 (2 bytes). [cap-020]
- **Assinatura de formato (magic number)** — arquivos se identificam pelos primeiros bytes: cartucho NES começa com "NES", PNG com 0x89+"PNG", JPEG com FF D8 FF E0, `.class` do Java com CAFE BABE. Executável ELF e cartucho trazem cabeçalho dizendo ao sistema o que vem depois. [cap-014][cap-018]
- **Hexadecimal é a lente natural** — cada dígito hexa = 4 bits, 2 dígitos = 1 byte. Metades e limites ficam óbvios (metade de FFFF é 7FFF; 1 KB = 0x400 = 1024 bytes); em decimal a estrutura binária some. [cap-014]
- **Bits crescem em exponencial** — cada bit a mais dobra o alcance: 32 bits ≈ 4,3 bilhões, 64 bits ≈ 1,8×10^19. A metade de 64 bits é 63 bits, não 32; o salto de 16 para 20 bits de endereço já multiplicou a memória por 16. [cap-014][cap-018]
- **Shift e bitwise** — deslocar à esquerda multiplica por 2, à direita divide (perdendo o resto). OR mescla flags; bit fields empacotam várias características num único byte, técnica comum em protocolos de rede. [cap-014]
- **Endianness** — 6502, Intel/AMD e a maioria dos ARM são little endian (byte baixo primeiro); o PowerPC era big endian. O autor recebia lixo num protocolo binário entre Intel e PowerPC até inverter os bytes; TCP/HTTP e Protobuf cuidam disso por você. [cap-014]
- **Inteiros têm tamanho e sinal** — i8 a i64 (Rust ainda tem i128). Unsigned de 8 bits vai de 0 a 255; signed vai de -128 a 127, com o bit mais significativo como sinal. O `int` do C muda de tamanho conforme compilador e arquitetura, o que gera bugs; linguagens modernas deixam o tamanho explícito. [cap-020][cap-021]
- **Complemento de dois** — subtrair é somar o negativo: inverte todos os bits e soma 1. Assim a CPU reaproveita o mesmo circuito somador para a subtração. [cap-020]
- **Overflow e números maiores que o registrador** — uma CPU de 8 bits soma números grandes por pedaços, propagando o "sobe um" no flag de carry entre os bytes. Um valor maior que o registrador transborda e é truncado; BigInteger faz a conta em partes no software e custa pelo menos o dobro. [cap-015][cap-020]
- **Ponto flutuante é aproximação** — IEEE 754 binary64 = 1 bit de sinal, 11 de expoente e 52 de mantissa (notação científica em base 2). 0.1, 0.2 e 0.3 não têm representação exata, por isso `0.1 + 0.2 == 0.3` dá falso em JS, Python, Ruby e Rust. O maior inteiro exato é 2^53. No C antigo, `float` era binary32 e `double` era binary64. [cap-020]
- **No JavaScript, todo número é float64** — não há inteiro primitivo: o limite de inteiro seguro é `MAX_SAFE_INTEGER` (2^53), não 2^64. Em 2021, decimal exato (BigDecimal) ainda exigia biblioteca. [cap-020]

### CPU e execução

- **Transistor → porta → somador → CPU** — transistores são interruptores; com eles se montam portas NOT/AND/OR/NAND/XOR. Meio somador = AND + XOR (soma e carry); somadores completos em cascata somam N bits; flip-flops formam memória. A "lógica" de programação vem dessa lógica de hardware. [cap-014][cap-015]
- **Clock é o maestro** — todos os componentes pulsam no mesmo clock e cada instrução gasta um número exato de ciclos. Quem lê fora de sincronia lê lixo. Um periférico mais lento (o LCD do protótipo a 1 MHz) para de funcionar se a CPU não perguntar antes se ele está pronto. [cap-014][cap-015]
- **Barramento de endereço × barramento de dados** — o 6502 tem 16 pinos de endereço (A0–A15, 64 KB) e 8 de dados (D0–D7); cada pino é um bit. O tamanho do registrador não é o tamanho do endereço: o 6502 tem dados de 8 bits e endereço de 16; o 8086, registradores de 16 e endereço de 20. [cap-014][cap-018]
- **Mapa de memória e I/O mapeado** — o projetista decide quais chips respondem a quais faixas de endereço (decodificação feita com portas lógicas). Escrever num endereço pode acionar um latch, um LCD, a PPU ou o mapper do cartucho. Um chip que ignora o bit alto responde "espelhado" em duas faixas. [cap-014][cap-015]
- **Contador de programa e vetor de reset** — ao ligar, o 6502 lê FFFC/FFFD para saber onde começa o programa; depois o PC avança instrução a instrução, e JMP sobrescreve o PC. O ELF também declara o endereço da primeira instrução. [cap-014][cap-020]
- **Instrução e dado são o mesmo byte** — só o lugar para onde o PC aponta e o tamanho fixo de cada opcode, que a CPU conhece, separam código de dado. Se a execução cair em dados, eles rodam como instruções. [cap-014][cap-015]
- **Registradores = variáveis globais rápidas** — são uma ordem de grandeza mais rápidos que a RAM, mas compartilhados: instruções deixam efeitos colaterais (o carry, o acumulador sobrescrito dentro de uma sub-rotina). Empilhar o registrador (PHA/PLA) protege, ao custo de espaço na pilha. [cap-014][cap-015]
- **Opcode, assembly e assembler** — o mesmo mnemônico (LDA) vira opcodes diferentes conforme o argumento (valor imediato ou endereço). O assembler monta o binário e calcula os endereços dos labels, eliminando endereços fixos escritos à mão. [cap-014]
- **Sub-rotina = JSR + pilha + RTS** — chamar empilha o endereço de retorno e pula; retornar desempilha e volta. Com o PC e o SP já existem loops e funções; com branch/compare existe o `if`. Sem RAM respondendo na faixa da pilha, o endereço de retorno se perde e o programa "trava". [cap-014]
- **Interrupção = evento** — amarrar código a um sinal externo (clock, botão) é a mesma orientação a eventos do front-end, que existe desde os primeiros computadores. [cap-014]
- **Janela de escrita e escrita em lote** — a PPU desenha linha a linha e só aceita ordens nas ~20 scanlines do VBlank. A CPU acumula as mudanças em RAM e despacha tudo de uma vez: é o princípio do Virtual DOM, que separa a lógica da apresentação e agrupa escritas. [cap-015]
- **Tempo amarrado ao clock × ao frame** — jogos antigos amarravam a lógica ao clock e ficavam rápidos demais em CPUs novas (daí o botão Turbo); hoje se amarra ao frame rate ou ao vsync. Um emulador precisa da contagem exata de ciclos por instrução. [cap-015]
- **Abstração no próprio hardware: tiles** — a CPU do NES não manda pixels: manda o índice de um tile 8×8 (1 byte em vez de 64 comandos) e a PPU desenha. É a mesma ideia das fontes, dos sprites de CSS e da Font Awesome. [cap-015]
- **Abstração tem custo** — cada conveniência (binário → assembly → C → linguagem cola; OpenGL/DirectX ≤11 → Metal/Vulkan/DX12) muda o equilíbrio entre conveniência e velocidade. O movimento recente das APIs gráficas foi descer de nível para cortar o overhead de plataformas que cresceram demais. [cap-014][cap-019]
- **CISC × RISC (números de 2020)** — no x86 a instrução tem de 1 a 15 bytes, o que limita quantas o processador despacha por clock (~4); o ARM usa tamanho fixo (2 ou 4 bytes) e permite pipeline mais largo (o M1 executa 8 por clock). O x86-64 tem 16 registradores de uso geral; o ARMv8, 32. [cap-017]
- **O compilador absorve a complexidade da ISA** — com instruções simples, quem trabalha mais é o compilador, não o programador. O LLVM separou o front-end da linguagem da otimização por meio de uma linguagem intermediária, e isso facilitou trocar de arquitetura (PowerPC → Intel → ARM). [cap-017][cap-019]
- **Energia, calor e clock** — mais desempenho exige mais energia e gera mais calor. Chips modernos variam o clock (base, turbo/boost) e fazem throttling ao esquentar; o TDP configurado limita o pico. Nem o GHz nominal nem o nome comercial (i3/i5/i7) comparam máquinas: geração, núcleos e cache pesam. [cap-019]
- **CPU × GPU** — a CPU tem poucos núcleos rápidos para trabalho sequencial; a GPU tem milhares de unidades pequenas para trabalho independente por elemento. Um shader/compute kernel é só o corpo do `for`, e quem controla o loop é a GPU; map/reduce/scan/scatter/gather são os mesmos padrões. [cap-019]

### Computabilidade e emulação

- **Máquina de Turing** — tem número finito de estados, fita infinita e uma cabeça que lê, escreve e move um símbolo por vez; a tabela de transições é o programa. Para o autor, uma máquina de Turing é como uma função: o "número descritivo" é o corpo dela. [cap-016][cap-017]
- **Máquina de estados finitos na prática** — estados mais transições definem sistemas reais, como o ciclo de um pedido (aguardando pagamento → aguardando envio → entregue). Toda linguagem moderna tem bibliotecas para isso. [cap-016]
- **Programa armazenado (Turing + von Neumann)** — o que separa computador de calculadora (Babbage, ENIAC, Colossus) é guardar programa e dados no mesmo armazenamento e poder simular qualquer outra máquina. A arquitetura von Neumann tem unidade de processamento (ULA + registradores), unidade de controle (registrador de instrução + PC), memória para dados e instruções, armazenamento externo e E/S. [cap-016]
- **Todo programa é um número** — os bits de um programa, ou de um cartucho inteiro, formam um único número gigante. A máquina universal recebe esse número e simula a máquina correspondente: isso é emulação. [cap-017]
- **Turing-completo** — basta repetição condicional (while, ou if com goto) e ler/escrever memória; no mundo funcional, basta abstrair e aplicar funções. Como computadores reais têm memória finita (autômatos linearmente limitados), ninguém é Turing-completo no rigor. A régua prática do autor: se a linguagem consegue escrever um emulador, é Turing-completa. [cap-016][cap-017]
- **Limites do computável** — existem números e, muito mais, funções não computáveis; o problema da parada não tem solução em nenhum computador. Um computador quântico continua sendo uma máquina de Turing universal, só mais eficiente. [cap-016][cap-017]
- **Emular, interceptar, traduzir** — um emulador simula o chip, e o programa hóspede não sabe. O Game Genie fica entre o cartucho e o console e troca o valor lido num endereço. O Wine não emula: intercepta chamadas de API e traduz. O DXVK traduz DirectX para Vulkan e converte shaders, daí o cache e as travadinhas na primeira execução. [cap-015][cap-017][cap-019]

### Memória

- **Memória é um array endereçado** — para a CPU, memória é uma lista de bytes indexada por endereço; um emulador modela a RAM como array e cada chip como uma estrutura. [cap-015][cap-018]
- **Carregar × endereçar direto** — a ROM do cartucho "é" a memória: a CPU lê direto dela e não existe loading. Disco, CD e SSD obrigam a copiar para a RAM, que no PC ainda divide espaço com o SO e os drivers. O I/O segue como gargalo dos jogos modernos. [cap-015][cap-018]
- **Bank switching** — quando faltam endereços, um controlador (mapper) troca "páginas" no mesmo intervalo: mesmo endereço, banco diferente, dado diferente. O truque reaparece no EMS do DOS e no PAE/Address Windowing dos 32 bits. [cap-015][cap-018]
- **Segmentação x86 (legado)** — endereço = segmento×16 + offset, o que dá 20 bits a partir de registradores de 16. Mais de 4 mil pares segmento:offset caem no mesmo endereço, então escrever "em outro endereço" pode sobrescrever o mesmo byte. Esquemas de endereço com aliasing corrompem dados em silêncio. [cap-018]
- **Memória virtual e modo protegido** — cada processo enxerga uma tabela de endereços própria, mantida pelo SO, e só o SO vê a tabela global; um processo não alcança a memória do outro. O 386 trouxe a MMU em hardware e o modo Virtual 8086 para isolar programas DOS. [cap-018]
- **Paginação, page fault e swap** — o SO divide a memória em páginas e, quando ela acaba, grava em disco as pouco usadas. Ao acessar uma página ausente ocorre um page fault: o SO pausa o processo, recarrega a página e devolve o controle. Troca desempenho por espaço, como um cache. [cap-018]
- **O espaço de endereço nunca é todo seu** — o Windows 32 bits reservava 2 GB dos 4 GB virtuais para o sistema (DLLs, estruturas internas). Gerenciar memória também consome memória, porque as tabelas de páginas ocupam espaço. [cap-018]
- **64 bits na prática** — registradores e instruções têm 64 bits, mas o endereçamento físico fica entre 40 e 48 bits (com 48, pouco mais de 280 TB). Um endereço de 64 bits ocupa 8 bytes. [cap-018][cap-020][cap-021]
- **NX/DEP e ASLR** — o bit NX marca regiões como não executáveis, e o DEP usa isso para impedir que código injetado na stack ou nos dados rode. O ASLR sorteia onde ficam stack, heap e bibliotecas para que um exploit não dependa de endereços fixos. São camadas de dificuldade, não segurança total: um vazamento de ponteiros burla o ASLR. [cap-018]
- **Segmentos de um processo** — Text (código de máquina), ROData (literais e constantes do fonte, só leitura), BSS (globais não inicializadas, zeradas), heap (cresce para cima a partir dos endereços baixos), stack (cresce para baixo a partir dos altos) e área da kernel. `objdump -h` lista os segmentos de um ELF. [cap-020][cap-021]
- **Stack frame e passagem por valor** — cada chamada empilha o endereço de retorno e as cópias dos argumentos; o `return` desempilha e salta de volta. Um tipo primitivo é copiado; um array ou string em C é passado como endereço, então só esses 8 bytes são copiados. [cap-020][cap-021]
- **Literal × cópia em C** — (complemento, não está no compêndio) `char *s = "..."` aponta para o literal no ROData; `char s[] = "..."` dentro de uma função cria uma cópia na stack. Em ambos os casos, passar `s` para outra função copia só o ponteiro.
- **Recursão e stack overflow** — a pilha é finita; recursão sem caso de parada a enche até o segfault. Tail recursion reaproveita o frame. O autor cita uma stack inicial de 8 MB; a do 6502 tinha cerca de 255 bytes. [cap-014][cap-020]
- **Tamanho da stack** — (complemento, não está no compêndio) no Linux o limite padrão costuma ser 8 MB por thread (`ulimit -s`); não dimensione nada contando com gigabytes de stack.
- **Heap, malloc e ponteiro** — `malloc` reserva bytes contíguos no heap e devolve o endereço; ponteiro é só esse número. `*p` lê o valor e `p + 6` aponta 6 bytes adiante sem copiar nada. Objetos de linguagens de alto nível vivem no heap, e na stack fica só a referência. [cap-020][cap-021]
- **O heap não se limpa sozinho** — sem `free`, a memória fica ocupada até o processo morrer. Num processo longo (servidor, navegador) isso vira vazamento até o OOM killer matar o app. O GC resolve à custa de mais memória e de pausas; o ARC do Swift e o borrow do Rust ficam no meio-termo. [cap-020][cap-021]

### Dados e estruturas: o custo por baixo da sintaxe

- **Array** — elementos de mesmo tamanho, contíguos: `a[i]` = endereço inicial + i × tamanho do elemento, acesso O(1). O tamanho é fixo; "crescer" significa alocar outro maior, copiar tudo e liberar o antigo, com custo oculto e pico de memória dobrado. [cap-020][cap-021]
- **Array "heterogêneo"** — em linguagens dinâmicas, um array que mistura tipos é, por baixo, um array de ponteiros de 8 bytes para objetos no heap (hipótese do autor para o JS). Em C o tipo define o layout binário; em Java, a semântica. [cap-021]
- **String** — em C é um array de chars terminado em nulo; em linguagens modernas é um tipo próprio sobre arrays, em geral imutável. Concatenar repetidamente recria a string, e por isso o Java tem StringBuffer. [cap-020]
- **Struct e tupla** — a struct é um "molde" que fatia bytes contíguos em campos de tamanhos diferentes (Person = 10 + 1 + 1 bytes); a tupla é uma struct anônima e imutável. O compilador decide o layout final. [cap-020][cap-021] (complemento, não está no compêndio: C preserva a ordem dos campos mas pode inserir padding; meça com `sizeof`, não some à mão.)
- **Símbolos e metadados** — nomes de variáveis, campos e funções são para humanos: o compilador os troca por endereços e só os guarda se você pedir símbolos de debug. Formatos que levam metadados junto com os dados (JSON/XML) gastam espaço, e é por isso que o Protobuf existe. [cap-020]
- **Ponteiro de função → callback → OO** — funções também têm endereço; passá-las como argumento é callback, ou função de ordem superior (`map`, `filter`). Um objeto é uma struct no heap com ponteiros de função que recebem `self`; o construtor é `malloc` mais a inicialização. [cap-020]
- **Lista ligada** — nós no heap apontando uns para os outros: inserir ou remover é O(1) e não exige pré-alocação. Achar o n-ésimo é O(n), porque não há aritmética de endereço, e cada nó custa um `malloc`. [cap-021]
- **Hashtable** — um array fixo de buckets mais uma função de hash, com módulo para virar índice; colisões viram lista ligada dentro do bucket. Com hash bem distribuído, inserir e buscar fica perto de O(1); com hash ruim, degrada para O(n). [cap-021]
- **Pilha × fila** — a pilha é LIFO e sustenta a execução de funções. A fila é FIFO e serve a requisições de servidor web, jobs em background e controle de quanto processamento paralelo roda de cada vez. [cap-021]
- **Lookup table** — pré-calcular resultados (tabuada, cache Redis/Memcache, índice de banco) troca memória por ciclos. Matemática encolhe a tabela: multiplicar por quadrados reduz ~16 KB para ~0,5 KB. Tabela grande demais desperdiça espaço e fica lenta de consultar. [cap-015]
- **Complexidade (Big O)** — O(1), O(log n), O(n), O(n log n), O(n²), exponencial, fatorial. O bubblesort com 15 itens faz n(n−1)/2 = 105 comparações; a força bruta sobre permutações (caixeiro viajante) é fatorial. Big O mostra a ordem de crescimento, não o tempo absoluto, e o caso médio esconde o pior caso. [cap-021]
- **Linguagens "cola"** — Python, JS, Ruby e PHP são camadas sobre bibliotecas em C/C++: o CPython é C, o `dict` é o `dictobject.c`, e TensorFlow e SciPy são C++/Fortran. São ótimas para orquestrar, não para "sistemas" (drivers, cripto, banco de dados, protocolos eficientes). [cap-020][cap-021]

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Valores monetários (preço, saldo, imposto) | Inteiros em centavos ou tipo decimal (BigDecimal); nunca float | O binary64 acumula erro de conversão que, em volume, vira diferença real | cap-020 |
| Resultado de conta com float | Não comparar com `==`; arredondar ou truncar nas casas que importam | 0.1 + 0.2 não é 0.3 em base 2 | cap-020 |
| IDs ou contadores que podem passar de 2^53 consumidos por JS | Usar tipo de inteiro grande e não confiar em `Number` | `MAX_SAFE_INTEGER` é 2^53 | cap-020 |
| Campo numérico em estrutura com milhões de itens | Menor tipo que cobre o domínio, com tamanho e sinal explícitos | int64 sem necessidade desperdiça 7 bytes por elemento; `int` implícito muda com o compilador | cap-020, cap-021 |
| Número que não cabe no registrador | BigInteger/BigDecimal, sabendo que cada operação custa ≥2× | A conta é feita por partes, propagando o carry | cap-015, cap-020 |
| Coleção com tamanho final conhecido | Pré-alocar a capacidade | Evita sucessivas realocações com cópia e o pico de memória | cap-021 |
| Acesso por posição × inserção no meio ou crescimento sem fim | Array no primeiro caso, lista ligada no segundo | O(1) indexar × O(1) inserir; cada estrutura paga caro no outro | cap-021 |
| Busca por chave | Usar a hashtable nativa da linguagem | Já é C otimizado (o `dict` do CPython tem 5 mil+ linhas) | cap-021 |
| Estrutura ou algoritmo básico em linguagem cola | Usar a biblioteca padrão/nativa, não reimplementar em Python/JS puro | Nunca será mais rápido que o nativo em C | cap-020, cap-021 |
| Escolher algoritmo de ordenação/busca | Avaliar pior caso e perfil real dos dados, não só o caso médio | Quicksort vira O(n²) em lista já ordenada; um pior caso pode derrubar o sistema | cap-021 |
| Operação genérica em caminho quente | Procurar o caso especial (×2 = shift; tabela de quadrados) | Fórmulas universais costumam ser lentas | cap-015 |
| Trocar memória por CPU ou vice-versa | Decidir explicitamente (lookup table, cache) e dimensionar a tabela | Algoritmo é balanço entre memória e ciclos | cap-015 |
| Validar desempenho | Testar com 10³, 10⁵ e 10⁶ itens e com a entrada de pior caso | Diferenças de complexidade só aparecem com volume | cap-015, cap-021 |
| Otimização × legibilidade | Escrever legível, medir e só então otimizar | O ganho de tirar JSR/RTS não compensa a sujeira; decide-se depois de mensurar | cap-014 |
| Bloco repetido por copy-paste | Extrair sub-rotina/função | Código sujo aumenta o binário e espalha bugs (o próprio autor esqueceu valores hardcoded) | cap-014, cap-021 |
| Endereços, tamanhos ou limites escritos à mão | Constantes nomeadas e labels; deixar o compilador/assembler calcular | Endereços mudam quando o código muda; números mágicos são ruins até em assembly | cap-014, cap-021 |
| Recursão sobre entrada de tamanho variável | Garantir caso de parada; preferir tail recursion ou iteração | A stack é finita e estoura em segfault | cap-020 |
| Processo de longa duração (servidor, worker) | Liberar toda alocação e monitorar o uso de memória | `malloc` sem `free` vaza até o OOM killer agir | cap-021 |
| Sistema sensível a latência ou de baixo nível | Pesar pausas e overhead do GC; considerar Rust, Swift ou C | Todo GC reserva mais memória e pausa a execução | cap-020, cap-021 |
| Protocolo ou arquivo binário próprio entre máquinas | Fixar a ordem de bytes e um cabeçalho identificável | Little × big endian geram lixo silencioso | cap-014 |
| Integração com componente mais lento | Perguntar se está pronto (handshake) ou enfileirar com limite | O LCD não acompanhava a CPU a 1 MHz; filas controlam a carga | cap-014, cap-021 |
| Muitas mudanças de estado na saída (UI, I/O) | Acumular em memória e aplicar em lote numa janela | Modelo VBlank / Virtual DOM: uma interrupção em vez de cinquenta | cap-015 |
| Lógica que depende de tempo | Amarrar a tempo real ou frame, nunca à velocidade da CPU | Jogos antigos ficaram rápidos demais em CPUs novas | cap-015 |
| Trabalho pesado sobre muitos elementos | Elementos independentes → GPU/kernels; trabalho sequencial e dependente → CPU | Nem todo trabalho dá para paralelizar | cap-019 |
| Dimensionar ou comparar máquinas | Olhar geração, núcleos, cache, TDP e throttling, não GHz nominal nem nome i5/i7 | O turbo dura pouco quando a solução térmica não acompanha | cap-019 |
| Portar para outra plataforma ou API | Considerar uma camada de tradução (DXVK, MoltenVK, Angle) antes de reescrever | Um adaptador de API é viável onde reimplementar tudo não é | cap-019 |
| Rodar binário de outra arquitetura | Emulação/tradução é viável, mas espere custo maior quando o hóspede tem mais registradores ou instruções mais simples que o hospedeiro | ARM emula x86 bem (Rosetta 2); o inverso é lento (Rosetta 1) | cap-017 |
| Dependência de driver ou SDK proprietário | Preferir padrão aberto quando a portabilidade importa (OpenCL × CUDA) | O suporte só vem quando o fornecedor quer (NVIDIA × Wayland) | cap-019 |
| Programa "trava" sem motivo aparente | Executar passo a passo observando endereços e registradores, começando pelo suspeito mais óbvio | Assim se viu que JSR/RTS gravavam o retorno numa pilha sem RAM | cap-014 |
| Achar onde o estado vive num binário sem fonte | Comparar dumps de memória antes e depois do evento; desassemblar e rotular o óbvio primeiro | É como se descobrem variáveis (vidas) e se reconstrói código perdido | cap-015 |
| Debate "a linguagem X é Turing-completa?" | Encerrar e avaliar a eficiência para o caso de uso | Em capacidade são todas equivalentes; em eficiência não | cap-016 |

## Armadilhas e mitos

- "Arquivo texto e arquivo binário são coisas diferentes" → ambos são bits; muda só a convenção de interpretação. [cap-014]
- "64 bits é o dobro de 32" → é 2^32 vezes maior; a metade de 64 bits é 63 bits. [cap-014][cap-018]
- "CPU de N bits endereça 2^N bytes" → registrador e endereço têm tamanhos independentes (6502: 8/16; 8086: 16/20; x86-64: 64 bits com 40–48 de endereço físico). [cap-018][cap-020]
- "Endereço de 64 bits ocupa 4 bytes" → ocupa 8; converter bits em bytes é dividir por 8, e o próprio autor errou isso por memória muscular. [cap-021]
- "0.1 + 0.2 == 0.3" → falso em quase toda linguagem; float é aproximação em base 2. [cap-020]
- "Um char é um byte" → só em C/ASCII; Java e C# usam UTF-16 e o UTF-8 é multibyte. [cap-020]
- "O array do JS/Python cresce de graça" → por baixo realoca e copia; em milhões de itens isso pesa e dobra o pico de memória. [cap-020][cap-021]
- "`obj['chave']` é um array" → é uma hashtable e funciona bem diferente. [cap-021]
- "Mesma Big O, mesmo desempenho" → só a mesma ordem de crescimento; inserir em array é mais rápido que em lista ligada, que faz `malloc` a cada nó. [cap-021]
- "O algoritmo com melhor caso médio é sempre o melhor" → o quicksort em lista já ordenada vira quadrático; Python usa Timsort pelo pior caso melhor. [cap-021]
- "Testei aqui e funciona" → multiplicar por somas sucessivas vai bem com 7 e fica inviável com 10 mil; complexidade só aparece com volume real. [cap-015][cap-021]
- "Hardware 10 milhões de vezes mais denso = software 10 milhões de vezes melhor" → não; boa parte do software ainda não roda da melhor forma possível. [cap-015]
- "Um índice fora do limite só mostra lixo na tela" → no Super Mario 3, um índice estourado escreveu num endereço do mapper, trocou o banco de código em execução, a pilha vazia deu a volta e dados viraram instruções. [cap-015]
- "Com GC não existe problema de memória" → o GC reserva mais memória e causa pausas; vazamento lógico continua possível. [cap-020][cap-021]
- "`free` é opcional" → só num programa que termina logo, porque aí o SO limpa; num processo longo, vaza. [cap-021]
- "Ponteiro é um conceito místico" → é só um número com um endereço; `[]` é aritmética de ponteiro. [cap-020]
- "Os nomes do código existem no executável" → só com símbolos de debug; para a máquina é tudo uma linguiça de bits. [cap-020]
- "Python é rápido para machine learning" → Python é a cola; o trabalho pesado roda em C/C++/Fortran. [cap-020]
- "ENIAC, Colossus e a máquina de Babbage foram os primeiros computadores" → sem programa armazenado nem universalidade, são calculadoras glorificadas. [cap-016]
- "O computador quântico vai resolver o que hoje é impossível" → continua sendo máquina de Turing universal: mais eficiente, não mais capaz. O problema da parada segue sem solução. [cap-016][cap-017]
- "Turing-completo = linguagem boa" → Brainfuck é Turing-completo e inútil; HTML não é e é muito útil. O autor também classifica regex e SQL como não Turing-completos. [cap-016] (complemento, não está no compêndio: SQL com CTE recursiva é considerado Turing-completo; o ponto prático do autor, julgar pela eficiência, não muda.)
- "Mais GHz = mais rápido" → turbo, throttling, TDP, geração e cache decidem. [cap-019]
- "Wine é um emulador" → ele intercepta e traduz chamadas de API, não simula hardware. [cap-019]
- "CUDA core e compute kernel são outra ciência" → um kernel de data science é, na origem, um pixel shader. [cap-019]
- "ASLR e NX tornam o sistema seguro" → são camadas extras de dificuldade; um vazamento de ponteiros burla o ASLR. [cap-018]

## Checklist para aplicar num sistema real

- [ ] Todo valor monetário ou que exige exatidão está em inteiro (centavos) ou decimal, nunca em float? [cap-020]
- [ ] Há comparação `==` entre floats ou acumulação de float em loop longo? [cap-020]
- [ ] Algum ID ou contador pode passar de 2^53 e ser lido por JavaScript? [cap-020]
- [ ] Os inteiros têm tamanho e sinal explícitos, cabem no domínio e o overflow foi considerado? [cap-020][cap-021]
- [ ] O encoding está definido em cada fronteira (arquivo, banco, rede) e os tamanhos são medidos conscientemente em bytes ou em caracteres? [cap-020]
- [ ] Protocolos e arquivos binários fixam a ordem de bytes e trazem cabeçalho/assinatura identificável? [cap-014]
- [ ] Cada estrutura de dados foi escolhida pelo padrão de acesso (posição → array; chave → hash; inserção no meio → lista; ordem de chegada → fila)? [cap-021]
- [ ] Coleções que crescem têm capacidade pré-alocada quando o tamanho é conhecido? [cap-021]
- [ ] Qual é a complexidade dos loops críticos? Existe loop aninhado sobre dados que crescem com o uso? [cap-021]
- [ ] Os testes de desempenho usam volume realista e a entrada de pior caso (por exemplo, dados já ordenados)? [cap-015][cap-021]
- [ ] Toda recursão tem caso de parada e profundidade limitada, sem depender de stack grande? [cap-020]
- [ ] Em processo longo, toda alocação tem liberação, e há monitoramento de memória contra vazamento e OOM? [cap-021]
- [ ] Com GC, as pausas e o overhead de memória cabem no requisito de latência? [cap-020][cap-021]
- [ ] A lógica temporal está amarrada a tempo real ou frame, e não à velocidade da máquina ou à contagem de iterações? [cap-015]
- [ ] Integrações com componentes mais lentos têm handshake ou fila com limite de concorrência? [cap-014][cap-021]
- [ ] As escritas de saída (UI, disco, rede) são agrupadas em lote em vez de uma interrupção por mudança? [cap-015]
- [ ] Trabalho por elemento independente pode ir para GPU ou biblioteca nativa, e o trabalho sequencial fica na CPU? [cap-019]
- [ ] O caminho quente em linguagem cola usa bibliotecas nativas em vez de reimplementação pura? [cap-020][cap-021]
- [ ] O dimensionamento de máquina considera throttling, TDP, núcleos, cache e RAM, e não só GHz ou marca? [cap-019]
- [ ] A memória do processo cabe no espaço realmente disponível (reservas do SO, overhead do runtime) e não no total teórico? [cap-018]
- [ ] NX/DEP e ASLR estão ativos, e nada no código ou no deploy depende de endereços fixos? [cap-018]
- [ ] A serialização em JSON/XML é aceitável no volume, ou um formato binário com schema compensa? [cap-020]
- [ ] Números mágicos, endereços e tamanhos hardcoded viraram constantes nomeadas? [cap-014][cap-021]
- [ ] Cada otimização proposta foi medida antes e depois, e o ganho paga a perda de legibilidade? [cap-014]
- [ ] Nas contas de tamanho e capacidade, bits × bytes estão conferidos (÷8; endereço de 64 bits = 8 bytes)? [cap-021]

## Frases-âncora

> "Muita gente acha que arquivos texto e binário são diferentes. Mas não, ambos são binários." — cap-014

> "Boa parte do que é programação é retrabalhar os dados pra ficar mais fácil de computar o que queremos." — cap-014

> "Algoritmos costumam ser um balanço de dois fatores: se você quiser economizar memória vai acabar gastando mais ciclos de processamento." — cap-015

> "Agora, ser capaz não é a mesma coisa que ser eficiente. Muita gente confunde isso." — cap-016

> "Memória não é tudo igual. Não é uma lista linear." — cap-018

> "CPUs são boas pra processar linearmente, trabalhos grandes sem parar. GPUs são boas pra processar milhares de pequenos trabalhos em paralelo." — cap-019

> "Tudo em computação tem um custo, não é automático nem tudo igual." — cap-020

> "E certamente sua primeira tentativa vai ter complexidade quadrática, então toma cuidado." — cap-021

## Onde aprofundar

| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-014 | [Akitando] #80 - O Guia +Hardcore de Introdução à COMPUTAÇÃO | https://www.akitaonrails.com/2020/06/04/akitando-80-o-guia-hardcore-de-introducao-a-computacao/ | Precisar do modelo mental de CPU, clock, barramentos, ROM/RAM, endianness, assembly, sub-rotinas e pilha montados do zero num 6502. |
| cap-015 | [Akitando] #81 - Aprendendo sobre Computadores com Super Mario (do jeito Hardcore++) | https://www.akitaonrails.com/2020/06/18/akitando-81-aprendendo-sobre-computadores-com-super-mario-do-jeito-hardcore/ | For discutir aritmética binária, trade-off memória × ciclos, lookup tables, mapa de memória, emulação, escrita em lote (VBlank) ou como um overflow vira execução arbitrária. |
| cap-016 | [Akitando] #86 - O Computador de Turing e Von Neumann \| Por que calculadoras não são computadores? | https://www.akitaonrails.com/2020/10/23/akitando-86-o-computador-de-turing-e-von-neumann-por-que-calculadoras-nao-sao-computadores/ | Surgir dúvida sobre máquina de Turing, máquinas de estado, programa armazenado, arquitetura von Neumann ou Turing-completude. |
| cap-017 | [Akitando] #87 Turing Complete, Emuladores e o Chip ARM M1 | https://www.akitaonrails.com/2020/11/27/akitando-87-turing-complete-emuladores-e-o-chip-arm-m1/ | Avaliar emulação, tradução binária ou diferenças RISC × CISC (números datados de 2020). |
| cap-018 | [Akitando] #97 - Só Precisamos de 640 kB de Memória? \| 16-bits até 64-bits! | https://www.akitaonrails.com/2021/05/13/akitando-97-so-precisamos-de-640-kb-de-memoria-16-bits-ate-64-bits/ | Precisar entender endereçamento, memória virtual, paginação/swap, limites de 32 × 64 bits, NX/DEP e ASLR (a parte DOS é legado, vale pelo princípio). |
| cap-019 | [Akitando] #98 - A Longa História de CPUs e GPUs \| Jogos de Windows em Linux?? | https://www.akitaonrails.com/2021/05/26/akitando-98-a-longa-historia-de-cpus-e-gpus-jogos-de-windows-em-linux/ | Decidir entre CPU e GPU, entender shaders/kernels, APIs gráficas, camadas de tradução (Wine/DXVK) ou throttling e TDP (detalhes de Optimus e fabricantes são de 2021). |
| cap-020 | [Akitando] #93 - Hello World Como Você Nunca Viu! \| Entendendo C | https://www.akitaonrails.com/2021/03/06/akitando-93-hello-world-como-voce-nunca-viu-entendendo-c/ | Explicar tipos (inteiros, float IEEE 754), arrays, strings, stack × heap, ponteiros, structs, callbacks e objetos vistos de baixo; leia junto com a errata do cap-021. |
| cap-021 | [Akitando] #94 - O que vem DEPOIS do Hello World \| Consertando meu C | https://www.akitaonrails.com/2021/03/22/akitando-94-o-que-vem-depois-do-hello-world-consertando-meu-c/ | Escolher estrutura de dados (array, lista ligada, hashtable, pilha, fila), raciocinar sobre Big O e pior caso, ou rever os segmentos de memória e o `free`/OOM. |
