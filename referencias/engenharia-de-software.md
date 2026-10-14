# Engenharia de Software
> Destilado de: cap-083, cap-084, cap-085, cap-086, cap-087, cap-088, cap-089, cap-090, cap-091, cap-092, cap-093, cap-094, cap-095, cap-096, cap-097, cap-098 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).

## Ideia central
Para o autor, software é "soft": o código-fonte é o próprio projeto (a planta), não a obra. Por isso não dá para especificar tudo antes, o design emerge do uso e o sistema nunca fica "pronto". Diante do desconhecido, o único método que funciona é tentativa e erro sistemática (método científico, PDCA, Kaizen): hipótese, experimento barato com prazo fixo, medição, ajuste e nunca o mesmo erro duas vezes, com testes e CI garantindo que o que já funciona não regride. Estimativa é ordem de grandeza: trava-se prazo e custo e o escopo é que encolhe. Nenhum processo, modelo copiado ou bala de prata substitui gente competente, poucas linhas de comunicação e alguém que decide na hora certa. Os princípios vêm de Brooks (1975/1995) e do XP dos anos 90 e continuam valendo; as ferramentas citadas em cada texto envelhecem.

## Conceitos que todo sistema depende

### Natureza do software e do design
- **O código é a planta, o compilador é o pedreiro** — Programar não é assentar tijolo: a "construção" (compilar/interpretar) é automática e o trabalho caro é escrever o código, que é a especificação de fato; todo programador faz trabalho de arquiteto. Diagramas, casos de uso e stories são rascunho de guardanapo. [cap-086, cap-093, cap-097]
- **Design emergente** — Como o código é o design e nunca sai perfeito de primeira, faz-se primeiro o código que funciona e depois se refatora com disciplina. O "design no papel, código depois" (RUP/UML) fazia sentido quando depurar num mainframe compartilhado era caro; hoje não faz. [cap-088]
- **Refatorar ≠ reescrever** — Refatorar (Fowler) é mudar a estrutura interna sem alterar o comportamento externo, de forma disciplinada para não introduzir bugs; reescrever é jogar fora e fazer outro comportamento, sem garantia de melhoria. [cap-088]
- **Software nunca está pronto** — Todo sistema teve um dia 1 do tamanho de um tutorial e cresceu respondendo a uso real, aquisições e prioridades que ninguém previu (o Magento foi de ~0,5 para 2 milhões de linhas de PHP). Modelagem e arquitetura são respostas a essas mudanças e emergem do software. [cap-087]
- **Atributos de qualidade em tensão** — Manutenibilidade, flexibilidade, portabilidade, reusabilidade, legibilidade, testabilidade e segurança competem entre si, e a prioridade muda com o uso (eficiência, depois segurança, depois manutenção). Código que roda mas é sujo é lixo, porque amanhã alguém precisa estendê-lo. [cap-088]
- **Integridade conceitual** — Para Brooks, o mais importante em design: melhor omitir features e refletir um só conjunto de ideias do que somar boas ideias descoordenadas. Exige alguém (ditador benevolente, core team) que decide o que entra; arquiteto que não consegue mostrar uma implementação não serve. [cap-089, cap-096]
- **Construção incremental** — Primeiro um esqueleto que roda, depois as funções preenchidas uma a uma, compilando e testando: em todo estágio existe um sistema funcionando (Mills, builds noturnos da Microsoft). É a base conceitual de CI/CD e de um staging sempre funcional. [cap-089]

### Modelagem e regras de negócio
- **Modelar é representar o mundo real** — Não se modela o que não se conhece. "Usuário" vira identificação, autenticação (salt/hash, seed de TOTP, token de reset de uso único com IP registrado), endereços de cobrança e de entrega, perfis (funcionário, fornecedor, cliente); "produto" vira um SKU por variante; preço é resultado de cálculo (custo, impostos por região, estratégia), não um campo. [cap-087]
- **Regra de negócio antes de pattern** — DDD, normalização e Design Patterns descrevem a implementação. Se a regra está errada, a implementação está errada. Cada indústria e cada país têm normas (PCI DSS, LGPD/GDPR) com nuances que não estão documentadas. [cap-087]
- **Política explícita, não guarda escondida** — Uma regra como "10% de overbooking" escrita como cláusula de guarda no meio do método some no código; extraída como `OverbookingPolicy` (Strategy) fica documentada e fácil de mudar de 10% para 5%. [cap-087]
- **Patterns e DDD são vocabulário** — Design Patterns é um catálogo de repetições com nome (pattern, não standard); DDD dá nomes (linguagem ubíqua, bounded context, aggregate) para comunicar abstrações. Saber onde usar vale mais que saber como. [cap-087, cap-088]
- **Domínio é referência móvel** — O que é um domínio (checkout dentro de "pagamento" ou de "vendas") muda de empresa para empresa e só se aprende vivendo a operação: fiscal, fulfillment, logística, suporte. [cap-087]
- **Tipo semântico único** — No dicionário de dados do SAP, campo → elemento de dados → domínio carrega tipo primitivo, valores permitidos, conversão ("1000" vira "0000001000"), validação e tradução. Mudar `PRICE` ou a regra de e-mail propaga para tabelas, programas e telas sem buscar e colar. Lição do autor: manter é mais difícil que escrever. [cap-098]

