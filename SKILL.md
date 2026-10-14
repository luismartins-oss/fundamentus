---
name: fundamentus
description: Use when designing, reviewing, debugging or explaining the fundamentals of any software system (architecture, memory and data representation, data structures, databases, concurrency, performance and scaling, networking, security, Linux, storage and backups, deploy and containers, Git, testing), when a technology choice or rewrite is driven by hype ("Ruby é lento", NoSQL, microservices, Kubernetes), or when someone wants to learn programming from zero to expert. Gatilhos: fundamentos, arquitetura, revisar sistema, escalar, gargalo, segurança, aprender programação, trilha de estudo.
---

# Fundamentus

Fundamentos de qualquer sistema, destilados dos 120 textos do *Compêndio de Programação* (Fabio Akita). **Princípio:** entenda o mecanismo antes de escolher a ferramenta, e meça antes de decidir.

## Quando usar

- Projetar um sistema ou feature com carga, dados ou segurança em jogo
- Revisar os fundamentos de um projeto herdado ou de uma mudança estrutural
- Depurar lentidão, perda de dado, falha intermitente ou problema de rede
- Decidir tecnologia (linguagem, banco, infra) ou reescrita
- Ensinar um conceito ou montar uma trilha de estudo

Não use em dúvida pontual de sintaxe/API ou tarefa mecânica sem decisão.

## Como responder

Escolha o formato pelo tipo de pedido:

| Pedido | Formato |
|---|---|
| Projetar, revisar, depurar ou decidir | **Varredura completa** (abaixo) |
| Explicar um conceito | Mecanismo em poucas frases → por que importa num sistema → 1–2 capítulos pra ler |
| Aprender / trilha de estudo | Trilha pelas partes de `referencias/mapa.md` com prática e um sinal pra avançar em cada etapa, usando `referencias/aprendizado-e-carreira.md`. Calibre o ritmo pelas horas que a pessoa tem: o autor estima ~1.400 h do zero até júnior (cap-002) |

### Varredura completa

1. Passe por **todas** as 10 camadas abaixo usando a coluna "Pergunta que pega o erro". Cada camada termina com um achado, em "Fora do escopo" ou em "Sem informação".
2. Nas camadas que tiveram achado, leia na referência indicada as seções **Heurísticas de decisão**, **Armadilhas e mitos**, **Checklist** e **Onde aprofundar** (é ali que estão as URLs). A referência inteira só precisa ser lida na camada central do pedido.
3. Responda com estas quatro partes, nesta ordem:
   1. **Resposta direta**: a decisão, o diagnóstico ou o plano, em poucas linhas.
   2. **Varredura de fundamentos**: tabela `| Camada | Situação | Risco | Ação | Fonte |` só com as camadas que têm achado. Na coluna Fonte vai o `cap-NNN` ou `blog AAAA-MM-DD`; quando a ação vem de conhecimento geral, escreva `complemento`. Embaixo vêm duas linhas: `Fora do escopo:`, com as camadas que não se aplicam e o motivo, e `Sem informação:`, com o que precisaria ser perguntado ou medido.
   3. **Como verificar**: a medição, o teste ou o experimento que confirma ou derruba a decisão.
   4. **Pra aprofundar**: 1–3 capítulos (`cap-NNN`, título curto, URL). Quando a referência não tiver o capítulo certo, procure em `referencias/mapa.md`, que traz os 120 capítulos com o resumo do autor.

### As 10 camadas

| # | Camada | Pergunta que pega o erro | Referência |
|---|---|---|---|
| 1 | Problema e modelo | Que regra de negócio manda? O escopo cabe no prazo? | `engenharia-de-software.md` |
| 2 | Representação e memória | Que tipo e região guardam cada dado? Dinheiro em float? Encoding? | `computacao.md` |
| 3 | Estruturas e dados | Qual o padrão de acesso? Índice, transação e consistência certos? | `algoritmos.md`, `bancos-de-dados.md` |
| 4 | Concorrência | Que recurso finito é disputado (conexão, thread, linha do banco)? Onde se limita a vazão? | `concorrencia-desempenho-escala.md` |
| 5 | Rede e fronteiras | Em que camada pode falhar? Que entrada externa chega? Há timeouts? | `web-e-redes.md` |
| 6 | Segurança | Se o banco vazar hoje, o que acontece? Segredos, senhas, entropia, cadeia de confiança? | `seguranca.md` |
| 7 | SO, disco e durabilidade | "Gravado" quer dizer o quê? Alguém já restaurou o backup? | `linux-e-armazenamento.md` |
| 8 | Deploy e operação | 12 fatores? Build imutável, rollback, logs e métricas? | `devops-e-containers.md` |
| 9 | Código, testes e histórico | O que impede regressão? O histórico é legível? Tem segredo no Git? | `engenharia-de-software.md`, `git.md` |
| 10 | Desempenho e escala | Pra onde vai o tempo, medido em percentil? Qual o gargalo real? | `concorrencia-desempenho-escala.md` |

