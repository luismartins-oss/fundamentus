# Programando com IA

> Destilado de: cap-109, cap-110, cap-111, cap-112, cap-113, cap-114, cap-115, cap-116 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).

## Ideia central

Para o autor, um LLM é um completador de texto probabilístico treinado no que é público. Reproduz muito bem o que já existe (CRUD, integração, port, mashup de peças conhecidas). Diante do inédito, sem um oráculo que diga o que é "certo", vira força bruta cara. O salto sentido em 2025–2026 veio do harness agêntico (busca sob demanda, ferramentas, loop de teste), não de uma inteligência de base maior, e isso já é "competente o suficiente" pra automatizar engenharia de software. A condição é ter a disciplina antiga do XP (TDD, CI, small releases, refactoring, rollback) e um engenheiro experiente decidindo o que pedir. Software nunca fica pronto: o valor aparece na iteração pós-deploy com usuários reais, e o código deve ser escrito pra esse ciclo, ou seja, pequeno, greppável e testável por um agente. Nomes de modelos, tamanhos de contexto e ferramentas citados nos textos envelhecem em meses (o próprio autor revisou em 2026 previsões que fez em 2023). O que continua valendo é "oráculo + teste + decisão humana", síntese que ele consolidou no livro gratuito *Competente o Suficiente* (https://ailair.akitaonrails.com/pt-br/book/).

## Conceitos que todo sistema depende

### O modelo

- **LLM é um completador de texto**: GPT quer dizer "transformer generativo pré-treinado". Dado o texto anterior, o modelo escolhe o próximo token pelos pesos aprendidos no treino. A "resposta" é só a continuação mais provável do prompt, por isso sai token a token e cada token gasta GPU. Não existem regras de gramática nem "ifs", só pesos. [cap-109]
- **Parâmetros são pesos, não sinapses**: um parâmetro isolado não significa nada, porque o resultado sai da rede inteira de tensores. A contagem não mede qualidade, já que algoritmo de treino, dados e quantização (float32 → float16) mudam o valor de cada parâmetro. Desde o GPT-4 os laboratórios nem publicam mais esse número. [cap-109, cap-116]
- **Temperatura é aleatoriedade, não criatividade**: ela controla quão espalhada fica a escolha entre os tokens candidatos. Com temperatura alta o texto "parece criativo", mas aumenta a chance de sair sem sentido. O autor rejeita o termo "alucinação": o texto errado sai do mesmo mecanismo probabilístico que gera o certo. [cap-109]
- **Contexto e atenção são finitos**: o modelo só enxerga a janela de contexto. Quanto mais cheia, mais cara fica e pior a precisão de detalhe, e isso acontece bem antes do limite declarado. Num agente, o código disputa a janela com CLAUDE.md, prompt de sistema, histórico, saída de ferramenta, logs e testes. [cap-109, cap-111]
- **O modelo é um arquivo estático**: depois do treino ele fica read-only. Cada sessão roda uma cópia isolada, e nada do que acontece nela volta pro arquivo. Conhecimento novo só entra na próxima rodada de treino. O que se acumula numa maratona de trabalho é o *seu* contexto (notas, CLAUDE.md, testes), e ele se perde se você não documentar. [cap-115]
- **Ponto cego do treino**: todo modelo foi treinado no que é público. Em 2025, 81,5% das contribuições no GitHub aconteceram em repositórios privados. Código proprietário, de jogos, embarcado e corporativo nunca entrou no treino. [cap-115]
- **Reprodução vs. novidade**: quando o problema já foi resolvido em público, os tokens certos têm probabilidade alta e o trabalho voa. No inédito, o caminho certo tem probabilidade baixa e o processo vira "força bruta direcionada": tenta, regride, reverte, tenta de novo. [cap-115]
- **Oráculo**: o que tira o agente do giro é um verificador do tipo "o resultado tem que ser igual a X", seja uma imagem de referência, uma trajetória gravada, um emulador ciclo-exato ou um teste. Os feitos "novos" de laboratório (AlphaEvolve, o3 no ARC, AlphaGo) também são busca cara em cima de um verificador barato. [cap-115]
- **RL só funciona onde há recompensa verificável**: "IA treinando IA" é aprendizado por reforço, que existe há décadas. Funciona onde há placar, prova ou teste (jogo, matemática, código) e produz um arquivo novo, tão congelado quanto o anterior. [cap-115]

### O agente (harness)

- **Harness pesa mais que a densidade do modelo**: busca e leitura sob demanda tiram dos pesos o conhecimento de baixa prioridade. O loop hipótese → implementa → testa → corrige traz informação real pra sessão. Os dois mecanismos atacam a alucinação na raiz. [cap-116]
- **Convergência**: nos benchmarks do autor, programação geral não melhora de forma relevante desde a geração do Opus 4.5, e os modelos (inclusive os chineses) estão no mesmo patamar. A escolha do modelo é quase indiferente. O que ainda faz diferença é o nível de esforço (effort). [cap-116]
- **Encaixe vale mais que densidade**: muita "tarefa de LLM" é classificação disfarçada (rotular ticket, rotear fila, aplicar política). Nos testes do autor, um classificador especializado pequeno empatou com o líder hospedado ou ficou a poucos pontos dele, custando centavos. [cap-116]
- **O loop precisa de mapa**: sem documentação pra buscar nem teste pra validar (por exemplo, uma linguagem nova demais pra ter corpus), o loop gira no vazio. [cap-116]
- **O agente faz o que você pede, e não faz o que você não pediu**: ele implementa o caminho feliz. Circuit breaker, retry, timeout, fallback, DI, DRY rigoroso e tipos em todo lugar só aparecem se você pedir ou escrever nas regras do projeto. [cap-110, cap-111, cap-114]

### O processo

- **Software nunca está pronto**: bugs e features aparecem no contato com dados, APIs e usuários reais. Exemplos do autor: Gmail cortando e-mail acima de 102 KB, a API Steam devolvendo datas como "fev", tipos MIME de .deb/.dmg faltando, UNC paths no Windows. Foram 125 commits depois do "pronto" em 4 projetos. Pós-produção é desenvolvimento, não manutenção. [cap-110]
- **One-shot prompt é mito**: a spec perfeita exigiria saber de antemão tudo que vai dar errado. Software bom é a soma de centenas de micro-decisões tomadas com o sistema rodando. [cap-110]
- **XP é o jeito certo de trabalhar com agente**: pair programming com o agente no lugar do par humano, TDD, CI apertado, small releases e refactoring contínuo. O teste é o feedback do agente e a rede de segurança da próxima mudança dele. Sem essa disciplina, a IA acumula dívida técnica ainda mais rápido. [cap-110, cap-111]
- **Humano decide, agente executa**: o gargalo é a experiência, não a velocidade de digitar. Isso inclui contexto operacional, saber o que pedir, decompor o problema e perceber quando o pedido estava mal formulado. Quando o agente erra, quase sempre o erro é de quem dirigiu o harness. [cap-110, cap-114, cap-116]
- **Corrigir ficou barato**: com revisão por LLM, git e containers, reverter e corrigir custa minutos. Segurar código numa fila de revisão manual virou pedágio pago por medo. Isso só vale em cima da engenharia básica: testes, CI, rollback e observabilidade. [cap-114]
- **Deploy frequente reduz falha**: XP, *Continuous Delivery* e os relatórios DORA mostram que, na média, times que deployam mais falham menos e se recuperam mais rápido. [cap-114]
- **Feedback real vale mais que a própria revisão**: revisão automatizada sem uso real só devolve suas próprias suposições. O que mantém o projeto no chão são usuários reais abrindo issues e PRs. [cap-114, cap-110]
- **Padronização habilita automação**: com o mesmo `bin/deploy`, a mesma release por tag e a mesma estrutura em todo projeto, dá pra pedir ao agente "faz o deploy" ou "solta a v0.5.0" sem que ele precise adivinhar nada. [cap-113]

### O código (o leitor primário agora é o agente)

- **Clean Code re-ranqueado**: em ordem de peso pra um agente, vêm funções e arquivos pequenos, SRP, nomes únicos e greppáveis, comentários de contexto/proveniência, tipos explícitos, DRY, testes que o agente consegue rodar, estrutura de diretório previsível, DI, pouco aninhamento, erros com contexto e formatador padrão. Por último, evitar comentário óbvio. [cap-111]
- **Tamanho cabe numa leitura**: as CLIs de agente leem arquivo em faixas (o Claude Code lê 2000 linhas por padrão). O autor recomenda arquivos abaixo de 500 linhas, idealmente 200–300, e funções de 4 a 20 linhas. Assim a unidade de sentido inteira cabe numa só chamada de ferramenta, com atenção cheia. [cap-111]
- **Grep é a API de navegação do agente**: o agente prefere buscar a ler o arquivo inteiro. Um nome distintivo devolve três resultados; um genérico (`data`, `handler`, `Manager`) devolve cinquenta. [cap-111]
- **Comentário de proveniência é contexto de primeira classe**: o que o agente não sabe é o porquê (o bug de produção, a restrição de negócio, o workaround de uma lib com bug conhecido, a issue ou o commit de origem). Não pode os comentários que o agente escreveu; remova só os óbvios. [cap-111]
- **DRY ficou mais crítico pra agente**: ele atualiza uma cópia e esquece as outras, porque nada na janela de atenção dele "lembra" que existem duplicatas. Fatorar é segurança de refactor automatizado. [cap-111]
- **Meta-documentação pra agentes**: CLAUDE.md, AGENTS.md e `.cursor/rules` devem ser curtos, imperativos e em bullets, porque são relidos a cada iteração e cada linha gasta token. Complementam isso um README com a arquitetura em alto nível, logs estruturados em JSON, comandos de verificação previsíveis e um `bin/setup` idempotente. [cap-111]
- **Linguagem "para LLM"**: o experimento mental de fevereiro de 2026, escrito pelo próprio Claude/GPT e que o autor chama de "achismo", imagina uma AST tipada com efeitos, contratos e proveniência, com o humano virando "autor de restrições". O benchmark de linguagens que ele cita em outubro de 2026 achou sete linguagens praticamente equivalentes; o que mudou o resultado foi a disciplina de teste. [cap-112, cap-116]

### Lendo o mercado

- **Hype vs. número**: "Seu grau de empolgação com IA é inversamente proporcional ao seu entendimento de IA." Os feitos de manchete, como a prova de Navier-Stokes (~130 bilhões de tokens) e o incidente da Hugging Face (~17.600 ações de ataque), são força bruta em escala industrial. O medo de uma "IA descontrolada" também é produto à venda. [cap-109, cap-115, cap-116]
- **Validade temporal**: em 2023 o autor dizia que uma janela de 2.048 tokens impedia projetos inteiros. Em fevereiro de 2026, ele colocou sozinho 4 projetos em produção em um mês com um agente. Manteve, porém, que não existe "mágica pra amador". Números, modelos (Opus, Fable, Sol, Grok…) e ferramentas envelhecem em meses; os mecanismos (probabilidade, contexto finito, arquivo estático, oráculo) não. [cap-109, cap-110, cap-116]

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Tarefa de reprodução (CRUD, formulário, relatório, integração com API documentada, migração de framework) | Delegar ao agente com testes e revisar pelo resultado | É território coberto pelo treino; a máquina reproduz melhor e mais rápido | cap-115 |
| Problema inédito, sem referência pública | Antes de gastar tokens, definir um oráculo (saída esperada, trajetória gravada, emulador/implementação de referência) e procurar material parecido (paper, projeto abandonado) pra pôr no contexto | Sem oráculo, o agente vira força bruta direcionada e gira | cap-115 |
| O agente começou a girar (tenta, regride, reverte, repete) | Parar e redefinir o "X" verificável, ou pivotar pra uma versão do problema que tenha mapa | Cada ciclo queima token sem convergir; o problema com referência anda em dias | cap-115 |
| Código privado ou legado sem corpus (ex.: COBOL de banco) | Não presumir que o modelo sabe; escrever as invariantes de domínio e esperar custo maior | O modelo não adivinha o que nunca viu | cap-115 |
| Classificar, rotear ou aplicar política | Usar classificador ou modelo pequeno especializado, não um LLM gigante de chat | Encaixe vale mais que densidade; custo, latência e erro de formato despencam | cap-116 |
| Escolher modelo | Não perseguir o último lançamento; ajustar o esforço (baixo pra tradução e resumo, alto pra varredura de vulnerabilidade, bug pesado e otimização) | Os modelos convergiram em programação geral | cap-116 |
| Projeto novo com agente | Criar CLAUDE.md/AGENTS.md com regras de estilo, testes, DI, estrutura e logging (o autor dá um template de ~500 tokens) | Nenhum LLM faz essas coisas por padrão | cap-111 |
| Defesa operacional (rate limit, timeout, API instável) | Decidir a política (ex.: circuit breaker de 120 s em vez de retry em manada) e listar no CLAUDE.md as categorias de código defensivo que o projeto precisa | O agente não conhece os pontos de falha operacional; implementa o caminho feliz | cap-110, cap-111 |
| Arquivo acima de 500 linhas ou função longa | Dividir por responsabilidade | Cabe numa leitura e o agente raciocina com atenção cheia | cap-111 |
| Nome que dá muito resultado no grep | Renomear pra algo específico (regra do autor: menos de 5 resultados) | Grep é a navegação primária do agente | cap-111 |
| Configuração espalhada (ex.: nome do modelo LLM em 24 arquivos) | Centralizar numa constante/config e injetar dependências | Trocar de provedor vira mudança de uma linha | cap-110, cap-111 |
| Teste que depende de seed manual, config fora do repo ou credencial | Fazer o teste rodar headless com um único comando documentado (README/CLAUDE.md/Makefile) | Sem isso o agente fica cego e o ciclo de feedback quebra | cap-111 |
| Feature com várias implementações possíveis | Montar um benchmark ou A/B antes de implementar | A escolha errada custa dias; o agente não sabe qual vence no seu caso | cap-110 |
| PR de terceiro | Auditar com LLM ("não confie na descrição do autor", checar regressão e cobertura); o humano decide entre mergear, corrigir ou recusar | A descrição conta o que a pessoa achou que fez | cap-113, cap-114 |
| Depois de muitos merges | Rodar uma auditoria de código morto, duplicação, valor mágico, cobertura e docs; consolidar à mão duplicações entre PRs | A auditoria por diff não enxerga padrão repetido entre PRs diferentes | cap-113, cap-114 |
| Revisar e subir demora mais que desenvolver | Automatizar a checagem mecânica, fazer deploy pequeno e frequente, medir contra uso real | Corrigir depois ficou rápido e barato | cap-114 |
| Não dá pra reverter rápido | Arrumar a infra (git, container, rollback, observabilidade) antes de acelerar | A velocidade só é segura com rollback de verdade | cap-114 |
| Efeito irreversível (pagamento, saúde, segurança física) | Régua mais rígida: testes de cenário (concorrência, falha parcial, processo morrendo no meio), mais de uma trava por modo de falha, staging e feature flag | Reverter o código não desfaz o dano que já saiu do sistema | cap-114 |
| Migration de banco | Usar expand-contract: adicionar o formato novo, migrar os consumidores, só então remover o velho | Estado não se desfaz com `git revert` | cap-114 |
| Biblioteca ou pacote público | Tratar semver como gate (breaking change só em major) | Quem paga a quebra é quem depende de você, e reage mais devagar | cap-114 |
| Publicar projeto aberto | Instalação em um comando, CI com testes, lint e auditoria de segurança, e README que começa pelo problema resolvido | Sem isso é um "arquivo morto público"; ninguém liga pra sua stack | cap-113 |
| Empacotar pra várias plataformas | Compilar uma vez por arquitetura e reempacotar o mesmo binário (tarball, AUR, Homebrew, Docker); release disparada por tag `v*.*.*`, com changelog extraído | Mesmo byte em todo formato; menos CI; agente consegue soltar a release | cap-113 |
| Segredos de deploy num repo público | Versionar só `config/deploy.env.example`; o real fica no `.gitignore` | Projeto público sem vazar host, usuário ou caminho | cap-113 |
| Ferramenta "que deve ser mais rápida" | Medir antes de adotar | A VM local do autor ficou mais lenta que o runner do GitHub, ao contrário do que ele apostava | cap-114 |

## Armadilhas e mitos

- "Um prompt bem detalhado gera o sistema pronto" → one-shot serve pra demo. Produção exige iteração, porque os problemas só aparecem quando o software encontra a realidade. [cap-110]
- "Terminei, está done" → software pronto é software morto. Pós-produção traz features, refactors e hardening. [cap-110]
- "O modelo raciocina e só às vezes alucina" → ele completa texto por probabilidade o tempo todo. Pesquisas citadas (GSM-Symbolic) mostram queda forte só de trocar números ou incluir uma frase irrelevante. [cap-109, cap-115]
- "Mais parâmetros, modelo melhor" → qualidade depende de dados, algoritmo e otimização. A guerra de parâmetros acabou e ninguém publica mais esse número. [cap-109, cap-116]
- "O agente aprende meu domínio com o tempo, como um estagiário" → o modelo é um arquivo congelado e cada sessão recomeça com amnésia. Quem aprende é você, e só fica o que você documentar. [cap-115]
- "IA treinando IA vai resolver" → isso é RL, que existe há décadas, só funciona com recompensa verificável barata e produz outro arquivo congelado. [cap-115]
- "Se a IA escreve CRUD, programador acabou" → ela substitui a parte de reprodução. Continua humano o sistema que ninguém construiu, o domínio sem documentação, o código que não pode ser mostrado ao modelo e o problema em que nem se sabe escrever o teste do "certo". [cap-115]
- "Benchmark prova que IA faz engenharia de software" → o SWE-bench é só Python, em 12 repositórios open source famosos, ou seja, território com mapa. No RCT da METR, devs experientes em repositórios maduros apostavam em 24% de aceleração e ficaram 19% mais lentos. [cap-115]
- "Comentário é code smell" → para o agente, o comentário que explica o porquê é o contexto mais acessível. Apagá-lo no review tira informação que o próprio agente vai querer ler. [cap-111]
- "Preciso migrar para a linguagem ideal para LLM" → no benchmark citado pelo autor as linguagens saíram equivalentes. O que muda o resultado é a disciplina de teste. [cap-112, cap-116]
- "Deploy rápido significa subir sem testar" → só é seguro depois que testes, CI, rollback e observabilidade existem. [cap-114]
- "Revisar linha a linha me protege" → em volume grande isso não faz diferença, e código parado não melhora sozinho. Só uso real mostra o que precisa melhorar. [cap-114]
- "Se der errado eu reverto" → o rollback não desfaz pagamento processado nem migration que apagou coluna. [cap-114]
- "Sou sênior; a IA errou três vezes, então não serve" → você também erra, e os testes pegam o erro da IA em minutos. Usar só como autocomplete deixa a maior parte do ganho na mesa. [cap-110]
- "O valor do projeto está na stack" → ninguém liga pra stack. O topo do README é o problema que o projeto resolve e pra quem; detalhe técnico vai em `docs/`. [cap-113]
- "Engenheiro de prompt / curso de prompt" → o autor chama de pega-trouxa. O que funciona é prompt conciso e estruturado e entender o mecanismo. [cap-109]
- "A IA fugiu do controle / provou um teorema sozinha" → é força bruta cara com verificação automática e apoiada no trabalho de humanos. O medo também é produto à venda. [cap-115, cap-116]

## Checklist para aplicar num sistema real

**Antes de delegar**
- [ ] Esta parte é reprodução (já existe pronta em público) ou é inédita? Se for inédita, qual é o oráculo que define "certo"?
- [ ] Existe material de referência (disassembly, implementação parecida, paper) pra colocar no contexto?
- [ ] Algum trecho é, na verdade, classificação, e caberia num modelo pequeno ou especializado em vez de um LLM de chat?
- [ ] Eu, humano, sei o que pedir, quais são os pontos de falha operacionais e quais decisões são minhas?

**Preparando o repositório pro agente**
- [ ] Existe CLAUDE.md/AGENTS.md curto e imperativo com regras de estilo, testes, DI, estrutura, logging e as categorias de código defensivo do projeto?
- [ ] Os testes rodam com um único comando, headless, sem credencial nem seed manual? `bin/setup` é idempotente?
- [ ] Os arquivos ficam abaixo de ~500 linhas, as funções são pequenas e os nomes retornam poucos resultados no grep?
- [ ] Os comentários explicam o porquê (bug, restrição, issue, commit) e não o óbvio?
- [ ] As configurações voláteis (modelo LLM, provedores, endpoints) estão centralizadas e as dependências injetadas?
- [ ] A estrutura de diretórios segue a convenção do framework e é previsível?
- [ ] Os logs são estruturados (JSON) e as mensagens de erro incluem o valor recebido e o formato esperado?

**Durante o trabalho com o agente**
- [ ] Cada feature vem com teste e cada bug fix com teste de regressão? O CI roda em todo push?
- [ ] Estou iterando em releases pequenas, ou tentando acertar tudo numa spec gigante?
- [ ] O agente está girando? Se sim: parar, redefinir o oráculo ou mudar a abordagem.
- [ ] Minhas descobertas (invariantes, decisões, hipóteses descartadas) estão sendo registradas em docs, CLAUDE.md ou testes, já que o modelo não guarda nada?
- [ ] Antes de implementar a opção cara, fiz benchmark ou A/B?

**Revisão e entrega**
- [ ] Os PRs passam por auditoria hostil (não confiar na descrição, checar regressão, cobertura, segredos, semver, prompt injection)?
- [ ] Depois de vários merges, alguém procura código morto, duplicação entre PRs, valores mágicos e docs desatualizadas?
- [ ] Consigo reverter em minutos (git + container)? Tenho observabilidade pra saber que quebrou?
- [ ] Há feature flag e staging de verdade limitando o raio de explosão?
- [ ] Algum efeito é irreversível (dinheiro, saúde, mundo físico, dado apagado)? Então há testes de cenário e mais de uma trava por modo de falha.
- [ ] As migrations seguem expand-contract? Bibliotecas públicas respeitam semver?
- [ ] Deploy e release saem com um comando ou uma tag, iguais em todos os projetos, a ponto do agente poder executar?
- [ ] O README começa pelo problema resolvido, e a instalação tem um caminho de um comando?
- [ ] Existe uso real (usuários, issues, telemetria) alimentando a próxima iteração, ou só a minha opinião?

## Frases-âncora

> "O agente escreve o código. Eu decido qual código escrever." — cap-110

> "Software pronto é software morto. Software vivo itera." — cap-110

> "Código limpo nunca foi moda. Virou infraestrutura." — cap-111

> "É melhor deployar mais rápido, porque corrigir depois ficou rápido e barato." — cap-114

> "LLM consegue replicar o que é aberto, mas não adivinha o que é privado." — cap-115

> "Seu agente queimando milhões de tokens em tentativa e erro está contribuindo exatamente zero de aprendizado novo pro modelo que ele usa." — cap-115

> "Ele nunca alucina, assim como nunca é sóbrio." — cap-109

> "LLM sozinho é um papagaio caro. LLM com agente é competente o suficiente pra trabalhar de verdade." — cap-116

## Onde aprofundar

| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-109 | [Akitando] #142 - Entendendo COMO ChatGPT Funciona - Rodando sua Própria IA (19/06/2023) | https://www.akitaonrails.com/2023/06/19/akitando-142-entendendo-como-chatgpt-funciona-rodando-sua-propria-ia/ | precisar explicar tokens, pesos, temperatura, transformers, quantização e GPU. Os números de contexto e as previsões são de 2023 e estão datados. |
| cap-110 | Software Nunca Está 'Pronto' — 4 Projetos, a Vida Pós-Deploy, e Por Que One-Shot Prompt É Mito (01/03/2026) | https://www.akitaonrails.com/2026/03/01/software-nunca-esta-pronto-4-projetos-a-vida-pos-deploy-e-por-que-one-shot-prompt-e-mito/ | alguém achar que um prompt entrega o sistema pronto, ou pra ver exemplos reais de bug que só aparece em produção. |
| cap-111 | Clean Code pra Agentes de IA (20/04/2026) | https://www.akitaonrails.com/2026/04/20/clean-code-para-agentes-de-ia/ | for organizar um repositório pra agentes ou escrever um CLAUDE.md (traz um template pronto). |
| cap-112 | AI Agents: Qual seria a melhor Linguagem de Programação para LLMs? (09/02/2026) | https://www.akitaonrails.com/2026/02/09/ai-agents-qual-seria-a-melhor-linguagem-de-programacao-para-llms/ | quiser o experimento mental (efeitos, proveniência, specs formais). É especulação gerada por LLM; o cap-116 relativiza. |
| cap-113 | Boas práticas de projetos de código aberto com LLM - O Mínimo (30/05/2026) | https://www.akitaonrails.com/2026/05/30/boas-praticas-projetos-codigo-aberto-llm-o-minimo/ | for publicar um projeto: instalação, CI, release por tag, empacotamento multiplataforma, `bin/deploy`, prompts de revisão de PR. |
| cap-114 | Parem de inventar desculpas e façam mais deploy! Com a IA, a premissa mudou. (22/09/2026) | https://www.akitaonrails.com/2026/09/22/parem-de-inventar-desculpas-e-facam-mais-deploy-a-premissa-mudou/ | for discutir ritmo de deploy, ambiente mínimo e os limites dessa conta (pagamento, banco, bibliotecas, lojas de app, empresas grandes). |
| cap-115 | Os Limites das LLMs: Boas em reproduzir o que existe. Caras pro que não existe (28/09/2026) | https://www.akitaonrails.com/2026/09/28/llms-sao-boas-em-reproduzir-o-que-existe-e-coisas-novas/ | o problema for inédito ou privado, o agente estiver girando, ou alguém prometer que a IA "aprende" ou "inventa". |
| cap-116 | LLMs + Agentes = competente o suficiente (12/10/2026) | https://www.akitaonrails.com/2026/10/12/llms-agentes-competente-o-suficiente/ | quiser a síntese 2019–2026 (convergência, harness, encaixe vs. densidade). |

Síntese completa do autor sobre 2026: livro gratuito *Competente o Suficiente*, em https://ailair.akitaonrails.com/pt-br/book/
