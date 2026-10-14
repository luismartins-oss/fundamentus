# Segurança e criptografia
> Destilado de: cap-077, cap-078, cap-079, cap-080, cap-081, cap-082 e posts do blog 2013-10-19, 2014-01-13, 2014-05-22, 2014-08-28, 2025-09-09, 2026-08-01, 2026-08-07 e 2026-08-16 (textos de Fabio Akita, CC BY-NC-SA 4.0).

## Ideia central

Para o autor, segurança não é um estado que se alcança, e sim gestão de risco: parta do pior caso (o banco *vai* vazar, a máquina *pode* estar comprometida, alguém *vai* cair em phishing) e projete para que o vazamento custe pouco, seja detectável e tenha recuperação. A força de um sistema vem menos do tamanho da chave e mais de como as primitivas são usadas, por isso programador não inventa criptografia nem escolhe algoritmo ou modo pelo tutorial mais simples: usa o padrão maduro do framework ou da biblioteca e o mantém atualizado. O agente deve checar sempre três coisas: entropia real na origem de todo segredo, chave privada que nunca sai do lugar e uma cadeia de confiança explícita. O elo mais fraco quase nunca é a matemática, e sim o humano (senha reusada, clique em link, permissão demais) e a ilusão de que selo, processo ou "ninguém achou nada até agora" significam segurança.

## Conceitos que todo sistema depende

### Primitivas: o que cada uma garante (e o que não garante)

- **Criptografia simétrica**: uma função recebe texto, chave secreta e IV e devolve texto embaralhado; a mesma chave desfaz a operação. Cifras de bloco modernas (AES/Rijndael, chaves de 128 a 256 bits) aplicam rodadas de substituição (S-boxes) para gerar *confusão* (nenhuma correlação visível entre entrada e saída) e *difusão* (mudar 1 bit muda cerca de metade da saída), dois princípios de Shannon. AES roda em instruções da própria CPU, o que acelera e também dificulta ataques de side-channel (tempo, energia, radiação). [cap-077]
- **Força bruta e criptanálise**: um algoritmo é bom quando a única saída é testar o espaço inteiro de chaves e esse espaço é inviável (cada bit a mais dobra o trabalho). Criptanálise usa padrões e estatística para cortar o espaço em ordens de grandeza. Turing quebrou a Enigma em ~20 min explorando duas coisas: uma letra nunca virava ela mesma e as mensagens tinham trechos previsíveis. O DES (56 bits) cai em ~26 h com FPGA (dado de 2012). [cap-077]
- **Modos de operação**: ECB cifra blocos iguais em saídas iguais, então vaza padrões, e não usa IV. CBC encadeia os blocos a partir de um IV aleatório; não paraleliza e fica vulnerável a padding oracle quando mal implementado. GCM é modo contador, paralelizável, e gera uma tag de autenticação. O IV pode ser público, mas precisa ser novo e aleatório a cada cifragem. A recomendação do autor: "Se precisar escolher, use GCM." [cap-078, cap-079, blog 2014-01-13]
- **Hash (message digest)**: função de mão única. Aceita entrada de qualquer tamanho, devolve saída de tamanho fixo e tem efeito avalanche. Serve de *impressão digital* para integridade (contrato, ISO, blocos no ZFS/Btrfs). MD5 e SHA-1 têm colisões que dá para controlar e não servem para assinatura; o caminho é SHA-2 ou SHA-3. MD5 ainda tem usos em que colisão não importa. [cap-077, cap-079]
- **Colisão e paradoxo do aniversário**: colisões são mais prováveis do que a intuição sugere (numa sala de 30 pessoas, ~70% de chance de dois aniversários no mesmo dia). Hash não é identificador único. Para ID use UUID/GUID: tem cara parecida, mas foi desenhado para que a colisão seja praticamente nula. [cap-077]
- **MAC, HMAC e extensão de comprimento**: um MAC é um hash usado para autenticar. Com `hash(segredo + mensagem)` em MD5/SHA-1, o atacante que conhece só o hash e o *tamanho* do segredo consegue anexar blocos e gerar um MAC válido (o hash é o próprio estado interno da função). O HMAC aplica o hash em duas camadas com a chave. Na errata, o autor avisa que a versão dele é simplificada: use a função pronta de uma biblioteca como a libsodium. [cap-077]
- **Codificação não é cifra**: Base64 (também chamado Radix-64 ou "ASCII armor") converte binário em texto (3 bytes viram 4 caracteres) e não tem chave nem embaralhamento. CRC/checksum detecta erro acidental de digitação ou transmissão. Nenhum dos dois esconde dados ou protege contra adulteração intencional. [cap-078]
- **Derivação de chave para senhas (KDF)**: PBKDF2, bcrypt, scrypt e Argon2 são lentos de propósito e têm custo configurável. Scrypt e Argon2 também exigem bastante memória, o que encarece ataques com GPU e ASIC. Eles transformam uma senha fraca num valor longo, e o hash gravado já carrega salt e custo. SHA-512 é rápido por projeto, o que é bom para integridade e ruim para senha: uma GPU calcula bilhões de SHA por segundo e só centenas de bcrypt. [cap-078, cap-081]
- **Salt**: valor aleatório por usuário, guardado em aberto junto do hash. Faz a mesma senha gerar hashes diferentes, inutiliza tabelas pré-computadas e obriga o atacante a quebrar uma conta de cada vez. [cap-078, cap-081]

### Chave pública, assinatura e cadeia de confiança

