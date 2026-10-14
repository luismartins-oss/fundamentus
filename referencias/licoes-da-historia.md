# Lições da história

> Destilado de: cap-117, cap-118, cap-119, cap-120 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).

## Ideia central

Na leitura do autor, o Ruby on Rails não venceu por ser "tecnicamente superior". Venceu porque foi extraído de um produto real, trocou configuração por convenção, veio com opinião forte, segurança e testes ligados por padrão, formou uma comunidade coesa e chegou na hora certa. Pra decidir framework hoje, as lições são: olhar o tipo de aplicação (a maioria é "estilo Basecamp"), quem depende da ferramenta pra sobreviver e qual a relação custo-benefício, e não a moda nem o "e se amanhã eu precisar?". Escala é problema de arquitetura, não de linguagem, e reescrever do zero raramente compensa. Licença e propriedade são parte da arquitetura: depender da boa vontade do dono de uma tecnologia é uma armadilha (o caso Java/Oracle), e combinado verbal não vale nada.

## Conceitos que todo sistema depende

### Por que um framework dá certo

- **Extraído de aplicação real**: o Rails saiu do Basecamp, um produto usado de verdade, ao contrário dos frameworks corporativos desenhados por comitê pra "qualquer projeto de qualquer tamanho". Ferramenta feita por quem depende dela tende a se ajustar melhor ao mercado do que ferramenta feita pra vender. [cap-117, cap-119]
- **Convention over Configuration + DRY**: se toda entidade precisa dos mesmos arquivos de boilerplate, o framework deve inferir isso em tempo de execução em vez de gerá-los. O autor estima que um projeto de ~100 arquivos em J2EE caía pra ~20 em Rails. [cap-117]
- **Estrutura previsível**: o gerador sempre cria o mesmo esqueleto (controllers, models, views, test), então quem troca de projeto sabe onde as coisas ficam. A estrutura mudou pouco do Rails 1 ao Rails 5. Convenção forte também ajuda agentes de IA a prever caminhos (ver programando-com-ia.md, cap-111). [cap-117, cap-111]
- **Opinião forte e integridade conceitual**: o Rails não é democracia nem comitê; foca em apps parecidos com o Basecamp. Quanto mais seu projeto se afasta do Rails original, mais caro fica manter. Um "ditador benevolente" assume as decisões e preserva a integridade. [cap-117]
- **Defaults seguros**: proteções contra CSRF, XSS, SQL injection e session hijacking já vinham ligadas; pra ficar sem elas, é preciso desligar à mão e assumir o risco. Vender "flexibilidade" pra cada um montar seus componentes faz a maioria escolher errado. [cap-117]
- **Testes por padrão**: o esqueleto já trazia o diretório `test`, e cada gerador criava o arquivo de teste correspondente. Teste deixou de ser algo que se adiciona depois e virou algo que você teria de remover. [cap-117]
- **Ecossistema novo absorve paradigma novo**: uma mudança radical (como TDD) é muito difícil de impor a um ecossistema grande e resistente. Ruby não tinha usuários pra convencer nem projetos pra migrar, por isso andou rápido. [cap-117]
- **YAGNI vs. "e se amanhã eu precisar?"**: o minimalismo pressupõe uma opinião clara do que é o mínimo. Abstrações e indireções "pra suportar qualquer escala" geram boilerplate e burocracia. [cap-117]
- **Coesão vs. fragmentação**: como a comunidade Ruby se concentrou num framework, o ecossistema inteiro (Heroku, Travis, CodeClimate, New Relic) mirou nele, e o dev já é produtivo no primeiro dia de uma empresa nova. Ecossistemas fragmentados (PHP, JavaScript) cobram uma curva de aprendizado nova em cada empresa. [cap-117, cap-119]
- **Janela histórica**: o sucesso dependeu de convergências daquele momento (fim da crise das pontocom, Web 2.0/Ajax, agilistas, Git e GitHub, cloud) e o autor acha que dificilmente se repete. Ele vê a influência do Rails forte até ~2012; depois, outras comunidades adotaram as mesmas práticas e surgiram novos problemas (mensageria em massa, containers, mobile). [cap-117]