### Testes e qualidade
- **Testes reduzem risco, não provam correção** — Pelo Problema da Parada, nenhum algoritmo decide se qualquer programa termina; testar uma ou duas dúzias de casos não garante os demais. Unidades testadas não garantem o todo, por isso existem testes funcionais e de integração. [cap-085]
- **Regressão é o pior desperdício** — Quebrar o que funcionava faz o time andar para trás, e um bug corrigido que volta por falta de teste é dinheiro jogado fora. Brooks já medira: corrigir um defeito tem 20–50% de chance de introduzir outro, por isso a suíte inteira roda depois de cada correção. [cap-086, cap-089]
- **Teste é seguro** — Não deixa ninguém mais rápido no dia 1, e nem deveria: protege o você futuro do você presente. Sem testes, a produtividade cai semana a semana até o time só apagar incêndio. [cap-086, cap-092]
- **Teste antes é uma mini prova de conceito** — Se não tem certeza de algo, comece escrevendo um teste. [cap-086]
- **Protocolo do time** — Nada entra na branch principal sem testes, rebase, CI verde e revisão de alguém mais experiente, e a branch principal fica sempre pronta para deploy. Furar o protocolo é desrespeitar a equipe. [cap-086, cap-096]
- **Qualidade, a quarta variável invisível** — Espremer escopo em pouco tempo e pouco custo derruba a qualidade, e o problema é empurrado para a operação. [cap-097]

### Resolver problemas e depurar
- **Método científico** — Pergunta certa, pesquisa breve ("pare, pense, continue"), hipótese, experimento repetível com critério definido, conclusão (aceitando que a própria hipótese pode cair) e comunicação para os outros tentarem derrubá-la. Hipótese não falsificada não é verdade. [cap-084, cap-095]
- **Cargo cult** — Repetir estruturas (patterns, comentários óbvios, injeção de dependência) por ritual, sem saber a razão. Intuição é conclusão rápida a partir da experiência: só presta se essa experiência foi racional. [cap-084]
- **Tentativa e erro sistemática** — PDCA, DMAIC, Kaizen, sprint e MVP são o mesmo ciclo, o único método conhecido para problemas desconhecidos. "Sistemática" quer dizer: não repetir erro (controle), testar logo, cortar desperdício e atacar um problema por vez (o elo mais fraco), com uma contramedida rápida seguida de correção na raiz (5 porquês). [cap-095, cap-096]
- **Prova de conceito descartável** — Pedaço pequeno, fora do código principal (ambiente separado ou branch), com prazo fixo curto, feito para ser jogado fora. É seguro, não perda. [cap-084, cap-086]
- **A primeira hipótese é você** — No bug do Premiere, a causa era desconhecimento (acelerar um clipe é um efeito caro), não hardware, sistema operacional ou ferramenta: milhares de pessoas usam a mesma ferramenta em trabalhos maiores. [cap-083]
- **Saber o que não sabe** — Os 20% de conhecimento resolvem 80% dos casos; os últimos 20% exigem os 80% que faltam. Repetir 50 vezes o mesmo processo não sobe de nível, só um problema novo faz isso. [cap-083, cap-087]

