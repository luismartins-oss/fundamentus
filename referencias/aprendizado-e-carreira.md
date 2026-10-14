# Aprendizado e carreira
> Destilado de: cap-001, cap-002, cap-003, cap-004, cap-005, cap-006, cap-007, cap-099, cap-100, cap-101, cap-102, cap-103, cap-104, cap-105, cap-106, cap-107, cap-108 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).

## Ideia central
Programação é uma profissão de prática. Começar é trivial, mas virar profissional exige milhares de horas de prática deliberada, dolorosa e conduzida pela própria pessoa sobre fundamentos que não envelhecem (algoritmos, estruturas de dados, matemática, como o computador funciona), e não sobre a ferramenta da moda. Cursos, livros e faculdade são "rodinhas na bicicleta": servem de introdução, mas o aprendizado real é resolver problemas sem passo a passo, achar o próprio material e se avaliar com honestidade. Na carreira, o valor de cada um é definido por oferta e procura num sistema não linear: atravessa bolhas e crises quem se define como solucionador de problemas (e não como "programador da linguagem X") e toma as próprias decisões aceitando perdas pequenas. Ao orientar alguém, o agente deve devolver critérios e perguntas, não receita pronta, porque, segundo o autor, quem não paga pelo erro (*skin in the game*) não pode decidir por você.

## Conceitos que todo sistema depende

### Como se aprende de verdade
- **Profissão de prática** — Programar é como cozinhar, desenhar ou praticar esporte: qualquer um começa, mas a barreira até técnica e qualidade profissionais é altíssima, e nenhum curso curto leva até ela. [cap-001, cap-002, cap-003, cap-099]
- **Aprender a aprender = "se virar"** — O método do autor cabe em três regras: receber problemas para resolver, não ter material oficial (achar o material faz parte do treino) e não ter chefe ou professor que entregue tudo mastigado. Aprender é ver o problema, aceitar que falta conhecimento, buscá-lo sozinho, testar hipóteses, falhar e criticar o que fez. [cap-001, cap-103]
- **Autodidata × passivo** — Quando um passo do procedimento falha, o passivo trava e espera que alguém conserte; o autodidata volta passos, pesquisa, pergunta e insiste. Por isso o autodidata avança com qualquer material, e o passivo não avança com nenhum. [cap-003]
- **A dor de aprender** — Frustração, vergonha de errar em público e cansaço indicam que se está no lugar certo ("no pain, no gain"). Quem vê a dor como defeito do curso vai trocar de curso para sempre. Adultos aprendem pior que crianças por medo de serem julgados: é preciso perder a vergonha de cair e desaprender os reflexos errados. [cap-001, cap-003, cap-007]
- **Prática deliberada** — Não é "praticar muito". É mirar exatamente no que ainda está ruim, ficar na zona de aprendizado (entre a zona de conforto e a de pânico), repetir em alto volume, com foco total e sozinho, e avaliar cada repetição contra um padrão relevante. As metas são de processo, não de resultado. [cap-006]
- **O trabalho não basta** — O emprego é quase sempre zona de conforto: serve para aplicar o que se estudou e aprender a se comunicar com colegas, chefes e clientes. Quem tenta aprender tudo só no expediente evolui devagar demais. [cap-006, cap-099]
- **Auto-avaliação e metacognição** — Os melhores se observam enquanto praticam, se julgam contra padrões específicos e assumem a culpa pelos próprios erros. Elogio de terceiros sabota a auto-avaliação; a crítica só vale como sinal de uma falha nela. [cap-006, cap-102]
- **Talento é mito** — Einstein e Darwin consolidaram o trabalho de contemporâneos (Poincaré, Lorentz, Wallace), e Mozart teve 18 anos de treino intensivo antes de compor a primeira obra hoje considerada prima. O que se chama talento é volume massivo de prática deliberada, somado a contexto e sorte. [cap-005, cap-006]
- **Referências, não ídolos; jogo de longo prazo** — Ninguém é semideus a ser copiado: use pessoas do passado como professores e trate a vida inteira como um processo deliberado de longo prazo. O autor esperou até os 34 anos para abrir a empresa porque sabia que aos 30 ainda não era bom. [cap-006, cap-100]
- **Copiar para ganhar fluência** — Nos primeiros meses, digitar código alheio (GitHub, livros) por horas, quebrar e combinar trechos sem se importar se está certo cria memória muscular e faz os padrões aparecerem. O objetivo do programa surge depois, quando já se sabe o que cada trecho faz. [cap-003]
- **Pirâmides do conhecimento e 80/20** — Aprenda o topo (a parte fácil) de muitas pirâmides e desça conforme a prática exigir; leia a introdução de dez livros e pratique mais do que lê. A armadilha: o topo (20% do conhecimento) resolve 80% dos problemas e dá a falsa impressão de já ser bom. [cap-101, cap-001]
- **Aprender fica mais rápido** — Com fundação e o hábito de aprender sozinho, o tempo cai de meses (Basic e dBase, sem referência) para semanas ou cerca de um mês até ficar produtivo numa linguagem nova. Produtivo, não expert. [cap-003, cap-103]
- **Foco e isolamento** — Estudar exige concentração total: celular em "não perturbe", redes sociais fora e horário sem interrupção (o autor usa a madrugada). Excesso de informação e opinião de influencer é custo, não ajuda. [cap-003, cap-006, cap-108]
- **Gente experiente por perto** — O iniciante precisa de contato direto com pessoas mais experientes e de ajudar quem está um passo atrás; o autor considera o home office prejudicial a quem está começando. (Datado: opinião de 2020; o princípio de manter correção rápida feita por quem sabe mais continua valendo.) [cap-001, cap-102]