- **Par de chaves assimétricas**: o que a chave pública cifra, só a privada abre, e vice-versa. A pública pode circular; a privada nunca sai da máquina ou do servidor. RSA nasce de dois primos gigantes; curva elíptica (ECC) dá chaves menores e mais fortes (ECC de 256 bits supera RSA de 3072 bits). RSA é caro demais para cifrar volume, por isso o padrão é **híbrido**: a assimétrica só transporta ou combina uma chave simétrica, e o AES cifra os dados. [cap-078, cap-079]
- **Diffie-Hellman**: duas partes chegam ao mesmo segredo trocando apenas valores públicos; a segurança vem do problema do logaritmo discreto. Sozinho, o DH não autentica ninguém e cai em man-in-the-middle. A versão efêmera (ECDHE) gera chaves novas a cada sessão e dá *forward secrecy*: quebrar a chave de identidade no futuro não abre as sessões passadas. [cap-078]
- **Impressão digital e assinatura digital**: a fingerprint é o hash do conteúdo; a assinatura é esse hash assinado com a chave privada de quem atesta. Baixar o `.sha512` pelo mesmo canal da ISO não prova nada, porque quem adulterou um adultera o outro. Verificar uma assinatura GPG com a chave do publicador, obtida antes por canal confiável, prova. [cap-079]
- **Certificado e cadeia de confiança**: o certificado X.509 empacota a chave pública e dados como o domínio, assinados por uma CA. A CA, por sua vez, é assinada por uma raiz que veio instalada no sistema operacional ou no navegador. A confiança é transitiva e acaba num ato físico: você instalou o SO de uma ISO oficial ou comprou a máquina nova numa loja. Máquina usada sem formatar ou navegador baixado de site genérico não dão garantia nenhuma. O autor chama isso de "triângulo de confiança": dois desconhecidos só confiam um no outro por meio de um terceiro em que ambos confiam. O Let's Encrypt prova a posse do domínio com um desafio no DNS. [cap-078, cap-079]
- **TLS é uma composição**: certificado para identidade, troca de chaves efêmera (ECDHE), cifra simétrica rápida (AES-GCM) e hash para integridade, negociados como *cipher suite* (ex.: `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`). "Conexão segura" não troca o cabo: a rede é a mesma, só que os pacotes vão cifrados. O SSH faz uma combinação parecida; ali a confiança vem de alguém de confiança ter colocado a sua chave pública em `authorized_keys`. [cap-078, cap-079]
- **Chaves SSH na prática**: use ed25519 (`ssh-keygen -t ed25519 -a 100`) com passphrase longa e permissão 600 em `~/.ssh` e nas chaves privadas. Um `ssh-agent` com expiração (`ssh-add -t 30m`) evita digitar a passphrase toda hora, e dá para ter uma chave por serviço. O nível acima é a chave de hardware (YubiKey 5 com PIV, ou `ed25519-sk` via FIDO2): a privada nasce e fica no dispositivo, e no disco sobra só um *stub*. Quando a chave estiver funcionando, desligue o login por senha no `sshd`. [blog 2025-09-09]

### Hash como estrutura: integridade, histórico e auditoria

- **Árvore de Merkle**: as folhas são hashes dos pedaços, cada nó é o hash dos filhos e a raiz resume tudo. Para provar que um pedaço pertence ao conjunto bastam ~log2(n) hashes (cerca de 11 para os ~1.800 pedaços de uma ISO), não todos. É assim que o BitTorrent valida pedaços vindos de peers não confiáveis, o ZFS detecta corrupção, Cassandra e Riak sincronizam réplicas e o Certificate Transparency mantém um log auditável dos certificados emitidos. [cap-079]
- **Encadeamento por hash (Git, blockchain)**: cada commit carrega o hash do pai (um Merkle DAG). Mudar um commit antigo obriga a reescrever todos os seguintes, e qualquer clone percebe. Isso *detecta* a adulteração, não *impede*; a resiliência vem de existirem várias cópias para comparar. No Bitcoin, o equivalente a um rebase custa refazer a prova de trabalho de todos os blocos seguintes, e é esse custo que dá valor ao ledger. [cap-079]
- **Compromisso (commit-reveal)**: você publica `hash(valor + nonce)` hoje e revela `(valor, nonce)` depois. Quem publicou não consegue trocar o valor (*binding*) e quem observa não descobre o valor antes da revelação (*hiding*), desde que o nonce seja aleatório. É a base de logs append-only verificáveis, de provas de conhecimento zero e de esquemas que fecham janelas de ataque (ex.: commit-reveal em mempools). [blog 2026-08-07, blog 2026-08-16]
- **Verificação pública versus custódia**: segurança baseada em sala trancada, lacre e credenciado é confiança em gente que você não vê. O desenho forte publica tudo de um jeito que qualquer um confere: log append-only replicado e assinado por observadores independentes, provas de inclusão e contagens que precisam fechar (ex.: número de folhas igual ao número de autenticações). Assim a falha fica *visível* em vez de silenciosa. Mas uma verificação que só especialista entende não é pública para quem precisa confiar nela. [blog 2026-08-07]
- **Público não é anônimo**: uma blockchain garante integridade, não sigilo. Endereços são pseudônimos, e heurísticas (inputs comuns, detecção de troco, contato com exchange que exige KYC) amarram uma identidade a todo o histórico, inclusive para trás. Uma saída pública derivada de um segredo fraco também vira *oráculo* para quem enumera candidatos offline. [blog 2026-08-07, blog 2026-08-01]

### Aleatoriedade: de onde vem o segredo

