# Linguagens de programação
> Destilado de: cap-030, cap-031, cap-032, cap-033, cap-034, cap-035, cap-036, cap-037, cap-038, cap-039, cap-040, cap-041, cap-042, cap-043, cap-044 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).

## Ideia central
Para o autor, nenhuma linguagem é especial: todas herdam ideias de décadas atrás (ALGOL, Simula, ML, Prolog, CSP), e linguagem é ferramenta escolhida pelo contexto, não identidade. O que decide o comportamento de um sistema não é o rótulo "compilada vs interpretada" (hoje quase tudo vira código nativo, antes via AOT ou durante via JIT), mas o que roda por baixo: runtime, bibliotecas em C, custo de FFI, alocador e garbage collector, modelo de concorrência e de falhas. Escolha pela categoria do problema, pela maturidade do ecossistema e pelo custo de hora de programador versus hora de máquina, e escreva no idioma da linguagem, para pessoas.

## Conceitos que todo sistema depende

### Linhagem e paradigmas
- **Toda linguagem é herdeira** — ALGOL deu origem a CPL → BCPL → B → C e, via ALGOL W, ao Pascal; C# é "filho bastardo" de Delphi com J++ (mesmo projetista, Anders Hejlsberg). Saber de onde a linguagem veio ajuda a prever para onde ela vai. [cap-030, cap-034]
- **Recursos "modernos" são antigos** — Objetos, classes, herança e GC vieram com Simula (1967); tipos abstratos e iteradores com CLU (Barbara Liskov, 1974); patterns de texto com SNOBOL; call-by-pattern com Hope/NPL. CSP com canais veio de OCCAM (Tony Hoare) e foi reciclado em Erlang e Go; inferência de tipos Hindley-Milner veio da família ML e hoje está em Rust, Kotlin, Swift, Scala e Java 10+. [cap-031]
- **Paradigmas além de "OO vs funcional"** — Há o imperativo, o orientado a objetos, o funcional puro (Miranda, Haskell), o lógico/declarativo (Prolog, primo do cálculo relacional do SQL) e o de arrays (APL, Julia). Erlang e JavaScript têm funções, mas não são puramente funcionais. [cap-031]
- **A gramática define a linguagem** — BNF (Backus-Naur Form) descreve gramáticas livres de contexto; a definição oficial e não ambígua de uma linguagem está no léxico e na gramática (yacc do C, especificação ECMAScript), não em blog post. [cap-030, cap-034]

### Compilação e execução
- **Lexer, parser e AST** — O lexer quebra o texto em tokens; o parser aplica a gramática e monta uma árvore (parse tree / AST). Linters e análise estática trabalham sobre a AST, e o DOM é o equivalente para HTML. [cap-034]
- **O compilador reescreve seu código** — Depois do parsing, o código vira representação intermediária (IR do LLVM, RTL do GCC, IL do .NET) e passa por várias passadas de otimização: pré-cálculo de constantes, eliminação de código morto, tail call, inlining. O que você escreve não é o que executa. [cap-034]
- **Front-end, middle-end, back-end** — Um front-end por linguagem (lexer + parser → IR), um otimizador comum e um back-end por arquitetura (x64, ARM). Linguagem nova só precisa do front-end (Rust, Julia, Crystal sobre LLVM); chip novo, só do back-end. [cap-034, cap-036]
- **Linker** — Cada arquivo vira um objeto (.o) com endereços externos pendentes; o linker resolve os endereços entre objetos e bibliotecas compartilhadas e ainda otimiza (ex.: inlining). Essa otimização estática é parte do porquê C e Rust geram binários enxutos e rápidos. [cap-034]
- **Bytecode e máquina virtual** — Java compila para bytecode de uma máquina que não existe (a JVM; todo `.class` começa com `cafe babe`), Python gera `.pyc` e o V8 gera bytecode próprio. Todo interpretador é um caso de máquina virtual, mas, diferente de um VirtualBox, não esconde o disco nem a rede do host. [cap-034]
- **Endereços resolvidos em tempo de carga** — O class loader da JVM resolve endereços ao carregar classes, o que permite compilar classes separadamente. Em troca, o compilador não pode eliminar métodos que outra classe ou a reflection podem chamar, o que limita a otimização estática. [cap-034]
- **AOT vs JIT** — AOT (C, Rust, Go) gera binário nativo antes de rodar; JIT (HotSpot da JVM, TurboFan do V8, BEAM) interpreta, mede os pontos quentes e compila esses trechos para nativo durante a execução. Daí o "aquecimento" e o consumo maior de memória; no fim, os dois geram código nativo. [cap-034, cap-036]
- **Estático vs dinâmico** — A diferença prática é poder alterar código e AST em execução: REPL, reflection, metaprogramação, carregar código sem reiniciar. Essa flexibilidade custa a otimização agressiva de compilador e linker, mais memória e mais CPU; o JIT reduziu muito a distância. [cap-034]
- **JIT não serve a todo perfil** — Em app mobile de vida curta, o JIT não tem tempo de otimizar e ainda gasta bateria; por isso o Android passou a pré-compilar (AOT) o bytecode DEX. O JIT brilha em servidores que ficam de pé por dias. [cap-036]
- **WebAssembly** — Com um back-end que gera JavaScript (Emscripten, asm.js) e depois WebAssembly, código C/C++ compilado via LLVM roda no navegador. O navegador vira mais uma "máquina" alvo de compilação. [cap-036]
- **A toolchain também é política** — A Apple trocou GCC por LLVM/Clang em parte por causa da GPLv3 (cláusulas anti-"Tivoização") e da baixa prioridade do GCC para Objective-C. Licença restritiva (GPL, "viral") vs permissiva (BSD, MIT, Apache) e risco de propriedade intelectual moldam a escolha de stack. [cap-036]