### Escolha de tecnologia e evolução de sistemas

- **Escala é arquitetura, não linguagem**: no "Rails não escala" do Twitter, o uso mudou de micro-blog pra broadcast massivo de mensagens, algo que nenhuma das linguagens da época aguentaria. O modelo certo era o de atores do Erlang (Scala/Akka na época; Elixir hoje). O GitHub segue em Rails. [cap-117]
- **Ferramenta certa pra classe de problema**: o autor estima que 80% das aplicações web são "estilo Basecamp" (experiência de consultoria). Pra elas, um framework completo dá o melhor custo-benefício, assim como Wordpress pra blog. Esticar a ferramenta pra outra classe de problema traz sofrimento. [cap-119]
- **Popularidade vira problema pelos mesmos motivos**: o que tornou a tecnologia popular (convenções, Active Record, monopólio) depois incomoda os devs avançados. O Spring nasceu contra a complexidade do J2EE e virou o novo gigante; "ou você morre como herói, ou vive o bastante pra virar o vilão". [cap-119]
- **Abstração é pra quem já sofreu sem ela**: devs avançados gostam de camadas, mas iniciantes sofrem com escolhas arquiteturais demais logo de início. O Rails venceu por ser simples pra iniciante, dar orientação clara a quem tem experiência e ter flexibilidade razoável pros avançados. [cap-119]
- **Evolução incremental**: em sistema legado, o caminho é adicionar testes, refatorar, remover duplicação, atualizar dependências e componentizar (engines, APIs separadas) um passo de cada vez. Se for preciso reescrever partes grandes, vale considerar outra stack de vez. [cap-119]
- **Fatores externos decidem o destino da ferramenta**: pergunte se o dono precisa dela pra sobreviver. O Google abandonou o GWT e reescreveu o Angular 2 de forma incompatível porque não depende deles; a migração Grunt → Gulp → Webpack mostra o custo da moda. [cap-119]

### Sistemas corporativos (ERP)

- **Integração é o cerne**: o valor de um ERP é ter processos integrados, sem dez sistemas repetindo os mesmos clientes, produtos, contas e regras de preço. Dado duplicado entre sistemas separados gera redigitação, erro humano e ineficiência. [cap-118]
- **Não subestime processos de negócio**: um ERP sério carrega décadas de conhecimento de domínio (estoque, preço por região e imposto, demanda, logística, finanças, ativos, RH, CRM/SRM). Entender um algoritmo e entender um processo de vendas exigem conhecimentos diferentes. [cap-118]
- **Automatize só o que já está estável**: sistema pesado só compensa quando os processos da empresa estão consolidados, nem que seja no papel. Uma empresa começando "não tem o que automatizar". Uma regra segura: comprar o que é commodity e desenvolver in-house o que é crítico. [cap-118]
- **Framework leve como front-end**: em vez de substituir o back-end robusto (SAP, EJBs), use o framework produtivo pra camada web que coleta e consulta dados e devolve pro back-end processar. Um front-end web costuma ser custo pra empresa, não fonte de receita. [cap-118]

### Licenças e propriedade