- **PRNG e entropia**: software não gera número "verdadeiramente" aleatório. Um PRNG expande uma semente e precisa ter saída uniforme, ser imprevisível e resistir a backtracking (ver a saída não deve revelar valores anteriores). A fonte certa é o sistema operacional (`/dev/random`), que pode ser reforçado com entropia de hardware (TPM via `rngd`). A força real de uma chave é a entropia inicial, não o tamanho do número. [cap-079]
- **A entropia efetiva é o tamanho real da chave**: no caso ColdCard (firmwares de 2021 a 2026), um `#ifndef` verificou se a macro *existia* em vez de checar o *valor* (`MICROPY_HW_ENABLE_RNG` definida como `0`). O build passou, e seeds de 256 bits passaram a nascer de um PRNG de software semeado pelo UID do chip e por timers: cerca de 40 bits efetivos. A 1 bilhão de testes por segundo, 40 bits se varrem em ~18 min; 128 bits levariam ~10²² anos. Usando a blockchain como oráculo, o atacante enumerou estados e drenou pelo menos 1.128 BTC sem tocar nos aparelhos. O commit do bug não passou por PR, e nenhum teste provava de onde a seed vinha. Hash e XOR de valores reproduzíveis não criam entropia, e uma saída que passa em teste estatístico ainda pode ser previsível. [blog 2026-08-01]

### Risco, responsabilidade e usabilidade

- **Segurança é gestão de risco, não risco zero**: computador útil está ligado e conectado, logo está exposto. Gerenciar risco é escolher quais riscos valem o retorno e preparar contingência (plano B, C, D) que limite o estrago quando o pior acontecer. Não basta resolver o incidente: é preciso garantir que ele não se repita ou que o impacto seja menor da próxima vez. [cap-080]
- **Proteção jurídica não é segurança técnica**: para muitas empresas, segurança é cumprir o que a regulação aceita (Sarbox, auditorias) para que, se houver brecha, ninguém responda criminalmente. Startup no início troca segurança por velocidade; na escala, uma invasão passa a significar negligência, e auditorias de M&A e IPO cobram processos e responsabilidades definidas. [cap-080]
- **Segurança contra usabilidade**: quanto mais usável se tenta fazer algo, mais inseguro tende a ficar. O autor criticou o JWT porque ele exigia que o desenvolvedor escolhesse algoritmos e modos e não bloqueava as combinações obviamente inseguras. Uma API de segurança deve vir com defaults seguros. [cap-079]
- **Privacidade reduz a superfície de ataque**: o motivo prático para proteger dados pessoais é que dado público torna phishing fácil para qualquer amador (nome dos pais, do cachorro, onde você está agora). [cap-080]
- **Valor em risco não é perda esperada**: "6,9 milhões de BTC expostos" mede a dependência de criptografia frágil, não o que um atacante de fato levaria. Da mesma forma, a probabilidade de uma etapa do ataque dar certo não é a probabilidade de roubo. Calibre antes de alarmar. [blog 2026-08-16]
- **Autenticação humana é o elo que a matemática não fecha**: nenhum algoritmo prova que a pessoa na sua frente é quem diz ser. O melhor desenho reduz a confiança a um evento único, público e caro, de modo que cada falsificação renda uma unidade só (fraude de varejo, visível) e as anomalias apareçam nas contagens. Um banco biométrico central é ponto único de falha permanente (Aadhaar; OPM perdeu 5,6 milhões de digitais em 2015). [blog 2026-08-07]

### Segredos, sessão, dependências e operação