### O que existe por baixo
- **Linguagem de script é casca sobre C** — Funções de Python, Ruby e PHP herdam nome e comportamento da libc (`strftime`, `mkdir`, `getenv`, `math.h`). numpy, SciPy, TensorFlow e PyTorch têm grande parte do código em C, C++ e Fortran; o event loop do Node é a libuv, em C, sobre epoll/kqueue/IOCP; até o OpenJDK tem o HotSpot em C++. [cap-031, cap-035]
- **O runtime muda conforme o SO** — O CPython de Windows não é o mesmo de Linux (blocos `#ifdef MS_WINDOWS` em dezenas de arquivos). O interpretador esconde o sistema operacional, mas não por completo. [cap-035]
- **Dialetos de biblioteca** — Regex no estilo Perl (PCRE) e no estilo ICU, e formatação de data no estilo `strftime` do C e no estilo ICU, têm sintaxes diferentes entre linguagens. O mesmo padrão pode não funcionar igual em outra plataforma. [cap-035]
- **ABI e drivers** — Kernel e drivers rodam no mesmo espaço de memória (ring 0) e expõem funções C; só linguagens compiladas que geram binário com ABI compatível com C (C, C++, Rust, Nim, D) conseguem escrever driver. Linguagens com VM ou interpretador ficam no userland (ring 3). [cap-035]
- **FFI e marshalling duplicam memória** — Tipos de linguagens diferentes não são binariamente iguais, então cruzar a fronteira (Python → C, JNI no Java, XML → JSON, chamadas remotas) exige converter e copiar. A cópia também evita que o GC de um lado libere o que o outro lado ainda usa. [cap-035]

### Memória
- **Hierarquia de memória** — Caches L1, L2 e L3 são pequenos e ultra rápidos; a RAM é ordens de grandeza mais lenta; o swap em disco, ordens de grandeza mais lento ainda. [cap-037]
- **Memória virtual** — Cada processo enxerga um espaço de endereços virtual próprio, que o SO mapeia para memória real sob demanda. Isso isola processos e permite prometer mais memória do que existe, mas parte do espaço fica reservada ao SO e às bibliotecas compartilhadas (no Windows 32 bits, um processo só usava 2 GB). [cap-037]
- **Fragmentação** — Toda alocação precisa de um bloco contíguo; liberar no meio deixa buracos, e o processo segura mais memória do que usa. No caso real contado pelo autor, a JVM não subia por falta de 1,5 GB contíguo, até ele fazer rebase das DLLs. [cap-037]
- **Alocadores** — O malloc da glibc (ptmalloc2, derivado do dlmalloc) divide a memória em arenas por thread (para evitar lock global) e em bins por tamanho (fast, small, large) para fragmentar menos. O tcmalloc (Google) adiciona cache por thread; o jemalloc (Firefox, Facebook) desperdiça ainda menos; o Go tem alocador próprio inspirado no tcmalloc. [cap-037]
- **Arenas demais** — Em 64 bits, a glibc permite até 8 × núcleos de arenas, o que infla a memória em máquina pequena. O Heroku recomenda `MALLOC_ARENA_MAX=2`; trocar o alocador do interpretador por jemalloc também ajuda Ruby e Python. [cap-037, cap-038]
- **OOM killer** — Linux, Android e iOS matam processos quando a memória acaba. Em mobile, sem swap, vazamento vira crash (o primeiro iPhone tinha 128 MB de RAM). [cap-036, cap-037, cap-038]

