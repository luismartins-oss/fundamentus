# Fundamentus

Skill para agentes de IA de código (Claude Code e compatíveis com `SKILL.md`) que acerta os **fundamentos de qualquer sistema de software**: projetar, revisar, depurar, decidir tecnologia e ensinar programação do zero ao expert. O conhecimento foi destilado dos textos de **Fabio Akita** (canal Akitando e blog [akitaonrails.com](https://www.akitaonrails.com)).

> *An agent skill for software-system fundamentals (architecture, data, concurrency, networking, security, storage, deploy, Git, testing, performance), distilled from Fabio Akita's writing. Content in Brazilian Portuguese.*

## O que ela faz

| Pedido | Como a skill responde |
|---|---|
| Projetar, revisar, depurar ou decidir tecnologia | Varre 10 camadas (problema e modelo, representação e memória, estruturas e dados, concorrência, rede, segurança, disco e durabilidade, deploy, testes e Git, desempenho). A resposta sai em formato fixo: resposta direta, tabela de achados com fonte, "fora do escopo" e "sem informação", como verificar e o que ler |
| Explicar um conceito | Mecanismo, por que importa num sistema e capítulos pra ler |
| Montar uma trilha de estudo | Etapas pelas partes do compêndio, com prática e um sinal pra avançar em cada etapa, no ritmo das horas disponíveis |

Cada afirmação aponta para o capítulo de origem (`cap-NNN`) e para a URL do post original.

## Instalação

Claude Code (macOS/Linux):

```bash
git clone https://github.com/luismartins-oss/fundamentus ~/.claude/skills/fundamentus
```

Windows (PowerShell):

```powershell
git clone https://github.com/luismartins-oss/fundamentus "$env:USERPROFILE\.claude\skills\fundamentus"
```

A skill é acionada sozinha quando o pedido tem a ver com fundamentos de sistemas e também pode ser chamada com `/fundamentus`. Outros agentes que leem `SKILL.md` usam `~/.agents/skills/fundamentus`.

## Conteúdo

```
SKILL.md                      método, camadas, formato de saída e princípios transversais
referencias/
  mapa.md                     os 120 capítulos com o resumo do autor e a URL
  computacao.md               hardware, memória, bits, C
  algoritmos.md               árvores, grafos, compressão
  linguagens.md               compilada × interpretada, GC, runtime, escolha de linguagem
  bancos-de-dados.md          B-Trees, índices, transações, SQL × NoSQL
  concorrencia-desempenho-escala.md   concorrência, filas, cache, Rinha de Backend, teste de carga
  web-e-redes.md              camadas de rede, sockets, HTTP, proxies, VPN
  seguranca.md                criptografia, senhas, segredos, entropia, vazamentos
  linux-e-armazenamento.md    boot, pacotes, discos, sistemas de arquivos, durabilidade
  devops-e-containers.md      12 fatores, deploy, containers por dentro
  git.md                      modelo interno, histórico limpo, recuperação
  engenharia-de-software.md   testes, modelagem, estimativa, Brooks, reescrita
  ambiente-dev.md             SO, terminal, ambiente reproduzível
  programando-com-ia.md       LLMs, agentes de código, oráculo, ciclo pós-deploy
  licoes-da-historia.md       por que o Rails deu certo, licenças, reescritas
  aprendizado-e-carreira.md   prática deliberada, trilha, mercado
```

Cada referência traz, nesta ordem: ideia central, conceitos, heurísticas de decisão, armadilhas e mitos, checklist, frases-âncora e onde aprofundar.

## Como foi feita

1. **Fonte.** Antologia de 120 textos do autor (de 2006 a 2026), tirada do repositório público do blog [akitaonrails/akitaonrails.github.io](https://github.com/akitaonrails/akitaonrails.github.io), commit `956fc9e`. Segurança inclui também 8 posts do blog fora da antologia.
2. **Destilação** por tema, lendo os capítulos inteiros.
3. **Fidelidade.**
   - As 122 frases-âncora foram conferidas automaticamente e são literais.
   - O que não está nos textos aparece marcado como **complemento**, inclusive correções de afirmações datadas ou erradas dos próprios textos.
   - Conselhos datados (ferramentas e versões da época) vêm sinalizados, junto com o princípio que continua valendo.
4. **Teste.** Os mesmos cenários foram respondidos por um agente com e sem a skill. Com ela, a varredura pegou o que a resposta sem skill deixou passar, como backup testado antes de migração, observabilidade durante pico e custo de CPU de KDF num pico de logins. As respostas também passaram a citar as fontes.

## Créditos e licença

Os textos e as ideias são de **Fabio Akita**, publicados em [akitaonrails.com](https://www.akitaonrails.com) sob [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.pt-br). Esta skill é uma obra derivada **não oficial**: o autor não participou nem a endossa.

Ela é distribuída sob a mesma licença, **CC BY-NC-SA 4.0** (veja `LICENSE`):

- dê crédito;
- não use comercialmente;
- obras derivadas saem sob a mesma licença.

Para a parte de IA, o autor consolidou 2026 no livro gratuito [*Competente o Suficiente*](https://ailair.akitaonrails.com/pt-br/book/).