- **Segredo fora do código**: a configuração de ambiente vai em variáveis de ambiente. Em desenvolvimento, o `.env` fica fora do Git (`.gitignore`) e um modelo (`.env.development`) vai versionado para o próximo dev. O token de produção é diferente do de dev e gerado por ferramenta (`rake secret`). *Datado:* é Rails 3/4 de 2013, com `secret_token.rb`. O princípio continua valendo: segredo nunca no repositório e um por ambiente. [blog 2013-10-19]
- **Entrada do usuário não indexa estado interno**: no desafio de 2014, `session[params[:token]]` deixava o atacante escolher a chave `_csrf_token`, que toda sessão Rails tem por padrão. Como `User.find("1abc")` vira `find(1)` por conversão implícita, bastava recarregar a página até o token CSRF começar com "1" para trocar a senha do admin pela rota de reset. *Datado:* Rails 4/ActiveRecord de 2014. Princípios que seguem valendo: nunca confiar em parâmetro, desconfiar de coerção implícita e usar autenticação pronta (Devise na época). [blog 2014-08-28]
- **Proteções do framework**: frameworks maduros já trazem sanitização de SQL via ORM e proteção contra CSRF, session hijacking e replay. Trocar o ORM por SQL escrito à mão "por performance", interpolar string em query ou desligar uma proteção sem entender o motivo reabre esses buracos. Login, "esqueci minha senha" e autenticação não se fazem do zero. [cap-077, cap-082, blog 2014-05-22]
- **Dependências são superfície de ataque**: vendorizar (copiar para dentro do projeto) OpenSSL, `cryptography` e afins congela versões com falhas conhecidas que ninguém atualiza; o vazamento do Twitch (2021) tinha dezenas de cópias assim. Declare as dependências no gerenciador (`package.json`, `Gemfile`, `requirements.txt`) para que auditores automáticos (dependabot; no Rails, bundler-audit e Brakeman) cruzem as versões com bases de vulnerabilidades. Dependência puxada direto de um repositório Git escapa dessa checagem. [cap-082, blog 2014-05-22]
- **Fator humano e engenharia social**: a maioria dos grandes ataques é banal. Alguém liga ou manda um e-mail convincente para um funcionário com permissão demais. No Twitter, em 2020, uma ferramenta interna trocou e-mails, desligou 2FA e tomou 130 contas. Os dados que as pessoas expõem em redes sociais alimentam o spear phishing. Processos de governança (ITIL, Cobit) servem para reduzir a variância das decisões humanas, inventariar máquinas e acessos e evitar o servidor que só uma pessoa sabe que existe. [cap-080]
- **Autenticação do usuário final**: o risco imediato é o reuso, porque quando um site vaza, robôs testam a mesma senha em todos os outros. Senha boa é longa e aleatória (com 92 símbolos, 10 caracteres dão ~43 quintilhões de combinações, e cada caractere a mais multiplica por 92), guardada num gerenciador; só a senha mestra é decorada, em forma de passphrase longa. TOTP (RFC 6238) é um HMAC-SHA1 do seed compartilhado com o horário em janelas de 30 a 60 s, truncado em 6 dígitos, por isso depende de relógio sincronizado por NTP. Os fatores são algo que você sabe, algo que tem e algo que é. Biometria é irrevogável: senha se troca, digital não. [cap-081, blog 2026-08-07]
- **Dados em repouso**: a senha de login não protege o disco, porque basta tirar o SSD e ler em outra máquina. A encriptação de disco (BitLocker, FileVault, LUKS) deriva da sua senha, com KDF, uma chave que só cifra a *chave de montagem*; por isso trocar a senha não exige recifrar o disco inteiro. O swap também precisa estar cifrado, porque guarda o que estava na memória. Enclaves (SGX, Secure Enclave/T2, TPM) deixam segredos fora do alcance até do próprio SO. [cap-078, cap-079, cap-081]
- **Backup é defesa**: contra ransomware, a única garantia é backup com histórico (snapshots de semanas ou meses), porque o backup mais recente pode já conter os arquivos cifrados. [cap-081]
- **A estação de trabalho também é superfície de ataque**: instaladores piratas, cheats e drivers de sites não oficiais são os principais vetores de malware e ransomware. Wi-Fi público permite DNS poisoning que entrega um instalador falso. Programa suspeito roda em VM ou Windows Sandbox, e Defender, TPM e Secure Boot ficam ligados. No caso Twitch, o autor suspeita que o vazamento saiu da máquina de um desenvolvedor (havia `node_modules`, builds e arquivos temporários). [cap-081, cap-082]
- **Ameaça quântica, sem exagero**: o algoritmo de Shor quebra RSA, DH e curvas elípticas se a máquina existir; o de Grover dá ganho pequeno contra AES e hashes (a mineração não corre risco). Estimativa de 2026: quebrar a secp256k1 exige menos de 500 mil qubits físicos (~1.200 lógicos) e ~9 min, enquanto o melhor hardware público tem ~12 lógicos. O que conta é a chave *pública* estar exposta (endereço reusado, Taproot, P2PK); senha forte não muda nada. O teorema de Mosca resume: o prazo real é a data da máquina menos o tempo que a sua migração leva. [cap-077, cap-078, blog 2026-08-16]
- **Prova de conhecimento zero (ZK)**: prova que você conhece um segredo (ou que um compromisso guarda um valor válido) sem revelá-lo. No protocolo de Schnorr, quem não sabe o segredo não responde a um desafio arbitrário, mas um simulador consegue fabricar uma transcrição indistinguível, e é por isso que a transcrição não carrega informação. A transformação Fiat-Shamir deriva o desafio por hash e torna a prova não interativa. A assinatura cega deixa uma autoridade assinar sem ver o conteúdo, o que separa a identidade do dado. [blog 2026-08-07]
- **Prova também tem implementação**: a prova de conhecimento zero que o Google publicou sobre o circuito quântico foi forjada pela Trail of Bits por causa de bugs no verificador em Rust. Confiar na prova passou a significar confiar no parser, no compilador e no simulador. O que sustentou a alegação foi uma reprodução aberta e independente. [blog 2026-08-16]

### Complementos (não estão nos textos do autor)