### Garbage collection
- **GC é um alocador em user-land** — Pede blocos grandes ao SO e gerencia tudo internamente. Existe porque programadores erram `malloc`/`free` (vazamento, ponteiro inválido). [cap-038]
- **Reference counting** — Cada objeto tem um contador e é liberado quando ele chega a zero (Python, Objective-C/Swift com ARC). Ciclos A↔B nunca zeram, por isso existem referências weak/unowned. [cap-036, cap-038]
- **Mark and sweep** — Marca o que ninguém mais referencia e depois varre. Exige pausar o programa (stop the world), e a pausa cresce com o número de objetos alocados. [cap-038]
- **Frequência é troca** — Coletar ou liberar com muita frequência gasta CPU e reduz performance; coletar pouco economiza CPU e infla a memória. Não existe configuração que ganhe nos dois. [cap-038]
- **Gerações** — Pela hipótese geracional, objetos novos morrem cedo: coletas menores (minor) olham só a geração nova, e quem sobrevive é promovido (Young/Old/Perm na JVM; Eden/Survivor/Old no G1; gerações 0/1/2 no Python). Isso reduz muito o custo da marcação. [cap-038]
- **Mover ou não mover objetos** — Sem ponteiros expostos (Java, Erlang, .NET), o GC pode copiar e compactar objetos e assim desfragmentar. Com ponteiros expostos ou extensões em C (Ruby, Python, Go), os objetos ficam no lugar e a fragmentação persiste. [cap-038]
- **Liberar não é devolver ao SO** — Slots liberados são reaproveitados, mas a memória continua alocada pelo processo. Um pico (arquivo inteiro na memória, SELECT sem filtro) incha o heap de forma duradoura. [cap-038]
- **Coleta incremental e concorrente** — A marcação em três cores (tri-color) permite marcar em etapas sem pausa longa (Erlang, coletas minor do Ruby); a JVM tem coletores paralelos e concorrentes (Parallel GC, CMS, G1GC). GC geracional custa mais memória que reference counting. [cap-038]

### Abstração, idioma e Ruby
- **Design pattern como sintoma** — Texto de 2006, tradução de artigo de Mark Dominus publicada pelo autor: um pattern recorrente indica algo que a linguagem ainda não absorveu. Sub-rotina era pattern em assembly, "classe" era pattern em C antes do C++, o Iterator some em linguagens com boas listas e o MVC foi absorvido pelo Rails como DSL. [cap-039]
- **Níveis de um pattern (Norvig)** — Um pattern pode ser invisível (parte da linguagem), formal (implementado uma vez, por exemplo como macro) ou informal (reescrito do zero a cada uso). O progresso é empurrar o pattern de informal para invisível. [cap-039]
- **Tudo é objeto, chamar é enviar mensagem** — Em Ruby, números, `true` e `nil` são objetos, e chamar um método é enviar uma mensagem (`send`). Com `method_missing`, um objeto responde a mensagens arbitrárias, o que viabiliza DSLs como o XML Builder. [cap-040, cap-041, cap-042]
- **Classes abertas e módulos** — Em Ruby, qualquer classe pode ser reaberta, inclusive as da biblioteca padrão (monkey patching), e módulos organizam comportamento com `include` (métodos de instância) e `extend` (métodos de classe). É assim que o ActiveSupport estende o Ruby e o ActiveRecord se divide em módulos. [cap-040]
- **Reflexão vs metaprogramação** — Reflexão é perguntar ao objeto o que ele sabe responder (`methods`, `respond_to?`); metaprogramação é alterar o comportamento em execução (gerar getters e setters com `module_eval`). [cap-042]
- **Closures e funções de ordem superior** — Um bloco (lambda/Proc) captura as variáveis do contexto em que foi criado e pode ser passado ou devolvido como valor. O pattern Command vira simplesmente um bloco guardado e chamado depois. [cap-041, cap-043]
- **Recurso gerenciado por bloco** — O método abre o recurso, faz `yield` para o código do usuário e garante a limpeza em `ensure` (`File.open`, transação do ActiveRecord). O usuário escreve só a lógica de negócio, e o encanamento fica escondido. [cap-041, cap-043]
- **Transformar, não iterar** — `map`, `select` e `inject` dizem o que transformar, filtrar ou reduzir; o "como iterar" fica escondido no iterador. [cap-041]