### Economia, dívida e estimativa
- **Variável Tempo e dívida técnica** — Toda decisão é um compromisso entre o futuro ideal e o agora. Atalhos (sem testes, microsserviços no dia 1, a "Lamborghini" da Netflix) são empréstimos com juros que travam o time. Super-engenharia custa tanto quanto o sujo. [cap-092]
- **Menor coisa que funciona** — Comece pelo mínimo bem feito (não rápido e sujo), meça e evolua continuamente: o dia 7.200 da Netflix não é o seu dia 1. [cap-092]
- **Pareto do escopo** — Escopo nunca é completo nem fechado, e ~80% do "esboço" pouco importa aos usuários. Priorize e entregue cedo os 20% que importam (o MVP). [cap-093]
- **Estimar não é prever** — Estimativa é chute educado em ordem de grandeza, tão boa quanto a experiência de quem tem "skin in the game". Nenhuma fórmula (COCOMO, contas sobre estimativas) transforma texto em horas. [cap-086, cap-093, cap-097]
- **Triângulo de ferro** — Rápido, bom e barato: escolha dois. Trave tempo e custo e deixe o escopo variar, priorizado por valor; o PO é o guardião do ROI. [cap-093, cap-097, cap-095]
- **Velocidade é termômetro** — Como o termômetro de um alto-forno: alta demais indica hora extra ou queda de qualidade; baixa demais, enrolação, backlog incompreensível, dívida ou burnout. O alvo é estabilidade. [cap-093, cap-086]
- **Ágil é gestão de risco** — Não é backlog, ritual nem métrica: é dar um passo, avaliar, ajustar e repetir, expondo pequenos erros cedo. Deploy big bang é o caminho do fracasso. [cap-093, cap-094]

### Pessoas, times e organização
- **Mítico homem-mês (Lei de Brooks)** — Pessoas e meses não se trocam. Adicionar gente a projeto atrasado o atrasa mais: é preciso reparticionar tarefas, treinar os novos e abrir mais canais de comunicação (n(n−1)/2: 10 pessoas = 45, 20 = 190). Divida em subsistemas com equipes "cirúrgicas" de até 10. [cap-089, cap-092]
- **Lei de Conway** — O sistema copia a estrutura de comunicação da organização; microsserviços muitas vezes tapam a incapacidade de dois times colaborarem. [cap-092, cap-096]
- **Efeito do segundo sistema** — A primeira grande reescrita é o sistema mais perigoso: tende a ser superprojetada com todos os adornos deixados de lado na primeira versão. [cap-089, cap-092, cap-088]
- **Legado** — Todo código vira legado no minuto em que fica pronto, inclusive o seu. O legado em produção gera um valor que o código novo ainda não gera, e o código ruim nasceu de um contexto (regras que mudaram, prazos, refatorações que não aconteceram). [cap-091]
- **Programador de produtividade negativa (NNPP)** — Um mau elemento faz o time andar de ré (dívida, retrabalho, ponto único de falha); o cowboy é aplaudido por resolver problemas que ele mesmo criou. Retirar um ruim pode render mais que adicionar um bom. [cap-090, cap-086]
- **Auto-organização com alinhamento** — Modelos (Spotify, Scrum, Lean) são fotografias de um momento. Organização eficiente emerge de gente boa (que busca mastery) com direção clara e autonomia sobre o próprio ofício, oscilando entre ordem e caos ("beira do caos"). Empresa não é democracia. [cap-096, cap-095]
- **Gestão é decidir** — Tomar decisões no momento certo, com informação incompleta: dizer não, escolher batalhas, priorizar ("se tudo é prioridade, nada é"). Projeto tem começo, meio e fim, e saber cancelar é uma arte. [cap-097, cap-087]
- **Não existe bala de prata** — Nenhuma linguagem, framework, arquitetura ou metodologia dá ganho de ordem de grandeza; software não tem Lei de Moore. [cap-089, cap-092, cap-096]