- **Injeção**: use consultas parametrizadas ou prepared statements. Nunca interpole entrada em SQL, shell ou caminho de arquivo, nem nos métodos de "SQL cru" do ORM.
- **XSS**: escape a saída conforme o contexto (HTML, atributo, JS, URL) e use templates com auto-escape. Não marque conteúdo do usuário como "seguro". CSP entra como segunda camada.
- **CSRF**: exija token por sessão em toda requisição que muda estado e use cookies `SameSite`; requisição GET nunca deve alterar estado.
- **Autorização não é autenticação**: verifique no servidor, a cada requisição, se *este* usuário pode acessar *este* objeto (IDOR) e negue por padrão.
- **Sessão**: troque o ID da sessão no login (evita fixação), use cookies `HttpOnly`, `Secure` e `SameSite` e invalide a sessão no logout e na troca de senha.
- **Tokens de reset, convite e API**: gere com CSPRNG (≥128 bits), com uso único e validade curta. Guarde só o hash e compare em tempo constante.
- **Aleatoriedade na aplicação**: use o CSPRNG da linguagem (`secrets`, `crypto.randomBytes`/`getRandomValues`, `SecureRandom`). `Math.random`/`rand()` nunca servem para segredo.
- **AEAD**: use AES-GCM ou ChaCha20-Poly1305 e nunca repita o nonce com a mesma chave (no GCM isso quebra sigilo e autenticidade). A tag *não* é secreta e viaja junto do texto cifrado. Ressalva ao cap-079: secreta é só a chave.
- **Extensão de comprimento**: também atinge SHA-256 e SHA-512 puros, que usam a mesma construção. O SHA-3 é imune, mas o caminho seguro continua sendo o HMAC (ou KMAC) de uma biblioteca. Ressalva ao cap-077.
- **bcrypt e "rainbow table"**: com salt único por usuário, tabela pré-computada não se aplica. O ataque real é dicionário com regras contra cada hash, e o custo do KDF é o que o freia. Ressalva ao cap-081.
- **Autenticação no TLS**: quem autentica é a assinatura do certificado (RSA ou ECDSA); o ECDHE cuida da troca efêmera e do forward secrecy. O cap-078 simplifica dizendo que a curva "substitui" o STS.
- **Pós-quântico**: o SIKE (isogenias supersingulares, citado no cap-078 de 2019) foi quebrado em 2022. Os padrões do NIST hoje são ML-KEM para troca de chaves e ML-DSA, SLH-DSA (SPHINCS+) e Falcon para assinatura.
- **Kerckhoffs**: a segurança não pode depender de esconder o algoritmo. Escolher uma cifra rara (argumento do VeraCrypt em cap-079/cap-081) não substitui chave e senha fortes.
- **JWT e afins**: fixe o algoritmo no servidor (rejeite `alg: none` e a troca RS→HS) e valide expiração e audiência. Nunca grave senha, token ou chave em log.

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Guardar senha de usuário | KDF lento (Argon2 ou scrypt; bcrypt/PBKDF2 aceitáveis) com salt único; nunca texto aberto, nunca "encriptada", nunca MD5/SHA puro | Senha encriptada vira ponto único de falha (vazou a chave, abrem-se 100% das senhas); hash rápido sai a bilhões por segundo em GPU/ASIC | cap-077, cap-078, cap-081, blog 2014-01-13 |
| Endurecer hashes antigos | Recalcular com custo maior no próximo login bem-sucedido | É o único momento em que você tem a senha original | cap-078 |
| Custo do KDF por ambiente | Alto em produção, baixo em dev e CI | Mantém a segurança sem travar os testes | cap-078 |
| Verificar integridade de arquivo | SHA-256/512 vindo de canal confiável ou, melhor, assinatura (GPG) com chave obtida antes | Checksum baixado junto do arquivo pode ter sido adulterado junto | cap-079 |
| Autenticar mensagem com segredo | HMAC de biblioteca; nunca `hash(segredo+msg)` | Extensão de comprimento forja MAC sem o segredo | cap-077 |
| Precisa de identificador único | UUID, não hash | Colisões de hash são estatisticamente esperadas | cap-077 |
| Cifrar dados | AES-256 via biblioteca, modo GCM, IV novo e aleatório a cada vez; nada de DES/3DES/ECB | ECB vaza padrões; DES já cai por força bruta; 3DES é lento | cap-077, cap-078, cap-079, blog 2014-01-13 |
| Escolher algoritmo, modo ou parâmetro | Usar o default de framework/biblioteca madura; não decidir a partir de tutorial; jamais inventar | Tutorial escolhe o mais fácil de explicar, que costuma ser o mais inseguro | cap-077, cap-079, blog 2014-01-13 |
| Projetar API de segurança | Não deixar combinações obviamente inseguras disponíveis | Crítica ao JWT: exigia decisões de algoritmo que quase ninguém sabe tomar | cap-079 |
| Passar chave simétrica a outra parte | Cifrar com a chave pública dela (ou ECDH); nunca por e-mail ou chat | Quem intercepta a chave lê e forja tudo dali em diante | cap-079 |
| Aceitar uma chave pública | Só por cadeia de confiança: CA, chave obtida antes por canal seguro ou `authorized_keys` posto por alguém confiável | Sem um terceiro confiável, não há como distinguir o legítimo do impostor | cap-078, cap-079 |
| Chave SSH | ed25519 com passphrase, permissão 600, agent com timeout; chave de hardware se possível; depois, `PasswordAuthentication no` | Bots fazem força bruta em login por senha | blog 2025-09-09, cap-078 |
| Onde guardar segredo da aplicação | Variável de ambiente; `.env` no `.gitignore` com modelo versionado; um segredo por ambiente | Segredo no repositório vaza junto com o código | blog 2013-10-19 |
| Chave privada ou certificado vazou | Gerar par novo e substituir imediatamente | Com a privada, qualquer um se passa por você | cap-079 |
| Gerar chave, seed ou token | CSPRNG do SO ou entropia de hardware; seed crítica combina fontes independentes; teste que prove a origem | 40 bits efetivos se enumeram com hardware barato | cap-079, blog 2026-08-01 |
| Parâmetro de URL vira chave de lookup | Nunca usar direto; whitelist e conversão estrita | A sessão tem chaves padrão, e coerção implícita abre atalhos | blog 2014-08-28 |
| Login, reset, "esqueci minha senha" | Biblioteca pronta do framework | Feito à mão, sempre acaba com buracos | blog 2014-08-28, cap-082 |
| Incluir lib de terceiro (sobretudo cripto) | Gerenciador de dependências e auditoria automática; nunca vendorizar | Cópia estática congela vulnerabilidades conhecidas | cap-082, blog 2014-05-22 |
| Tentativas repetidas de login | Rate limit/bloqueio (Rack::Attack, fail2ban) | Força bruta automatizada é o padrão | blog 2014-05-22 |
| Oferecer 2FA | TOTP por app ou chave de hardware; SMS não é recomendado; relógio via NTP | SIM swap e engenharia social na operadora | cap-081 |
| Notebook ou servidor com dados sensíveis | Disco e swap cifrados; TPM e Secure Boot ligados | Quem tem acesso físico ignora o login | cap-078, cap-079, cap-081 |
| Política de backup | Versionado (snapshots de semanas ou meses), cópia física local | O ransomware cifra aos poucos e contamina o backup recente | cap-081 |
| Ferramentas internas de admin | Privilégio mínimo, processo e treinamento anti-phishing | Basta uma pessoa enganada com acesso demais | cap-080 |
| Dados que precisam ser auditáveis | Log append-only com hashes encadeados/Merkle, replicado fora do controle de quem escreve | Detecta adulteração e prova inclusão com log2(n) hashes | cap-079, blog 2026-08-07 |
| Checar consistência de dados | Conferir com fonte independente do software que gerou os dados | Software adulterado produz registros adulterados e consistentes entre si | blog 2026-08-07 |
| Garantir que a verificação aconteça | Criar incentivo para quem verifica (ex.: sorteio para quem confere o recibo) | Prova que ninguém confere vira decoração | blog 2026-08-07 |
| Dados ou assinaturas de vida longa diante do quântico | Começar a migrar já, em camadas, com assinaturas híbridas; minimizar exposição e reuso de chave pública | Teorema de Mosca: migrar leva anos | blog 2026-08-16 |
| Relatar o estado de segurança | Dizer "sem vulnerabilidade conhecida", nunca "seguro" | Não ter achado não prova que não existe | blog 2014-05-22, cap-080 |
| Estimar ou comunicar um risco | Separar valor em risco de perda esperada e a probabilidade de uma etapa da probabilidade de roubo | Somar exposições numa manchete engana | blog 2026-08-16 |
| Rodar executável de origem duvidosa | VM ou Windows Sandbox, nunca na máquina principal | Pirataria e drivers falsos são vetor de malware e ransomware | cap-081 |
| Baixar ou instalar a partir de rede pública | VPN e verificação de assinatura antes de instalar | DNS envenenado entrega instalador falso | cap-081, cap-079 |
| Criptografia deixa a máquina lenta | Melhorar o hardware (SSD), não desligar a encriptação ou o antivírus | O ganho de performance não compensa o risco | cap-079, cap-081 |
| Fluxo que depende de identidade presencial | Fazer cada falsificação render uma unidade, ser cara e aparecer nas contagens | Nenhum algoritmo prova quem é a pessoa | blog 2026-08-07 |