### Concorrência e falhas (Erlang/Elixir)
- **Imutabilidade e mensagens opacas** — Dados mutáveis e estado compartilhado otimizam performance, mas tornam a concorrência difícil. Erlang força imutabilidade e processos isolados que só conversam por mensagens (passadas por referência, sem cópia nas chamadas). [cap-044]
- **Scheduler em user-land** — Uma thread real por núcleo, cada uma com um scheduler que suspende processos leves depois de um certo número de "reduções". O I/O assíncrono é transparente: sem pirâmide de callbacks e sem depender do programador lembrar de ceder o controle ao event loop. [cap-044]
- **Let it crash + supervisores** — O processo com defeito morre limpo e avisa seu supervisor, que o reinicia a partir de um estado bom (one_for_one, one_for_all). Uma árvore de supervisores isola a falha sem deixar o resto do sistema em estado inconsistente. [cap-044]
- **Hot swap de código** — Sem hierarquia de tipos rígida e com estado isolado, dá para trocar um módulo em produção, com o callback `code_change` migrando o estado antigo. Com assinaturas estáticas isso é bem mais difícil. [cap-044]
- **Tipagem** — Inferência de tipos (Hindley-Milner) é um meio-termo: verificação estática com menos boilerplate. A tipagem dinâmica dá flexibilidade e exige testes de comportamento, que não deveriam ser opcionais em nenhum dos dois casos. [cap-031, cap-044]

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Escolher linguagem para um projeto | Decidir pela categoria do problema e pela maturidade do ecossistema, não por gosto | Cada linguagem estava "no lugar certo na hora certa"; código não liga para lealdade | cap-035, cap-044 |
| Kernel, driver, stack TCP | C, C++ ou Rust (Nim, D) | Precisa de ABI C e do menor número de abstrações | cap-035 |
| Ferramenta de linha de comando ou infra | Go ou Rust | Binário quase tão eficiente quanto C, sintaxe moderna; a JVM sofre com o aquecimento | cap-035, cap-044 |
| Sistema distribuído (banco, fila, coordenação) | Ecossistema maduro (Java/Apache: ZooKeeper, Kafka, Cassandra) ou Erlang/Elixir | Consenso e cluster já estão resolvidos e testados em produção | cap-035 |
| Alta concorrência: tempo real, chat, notificações | Erlang/Elixir (BEAM) | Processos isolados, scheduler e supervisores | cap-044 |
| Site de conteúdo comum | Framework maduro (o autor usa Rails) e Phoenix só para WebSockets e tempo real | A maioria dos sites não precisa da concorrência; ecossistema maduro entrega mais rápido | cap-044 |
| Poder bruto de computação | Go, Java ou C/C++, não Erlang | A BEAM não é rápida em CPU pesada nem serve para data science | cap-044 |
| App comercial que compete por funcionalidade | Linguagem produtiva (Ruby, Python, PHP, JS/TS, Kotlin) | Hora de programador custa mais que hora de CPU, e o app precisa mudar rápido | cap-035 |
| Trecho pesado dentro de app em linguagem de script | Escrever só esse trecho em C, Go ou Rust e integrar via FFI | Os outros 80% "mundanos" ficam na linguagem produtiva | cap-035 |
| Integração FFI frequente ou com dados grandes | Reduzir as travessias e, se preciso, trabalhar direto no tipo do outro lado (`CString` no Rust) | Cada conversão duplica memória | cap-035 |
| Mobile simples | React Native ou Flutter, com componentes nativos (Swift, Kotlin) para se diferenciar | Produtividade; smartphones já têm RAM para gastar um pouco | cap-035 |
| Mobile com memória apertada | Evitar casca de webview e controlar os picos | Sem swap, o OOM killer derruba o app | cap-036, cap-038 |
| Data science e ML | Usar Python como cola, sabendo que o motor é C, C++, Fortran ou CUDA | A linguagem não é a parte importante; o acesso à GPU e ao vetor é | cap-035 |
| Sistema maduro em linguagem "velha" | Não reescrever | Maturidade vale mais que novidade | cap-035, cap-044 |
| Processo de vida longa (servidor) | Um runtime com JIT compensa | Otimiza os pontos quentes depois do aquecimento | cap-034, cap-036 |
| Processo de vida curta (app mobile, CLI) | Preferir AOT ou runtime sem aquecimento | O JIT não tem tempo de otimizar e gasta bateria e memória | cap-036, cap-044 |
| Vontade de escrever assembly ou micro-otimizar | Deixar o compilador otimizar e escrever código claro | O compilador gera assembly melhor; boas práticas economizam hora humana | cap-034 |
| Testar app de outra arquitetura | Preferir compilar para o host (simulador) a emular instruções | Emular traduz instrução a instrução e é lento | cap-036 |
| Processo usando memória demais | Corrigir o código primeiro (streams, LIMIT, evitar picos); tuning de GC/alocador só depois de medir em produção | Quase sempre o desperdício está no código | cap-038 |
| Arquivo grande ou query sem limite | Ler em stream ou em pedaços, usar LIMIT e checar o tamanho antes | O heap infla e não volta | cap-038 |
| Servidor Linux pequeno com glibc e várias threads | `MALLOC_ARENA_MAX=2` ou jemalloc | Arenas demais desperdiçam memória | cap-037, cap-038 |
| Grafo de objetos com reference counting | Usar referência weak/unowned no lado "de volta" | Quebra o ciclo e evita vazamento | cap-038 |
| Problema de pausa e latência por GC | Entender gerações, frequência e o tipo de coletor (paralelo, concorrente) | A frequência troca CPU por memória | cap-038 |
| Concorrência | Preferir a abstração do runtime (processos, mensagens, filas) a threads com lock feitas à mão | "não use threads, é provável que você vá errar" | cap-044 |
| Boilerplate repetido (pattern) | Procurar a abstração na linguagem ou biblioteca (bloco, função de ordem superior, DSL) antes de replicar | Múltiplas implementações da mesma ideia são quase sempre um erro | cap-039, cap-041 |
| Recurso que precisa ser fechado | Usar método com bloco + `ensure` (ou equivalente) | A limpeza fica garantida por construção | cap-041, cap-043 |
| Montar SQL com entrada do usuário | Nunca concatenar; usar ORM ou parâmetros | Concatenar abre SQL injection | cap-042 |
| Reabrir classe ou fazer monkey patching | Usar com parcimônia e organizar em módulos | É poder e risco ao mesmo tempo | cap-040 |
| Escrever numa linguagem nova | Seguir os idiomas dela (nomes, blocos, convenções) | "Quando se está em Roma, faça como os romanos" | cap-040, cap-042 |
| Tratamento de erros | Não envolver tudo em try/catch genérico; deixar falhar de forma visível e supervisionada | Erro engolido vira bug lógico que ninguém percebe | cap-044 |
| Sessão de usuário em memória atrás de load balancer | Guardar a sessão num store compartilhado, em vez de depender de session affinity | Escala horizontal "shared-nothing" | cap-032 |
| Árvore (hierarquia) em banco SQL | Nested set em vez de parent-child, ou documento serializado (XML/JSONB) | Parent-child não traz a árvore inteira num único select | cap-033 |
| Investir tempo de carreira numa tecnologia | Olhar a curva de adoção e sair das que estão em late majority/laggard | O valor do conhecimento é inverso à adoção | cap-032 |
| Escolher toolchain e dependências | Verificar a licença (GPL vs permissiva) e o risco de propriedade intelectual | Licença move ecossistemas inteiros, como o da Apple | cap-036 |