### Enterprise
- **Enterprise é regra de negócio em escala** — Um ERP como o SAP tem dezenas de milhares de tabelas, precificação como matriz multidimensional (variante, centro, região, imposto) e localização que é regra fiscal, contábil e trabalhista (SPED, retenções, CLT), não tradução. Só se aprende passando por várias empresas. [cap-098, cap-087]
- **Customizar sem clonar** — Copiar uma transação padrão para um "Z" e mudar duas linhas mata a capacidade de upgrade e cria um mercado de manutenção do "Z/3". Estenda pelos mecanismos previstos. [cap-098]

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Bug, crash ou lentidão inesperada | Assumir primeiro que o erro é de uso/configuração/conhecimento seu; trocar hardware, SO ou ferramenta é a última opção | Milhares usam a mesma ferramenta em casos maiores; culpar a ferramenta não ensina nada | cap-083 |
| Descrever o sintoma | Separar "trava" de "fica lento" e anotar o que mudou (ex.: dezenas de clipes acelerados a 1000–6000%) | Hardware fraco deixa lento, raramente faz travar; o sintoma aponta a classe de causa | cap-083 |
| Escolher qual hipótese testar | Ordenar por probabilidade e custo; desconfiar da ideia "geek" improvável e cara (exportar proxies por 1 hora de madrugada) | Experimento caro e improvável queima o prazo sem informação útil | cap-083 |
| Experimento "não fez diferença" | Conferir se ele testou mesmo a hipótese (Render and Replace sem o checkbox "com efeitos" só trocava de codec) | Um experimento mal montado derruba uma hipótese correta | cap-083 |
| Travado numa investigação | Conversar com alguém (método socrático); sozinho, escrever todos os sintomas e hipóteses; tarde da noite, ir dormir | O diálogo organiza o raciocínio; cansaço prolonga o problema | cap-083 |
| Trocar ferramenta, linguagem ou framework | Nunca no meio de uma emergência com prazo apertado; avaliar ecossistema, migração e curva de aprendizado | Mudar não é trivial e cria novos problemas inesperados | cap-083 |
| Corrigir um bug | Escrever antes um teste que reproduz o bug (falhando); corrigir até passar | Prova o conserto e impede que esse bug volte | cap-086 |
| Entregar funcionalidade | Cobrir com testes; PR sem teste é rejeitado na revisão | Cada regressão é tempo e dinheiro jogados fora | cap-086 |
| Projeto com zero testes | Escrever o primeiro teste na funcionalidade que acabou de entregar; ignorar a discussão de 100% de cobertura | De um em um a cobertura cresce; o teste mostra quem quebrou e em qual commit | cap-086 |
| Suíte lenta | Rodar localmente só os testes ao redor da mudança; o CI roda a suíte inteira em cada PR | Remove a desculpa sem abrir mão da rede de proteção | cap-086 |
| Atualizar linguagem, framework ou libs | Rodar a suíte antes de subir; não subir se falha | O blog do autor atualiza há 10 anos assim, em ~30 min por ano | cap-086 |
| Dúvida técnica cara (React ou Vue, Postgres ou Firebase, REST ou GraphQL) | Prova de conceito em branch, prazo fixo de 2 dias a 1 semana (um lote de dúvidas: 2 semanas), descartável | No exemplo, 2 semanas evitam mais de meio ano em 7 de 10 apostas erradas | cap-086, cap-084 |
| Tecnologia ou arquitetura desconhecida num projeto grande | Parar para prototipar e testar hipóteses antes de implementar | YellowPages: 4 meses de protótipos impediram que 4 meses de implementação virassem 20 | cap-084 |
| Discussão técnica sem consenso | Alguém experiente decide; se falhar, a evidência alimenta a decisão seguinte | Errar cedo e consertar rápido é melhor que bikeshedding | cap-086 |
| Regra de negócio enterrada num `if` | Extrair como política nomeada (Strategy) | Documenta a regra e localiza a mudança | cap-087 |
| Mesmo conceito (preço, e-mail, código) repetido em muitos campos | Definir um tipo de domínio único com validação, conversão e rótulo | Uma mudança propaga para todo lugar sem buscar e colar | cap-098 |
| Precisa mudar comportamento de um sistema padrão (ERP, framework) | Usar os pontos de extensão; não clonar o código padrão | Clones ficam para trás e impedem upgrade | cap-098 |
| Herdou código ruim | Entender regra de negócio, origem dos dados e por que falha; limpeza mínima e o mínimo necessário para atingir o objetivo | O autor salvou em 2 semanas um ASP+DCOM condenado; "eu faria melhor" não justifica reescrita | cap-091 |
| Vai mexer em legado sem testes | Antes de alterar, escrever testes que fixam o comportamento atual (testes de caracterização) (complemento, não está no compêndio) | Sem rede, refatorar vira reescrever às cegas | — |
| Vai reescrever (versão 2.0) | Manter o escopo enxuto e resistir aos adornos guardados | Efeito do segundo sistema | cap-089, cap-092 |
| Arquitetura de time pequeno | Monolito organizado, "Agile e OO de verdade", menor coisa que funciona; microsserviço só quando o tamanho do time pede (o autor cita mais de 5 devs como o ponto em que a dor aparece) | Microsserviços no dia 1 são dívida com juros: deploys quebrando sem monitoramento nem testes integrados | cap-092 |
| App web lento | Procurar código ruim, usar cache e CDN antes de trocar linguagem | Servidores ficam ociosos; nenhuma linguagem salva programação ruim | cap-092 |
| Escolher stack (datado, 2017) | Go para infra/CLI/daemons, Rust para libs embutidas/drivers, Swift/Kotlin no mobile, Elixir para o realmente distribuído, Rails para CRUD web; princípio: escolher pelo tipo de problema | Ferramenta de nicho para problema comum é desperdício | cap-092 |
| Estimar | Dar faixas de ballpark (1 mês: menos de 2; 3 meses: entre 2 e 6; 6 meses: entre 4 e 9; mais de 6, menos de 1 ano) com premissas explícitas | Granular mais que isso é inútil | cap-093 |
| Quebrar o backlog | Stories de tamanho parecido: se a menor é meio dia, a maior não passa de 2–3 dias; maior é épico e se divide | Mantém a medida grosseira mas comparável | cap-097 |
| Planejar | Lista curta (horas, não dias); detalhe só no curto prazo (1 mês, no máximo 2); longo prazo só como direção | Quanto mais detalhado o plano do desconhecido, mais errado | cap-086 |
| Montar o projeto | Fixar data e orçamento, ordenar objetivos por prioridade, colocar qualidade como restrição e encaixar o escopo que couber | Fazer custo e prazo caberem num escopo fechado é impossível | cap-093, cap-097, cap-095 |
| Backlog não cabe na semana | Cortar ou simplificar escopo; nunca pular testes | Ceder uma vez acumula e cobra nas semanas seguintes | cap-086 |
| Projeto atrasando | Não adicionar gente nem tornar hora extra rotina; mexer em escopo, equipe ou arquitetura, não na data | Lei de Brooks; hora extra contínua derruba a qualidade em bola de neve | cap-089, cap-097 |
| Sistema grande, time grande | Subsistemas com equipes de até 10, cada uma com um líder claro | Linhas de comunicação crescem quadraticamente | cap-089 |
| Duração da iteração | Uma semana, duas no máximo; se as cerimônias não cabem, cortar cerimônias | Sprint de 3 semanas sinaliza reuniões demais e falta de alinhamento | cap-096, cap-097 |
| Contratou fornecedor ou time novo | Exigir entregas testáveis em staging numa URL a cada iteração; se após ~2 semanas não gostar, encerrar | Perder 2 semanas é gestão de risco; perder 6 meses é ingenuidade | cap-093 |
| Requisito mal definido | Devolver para quem pediu escolher; não começar projeto inseguro demais | Não começar também evita desperdício | cap-094, cap-096 |
| Deploy | Pequeno e frequente; review app por PR; rollback pronto | Lote grande é pesadelo para achar o que quebrou; medo de sexta-feira é sintoma de time disfuncional | cap-086, cap-096 |
| Escolher métricas | Velocidade como termômetro, ROI e custo total; não medir "bugs corrigidos" nem fazer contas sobre estimativas | Métrica ruim incentiva o oposto; números não viram ciência | cap-097, cap-094, cap-086 |
| Programador que piora o time (cowboy, SPOF, contra testes e revisão) | Orientar; se resistir, remover em vez de contratar mais um | A produtividade sobe só de parar o retrabalho | cap-090, cap-086 |
| Projeto novo com todos ocupados | Seniors internos lideram; terceiros entram como juniors sob eles | Terceirizar o projeto inteiro perde o conhecimento | cap-097 |
| Composição de equipe | Misturar seniors e juniors e rotacionar | Só seniors é desperdício; só juniors é desastre anunciado | cap-097 |
| Comunicação | E-mail para o não urgente; reunião só para bater o martelo; alerta vermelho só no realmente urgente | Toda interrupção é desperdício; se tudo é urgente, nada é | cap-097 |
| Chefe empurrando o time para o precipício | Dizer não, cara a cara, com realismo (PACE da aviação: sondar, alertar, desafiar, emergência) | O silêncio hierárquico derruba aviões e projetos | cap-090 |
| Estudar modelagem, DDD e patterns | Folhear, guardar, voltar com quilometragem; usar como consulta | Sem experiência real, o "porquê" não faz sentido | cap-087, cap-088 |