## Armadilhas e mitos

- **"Encriptar a senha antes de gravar resolve."** → Quem roubar a chave abre todas as senhas. Senha se guarda como hash de KDF com salt. [cap-077]
- **"Hash com salt já basta."** → Salt mata tabela pré-computada, mas não resiste a GPU/ASIC calculando hash rápido na hora. Precisa de KDF caro. [cap-078]
- **"Trocar A por @ e I por ! deixa a senha forte."** → Essas regras já estão nos crackers; uma senha de 17 caracteres em leet caiu em menos de 1 minuto com 8 GPUs de 2018. Só senha aleatória e longa escapa do dicionário. [cap-081]
- **"Base64 ofusca."** → É conversão trivialmente reversível, sem chave. CRC também não tem nada a ver com segurança. [cap-078]
- **"SHA-256 é único, dá para usar como ID."** → Colisões são estatisticamente esperadas (paradoxo do aniversário). Use UUID. [cap-077]
- **"Concatenar um segredo antes do hash deixa tudo seguro."** → Extensão de comprimento forja o MAC. A força depende de como você usa o algoritmo, não do tamanho da chave. [cap-077]
- **"Está encriptado, então está seguro."** → TripleDES em ECB com chave derivada de MD5, copiado do CodeProject, roda em grandes sistemas. Entenda o que copia. [blog 2014-01-13]
- **"Minha senha de login protege o notebook."** → Tire o disco e leia em outra máquina; Mac em Target Disk Mode vira HD externo. Sem encriptação de disco, o login não protege nada. [cap-079, cap-081]
- **"Ninguém se interessa por mim."** → Sites vazam todo ano (o HIBP registrava 600+ sites e 11+ bilhões de contas em 2022) e robôs testam seu e-mail e sua senha em todo lugar. [cap-081]
- **"Ninguém achou vulnerabilidade, então é seguro."** → Só quer dizer que ninguém detectou ainda. Não existe 100% seguro, nem desligado e enterrado. [cap-080, blog 2014-05-22]
- **"Tem selo, auditoria e consultoria, então é seguro."** → Boa parte disso serve para delimitar responsabilidade jurídica (Sarbox) e para marketing, não para fechar buracos. [cap-080]
- **"Hacker é gênio russo."** → O nível de segurança do software comum é tão baixo que amadores bastam: o golpe no Twitter foi obra de adolescentes. [cap-080]
- **"Coloquei proxy, whitelist e processo, então estou protegido."** → O autor furou exatamente isso com túnel HTTP, modem próprio e BIOS sem senha. Processo sem entender o mecanismo é teatro. [cap-080]
- **"Está fora da internet (air gap), então é honesto."** → Isolamento não diz nada sobre o que o software faz: malware chega por fábrica, atualização ou pendrive (Stuxnet), e os dados precisam sair de algum jeito. [blog 2026-08-07]
- **"Passar por hash ou XOR aumenta a aleatoriedade."** → Com 40 bits de entrada existem no máximo 2^40 saídas. O atacante enumera as entradas, não os hashes. [blog 2026-08-01]
- **"Passou no teste estatístico, então é aleatório."** → Com estado inicial previsível, a sequência inteira é previsível. [blog 2026-08-01]
- **"Os números batem entre si, então estão certos."** → Consistência interna (BU × RDV) não prova correção se tudo sai do mesmo software. [blog 2026-08-07]
- **"Tem prova criptográfica, não preciso confiar em ninguém."** → O verificador da prova ZK do Google tinha bug e aceitou prova forjada. A confiança só mudou para o código do verificador. [blog 2026-08-16]
- **"Blockchain é anônima."** → É pública e pseudônima; análise de cadeia liga identidades ao histórico inteiro. [blog 2026-08-07]
- **"Curva elíptica (ed25519) resiste a computador quântico."** → Falso: Shor quebra ECC (cap-078, blog 2026-08-16). O post de 2025 afirma o contrário; trate como erro. [blog 2025-09-09]
- **"O quântico quebra tudo, inclusive AES e mineração."** → Grover dá ganho pequeno; AES e prova de trabalho seguem de pé. O alvo é a criptografia de chave pública. [cap-077, blog 2026-08-16]
- **"Senha mais forte protege do quântico."** → O quântico calcula a privada a partir da pública exposta; a força da senha não entra na conta. [blog 2026-08-16]
- **"Merkle tree e Git impedem adulteração."** → Só tornam a adulteração detectável e barata de checar. [cap-079]
- **"O antivírus não achou nada, então estou limpo."** → Vírus se escondem do antivírus; o mínimo é escanear bootando por fora do sistema. Software pirata é o vetor campeão. [cap-081]
- **"Vendorizar a lib dá estabilidade."** → Cria bomba-relógio de versões vulneráveis que ninguém audita. [cap-082]
- **"Biometria é a senha definitiva."** → É irrevogável (senha se troca, digital não), e um cadastro central vira ponto único de falha catastrófica. [blog 2026-08-07]
- **"Encriptação de disco deixa lento, melhor desligar."** → Em CPU moderna, com AES em hardware, quase não se sente a diferença. Para ganhar performance, compre um SSD melhor. [cap-079]