### Fundamentos
- **Codificar ≠ programar** — Montar frameworks, plugins e templates é commodity, e o valor de uma commodity só cai. O valor está em extrair mais resultado com menos custo, e isso exige dominar a base: busca por *vector space model* (álgebra linear) em vez de LIKE, filtro com classificador bayesiano (estatística), roteador com parser (compiladores). [cap-004]
- **Fundação antes do hype** — Algoritmos e estruturas de dados separam o hobista do profissional e exigem estudo formal com centenas de repetições; design patterns e arquitetura vêm depois. Quem conhece processos, threads, alocação de memória, schedulers e a diferença entre hypervisor e container entende o "porquê" e não se perde a cada ferramenta nova. [cap-003, cap-001]
- **Conhecimento de base × temporário** — Quanto mais específico e de alto nível (o Flash dos anos 2000, por exemplo), mais útil no dia 1 e mais rápido envelhece. Quanto mais de baixo nível (cálculo, estatística, compiladores), menos prático no começo e mais tempo dura. [cap-101, cap-102, cap-103]
- **Pensamento matemático** — Não se trata de decorar equações, e sim de enxergar dados, limites e complexidade e desconfiar de procedimentos. Exemplo do autor: 0,5% ao mês em 7 fundos não rende 42% ao ano. Cursos não ensinam isso porque os alunos não pagariam para aprender. [cap-103, cap-004]
- **Pattern ≠ standard ≠ default** — Em português tudo vira "padrão", mas pattern é só algo que se repete: pode ser um anti-pattern e envelhece. Design patterns servem para dar nome, depois de muito código, ao que você já fazia. [cap-007, cap-003]
- **Ferramenta não é identidade** — Linguagens são ferramentas e não merecem lealdade. Quem se apega à primeira vira escravo de marca, como o programador de Visual Basic que perdeu todas as ondas seguintes. [cap-001, cap-004, cap-103]
- **Ombros de gigantes e história** — A ciência da computação é cumulativa e a maior parte da "inovação" é redescoberta. Leia os clássicos e os autores originais (Knuth, Dijkstra, Wirth, SICP, K&R, o livro do Dragão) antes dos gurus de técnica, e estude a história da área para reconhecer para onde as coisas tendem a ir. [cap-004, cap-007, cap-108]
- **Inglês como pré-requisito** — Tudo de novo sai primeiro em inglês, e depender de tradução é ficar atrasado. Quem aprende uma tecnologia antes da versão 1.0 cobra muito mais que quem chega na versão 4, com o mercado já cheio de cursos. Escreva código em inglês para não fechar a porta ao open source nem a times de fora. [cap-004, cap-007, cap-001]

### Formação
- **Tipos de curso superior** — Ciência da Computação é, na visão do autor, uma especialização da matemática aplicada (dois anos quase só de matemática). Engenharia da Computação é hardware, uma especialização da elétrica. Sistemas de Informação é a herdeira de Processamento de Dados (automação de processos corporativos). Engenharia de Software aplica ciência para chegar a soluções economicamente viáveis. O bacharelado é mais generalista; o tecnólogo, mais curto e focado. [cap-102]
- **Faculdade não forma pronto** — Ao final de qualquer curso a pessoa ainda está longe de pronta; a escola condicionou você a seguir currículo e tirar 5, e no mercado "todo dia é uma prova". O sucesso depende mais do tipo de pessoa do que do curso escolhido. [cap-102, cap-105, cap-103]
- **Seleção** — Universidade pública com concorrência alta seleciona melhor que curso pago sem filtro, que tem incentivo para manter qualquer pagante. O mesmo vale para um curso comercial que nunca diz a alguém que ele não leva jeito. [cap-001]
- **Estágio** — No Brasil é regido por termo de compromisso com a instituição de ensino, com limite de 2 anos; o autor lamenta que não exista mais estágio sem vínculo com escola. (Regra citada em 2019; confira a legislação vigente, complemento, não está no compêndio.) [cap-102]