## Armadilhas e mitos
- "Se compila, funciona; tipagem estática dispensa testes" → Spring Boot e Tokio (Rust) rodam milhares de testes; o TDD nasceu com o jUnit, em Java. Compilar não garante ausência de bug. [cap-086]
- "Front-end e CSS não precisam de teste" → React (mais de 7 mil testes) e Tailwind (quase 900) têm suítes; todo software maduro tem. [cap-086]
- "Teste é perda de tempo" → a desculpa real costuma ser não saber fazer, e cria o ciclo "sem tempo porque tem bug, tem bug porque não tem teste, não tem teste porque falta tempo". [cap-086, cap-095]
- "Funcionou nos casos que testei, então funciona sempre" → o Problema da Parada mostra que não dá para garantir todos os casos; testes reduzem risco. [cap-085]
- "Quanto mais requisitos, UML e diagramas, melhor" → a premissa da engenharia civil não vale: o código é a planta. Exigir que o programador liste classes e métodos antes de codar é "noção boçal". [cap-086, cap-088, cap-097]
- "Estimativa é previsão" → é ordem de grandeza; quem cobra 2 dias e recebe em 5 confundiu as palavras. [cap-086, cap-093, cap-097]
- "Gráficos, derivadas e Monte Carlo sobre estimativas são ciência" → multiplicam um erro por outro; é numerologia. Velocidade e simulações servem como termômetro, não como ferramenta de estimativa. [cap-086, cap-093, cap-094]
- "Dá para travar escopo, prazo e custo" → triângulo de ferro: escolha dois. [cap-093]
- "Contratar mais gente resolve" → atrasa projeto atrasado, e um time dobrado sem qualidade volta a entregar 1–2 tarefas por semana. [cap-086, cap-089, cap-092]
- "Hora extra resolve prazo" → só numa emergência de poucos dias; como rotina, a qualidade despenca. [cap-097]
- "Legado ruim: jogue fora e reescreva" → o Big Rewrite que nunca vai ao ar é comum; quem critica teria entregado igual ou pior no mesmo contexto. Lidar com legado é para adultos. [cap-091]
- "Agora sei como deveria ser: a 2.0 vai ser perfeita" → é a armadilha do segundo sistema. [cap-088, cap-089]
- "Waterfall era burrice e o Ágil inventou testes, CI e entrega incremental" → Royce (1970) já tinha feedback; Brooks (1975/1995) descreve testes de regressão, construção incremental e builds noturnos; o Manifesto resume 20 anos de prática. [cap-089, cap-088]
- "Ágil é Scrum, Kanban e cerimônias que se implementam" → ágil é adjetivo; o Scrum Guide nem usa o termo, e o Kanban de software não tem paralelo com o da Toyota. O que importa são as práticas do XP (TDD, small releases, CI, design simples, propriedade coletiva). [cap-094, cap-096]
- "Copiar o modelo Spotify, Netflix ou Google" → nem o Spotify segue o próprio paper; é uma fotografia de outro contexto, orçamento e momento. [cap-096, cap-097, cap-092]
- "Microsserviço, framework novo ou linguagem nova vão nos salvar" → não existe bala de prata. [cap-089, cap-092, cap-096]
- "Bounded context é microsserviço; DDD é Event Sourcing com CQRS" → bounded context cabe num monolito com namespaces e módulos; aplicar tudo de DDD em todo lugar é overengineering. [cap-087]
- "Design Patterns são os padrões oficiais de design" → é um catálogo de patterns (repetições com nome); empilhar decorator, factory, singleton e observer é pose de arquiteto. [cap-088, cap-084]
- "Trocar hardware, sistema operacional ou ferramenta resolve" → são as últimas opções, e no meio de um prazo são as piores. [cap-083]
- "Performance exige uma linguagem mais rápida" → na web a lentidão vem da programação ruim; quase tudo pode ser cacheado. [cap-092]
- "Toda linha escrita precisa ir para produção" → PoC e iteração descartadas são seguro; programar é jogar fora a maior parte do que se digita. [cap-084, cap-092, cap-095]
- "O curso ou livro já ensina o sistema de verdade" → o e-commerce do livro de Rails tem ~2.700 linhas de Ruby, o Spree tem 34 vezes isso e o Magento 2 milhões de linhas de PHP. [cap-087]
- "Programador ruim é só mais lento" e "o cowboy é herói" → o ruim dá marcha a ré; "só ele sabe como funciona" é motivo a mais para resolver o ponto único de falha. [cap-090]
- "Certificado, PMP ou MBA provam competência" → são papel; competência se demonstra em código e entrega. [cap-097, cap-090, cap-084]
- "Autonomia é democracia" → autonomia vale sobre o próprio ofício (ferramentas, equipes, técnica); quem não arca com o prejuízo não decide o orçamento. [cap-096]
- "O gestor motiva a equipe" → motivação é intrínseca; o gestor no máximo evita desmotivar. [cap-097, cap-096]
- "Burocracia protege o projeto" → é a muleta do "segui o procedimento, a culpa não é minha". [cap-086, cap-094]
- "Localização é traduzir telas" → é implementar regra fiscal, contábil e trabalhista de cada país. [cap-098]