- **A GPL nasceu de uma briga de copyright**: o GNU Emacs partiu do código do GosMacs de Gosling, que depois foi vendido à Unipress. A única permissão de distribuição era um e-mail que se perdeu. A Unipress passou a dizer que Stallman não podia distribuir o código, o medo de processo afastou usuários e, diz a lenda, daí nasceu o copyleft. Na versão de Gosling, a Unipress processou IBM e DEC e ganhou. Na leitura do autor, todos foram ingênuos sobre copyright. [cap-120]
- **Código-fonte aberto não é software livre**: por mais de 10 anos o Java teve fonte visível sob uma licença que não deixava usar o código; o mercado confiou que "a Sun é boazinha". Quando a Oracle comprou a Sun, veio o processo contra o Google: a "armadilha Java" que Stallman descreveu. [cap-120]
- **Detalhe de licença importa**: o OpenJDK saiu como GPL com classpath exception (senão, quem linkasse as bibliotecas seria "contaminado") e partes como o JavaFX demoraram a ganhar a exceção. Desde janeiro de 2019, receber atualizações do Java da Oracle exige licença; o conselho do autor é usar OpenJDK. [cap-120]
- **Copyright de API**: o caso Oracle vs. Google discutia se reimplementar uma API (o autor cita 37 APIs e ~11.500 linhas) infringe copyright. Se sim, isso ameaçaria até o Linux, que reimplementa o POSIX. O caso era ambíguo, porque além da API havia código original copiado. [cap-120]
- **Reimplementação clean room**: reimplementar do zero sem olhar o código original (como Harmony e Classpath) é a defesa contra plágio. Essas versões, porém, tendem a ficar atrás da original em compatibilidade e desempenho. [cap-120]
- **Pessoas não são o software**: Stallman, Gosling, Torvalds, DHH erram e têm ego. O autor avalia o trabalho, não a pessoa, e sem maniqueísmo. Até o Emacs passou quase dois anos (2009–2011) distribuindo binários sem o fonte completo, por erro de automação de release. [cap-120, cap-119]

### Contexto datado

- **Leia como história**: são textos de 2008, 2016, 2019 e 2020. As previsões (JRuby/Rubinius como futuro do Rails no mundo corporativo; "em 2020 ainda não há veredito") e as listas de quem usa o quê envelhecem. Os princípios (convenção, integração, custo-benefício, licença por escrito) não. [cap-118, cap-119, cap-120]
- **Desfecho do caso Oracle vs. Google**: em abril de 2021 a Suprema Corte dos EUA decidiu que a reimplementação da API do Java pelo Google foi fair use, sem decidir se APIs são passíveis de copyright. (complemento, não está no compêndio)
- **Licenças da Oracle mudam com frequência**: os termos de licença do Java da Oracle mudaram mais de uma vez depois de 2020; verifique os termos atuais antes de decidir. (complemento, não está no compêndio)

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Escolher framework pra app web típico (CRUD, colaboração, SaaS pequeno/médio) | Preferir um framework completo e opinativo, com convenções e ecossistema coeso | Cerca de 80–90% dos projetos são desse tipo; é o melhor custo-benefício | cap-119, cap-117 |
| Avaliar um framework novo | Perguntar de qual aplicação real ele foi extraído e quem depende dele pra sobreviver | Ferramenta feita por quem depende dela tende a se ajustar melhor e a não ser abandonada | cap-117, cap-119 |
| Tecnologia mantida por big tech que não depende dela | Usar se ajudar, mas não apostar o negócio inteiro nela | O dono pode abandonar ou quebrar a compatibilidade (GWT, Angular 1 → 2) | cap-119 |
| Escolher entre "flexível, monte você mesmo" e "defaults opinativos" | Ficar com os defaults seguros e desligar só com motivo explícito | A maioria escolhe errado quando tudo é opcional | cap-117 |
| Querer customizar muito o framework | Ficar perto da convenção; se a divergência for grande, reconsiderar a escolha | Quanto mais longe do original, mais cara a manutenção | cap-117 |
| Sistema "não escala" | Diagnosticar a classe do problema (ex.: broadcast de mensagens) e trocar só a parte com outra arquitetura | O problema costuma ser arquitetura, não linguagem | cap-117 |
| Pressão pra reescrever do zero | Evoluir em passos: testes, refatoração, dependências, extrair componentes ou APIs | Raramente se ganha com grandes rewrites | cap-119 |
| "Tecnologia X serve pra minha grande aplicação?" | Antes, levantar processos, lacunas, recursos (custo, tempo, pessoas) e objetivos | Não se escolhe trator sem a planta da obra | cap-118 |
| Empresa sem processos estáveis pedindo ERP/sistema grande | Começar pelo básico (contabilidade, planilha) e automatizar depois que o processo estabilizar | Automatizar caos gera catástrofe | cap-118 |
| Vários sistemas com os mesmos cadastros | Priorizar integração e uma única fonte de verdade | Dado duplicado gera erro humano e ineficiência | cap-118 |
| Já existe um back-end corporativo robusto | Usar o framework produtivo como front-end ou camada de integração, não como substituto | Front-end web é commodity; o back-end carrega o processo | cap-118 |
| Buy vs. build | Comprar o que é commodity, desenvolver o que é crítico | Estratégia mais segura de grandes corporações | cap-118 |
| Adotar ou distribuir software de terceiros | Ler a licença: GPL, LGPL ou exceção de classpath, e se o fonte "aberto" pode de fato ser usado | Contaminação e armadilha de dependência | cap-120 |
| Plataforma Java hoje | Usar OpenJDK (ou suporte pago de terceiros) | Atualizações do JDK da Oracle exigem licença desde 2019 | cap-120 |
| Acordo de uso, cessão ou distribuição de código | Pôr tudo por escrito, assinado, e guardar | No mundo jurídico a palavra verbal não vale; foi assim que o GosMacs virou litígio | cap-120 |
| Reimplementar uma API ou produto concorrente | Clean room de verdade, sem copiar código original | A disputa Oracle vs. Google envolvia código copiado, não só a API | cap-120 |