### Carreira e mercado
- **Oferta e procura** — Salário é preço: você raramente vale o que acha. Na bolha, a demanda infla o preço de qualquer um; na crise, sobra oferta e o preço cai. O que é fácil de aprender vale pouco, porque qualquer um aprende. [cap-002, cap-103, cap-001]
- **Ciclos de mercado** — O autor descreve a bolha de 2009 a 2020 (startups queimando caixa, equity como promessa, contratação em atacado) estourando com a COVID, e antes dela a pontocom de 2001. (Datado; o princípio vale: abundância esconde quem não entrega, e nas crises ficam os que resolvem problemas reais.) [cap-001, cap-002, cap-103, cap-108]
- **Média salarial não existe** — Salários, seguidores e valor de empresas seguem *power laws* em redes complexas e não lineares, não a distribuição normal, então não há "média" onde se encaixar. A curva pessoal pode crescer de forma não linear (é possibilidade, não garantia) para quem investe em técnica, comunicação, negociação e rede de contatos real. [cap-104]
- **Níveis de experiência** — Júnior, pleno e sênior podem entregar a mesma qualidade; o que muda é o tempo e o número de tentativas. Sênior é quem toma decisões confiáveis e assume o ônus delas, delega, orienta e "escala horizontalmente" os júniors; o "10X" raro é o especialista em problemas de escala. O pleno corre o risco do efeito Dunning-Kruger e de fugir para as tarefas que já domina. [cap-102]
- **Confiança e reputação** — Subir de nível é acumular confiança entregando com qualidade constante. Manter testes que ninguém pediu constrói reputação, cortar caminho a destrói, e confiança quebrada custa caro para reconstruir. [cap-102, cap-101]
- **Networking de verdade** — Não é evento, cartão nem LinkedIn: é resolver problemas de forma consistente até os colegas contarem com você. O júnior que está ao seu lado hoje vira diretor, CTO ou fundador amanhã. [cap-099, cap-108, cap-104]
- **Tipos de empresa** — Públicas; grandes empresas com sistemas de back-office; empresas de produto e startups; e terceirizadas (agências, body-shops, consultorias). Iniciante precisa de estrutura e de gente experiente ao lado; body-shop e freelance cedo demais levam à estagnação. [cap-099]
- **Curva de adoção em S** — Toda tecnologia começa devagar, acelera na *early majority* e desacelera. Ganha mais quem já domina a tecnologia quando a curva acelera. O autor procura ser pleno no que está na *early majority* e *early adopter* de algo novo, reavaliando a cada ~5 anos em ciclos de ~10. [cap-102, cap-105, cap-007]
- **Ame o seu esforço, não "o que você ama"** — Satisfação vem da qualidade do próprio resultado, não do cargo nem da empresa. Melhorar um pouco por dia e ir entender vendas, marketing, suporte e jurídico gera a experiência que depois vira oportunidade. [cap-106]
- **Legado e trabalho "chato" como oportunidade** — Domine a ferramenta do trabalho e exercite as novas nas horas vagas: competência técnica só dá voz quando é óbvia. Um conserto desprezado num sistema ASP levou ao maior projeto Java do autor; os pontos só se conectam olhando para trás. [cap-100]
- **Especialista em resolver problemas** — De 1996 a 2001, o autor aprendeu e produziu numa plataforma nova por ano (Director, Flash, ASP, PHP, Perl, SQL Server...) e passou ileso pelo crash porque nunca se definiu por ferramenta nem por modelo de negócio. [cap-108]