## Armadilhas e mitos
- "Minha linguagem inventou X" → Pattern matching vem de SNOBOL, Prolog e Hope; CSP vem de OCCAM; GC vem de Simula e Basic; inferência de tipos vem de ML. Segundo o autor, até o JSON se inspirou no REBOL. [cap-031]
- "Fortran gera binário mais rápido que C" → Desde o C99, com `restrict` (sem aliasing de ponteiros), o C gera binário igualmente rápido. [cap-030]
- "Java é compilado e JavaScript é interpretado, então Java é mais rápido" → Os dois compilam para bytecode de VM e usam JIT, e os dois acabam gerando código nativo. [cap-034]
- "Linguagem interpretada nunca chega perto de compilada" → Isso era verdade nos anos 90. JIT e VMs evoluíram, e Erlang, que roda em VM com JIT, sustenta o WhatsApp. [cap-034]
- "Escrever assembly na mão é mais eficiente" → Tirando raríssimas exceções, nenhum programador hoje é melhor que um compilador. [cap-034]
- "O compilador conserta código ruim" → Ele não tem como saber que o `SELECT *` vai trazer um milhão de linhas. [cap-034]
- "Código limpo é para o computador" → O computador reescreve tudo. Código limpo é para pessoas, incluindo você às 2 da manhã numa emergência. [cap-034]
- "Docker é máquina virtual" → Não é. [cap-034]
- "Linguagem escrita nela mesma (bootstrapped) é melhor" → Não a torna melhor nem pior; só facilita a vida de quem mantém a linguagem. [cap-035]
- "Basta Python para data science e ML" → A interface é Python; o motor é C, C++, Fortran e GPU. [cap-035]
- "Tem garbage collector, então não preciso pensar em memória" → O GC custa CPU, memória e pausas, e picos incham o heap de forma duradoura. [cap-037, cap-038]
- "Tenho 4 GB e o processo usa 1,5 GB, então sobra" → O espaço de endereço reservado e a fragmentação podem derrubar o processo antes disso. [cap-037]
- "Liberar o objeto devolve a memória ao SO" → Os slots são reaproveitados, mas o heap não encolhe. [cap-038]
- "Tuning de GC resolve vazamento" → A configuração padrão serve para a esmagadora maioria dos casos; o desperdício está no código. [cap-038]
- "GC geracional é sempre melhor" → Ele usa mais memória que reference counting. É troca, não vitória. [cap-038]
- "Design patterns são boas práticas universais" → São sinal de deficiência da linguagem; em outra linguagem, o mesmo pattern é invisível. [cap-039]
- "Um pattern complexo como MVC não pode virar parte da linguagem" → O Rails absorveu o MVC numa DSL web. [cap-039]
- "`private` em Ruby protege de verdade" → `send` contorna; Ruby "não é paternalista" e só sinaliza a intenção. [cap-041]
- "Bloco é a mesma coisa que classe anônima" → Bloco é closure: captura o contexto e pode alterar variáveis de fora (o Java exige `final`). [cap-043]
- "Tipagem estática dispensa testes" → Dá uma falsa sensação de segurança. Teste comportamento, não tipos (o que o autor chama de "chicken typing"). [cap-044]
- "Mais try/catch deixa o sistema mais robusto" → Engole os erros e esconde bugs lógicos que nunca são notados. [cap-044]
- "Seguir o que os grandes nomes recomendam é o caminho seguro" → Para o autor, quem segue já está atrasado; teste você mesmo para que a peça serve. [cap-033]
- **Detalhes datados (o princípio continua valendo)**:
  - Os tutoriais de Ruby (2007/2008, Ruby 1.8) usam `Fixnum`, que foi unificado em `Integer` no Ruby 2.4 (complemento, não está no compêndio). Também usam APIs antigas do Rails (`find :first`) e a diferença `proc`/`Proc.new` que o próprio texto avisa que muda no 1.9. Continuam valendo os conceitos de mensagens, closures, blocos para gerenciar recursos e metaprogramação.
  - cap-037 (2019) diz que o Rust sempre usou jemalloc, mas o Rust passou a usar o alocador do sistema por padrão a partir da versão 1.32, de janeiro de 2019 (complemento, não está no compêndio). Continua valendo que trocar o alocador muda consumo e fragmentação.
  - cap-044 (2015) cita o Elixir 1.2 "futuro", o OTP R18 e o compilador nativo HiPE; cap-034 (2022) já descreve a BEAM com JIT. Continuam valendo a imutabilidade, o isolamento de processos e os supervisores.
  - A errata de cap-036 (2019) já avisa que hoje existem imagens x86 do Android para emuladores.
  - Os percentuais de C, C++ e Fortran nos repositórios (cap-031 em 2018, cap-035 em 2023) são retratos daquele momento.