## Checklist para aplicar num sistema real
- [ ] Qual é o problema real e a regra de negócio por trás dele? Alguém do domínio a validou (overbooking, SKU, impostos, regulação)?
- [ ] As regras de negócio estão nomeadas e isoladas (políticas/estratégias) ou escondidas em condicionais?
- [ ] Conceitos recorrentes (preço, e-mail, documento) têm definição única com validação e conversão?
- [ ] Esta é a menor coisa que funciona? Há microsserviços, patterns ou otimizações sem necessidade demonstrada?
- [ ] As decisões técnicas incertas passaram por uma PoC com prazo fixo antes de virarem compromisso de longo prazo?
- [ ] A divisão em serviços reflete uma decisão de design ou só o fato de dois times não conversarem (Conway)?
- [ ] Alguém responde pela integridade conceitual e decide o que entra?
- [ ] Customizações de plataformas usam pontos de extensão em vez de clones do código padrão?
- [ ] Toda feature nova e todo bug corrigido vêm com teste (o do bug falhando antes do conserto)?
- [ ] Existem testes de integração, além dos unitários, para o que é montado de várias peças?
- [ ] O CI roda a suíte em cada PR, e o merge exige CI verde mais revisão?
- [ ] A branch principal está sempre pronta para deploy, os deploys são pequenos e frequentes e o rollback existe?
- [ ] Upgrades de linguagem e bibliotecas passam pela suíte antes de ir para produção?
- [ ] Há staging (ou review app por PR) onde QA e cliente testam as entregas?
- [ ] Ao depurar: o sintoma está descrito com precisão (trava ou fica lento, desde quando, o que mudou)?
- [ ] A primeira hipótese examinada foi uso, configuração ou conhecimento próprio, antes de culpar ferramenta, SO ou hardware?
- [ ] As hipóteses estão escritas e ordenadas por probabilidade e custo, e cada experimento testa mesmo a hipótese?
- [ ] A causa raiz foi buscada (5 porquês) e um teste impede o mesmo erro de voltar?
- [ ] A estimativa é uma faixa de ordem de grandeza, com premissas explícitas, dada por quem vai fazer?
- [ ] Prazo e custo estão travados, com o backlog priorizado por valor e detalhado só para 1–2 meses?
- [ ] A velocidade é acompanhada como termômetro, sem cálculos sobre estimativas?
- [ ] As stories têm tamanhos comparáveis e os épicos foram quebrados?
- [ ] As iterações duram uma semana (no máximo duas), com ponto de reavaliação e de corte?
- [ ] Quando não cabe, corta-se escopo, sem abrir mão de testes nem tornar hora extra rotina?
- [ ] Os times têm até 10 pessoas, misturam seniors e juniors e não têm ponto único de falha (código só numa máquina, "só fulano sabe")?
- [ ] Alguma métrica incentiva o comportamento oposto ao desejado (ex.: contar bugs corrigidos)?
- [ ] Antes de propor reescrever um legado, o contexto, as regras e os dados foram entendidos, e as qualidades foram pesadas contra os defeitos?