### Decisão
- **Não terceirize decisões** — Um influencer não conhece sua história nem seus objetivos e não paga pelo seu erro. Quem dá resposta fácil para pergunta subjetiva provavelmente quer vender algo. Estude, decida, erre, assuma o prejuízo e não repita o erro. [cap-105]
- **Antifragilidade e custo perdido** — Faça apostas em que a perda é pequena e o ganho possível é grande, e largue o livro ou curso ruim no meio em vez de "honrar" o tempo já gasto. [cap-105]
- **Contexto, contexto, contexto** — As soluções de Netflix, Google, Spotify ou Nubank valem no contexto e na escala dessas empresas. No dia 1, a sua preocupação é durar até o dia 2. [cap-105]
- **Software livre ≠ open source** — Software livre (GPL e AGPL, as quatro liberdades) protege o usuário; o open source com licença permissiva (MIT, BSD) protege a liberdade de programadores e empresas, que abrem o que é commodity e fecham o núcleo do negócio. Para quem aprende, o valor está em código de graça para ler e feedback de graça nas contribuições. (Exemplos de AWS e SaaS de 2020 são datados.) [cap-107]

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Zero absoluto: por onde começar? | Use qualquer curso ou tutorial só para dominar o ambiente (editor, salvar, rodar, terminal, Git e GitHub); depois passe dias copiando código alheio | O curso entrega o "1%" inicial; a fluência vem do volume de código | cap-003, cap-001 |
| Qual curso ou livro escolher? | Documentação oficial primeiro; depois pesquise "best books of X", pegue o primeiro e troque se não servir | A diferença entre materiais é marginal, e avaliar material faz parte do treino | cap-001, cap-003, cap-105 |
| Curso, livro ou prática? | Curso e livro como aquecimento (com base, até em velocidade 2x); o aprendizado acontece na prática deliberada sobre problemas | Cursos ensinam o "o quê" de forma linear; o mundo real exige o "porquê" | cap-001, cap-003, cap-103 |
| Iniciante quer "fazer um app do zero" | Adie: copie, quebre e combine código pronto até saber o que cada trecho faz; o objetivo aparece sozinho | As ideias iniciais são simplórias e quase tudo que se imagina já existe | cap-003 |
| Comprou o livro de Design Patterns ou quer estudar arquitetura | Antes, algoritmos e estruturas de dados com material formal e muitos exercícios; patterns só depois de muito código | Patterns decorados sem experiência são aplicados errado | cap-003 |
| Fazer faculdade? | Se tem tempo e dinheiro, faça; para o longo prazo prefira Ciência da Computação ou engenharias, em bacharelado; SI ou tecnólogo servem se há dúvida | Matérias de base sobrevivem ao tempo; os 2 anos de matemática pouparam anos ao autor, que largou o curso mas não recomenda largar como primeira opção | cap-101, cap-102, cap-103 |
| Sem faculdade | Monte o próprio currículo de base: aulas gravadas de grandes universidades (cálculo, álgebra, estatística) e os livros clássicos | A base está disponível de graça; falta disposição para uns 2 anos de estudo | cap-103, cap-004 |
| Mudar de carreira aos 40 | Aceite começar no nível de um calouro e engula o ego por anos; procure empresas ativamente; evite concurso público | Ego frágil e dependência de validação inviabilizam a troca | cap-099 |
| Primeiro emprego | Priorize estar perto de gente mais experiente, com desafios e revisão de código: consultoria séria, empresa de produto ou grande empresa (agência para quem é mais arrojado) | O júnior cresce quando alguém aponta seus erros rápido | cap-099, cap-102 |
| Oferta de body-shop ou freelance logo no início | Não aceite como primeiro emprego | Sem orientação, trabalha muito por pouco resultado e estagna | cap-099 |
| Quanto ganhar no começo | Não otimize salário nos primeiros anos (salvo abuso ou falta do mínimo para viver); pense no Δt, não no t0 | O iniciante está na faixa com mais oferta, e a curva é não linear | cap-099, cap-102, cap-104 |
| Comparar-se com a "média salarial" | Ignore a média; avalie o que você produz, negocie e aumente competência e rede de contatos | Salário segue *power law*; o que você "acha" que vale é só o preço de abertura da negociação | cap-104, cap-102 |
| Cidade pequena, poucas vagas | Aprenda o que as empresas locais usam (em 2018 o autor citava Java, C#, PHP e JS, recomendação datada); estude novidades nas horas vagas sem esperar emprego nelas; considere trabalho remoto ou mudança; use o open source para ter contato e feedback | Dominar a ferramenta do trabalho dá voz, e conhecimento é o único bem que não se perde | cap-100, cap-104, cap-107 |
| Caiu num sistema legado "feio" | Aceite e resolva o problema real do usuário antes de propor reescrever | Melhorar legado é uma competência rara e abre as próximas oportunidades | cap-100 |
| Iniciante afogado em eventos, fóruns e influencers | Corte o ruído nos primeiros anos e foque no próprio esforço | Questionar só funciona com experiência; correr atrás do hype seria desastroso | cap-100, cap-108 |
| Escolher tecnologia "porque o Google usa" | Pergunte pelo seu contexto, faça prova de conceito e lembre que no dia 1 a meta é chegar ao dia 2 | Soluções de big tech pressupõem escala e recursos que você não tem | cap-105 |
| Pedir conselho de carreira | Faça perguntas objetivas, dê peso só a quem tem *skin in the game* e desconfie de respostas fáceis | Opinião de quem não paga pelo erro vale tanto quanto jogar uma moeda | cap-105 |
| Em que tecnologia apostar | Monte um portfólio: domine o que está estabelecido, seja pleno no que está acelerando e experimente cedo algo novo; reavalie a cada ~5 anos | Ninguém prevê a próxima curva, e apostas pequenas limitam a perda | cap-102, cap-105, cap-007 |
| Livro ou curso ruim pela metade | Pare | Insistir só soma custo perdido | cap-105 |
| Ler o livro técnico inteiro? | Com pouca maturidade (por volta dos 20 anos), leia do começo ao fim; com base, leia a introdução e volte quando precisar | A técnica de estudar amadurece; não queime etapas | cap-105, cap-101 |
| Inglês fraco | Imersão de graça já: sistema e navegador em inglês, legendas em inglês (depois sem), ler e ouvir todo dia, código em inglês e aceitar correções | Novidades e oportunidades chegam antes em inglês, e ser compreensível já basta | cap-007, cap-004 |
| Insatisfeito no emprego | Antes de trocar, faça seu trabalho melhor, um pouco por dia, e vá entender as áreas que geram o problema (vendas, marketing, suporte) | A frustração que vem do esforço mínimo acompanha você no próximo emprego | cap-106 |
| Mercado aquecido oferece vaga após curso curto | Pode aceitar, mas não pare de estudar fundamentos (na 42 SP, quem larga no "nível 7" de 21 é o exemplo) | O topo da pirâmide engana, e quem parou fica frágil quando o ciclo vira | cap-001, cap-101 |
| Crise ou layoff | Use o tempo para reforçar a fundação e procure vaga ativamente: as empresas continuam precisando manter tecnologia | Na baixa, o valor vai para quem resolve problemas reais (o autor ficou uma semana desempregado no crash de 2001) | cap-001, cap-108, cap-002 |
| Contribuir com open source | É opcional para iniciante; leia muito código aberto, contribua para "coçar a própria coceira" e nunca use código sem licença | Rende feedback de graça e portfólio; GPL e AGPL são licenças virais | cap-107, cap-003 |
| Quanto praticar por dia | Sessões de 60 a 90 min, no máximo 4 a 5 h/dia de prática deliberada, sozinho e sem distrações | O limite é a capacidade de manter a concentração | cap-006 |
| Saber se está ficando competente | Use o termômetro do autor: entregar com poucos bugs e dentro do horário; se precisa varar noite, ainda falta experiência | Evita confundir 20% de conhecimento com competência | cap-101, cap-108 |

## Armadilhas e mitos
- "Programar é fácil, qualquer um consegue" → começar é fácil; a profissão não é para todos, assim como o esporte profissional. O autor diz que, de propósito, nunca incentivou ninguém a virar programador. [cap-002]
- "Viro programador com 2 semanas ou 2 meses de bootcamp" → isso dá no máximo para tarefas simples. Pela conta do autor, do zero ao primeiro nível de júnior são ~1.400 h (≈700 h de curso + ≈700 h de estágio), e um workshop de 80 h cobre ~5% do caminho. [cap-002]
- "Fulano nasceu com talento" → não existe gene do gênio; existe prática deliberada, contexto e sorte. Recordes olímpicos de 1908 hoje são batidos por colegiais porque o treino melhorou, não o corpo. [cap-005, cap-006]
- "10 mil horas resolvem" ou "anos de casa = senioridade" → anos na área quase não se correlacionam com excelência; repetir o mesmo tipo de projeto é quantidade, não qualidade. [cap-006, cap-104]
- "O curso é que era ruim; falta achar o livro certo" → nenhum material corrige a falta de disposição para a dor; quem desiste de um programa difícil não aprende melhor em outro lugar. [cap-003, cap-001]
- "Certificado prova que eu sei" → a certificação atesta que você assistiu às aulas e pagou. O autor tirou o PMP e continuou júnior em gestão por anos. [cap-001]
- "Montar site com framework já é programar" → isso é codificar; o valor está nos fundamentos que permitem soluções mais baratas e melhores. [cap-004]
- "Matemática é inútil para quem programa" → busca, classificadores, machine learning, análise de complexidade e até a conta de juros dependem dela. [cap-004, cap-103]
- "Design pattern é o jeito certo" → pattern é repetição observada, não norma; muitas vezes é defeito da linguagem virando boilerplate. [cap-003, cap-007]
- "Minha linguagem é a melhor" → ferramenta não merece lealdade, e quem só tem martelo acaba martelando parafuso. [cap-001, cap-103]
- "Existe média salarial, e o normal é ficar na média" → salário segue *power law*, não a curva normal; "mediano" e "medíocre" têm a mesma raiz. [cap-104]
- "Sênior é quem escreve o código mais bonito" → sênior é quem decide bem, assume a responsabilidade e multiplica o resultado dos outros. [cap-102]
- "Elogio mostra que estou indo bem" → elogio confunde a auto-avaliação e prende na zona de conforto; é a crítica que pode apontar uma falha. [cap-102, cap-006]
- "Aprendo tudo no trabalho" → o trabalho aplica o que foi estudado e fica quase todo na zona de conforto. [cap-006, cap-099]
- "Faça o que ama e o dinheiro vem" → ame o esforço por trás do resultado; todo trabalho tem partes chatas. [cap-106]
- "Gates largou a faculdade, então eu posso largar" → primeiro entre em Harvard. O autor largou o curso, mas não recomenda isso como primeira opção. [cap-105, cap-101]
- "Se o Google usa, serve para mim" → o problema não é a tecnologia, é decidir sem contexto. [cap-105]
- "Inglês é imperialismo" ou "preciso de curso caro e intercâmbio" → é desculpa; a imersão em casa custa zero, e o sotaque importa menos do que se pensa. [cap-007, cap-002]
- "Se o mercado me contrata, é porque sou bom" → na bolha os critérios caem e iniciantes são supervalorizados; quando o ciclo vira, ficam os que entregam. [cap-001, cap-002, cap-103]
- "Networking é evento e LinkedIn" → é reputação de entrega entre os colegas. [cap-099, cap-108]
- "Preciso de um plano de 10 anos e saber se a tecnologia vai vingar" → os pontos só se conectam olhando para trás, e quem espera 2 ou 3 anos "para ver" perde a onda. [cap-100, cap-108]
- "Usar Linux e JavaScript é defender software livre" ou "abrir o código atrai contribuidores" → quase tudo que se usa no dia a dia é proprietário ou SaaS; abrir o código não garante adoção e mantê-lo custa caro. [cap-107]
- "Varar noite é admirável" → o autor varava noites porque prometia mais do que conseguia entregar, e não recomenda copiar isso. [cap-108, cap-101]
- "Chegar criticando o código de todos mostra competência" → o e-mail crítico que o autor mandou na primeira semana fez todos o odiarem, e a equipe já sabia dos problemas. Se os outros não fazem do seu jeito, talvez o errado seja o seu jeito. [cap-108, cap-099]

## Checklist para aplicar num sistema real
(Para montar um plano de estudo do zero ao expert ou avaliar uma trilha de aprendizado.)

### Antes de começar
- [ ] Teste de interesse feito: tecnologia prende a atenção por horas, e 1.000 a 2.000 horas de prática não parecem maçantes? Se não, o autor nem começaria. [cap-002]
- [ ] Expectativa calibrada: júnior em ~1.400 h e profissional em anos; nenhum curso curto encurta isso. [cap-002]
- [ ] Inglês em imersão desde o dia 1. [cap-007]
- [ ] Ambiente de foco montado: celular em "não perturbe" e redes sociais fora do horário de estudo. [cap-003]

### Iniciante
- [ ] Ambiente básico dominado: editor, rodar código, terminal, Git e GitHub. [cap-003]
- [ ] Centenas de horas copiando, quebrando e combinando código alheio. [cap-003]
- [ ] Algoritmos e estruturas de dados com material formal e muitos exercícios, antes de patterns, arquitetura ou framework da moda. [cap-003]
- [ ] Conceitos de base estudados: processos, threads, memória, scheduler, hypervisor × container. [cap-001]
- [ ] Matemática estudada: cálculo, álgebra linear, estatística e probabilidade (na faculdade ou em aulas gravadas). [cap-004, cap-103]
- [ ] Rotina de prática deliberada: metas de processo, sessões curtas, auto-avaliação contra um padrão explícito. [cap-006]
- [ ] Leitura de código de projetos abertos virou hábito. [cap-107]

### Júnior → pleno
- [ ] Primeiro emprego com gente mais experiente por perto e revisão de código obrigatória ("quem fez não mergeia"). [cap-099, cap-102]
- [ ] Estudo continua fora do expediente. [cap-099, cap-100]
- [ ] Qualidade que ninguém pediu (testes) mantida em todo o código. [cap-102]
- [ ] Busca deliberada de tarefas em que ainda é júnior, contra o efeito Dunning-Kruger. [cap-102]
- [ ] Patterns e arquitetura estudados agora, para dar nome ao que já faz. [cap-003]
- [ ] Clássicos e autores originais no lugar de resumos de influencer. [cap-004, cap-108]

### Pleno → sênior → expert
- [ ] Toma decisões e assume o ônus; orienta júniors e revisa o código deles. [cap-102]
- [ ] Mantém portfólio de tecnologias em pontos diferentes da curva de adoção, revisto a cada ~5 anos. [cap-102]
- [ ] Se apresenta como solucionador de problemas, não como "dev da linguagem X". [cap-108]
- [ ] Estuda a história da área para ler tendências. [cap-007]

### Avaliar uma trilha, curso ou escola
- [ ] Tem fundação obrigatória antes do conteúdo "cool" (por exemplo, algoritmos antes de JavaScript)? [cap-001]
- [ ] É difícil o bastante para filtrar? Seleciona quem entra? [cap-001]
- [ ] Entrega problemas em vez de passo a passo e obriga o aluno a achar material e formular perguntas? [cap-001]
- [ ] Põe o aprendiz ao lado de gente mais experiente e o faz ajudar quem está atrás? [cap-001]
- [ ] Ensina o "porquê", ou só a ferramenta, que tem validade curta? [cap-103, cap-003]
- [ ] Promete emprego ou salário garantido? Desconfie. [cap-105, cap-002]

## Frases-âncora

> "Programar não é escrever qualquer código da mesma forma que culinária não é jogar qualquer ingrediente numa panela." — cap-004

> "Esquece estudar Design Patterns ou Arquiteturas de Sistemas antes de aprender Estruturas de Dados e Algoritmos." — cap-003

> "Alta repetição é a diferença mais importante entre prática deliberada de uma tarefa e performar a tarefa na realidade, só quando conta. Você não vai evoluir só no horário do seu trabalho." — cap-006

> "Deixa eu te contar um dos grandes segredos da nossa área: qualquer imbecil consegue começar um projeto novo do zero. Mas só caras realmente bons conseguem melhorar sistemas legados." — cap-100

> "Faculdades, cursos, e qualquer tipo de instituição de ensino tradicional não são ruins. Mas elas são rodinhas na bicicleta." — cap-103

> "Você não faz parte de uma média, você faz parte de uma cauda longa. E na cauda longa, se você não fizer nada de diferente, o limite tende a zero." — cap-104

> "O que você estuda é menos importante do que “como” você estuda." — cap-105

> "Sem planejar muito, eu só me especializei em resolver problemas de tecnologia. Minha arma secreta foi Aprender a Aprender." — cap-108

## Onde aprofundar

| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-001 | [Akitando] #76 - Guia DEFINITIVO de Aprendendo a Aprender \| A maior BRONCA da sua vida [RATED R] (01/04/2020) | https://www.akitaonrails.com/2020/04/01/akitando-76-guia-definitivo-de-aprendendo-a-aprender-a-maior-bronca-da-sua-vida-rated-r/ | precisar do método completo (problemas, sem material oficial, sem chefe), da crítica a cursos e certificados e do modelo da 42 SP |
| cap-002 | [Akitando] #72 - RANT: Programação NÃO É Fácil (19/02/2020) | https://www.akitaonrails.com/2020/02/19/akitando-72-rant-programacao-nao-e-facil/ | alguém achar que vira dev em semanas, ou quiser a conta das ~1.400 h e o teste de vocação |
| cap-003 | [Akitando] #65 - A Dor de Aprender \| Que Cursos/Livros? (23/10/2019) | https://www.akitaonrails.com/2019/10/23/akitando-65-a-dor-de-aprender-que-cursos-livros/ | for montar a primeira fase de estudo: copiar código, ordem fundamentos → patterns, foco |
| cap-004 | [Off-Topic] Carreira em Programação - Codificar não é Programar (02/05/2014) | https://www.akitaonrails.com/2014/05/02/off-topic-carreira-em-programacao-codificar-nao-e-programar/ | precisar justificar matemática e CS com exemplos concretos (VSM, Bayes, parsers) e a lista de clássicos |
| cap-005 | [Akitando] #26 - Talento: Matando Semi-Deuses (Parte 1) (13/11/2018) | https://www.akitaonrails.com/2018/11/13/akitando-26-talento-matando-semi-deuses-parte-1/ | o aprendiz se achar sem "dom", ou para mostrar que conhecimento é cumulativo |
| cap-006 | [Akitando] #27 - Talento: Matando Semi-Deuses (Parte 2) (15/11/2018) | https://www.akitaonrails.com/2018/11/15/akitando-27-talento-matando-semi-deuses-parte-2/ | for desenhar a rotina de prática deliberada (zonas, repetição, foco, auto-avaliação) |
| cap-007 | [Akitando] #32 - Como eu aprendi Inglês? E entendendo "padrões"! (11/12/2018) | https://www.akitaonrails.com/2018/12/11/akitando-32-como-eu-aprendi-ingles-e-entendendo-padroes/ | o bloqueio for o inglês, ou para separar pattern de standard e default |
| cap-099 | [Akitando] #36 - O Mercado de TI para Iniciantes em Programação \| Série "Começando aos 40" (23/01/2019) | https://www.akitaonrails.com/2019/01/23/akitando-36-o-mercado-de-ti-para-iniciantes-em-programacao-serie-comecando-aos-40/ | alguém mudar de carreira ou escolher o primeiro tipo de empresa |
| cap-100 | [Akitando] #29 - O que eu devo estudar? Vou conseguir emprego? (27/11/2018) | https://www.akitaonrails.com/2018/11/27/akitando-29-o-que-eu-devo-estudar-vou-conseguir-emprego/ | o dilema for "linguagem com vaga × linguagem nova", ou o trabalho atual for legado |
| cap-101 | [Akitando] #19 - Devo fazer Faculdade? (Eu, 22 anos depois) (18/10/2018) | https://www.akitaonrails.com/2018/10/18/akitando-19-devo-fazer-faculdade-eu-22-anos-depois/ | discutir largar a faculdade, a teoria das pirâmides e a armadilha do 80/20 |
| cap-102 | [Akitando] #64 - Começando na Carreira de TI \| Faculdade? Níveis de Experiência? (16/10/2019) | https://www.akitaonrails.com/2019/10/16/akitando-64-comecando-na-carreira-de-ti-faculdade-niveis-de-experiencia/ | precisar comparar cursos ou definir júnior, pleno, sênior, 10X, revisão de código e curva de adoção |
| cap-103 | [Akitando] #90 - O que os Cursos NÃO te Ensinam sobre Mercados (06/01/2021) | https://www.akitaonrails.com/2021/01/06/akitando-90-o-que-os-cursos-nao-te-ensinam-sobre-mercados/ | quiser um resumo de oferta e procura, pensamento matemático e "curso de ferramenta × curso que ensina a aprender" |
| cap-104 | [Akitando] #111 - RANT: Média Salarial NÃO Existe \| Entendendo Power Laws (20/12/2021) | https://www.akitaonrails.com/2021/12/20/akitando-111-rant-media-salarial-nao-existe-entendendo-power-laws/ | alguém usar média salarial para decidir, ou para entender crescimento não linear |
| cap-105 | [Akitando] #63 - Não Terceirize suas Decisões! \| A Lição MAIS Importante da sua Vida (09/10/2019) | https://www.akitaonrails.com/2019/10/09/akitando-63-nao-terceirize-suas-decisoes-a-licao-mais-importante-da-sua-vida/ | a pergunta for "o que eu devo fazer?", ou houver cópia de solução de big tech sem contexto |
| cap-106 | [Akitando] #8 - Procure o que você Ama ... SÓ QUE NÃO! (06/09/2018) | https://www.akitaonrails.com/2018/09/06/akitando-8-procure-o-que-voce-ama-so-que-nao/ | houver insatisfação no emprego ou espera pela "grande oportunidade" |
| cap-107 | [Akitando] #73 - RANT: A Realidade do "Software Livre" (04/03/2020) | https://www.akitaonrails.com/2020/03/04/akitando-73-rant-a-realidade-do-software-livre/ | for decidir sobre licenças ou contribuição em open source, ou separar software livre de open source |
| cap-108 | [Akitando] #138 - Meu Começo de Carreira Durante a Bolha - 1997 a 2002, Bug do Milênio, Crash (14/02/2023) | https://www.akitaonrails.com/2023/02/14/akitando-138-meu-comeco-de-carreira-durante-a-bolha-1997-a-2002-bug-do-milenio-crash/ | quiser um exemplo de trajetória atravessando bolha e crash sem depender de uma tecnologia |

Os capítulos deste tema são de 2014 a 2023, anteriores aos agentes de IA. O efeito dos LLMs sobre a profissão aparece na seção "Programando na Era da IA" do compêndio (cap-109 a cap-116; ver `mapa.md`). (complemento, não está no compêndio destilado aqui)