## Armadilhas e mitos

- "A tecnologia tecnicamente superior deveria vencer" → mercado não funciona assim. Adoção, custo-benefício e quem mantém pesam mais; ferramenta que ninguém usa não importa. [cap-119]
- "Mais opções no ecossistema é melhor" → fragmentação faz cada empresa ter uma stack diferente e você nunca domina nenhuma. A coesão acelerou o ecossistema Rails inteiro. [cap-117, cap-119]
- "Um framework enterprise deve ser infinitamente configurável" → isso vira boilerplate, XML e comitê. Consenso técnico é o mínimo denominador comum. [cap-117]
- "Rails/linguagem X não escala" → no caso do Twitter o gargalo era a arquitetura de broadcast; nenhuma outra linguagem da época aguentaria. [cap-117]
- "Dá pra fazer um ERP num fim de semana, é só um banco e umas telas" → ERP é décadas de processo integrado. O programador mais inteligente provavelmente será um péssimo analista de negócios. [cap-118]
- "Se Rails virou problema, reescreve tudo do zero" → evoluir em passos é quase sempre melhor. Se for mesmo reescrever partes grandes, a troca de stack inteira entra na conta. [cap-119]
- "O fonte é aberto, então estou seguro" → aberto não é livre. O Java dependeu por anos da boa vontade da Sun até a Oracle comprá-la. [cap-120]
- "A empresa dona é boazinha e nunca vai dar ruim" → donos mudam (Sun → Oracle) e o almoço grátis acaba (licença Java em 2019). [cap-120]
- "Há um lado do bem e um do mal (Stallman vs. Gosling, Oracle vs. Google)" → na leitura do autor, nenhum dos dois está totalmente certo; ingenuidade e apostas arriscadas explicam mais que má-fé. [cap-120]
- "Pergunte a quem entende e siga a resposta" → as pessoas respondem o que é melhor pra elas. Avalie você mesmo com pilotos pequenos e assuma a decisão. [cap-118]
- "Vale a pena porque eu gosto da tecnologia" → o lado empreendedor tem que falar mais alto que o lado programador; é custo-benefício, sem fanatismo. [cap-118]

## Checklist para aplicar num sistema real