## Frases-âncora
> "Na realidade o código "é" o design e ele nunca vai estar perfeito, muito menos na primeira vez." — cap-088

> "Ter testes não é burocracia, não é perda de tempo. É um seguro." — cap-086

> "O importante não é andar rápido, o importante é não estragar o que já funcionava." — cap-086

> "Estimativas são ordens de grandeza, não está certo nem errado." — cap-086

> "Mas na grande maioria das vezes é o oposto: considere sempre como primeira hipótese que é você que não está sabendo usar as ferramentas." — cap-083

> "Se a regra estiver errada, a implementação vai estar errada, independente se seguiu o livro do Eric Evans ou pé da letra." — cap-087

> "O pior tipo de software é aquele que não gera valor." — cap-091

> "Gerenciar projetos é gerenciar pessoas. E gerenciar pessoas é gerenciar expectativas." — cap-097

## Onde aprofundar

| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-083 | [Akitando] #55 - Refletindo sobre RESOLUÇÃO de Problemas \| O bug do Premiere (2019) | https://www.akitaonrails.com/2019/07/16/akitando-55-refletindo-sobre-resolucao-de-problemas-o-bug-do-premiere/ | for depurar algo travado e a tentação for culpar ferramenta, SO ou hardware |
| cap-084 | Off-Topic: Método Científico vs Cargo Cult (2008) | https://www.akitaonrails.com/2008/12/16/off-topic-m-todo-cient-fico-vs-cargo-cult/ | precisar justificar protótipos descartáveis ou desmontar decisões por modismo (a lista de "crenças" sobre tecnologias é datada) |
| cap-085 | Por que Testar? (2007) | https://www.akitaonrails.com/2007/08/25/por-que-testar/ | quiser o fundamento teórico (Problema da Parada) de por que testes não provam correção (ZenTest/rcov estão obsoletos) |
| cap-086 | [Akitando] #130 - Rant: Projetos, TESTES e Estimativa??? \| Rated-R (2022) | https://www.akitaonrails.com/2022/10/25/akitando-130-rant-projetos-testes-e-estimativa-rated-r/ | for montar o fluxo PR + testes + CI + staging, argumentar por testes ou ajustar estimativas |
| cap-087 | [Akitando] #144 - Modelagem de Software é Difícil? \| "Ver" vs "Enxergar" (2023) | https://www.akitaonrails.com/2023/08/11/akitando-144-modelagem-de-software-e-dificil-ver-vs-enxergar/ | for modelar domínio (usuário, SKU, preço, regulação) ou decidir quando usar DDD e patterns |
| cap-088 | [Akitando] #106 - Recomendação de Livros - Introdução à Design Emergente (2021) | https://www.akitaonrails.com/2021/10/11/akitando-106-recomendacao-de-livros-introducao-a-design-emergente/ | quiser entender design emergente, refatorar vs reescrever e de onde vêm UML, XP e DDD |
| cap-089 | [Akitando] #23 - The MM-M: O Melhor Livro de Software? (2018) | https://www.akitaonrails.com/2018/11/01/akitando-23-the-mm-m-o-melhor-livro-de-software/ | for dimensionar time, discutir prazo ou reescrita (Lei de Brooks, integridade conceitual, segundo sistema) |
| cap-090 | [Off-Topic] Net Negative Producing Programmer (2009) | https://www.akitaonrails.com/2009/03/30/off-topic-net-negative-producing-programmer/ | o time estiver regredindo por causa de alguém, ou houver ponto único de falha (as recomendações de RH são opinião de 2009) |
| cap-091 | [Off-Topic] O Mito do "Legado" (2012) | https://www.akitaonrails.com/2012/08/15/off-topic-o-mito-do-legado/ | surgir a proposta de jogar fora e reescrever um sistema em produção |
| cap-092 | A Economia do Desenvolvimento de Software (2017) | https://www.akitaonrails.com/2017/06/22/the-economics-of-software-development/ | for escolher arquitetura e stack para time pequeno, ou pesar dívida técnica e microsserviços (o mapa de stacks é datado) |
| cap-093 | Estimativas São Promessas - Uma Metáfora Melhor (2017) | https://www.akitaonrails.com/2017/06/26/estimates-are-promises-a-better-metaphor/ | precisar dar estimativa a cliente, travar prazo e custo e usar a velocidade como termômetro |
| cap-094 | [Akitando] #51 - Esqueça Metodologias "Ágeis" \| [Rated R] (2019) | https://www.akitaonrails.com/2019/06/18/akitando-51-esqueca-metodologias-ageis-rated-r/ | alguém propuser "implementar Ágil" ou métricas de processo no lugar de entrega de valor |
| cap-095 | [Akitando] #119 - Rant: Aprendizado na Beira do Caos \| Rated R (2022) | https://www.akitaonrails.com/2022/05/30/akitando-119-rant-aprendizado-na-beira-do-caos-rated-r/ | quiser a base comum de PDCA, DMAIC, Kaizen, 5 porquês e MVP como tentativa e erro sistemática |
| cap-096 | [Akitando] #57 - O Guia DEFINITIVO de Organizações \| Desconstruindo o Modelo Spotify [RATED R] (2019) | https://www.akitaonrails.com/2019/07/31/akitando-57-o-guia-definitivo-de-organizacoes-desconstruindo-o-modelo-spotify-rated-r/ | for organizar times (squads, autonomia, alinhamento, Conway) ou alguém quiser copiar um modelo |
| cap-097 | [Akitando] #120 - Discutindo Gestão (2022) | https://www.akitaonrails.com/2022/06/13/akitando-120-discutindo-gestao/ | assumir papel de gestor ou PO: priorizar, dizer não, métricas, composição de equipe, comunicação |
| cap-098 | [Akitando] #34 - Você não sabe nada de Enterprise. Conhecendo a SAP! (2018) | https://www.akitaonrails.com/2018/12/26/akitando-34-voce-nao-sabe-nada-de-enterprise-conhecendo-a-sap/ | for integrar ou modelar sistema corporativo (dicionário de dados, localização, customização sem clones); o relato é do R/3 do início dos anos 2000 |