## Checklist para aplicar num sistema real
- [ ] A linguagem e o runtime foram escolhidos pela categoria do problema (sistema, ferramenta, distribuído, app comercial, mobile, dados) e pela maturidade do ecossistema?
- [ ] Antes de propor reescrita numa linguagem "mais rápida", confirmei que o gargalo está mesmo na linguagem e não no código, nos dados ou no I/O?
- [ ] Sei quais dependências são nativas (C, C++, Fortran) e se o build e o deploy têm a toolchain necessária?
- [ ] Onde há fronteira de FFI ou serialização, quanto de memória é copiado por requisição?
- [ ] O modelo de execução (AOT ou JIT) combina com o perfil do processo (vida longa ou curta, aquecimento, memória)?
- [ ] O runtime e as bibliotecas (regex, datas) se comportam igual em todos os SOs e plataformas-alvo?
- [ ] Algum caminho carrega dados sem limite na memória (arquivo inteiro, SELECT sem WHERE/LIMIT, payload sem teto)?
- [ ] O consumo de memória foi medido no pico, em produção e com dados reais, e não só no boot?
- [ ] Com reference counting, há ciclos que precisam de referência weak?
- [ ] A configuração de alocador e GC está no padrão, e todo tuning tem uma medição que o justifica?
- [ ] Em ambiente sem swap (mobile, container com limite), o que acontece no pico de memória?
- [ ] O estado mutável compartilhado entre threads está no mínimo, e a concorrência usa abstrações do runtime em vez de locks feitos à mão?
- [ ] As falhas ficam visíveis e supervisionadas (reinício, alerta), ou um try/catch genérico as engole?
- [ ] Recursos (arquivos, conexões, transações) são liberados por construção (bloco/`ensure`/`with`)?
- [ ] Nenhum SQL é montado concatenando entrada do usuário?
- [ ] O código segue o idioma da linguagem e é legível para a próxima pessoa? Há testes, que servem "pra você mesmo de amanhã" (cap-033)?
- [ ] Algum boilerplate repetido é um pattern que a linguagem ou uma biblioteca já resolve?
- [ ] As licenças das dependências e da toolchain são compatíveis com o modelo do produto?