## Checklist para aplicar num sistema real

- [ ] Senhas guardadas com Argon2/scrypt (ou bcrypt/PBKDF2), salt único e custo de produção; existe plano de rehash no login.
- [ ] Nenhum fluxo devolve a senha ao usuário (recuperação manda link ou token, não a senha) e o campo aceita senhas longas, sem limite de 8–10 caracteres.
- [ ] Login, reset e sessão vêm do framework ou de biblioteca madura; nenhuma proteção padrão (CSRF, sanitização de SQL) foi desligada ou contornada "por performance".
- [ ] Nenhum parâmetro de entrada é usado direto como chave de sessão, cache ou lookup sensível; conversões são explícitas e estritas.
- [ ] Login e SSH têm rate limit ou bloqueio; o SSH aceita só chave pública.
- [ ] Existe 2FA por TOTP ou chave de hardware (não só SMS), e os servidores estão sincronizados por NTP.
- [ ] Nenhuma criptografia caseira: só biblioteca reconhecida; nada de DES/3DES/ECB/MD5/SHA-1 em função de segurança; modo autenticado (GCM) com IV/nonce novo a cada mensagem.
- [ ] MACs usam HMAC de biblioteca; nenhum `hash(segredo+msg)`; nenhum hash usado como ID único; Base64 e CRC não aparecem como "proteção".
- [ ] Toda chave, seed e token vem do CSPRNG do SO ou de entropia de hardware; um teste prova de onde a entropia vem; mudanças nesse caminho passam por revisão (no caso ColdCard foram zero reviews).
- [ ] Nenhum segredo no repositório nem no histórico do Git (`.env` ignorado, modelo versionado); segredos distintos por ambiente.
- [ ] Chaves privadas nunca trafegam, ficam com permissão 600 e têm passphrase (ou ficam em hardware); existe procedimento para revogar e substituir se vazarem.
- [ ] Toda chave pública ou artefato aceito tem caminho de confiança (CA, chave obtida antes, assinatura GPG), não só checksum baixado pelo mesmo canal.
- [ ] Todo tráfego passa por TLS, com suites ECDHE (forward secrecy) e AES-GCM.
- [ ] Dependências declaradas no gerenciador, nada vendorizado; auditoria automática (dependabot, bundler-audit, Brakeman) rodando no CI; avisos de segurança do framework acompanhados.
- [ ] Inventário de máquinas e sistemas e de quem acessa o quê; privilégio mínimo, sobretudo em ferramentas internas de admin.
- [ ] Discos e swap cifrados em máquinas com dados sensíveis (incluindo as de dev); TPM e Secure Boot ligados; mídia descartada é destruída.
- [ ] Backups versionados (semanas ou meses de snapshots), com restauração testada.
- [ ] Dados auditáveis ficam em log append-only encadeado por hash e replicado fora do controle de quem escreve, e alguém de fato verifica.
- [ ] A verificação de consistência usa fonte independente do software que gerou os dados.
- [ ] Há treinamento e teste anti-phishing; a regra é não clicar em link de mensagem e não plugar pendrive desconhecido.
- [ ] Máquinas de dev e de operação sem software pirata nem drivers de site não oficial; programas suspeitos só rodam em VM ou sandbox.
- [ ] Toda etapa que depende de identidade humana tem custo por falsificação e contagens públicas que mostram anomalias.
- [ ] Riscos são comunicados com perda esperada e prazo, não com a soma de tudo que está exposto.
- [ ] Existe plano de incidente para quando (não "se") vazar: rotação de chaves e certificados, comunicação e post-mortem que impeça a repetição.
- [ ] Chaves públicas de vida longa estão mapeadas, e há plano de migração pós-quântica (híbrido, sem reuso de chave exposta).
- [ ] A comunicação externa diz "nenhuma vulnerabilidade conhecida", nunca "sistema seguro".