- [ ] Que classe de aplicação é esta (estilo Basecamp/CRUD, blog, e-commerce, mensageria em massa, ERP)? A ferramenta escolhida foi feita pra essa classe?
- [ ] O framework foi extraído de um produto real? Quem depende dele pra sobreviver e mantém?
- [ ] Estamos perto das convenções do framework, ou acumulando customização que vai encarecer a manutenção?
- [ ] Os defaults de segurança (CSRF, XSS, SQL injection, sessão) e de teste estão ligados? Quem desligou algum, desligou com motivo documentado?
- [ ] A estrutura do projeto é previsível pra um dev novo (e pra um agente) sem tour guiado?
- [ ] O problema de desempenho é de linguagem ou de arquitetura (fan-out, broadcast, concorrência)? Dá pra isolar e trocar só essa parte?
- [ ] Há proposta de rewrite? Já foram tentados testes, refatoração, atualização de dependências e extração de componentes?
- [ ] Existem dados duplicados entre sistemas? Qual é a fonte única de verdade e como os sistemas se integram?
- [ ] Os processos que vamos automatizar estão estáveis e descritos, ou estamos automatizando caos?
- [ ] Isto é commodity (comprar) ou diferencial crítico (construir)?
- [ ] A decisão considerou custo, tempo, pessoas disponíveis, motivação e risco, e foi validada com um piloto pequeno?
- [ ] Toda dependência tem licença lida e compatível com o modo de distribuição (SaaS, binário, biblioteca)? Há risco de copyleft ou de licença comercial?
- [ ] A plataforma depende da boa vontade de um único dono? Há plano B (implementação aberta, fork, OpenJDK)?
- [ ] Permissões de uso, cessões e contratos de código estão por escrito e arquivados?

## Frases-âncora

> "Aliás, primeira dica, nunca acredite num framework que não foi extraído de um aplicativo de verdade." — cap-117

> "Já não era sem tempo, consenso em mundo técnico é só o mínimo denominador comum dos membros." — cap-117

> "Todo framework que te vende “flexibilidade” porque você mesmo pode escolher quais componentes por adicionar, sempre vai ser pior porque a grande maioria vai sempre escolher errado." — cap-117

> "O problema nunca foi a linguagem, foi a arquitetura." — cap-117

> "As pessoas pensam que porque algo é "tecnicamente superior" todos os outros deveriam adotar cegamente. Mas não é assim que o mercado funciona." — cap-119

> "Raramente se beneficia de grandes rewrites do zero." — cap-119

> "Negócios não é lugar para fanatismo, fanboys, ou decisões dogmatizadas e muito menos ignorantes." — cap-118

> "Sempre se lembre disso: tenha tudo por escrito em papel, assinado e com testemunha, senão você é o trouxa da história." — cap-120

## Onde aprofundar

| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-117 | [Akitando] #59 - A História de Ruby on Rails, Por que deu certo? (21/08/2019) | https://www.akitaonrails.com/2019/08/21/akitando-59-a-historia-de-ruby-on-rails-por-que-deu-certo/ | quiser entender convenção, opinião forte, defaults seguros, testes por padrão e o caso "Rails não escala". |
| cap-118 | Rails serve para grandes aplicações? (12/03/2008) | https://www.akitaonrails.com/2008/03/12/rails-serve-para-grandes-aplica-es/ | o projeto envolver ERP, integração corporativa, buy vs. build, ou a pergunta "tecnologia X serve pra coisa grande?". |
| cap-119 | Rails Venceu: O Elefante na Sala (23/05/2016) | https://www.akitaonrails.com/2016/05/23/rails-has-won-the-elephant-in-the-room/ | for decidir framework pelo critério de mercado, avaliar quem mantém uma ferramenta, ou resistir a um rewrite. |
| cap-120 | [Akitando] #74 - Emacs vs Java, Oracle vs Google (18/03/2020) | https://www.akitaonrails.com/2020/03/18/akitando-74-emacs-vs-java-oracle-vs-google/ | houver decisão de licença (GPL/LGPL/classpath), dependência de plataforma de um único dono, ou reimplementação de API. |