## Frases-âncora
> "Linguagens de programação são meras ferramentas. Ferramentas foram feitas para servir seus mestres, e não o contrário." — cap-030

> "Toda boa prática de código é feito pra economizar hora de programador e não hora da máquina." — cap-034

> "Esse é o motivo de porque não jogamos fora Python ou Javascript: porque taxa homem é mais caro do que custo de CPU por hora." — cap-035

> "Maturidade é sempre mais importante que novidade." — cap-035

> "Tudo tem um preço. Quanto mais alguma coisa é conveniente, menos ele tende a ser eficiente." — cap-038

> "Não seja ingênuo de achar que seu programa vai ficar muito melhor fazendo tuning dos alocadores ou garbage collectors, praticamente em todos os casos é seu código que desperdiça memória e tuning nenhum vai resolver isso." — cap-038

> "Patterns são sinais de fraquezas nas linguagens de programação." — cap-039

> "Esse é o objetivo último da boa programação: não ser defensivo, não colocar try/catch em todo lugar porque você está com medo." — cap-044

## Onde aprofundar

| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-030 | [Akitando] #15 - Sua Linguagem NÃO É Especial! (Parte 1) (02/10/2018) | https://www.akitaonrails.com/2018/10/02/akitando-15-sua-linguagem-nao-e-especial-parte-1/ | precisar da genealogia ALGOL → C/Pascal → Java/C#, ou desmontar a ideia de linguagem como identidade |
| cap-031 | [Akitando] #16 - Sua Linguagem NÃO É Especial! (Parte 2) (04/10/2018) | https://www.akitaonrails.com/2018/10/04/akitando-16-sua-linguagem-nao-e-especial-parte-2/ | quiser saber de onde vieram OO, pattern matching, CSP, inferência de tipos e paradigmas como lógico e de arrays |
| cap-032 | [Akitando] #24 - Sua Linguagem É Especial? Parte 1 em 2001 (06/11/2018) | https://www.akitaonrails.com/2018/11/06/akitando-24-sua-linguagem-e-especial-parte-1-em-2001/ | for avaliar quando abandonar uma tecnologia (curva de adoção) ou pensar em sessão compartilhada para escalar horizontalmente |
| cap-033 | [Akitando] #25 - Sua Linguagem É Especial? Parte 2 em 2001 (08/11/2018) | https://www.akitaonrails.com/2018/11/08/akitando-25-sua-linguagem-e-especial-parte-2-em-2001/ | for guardar árvores em SQL (parent-child vs nested set vs documento) ou quiser um exemplo de overengineering consciente |
| cap-034 | [Akitando] #117 - Linguagem Compilada vs Interpretada \| Qual é melhor? (15/04/2022) | https://www.akitaonrails.com/2022/04/15/akitando-117-linguagem-compilada-vs-interpretada-qual-e-melhor/ | precisar explicar lexer, parser, AST, IR, linker, bytecode, JIT vs AOT e linguagem estática vs dinâmica |
| cap-035 | [Akitando] #136 - Python? Java? Rust? Qual a Diferença? \| Discutindo Linguagens (18/01/2023) | https://www.akitaonrails.com/2023/01/18/akitando-136-python-java-rust-qual-a-diferenca-discutindo-linguagens/ | escolher linguagem por categoria de software, entender FFI e marshalling ou o que há em C por baixo de Python, Node e Java |
| cap-036 | [Akitando] #42 - Entendendo Apple, GPL e Compiladores (06/03/2019) | https://www.akitaonrails.com/2019/03/06/akitando-42-entendendo-apple-gpl-e-compiladores/ | precisar entender LLVM vs GCC, licenças GPL vs permissivas, ARC, JIT vs AOT no mobile ou WebAssembly |
| cap-037 | [Akitando] #45 - Gerenciamento de Memória (Parte 1) \| Entendendo Back-end para Iniciantes (Parte 5) (27/03/2019) | https://www.akitaonrails.com/2019/03/27/akitando-45-gerenciamento-de-memoria-parte-1-entendendo-back-end-para-iniciantes-parte-5/ | investigar memória virtual, fragmentação, arenas, ou a escolha entre ptmalloc2, tcmalloc e jemalloc |
| cap-038 | [Akitando] #46 - Gerenciamento de Memória (Parte 2) \| Entendendo Back-end para Iniciantes (Parte 6) (03/04/2019) | https://www.akitaonrails.com/2019/04/03/akitando-46-gerenciamento-de-memoria-parte-2-entendendo-back-end-para-iniciantes-parte-6/ | depurar consumo de memória, pausas de GC, heap inchado ou comparar reference counting, mark and sweep, gerações e compactação |
| cap-039 | Design Patterns representam defeitos nas Linguagens (30/10/2006) | https://www.akitaonrails.com/2006/10/30/design-patterns-representam-defeitos-nas-linguagens/ | for julgar se um pattern está compensando uma falta da linguagem e se existe abstração melhor |
| cap-040 | Micro-Tutorial de Ruby - Parte I (10/11/2008) | https://www.akitaonrails.com/2008/11/10/micro-tutorial-de-ruby-parte-i/ | quiser entender objetos, classes abertas, monkey patching e módulos em Ruby (versão antiga) |
| cap-041 | Micro-Tutorial de Ruby - Parte II (10/11/2008) | https://www.akitaonrails.com/2008/11/10/micro-tutorial-de-ruby-parte-ii/ | precisar de mensagens, `send`/`method_missing`, blocos, lambdas e `map`/`select`/`inject` |
| cap-042 | Micro-Tutorial de Ruby - Parte III (10/11/2008) | https://www.akitaonrails.com/2008/11/10/micro-tutorial-de-ruby-parte-iii/ | olhar Symbols, parâmetros opcionais, splat, reflexão vs metaprogramação, `nil` e o alerta sobre SQL injection |
| cap-043 | Anatomia de Ruby Blocks/Closures (30/11/2007) | https://www.akitaonrails.com/2007/11/30/anatomia-de-ruby-blocks-closures/ | precisar explicar closures, `yield`, `&`, Proc e lambda, ou o padrão de bloco para recursos e transações |
| cap-044 | O Obrigatório "Por Que Elixir?" - Visão Pessoal (01/12/2015) | https://www.akitaonrails.com/2015/12/01/the-obligatory-why-elixir-personal-take/ | projetar concorrência e tolerância a falhas (imutabilidade, schedulers, supervisores, hot swap) ou decidir quando Erlang/Elixir vale a pena |