## Frases-âncora

> "Toda tabela de senhas uma hora vai ser roubada, não é uma questão de “se”, é só uma questão de quando." — cap-077

> "O que determina a força de um algoritmo não é só o tamanho da chave, é como você usa o algoritmo." — cap-077

> "Se você não é um bom matemático teórico, nunca, jamais tente inventar seu próprio algoritmo. Sua melhor tentativa vai ser pior do que o Enigma." — cap-077

> "Lembrem-se desse conceito: chaves privadas nunca podem tocar a internet, jamais." — cap-079

> "O maior problema de segurança é a ilusão de segurança." — cap-080

> "Só porque você não conhece, não significa que as vulnerabilidades não existam, e as chances são que existam e você simplesmente a ignora." — blog 2014-05-22

> "Hash não cria entropia." — blog 2026-08-01

> "A segurança não mora na ausência de rede; mora na verificação pública." — blog 2026-08-07

## Onde aprofundar

| Fonte | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-077 | Akitando #67 – Entendendo Conceitos Básicos de Criptografia, Parte 1/2 (2019) | https://www.akitaonrails.com/2019/11/21/akitando-67-entendendo-conceitos-basicos-de-criptografia-parte-1-2/ | precisar entender cifra simétrica, AES, hash, colisão, extensão de comprimento e HMAC |
| cap-078 | Akitando #68 – Entendendo Conceitos Básicos de Criptografia, Parte 2/2 (2019) | https://www.akitaonrails.com/2019/11/26/akitando-68-entendendo-conceitos-basicos-de-criptografia-parte-2-2/ | for decidir como guardar senha (KDF, salt) ou quiser entender DH, RSA, ECC, forward secrecy e TLS/SSH |
| cap-079 | Akitando #147 – Criptografia na Prática: Certificados, BitTorrent, Git, Bitcoin (2023) | https://www.akitaonrails.com/2023/11/10/akitando-147-criptografia-na-pratica-certificados-bittorrent-git-bitcoin/ | quiser ver OpenSSL, certificados, CA, GPG e árvores de Merkle na prática |
| cap-080 | Akitando #88 – RANT: Selo de Segurança é Marketing, o Fator Humano (2020) | https://www.akitaonrails.com/2020/12/08/akitando-88-rant-selo-de-seguranca-e-marketing-entendendo-o-fator-humano/ | discutir engenharia social, governança, compliance e gestão de risco |
| cap-081 | Akitando #131 – Sua Segurança é uma DROGA: Gerenciadores de Senhas, 2FA, Encriptação (2022) | https://www.akitaonrails.com/2022/11/08/akitando-131-sua-seguranca-e-uma-droga-gerenciadores-de-senhas-2fa-encriptacao/ | projetar 2FA/TOTP ou orientar sobre senhas, encriptação de disco e ransomware |
| cap-082 | Examinando o Vazamento do Twitch (2021) | https://www.akitaonrails.com/2021/10/21/examinando-o-vazamento-do-twitch/ | avaliar higiene de dependências (vendorização) e a forma real de um código grande vazado |
| blog 2013-10-19 (fora do compêndio) | [Iniciante] Configurações de ambiente com Dotenv | https://www.akitaonrails.com/2013/10/19/iniciante-configuracoes-de-ambiente-com-dotenv/ | for tirar segredos do código e separar por ambiente (Rails de 2013, datado) |
| blog 2014-01-13 (fora do compêndio) | [Criptografia] Não use TripleDES/ECB | https://www.akitaonrails.com/2014/01/13/criptografia-nao-use-tripledes-ecb-e-uma-curiosidade-sobre-cipher-key-do-net/ | encontrar cripto copiada de tutorial ou integrar cifra entre linguagens |
| blog 2014-05-22 (fora do compêndio) | Segurança em Rails | https://www.akitaonrails.com/2014/05/22/seguranca-em-rails/ | montar auditoria de dependências e rate limit (ferramentas de 2014, datadas) |
| blog 2014-08-28 (fora do compêndio) | [Small Bite] Session Injection Challenge | https://www.akitaonrails.com/2014/08/28/small-bite-session-injection-challenge/ | revisar fluxo de reset de senha ou uso de parâmetro como chave de sessão |
| blog 2025-09-09 (fora do compêndio) | Omarchy 2.0 – Entendendo SSH e Yubikeys | https://www.akitaonrails.com/2025/09/09/omarchy-2-0-entendendo-ssh-e-yubikeys/ | configurar chaves SSH, ssh-agent, YubiKey/FIDO2 e endurecer o `sshd` |
| blog 2026-08-01 (fora do compêndio) | Explorando o problema escandaloso do RNG da Coinkite | https://www.akitaonrails.com/2026/08/01/explorando-o-problema-escandaloso-do-rng-da-coinkite/ | revisar geração de chaves e seeds, ou precisar de um caso real de entropia fraca |
| blog 2026-08-07 (fora do compêndio) | Por que uma eleição digital perfeita ainda não seria viável? | https://www.akitaonrails.com/2026/08/07/por-que-uma-eleicao-digital-perfeita-ainda-nao-seria-viavel/ | projetar sistema auditável (compromissos, Merkle, ZK, assinatura cega) e discutir os limites da confiança |
| blog 2026-08-16 (fora do compêndio) | O que significa o Paper do Google sobre Vulnerabilidade Quântica | https://www.akitaonrails.com/2026/08/16/o-que-significa-o-paper-do-google-sobre-vulnerabilidade-quantica/ | planejar migração pós-quântica ou calibrar o risco quântico sem pânico |