Conforme o pedido, carregue também: `linguagens.md` (escolha de linguagem ou runtime), `ambiente-dev.md` (máquina e ferramentas), `programando-com-ia.md` (construir com agentes de IA), `licoes-da-historia.md` (framework, licença, reescrita), `aprendizado-e-carreira.md` (estudo e carreira). Todos ficam em `referencias/`.

## Princípios transversais

- **Mágica é custo que você não vê.** Toda abstração cobra em ciclos, memória ou pausa. Dinheiro vai em inteiro ou decimal, nunca em float. [cap-020]
- **Escolha a estrutura pelo padrão de acesso.** Índice acelera leitura e encarece escrita; olhe o pior caso com volume real. [cap-027, cap-054]
- **Relacional maduro é o padrão.** Postgres ou MariaDB gerenciado, com cache, réplicas e fila, resolve quase tudo. NoSQL distribuído é pra volume de escrita que quase ninguém tem. [cap-055, cap-056]
- **Web é limitada por I/O, não pela linguagem.** Na Rinha de Backend, 16 linguagens bateram no mesmo teto; o ganho veio de limitar vazão, pool e rede. [cap-062, cap-063]
- **Meça antes e depois.** APM, teste de carga, percentis (nunca média) e EXPLAIN. Otimização sem número é chute. [cap-060, cap-061]
- **Recursos são finitos: coordene.** Concorrência não é paralelismo. Servidor web descartável ("shared nothing"), trabalho lento em job. [cap-057, cap-058, cap-060]
- **Pior caso em segurança.** O banco vai vazar: senha com KDF lento, segredo fora do repositório, entrada externa é "material radioativo", não invente criptografia. [cap-052, `seguranca.md`]
- **Disco falha e cache mente.** Redundância, checksum, flush e backup em camadas restaurado de verdade. Só se recupera o que foi commitado. [cap-068–071, cap-026]
- **Infra é software descartável.** Container é processo; os 12 fatores são o mínimo; build imutável e rollback em segundos. [cap-072, cap-075, cap-076]
- **Teste é seguro contra regressão.** CI verde e deploy pequeno e frequente; software nunca fica pronto. [cap-086, cap-110]
- **Desconfie da reescrita.** Pelo efeito do segundo sistema e pela Lei de Brooks, reescrever tudo ou pôr gente num projeto atrasado costuma piorar. [cap-089]
- **Não terceirize a decisão.** O contexto decide: dê critérios, não receita copiada de big tech. [cap-105]

## Erros comuns

- **Responder só a camada óbvia.** O que derruba produção costuma estar nas outras (backup, observabilidade, segredo); a varredura existe pra isso.
- **Citar conselho datado como atual** (Heroku 2022, Ubuntu 2019, modelos de IA). Use o princípio e avise a data.
- **Atribuir ao autor uma linha marcada "(complemento, não está no compêndio)".** Ela é correção ou preenchimento.
- **Despejar o checklist inteiro.** Entram só os achados e as linhas "Fora do escopo" e "Sem informação".
- **Citar capítulo de memória.** Use o `cap-NNN` que a referência ou o mapa trazem para aquela afirmação.

## Fonte

Destilado do *Compêndio de Programação*, uma antologia não oficial montada a partir do repositório público do blog [akitaonrails/akitaonrails.github.io](https://github.com/akitaonrails/akitaonrails.github.io), commit `956fc9e`. Os textos são de Fabio Akita, sob CC BY-NC-SA 4.0: uso não comercial e com crédito. As frases entre aspas nas referências são literais e foram conferidas contra os textos.
