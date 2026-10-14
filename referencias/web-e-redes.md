# Web, back-end e redes
> Destilado de: cap-045, cap-046, cap-047, cap-048, cap-049, cap-050, cap-051, cap-052, cap-053 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).

## Ideia central
Para o autor, quase tudo na Web e em redes é a mesma ideia encadeada: um processo escuta numa porta, outro processo liga nela, e trocam blocos de bits (pacotes) segundo um formato combinado (protocolo) por camadas que não precisam saber umas das outras. As "sopas de letrinhas" (HTML, CSS, CGI, PHP, AJAX, SPA, TCP/IP, DNS, NAT, SSH, VPN) não são arbitrárias: cada uma nasceu pra resolver um problema concreto de um momento histórico, e você só acerta o fundamento quando sabe qual problema era. Um agente que projeta, depura ou explica um sistema web deve raciocinar nessas camadas (física → enlace → rede → transporte → aplicação) e tratar toda entrada externa como hostil. A regra de ouro de rede do autor: nunca confie em ninguém, e meça o gargalo antes de trocar a stack.

## Conceitos que todo sistema depende

- **Cliente/servidor é só um processo escutando numa porta** — Um servidor pede `bind` de uma porta ao SO, entra em `listen`, e fica num loop esperando `accept`; um cliente só liga no IP+porta do servidor. Navegador é cliente TCP, servidor web é servidor TCP. [cap-045][cap-051]
- **Porta é um número arbitrário numa tabela do SO** — Vai de 0 a 65535; 0–1023 são de sistema (exigem root), 1024–49151 usáveis sem privilégio (3000/4000/8080), 49152+ são efêmeras/dinâmicas. Convenções: 80 HTTP, 443 HTTPS, 22 SSH, 21 FTP, 25 SMTP, 53 DNS. Nada de mágico nos números. [cap-045][cap-051]
- **Socket = IP origem + porta origem + IP destino + porta destino** — É a generalização das primitivas de arquivo (open/read/write) pra comunicação entre processos, mesmo em máquinas diferentes; nasceu como Berkeley/BSD Sockets no UNIX BSD 4.2 em 1983. [cap-051]
- **URL / servidor mais simples** — protocolo + domínio + caminho do arquivo. O servidor web mais simples lê um arquivo e cospe; se traduz a URL direto para caminho no disco sem checagem, vira buraco de segurança (dá pra vasculhar arquivos não públicos). [cap-045]
- **Site dinâmico nasce de rodar executável no servidor (CGI)** — Em vez de ler um arquivo, o servidor roda um programa e devolve o output HTML. Começou com binários C (Common Gateway Interface), depois Perl (e REGEX); depois PHP/ASP embutidos no próprio servidor. [cap-045]
- **Servidor web ≠ servidor de aplicação** — Servir HTML estático (Apache, NGINX) é diferente de executar programas que cospem HTML (application servers). Todo application server é web server, mas não o contrário. [cap-045]
- **HTML/CSS/árvores** — HTML descende de SGML (marcar estrutura, não estilo); CSS separou estilo do conteúdo. "Quase tudo na Web é baseado em árvore": HTML é árvore de elementos, CSS árvore de regras cascateadas, sites árvore de páginas linkadas. [cap-045]
- **AJAX → SPA → frameworks de componentes** — Tirar lógica/estado do servidor e empurrar pra perto da tela gerou Ember/Angular/React/Vue; o React otimiza redesenho via Virtual DOM (muda-se o estado, não o elemento). SPA = app inteira numa página, trocando estado via AJAX. [cap-045]
- **Asset pipeline e minificação** — Muitos assets deixam o navegador lento; junta-se arquivos e remove-se espaços/nomes longos (minificar), transpila SASS→CSS e ES6/CoffeeScript→JS (Webpack/Grunt/Gulp/Babel). Transpiler traduz linguagem→linguagem, não para binário nativo. [cap-045]
- **Compilado vs interpretado** — C vira binário nativo (instruções do SO+CPU); interpretador lê o fonte e executa a cada carga via parser→AST. Máquina virtual (JVM) é a evolução do interpretador que quer "ser" o SO; usa bytecode portável + JIT (compila em tempo de execução só o código quente). [cap-046]
- **Linkagem estática vs dinâmica** — Estática funde bibliotecas no binário (gigante, mas copia e roda — jeito do Go); dinâmica (.so/.dll) reusa o binário instalado (menor, mas precisa da versão certa presente — "DLL Hell"). Vantagem da dinâmica: corrigir bug crítico numa lib só recompila a lib. [cap-046][cap-047]
- **Processo, fork e thread** — Cada processo tem memória virtual isolada (começa na "página 1" virtual, o SO traduz pra real), então não pisa na memória de outro. Servir vários clientes: fork (processo isolado, barato em UNIX via Copy-on-Write, caro no Windows) ou threads (baratas, mas compartilham memória → exigem locks e thread-safety). [cap-046]
- **Dependências são inevitáveis e perseguem o projeto** — "Você nunca vai fazer software 100% do zero". Cada ecossistema tem gerenciador (Maven/Gradle, NPM/Yarn, Bundler, PIP, Cargo, Hex, Composer, CPAN). Atualizar versões quebra código; ficar velho expõe a falhas de segurança. [cap-047]
- **Evolução das camadas de app: 1 → 2 → 3 tiers** — Arquivo local → arquivo compartilhado na rede (corrompia, exigia lock do arquivo inteiro) → servidor de banco mediador → servidor de aplicação no meio (3 camadas). RPC remoto: COM/DCOM, CORBA, J2EE/servlets. [cap-047]
- **Sinais, modulação e banda** — Bits viram ondas (modular/demodular = "modem"); amplitude ≈ bit 1/0, frequência ≈ taxa. Banda (bandwidth) é a faixa de frequências que o meio aguenta; distância degrada sinal. Multiplexação combina vários canais num mesmo meio físico. [cap-048]
- **Circuit switching → packet switching** — Circuito fecha uma sessão exclusiva stateful (desperdiça banda ociosa, tarifada por minuto); pacotes fragmentam a informação em blocos de tamanho fixo (datagramas) com metadados (destino, número de ordem), stateless, compartilhando o meio. É o que permite escalar. [cap-048]
- **ACK/NAK e remontagem** — Cada pacote recebido é confirmado (ACK) ou rejeitado (NAK → reenvia). Pacotes chegam fora de ordem e são remontados pelo número; velocidade de download caindo geralmente é repetição de pacote, não correção. [cap-048][cap-049]
- **Detecção vs correção de erros** — Corrupção de bit (bit flip/bit rot, até por raios cósmicos) é inevitável. Hamming Code detecta E corrige 1 bit via paridade (procura binária: bloco maior precisa de proporção menor de bits de redundância); Reed-Solomon em CD/Blu-ray/HD; checksum (Internet 16-bit, CRC, Fletcher) só detecta. Em rede prefere-se checar e repetir; em armazenamento (sem a quem pedir cópia) prefere-se corrigir. [cap-049]
- **RAID e integridade de armazenamento** — RAID-0 = performance, zero segurança (um HD morre, perde tudo); RAID-1 = espelho, sobrevive a 1 falha. ECC na RAM usa Hamming; ZFS/Btrfs/RAID fazem data scrubbing periódico (checksum por arquivo, reconstrói pela cópia redundante). [cap-049][cap-053]
- **Modelo OSI/TCP-IP em camadas** — 7 camadas; cada uma pode ser trocada sem mexer nas de cima. Física (bits), enlace/data link (checagem de erro, MAC/Ethernet), rede (IP, roteamento), transporte (TCP confiável / UDP leve e sem conexão), e aplicação (HTTP, DNS, SMTP...). Em TCP/IP as camadas altas se acumulam na de aplicação. [cap-050][cap-051]
- **Endereço IP + máscara de sub-rede** — IPv4 é 32 bits em 4 blocos. A máscara (ex. 255.255.255.0 = /24) separa, via AND bitwise, prefixo de rede e host. O prefixo deixa o roteador guardar uma linha por rede, não por host (como DDD no telefone). [cap-050]
- **ARP, MAC, DHCP, default gateway** — Dentro da LAN entrega-se por MAC (48 bits, de fábrica), não por IP: ARP "grita no corredor" perguntando quem tem tal IP e cacheia o MAC. DHCP (o "síndico") atribui IPs sem colisão. Endereços fora da rede vão pro default gateway (o "porteiro", IP mais baixo). [cap-050]
- **NAT/CGNAT** — Faltam IPs IPv4 (4 bilhões < dispositivos). Redes privadas usam 10/8, 172.16/12, 192.168/16; o provedor traduz (CGNAT, estilo PBX com ramais) muitos clientes num IP público. Efeito colateral: de fora ninguém te acha diretamente — você precisa iniciar a conexão (complica VoIP/P2P → STUN/TURN/ICE). [cap-050]
- **DNS** — Traduz nome (texto) → IP; a rede só entende número. Resposta traz vários IPs (round-robin + proximidade geográfica) e um TTL pra cache. DNS normal vai em texto aberto (provedor vê seus domínios) → DNS over HTTPS/TLS (1.1.1.1, 8.8.8.8), ou servidor caseiro (Pi-Hole). [cap-050][cap-051][cap-053]
- **loopback vs qualquer interface** — 127.0.0.1 = localhost (só acessível de dentro da máquina); 0.0.0.0 = "qualquer placa de rede" (acessível de fora). Dev usa localhost; produção dá bind em 0.0.0.0. [cap-051]
- **HTTP é texto; request/response** — Comandos texto (GET/POST/PUT) + status (200) + headers (Content-Type/MIME). Dá pra simular com `telnet`/`curl`/`wget` ou ver no DevTools (aba Network). TLS/HTTPS é camada à parte: a app recebe já descriptografado. [cap-051]
- **Protocolos de aplicação acumulados no HTTP(S)** — FTP, Telnet, NNTP (Usenet), IRC, Gopher foram absorvidos pelo HTTPS; isso centralizou tudo em "walled gardens" e tirou o caráter distribuído (sobraram BitTorrent/Bitcoin). SSH substituiu Telnet e FTP (SFTP). [cap-051]
- **Injeção de comando e sanitização** — Interpolar entrada do usuário direto num comando de shell (`;` encadeia comandos) dá RCE — "ownei a máquina, game over". Sanitizar = remover/escapar caracteres especiais. Trate entrada externa como "material radioativo"; mas não sanitize funções internas à toa. [cap-052]
- **Firewall** — Programa com privilégio de root que intercepta pacotes e aplica regras allow/deny por porta/direção. Sem firewall = allow *; fechado demais = servidor inútil. Também bloqueia máquina interna acessar porta de outra interna (contra malware lateral). [cap-052]
- **Proxy** — Intermediário de tráfego Web (o site só vê o IP do proxy, como no NAT). Empresas usam com whitelist/blacklist e logging; HTTPS dificulta ver o conteúdo, mas o destino ainda vaza. [cap-052]
- **Túnel SSH (poking a hole)** — Firewall que bloqueia entrada mas libera saída (porta 80) pode ser furado: a conexão nasce de dentro e, uma vez aberta, flui nos dois sentidos. Remote Forwarding (`ssh -R`) expõe serviço interno; Dynamic/Local Forwarding (`ssh -D`, SOCKS5) cria proxy pessoal pra sair por fora. [cap-052]
- **VPN, TUN/TAP e tabela de rotas** — Um driver pode "mentir" criando placa de rede virtual: TUN (camada 3) / TAP (camada 2). A VPN cria o dispositivo virtual, fura firewall/CGNAT abrindo túnel criptografado e reescreve a tabela de rotas (a "métrica"/prioridade decide a saída). Modos: client-to-site (acesso remoto) e site-to-site. ZeroTier/Tailscale (Wireguard)/OpenVPN/NordVPN. PPTP é obsoleto/inseguro. [cap-053]
- **Rede pública é hostil** — Em hub (ou Wi-Fi aberto) qualquer um em modo promíscuo lê os pacotes; use switch em casa e VPN em rede pública; nunca digite senha de banco sem criptografia. [cap-050][cap-053]
- **Assembly, syscalls e abstração** — A CPU vem de fábrica com um conjunto de instruções (Assembly, dependente da arquitetura: x86≠ARM≠PowerPC); o SO acrescenta syscalls. Abstração é encapsular uma sequência de instruções difíceis num nome descritivo (função). C existe pra não reescrever Assembly por arquitetura. [cap-046]
- **CGI → mod_perl/mod_php → FastCGI; ISAPI/NSAPI** — Reiniciar o interpretador a cada request é caro; mod_perl/mod_php embutem o interpretador no Apache (rápido, mas bug no script dá acesso ao Apache inteiro); FastCGI mantém processo separado e isolado comunicando via TCP/Unix socket. No Windows (sem fork barato) o IIS usa ISAPI (DLL thread-safe), a Netscape usava NSAPI. [cap-046]
- **Hub vs switch; modo promíscuo; MAC spoofing** — Hub manda o pacote pra todos (barato, inseguro); switch liga direto cada dispositivo (os outros não veem o tráfego). Placa em modo promíscuo lê tudo (Wireshark); MAC pode ser forjado, e como ele não muda com o DHCP, o provedor rastreia (por isso iPhone randomiza MAC). [cap-050]
- **CIDR e classful** — Máscaras definem sub-redes dentro de uma organização (Classless InterDomain Routing); antes de 1993 usava-se classes A–E (classful). É o que decide como roteadores encaminham quais pacotes pra quais redes. [cap-050]
- **Stream e buffer** — Abrir arquivo/socket dá um stream (cano de bits bidirecional); lê-se bit a bit ou enche-se um buffer (balde) e processa-se por linha. "Ver" os arquivos na rede ≠ "ter" os arquivos; puxa-se pedaço a pedaço. Linguagens de alto nível escondem o buffer; C/Rust/Java expõem mais. [cap-047][cap-051]
- **IPC: pipes e arquivos** — Memória é isolada entre processos; pra comunicar, usa-se pipe (stdout→stdin, serial, um termina e o outro começa) ou arquivo compartilhado (paralelo, um escreve e outro lê, ex. log + Prometheus). Socket é a generalização disso pra rede. [cap-051]
- **Short-lived vs long-lived; Big Loop** — Programas de tiro curto (grep/sed/awk) recebem input, processam, cospem e saem; servidores/GUI/games são loops infinitos (`while(true)`) que esperam evento, reagem e voltam a esperar, até um sinal (SIGTERM/kill). É a base da programação orientada a eventos. [cap-051]
- **Daemon** — Programa que o SO/gerenciador (systemd/OpenRC/runit) sobe após o boot e vigia (reinicia se cair). `sshd` é o ssh daemon; portas 80/443 de app oficial ficam abertas, o resto fechado. [cap-052]
- **Driver pode mentir (placa virtual)** — Pro SO basta existir um driver dizendo que o hardware existe; é o que VM (VirtualBox/VMware) e VPN fazem — emulam placa de rede/vídeo inexistente. TUN/TAP são placas de rede virtuais que a VPN usa pra criar o túnel. [cap-053]
- **Container: rede host vs bridge; port mapping** — Em host, o container usa a rede do PC (dois serviços na porta 80 colidem); em bridge, cada container ganha rede privada 172.x isolada (ambos podem ouvir a "porta 80" interna), e mapeia-se pra portas externas livres (8080/8081). "Containers estão na Matrix e não sabem disso." [cap-053]
- **Media types/queries e MIME** — Layout era pensado pra papel; adaptar pra telas diferentes (impressão, braille, som, celular) gerou `@media`/media queries. MIME (text/html, image/png) veio de e-mail e virou base pra todo navegador de arquivos saber o tipo. [cap-045]
- **Por que Flash/applets/ActiveX/Silverlight morreram** — Eram sandboxes/plugins pra rodar código rico no navegador; smartphone (pouca CPU/bateria, só touch) matou Flash (carta de Steve Jobs), e a padronização de HTML5/vídeo acelerado/canvas/WebGL tornou os plugins obsoletos. [cap-045]
- **Padronização tardia HTML5/CSS3/ES6** — 2008–2015 foi transição caótica (cada navegador implementava partes); JQuery virou necessidade, e transpilers (SASS, CoffeeScript, Babel) blindavam a bagunça até ES6/HTML5/CSS3 oficializarem (~2014–2015). [cap-045]
- **Tipagem estática vs dinâmica e inferência** — Tipo é "molde de biscoito": o compilador precisa saber quanto de memória alocar. Estática declara no código (tedioso, erra menos); dinâmica o interpretador decide em runtime (conveniente, erra na execução); inferência (Hindley-Milner, Scala→Rust/Swift/Kotlin/TS) é o meio-termo. [cap-047]
- **Incompatibilidade binária (Scala) vs recompilar tudo (Go)** — Bytecode de Scala quebra entre versões (LinkageError), forçando recompilar as libs; Go/Rust/Elixir preferem baixar o código-fonte das dependências e compilar tudo junto, com compilador veloz, evitando binários pré-compilados. [cap-047]
- **ADSL (CAP/DMT), PPP, handshake** — A voz usa só ~0–4 kHz de um cabo de ~1,5 MHz; ADSL (Speedy) aproveitou o resto pra dados no mesmo fio. CAP reservava faixas; DMT divide em 247 canais de 4 kHz e troca de canal ruidoso. Discada usava PPP/SLIP com handshake (o "barulhinho"). [cap-048]
- **Usenet/NNTP, uuencode, par2, M-Disc** — Usenet (NNTP) era texto de 8 bits; pra passar binário convertia-se em Base64 (uuencode), fatiava em .rar e gerava .par2 (bits de redundância Reed-Solomon) pra recuperar corrupção. sha1/md5 confirmam integridade de download (1 bit muda o hash todo). [cap-049]
- **Interface (contrato) separa consumo de implementação** — Quem consome uma lib/componente/serviço remoto só precisa do arquivo que declara funções, argumentos e tipos; a implementação por trás pode mudar. É o mesmo princípio de um botão "OK" ser igual em Windows/macOS/GNOME com código diferente, e de COM exigir `IUnknown`. [cap-047]
- **Componentização: OLE/OCX/COM** — Windows evoluiu de DLL (ligada em tempo de compilação) pra OLE/OCX/COM, que permite embutir componente em app já existente sem recompilar (ex. tabela Excel dentro do Word). Serviu pra comercializar componentes num formato padrão. [cap-047]
- **Stored procedures e XML enterprise** — Mover lógica pro banco (PL/SQL, T-SQL) reduz o tamanho/atualização dos clientes, mas sobrecarrega o banco. A onda enterprise do fim dos anos 90 (J2EE, EJB, SOAP, UML, config) grudou tudo em XML — e o mercado aprendeu a odiar XML. [cap-047]
- **Lambdas, closures e DSLs** — Funções anônimas que recebem/devolvem funções (de Lisp/Lambda Calculus) tornaram o estilo imperativo mais maleável e abriram metaprogramação e DSLs (mini-linguagens sobre a linguagem). Hoje até Java/C# têm lambdas. [cap-047]
- **Guerra dos navegadores e o medo de Bill Gates** — IE×Netscape disputaram a Web implementando CSS/interatividade; a Microsoft embutiu o IE no Windows (antitruste). O GMAIL (2004) mostrou o que a MS temia: a Web substituindo o SO na função de rodar programas (substituiu o Outlook). [cap-045]
- **Gerenciadores de pacotes nasceram pra parar de baixar zip na mão** — Da era de clicar/descomprimir manualmente a CPAN (Perl), Bundler (Ruby), NPM/Yarn (JS), Maven/Gradle (Java), NuGet (.NET) — baixar e resolver versões automaticamente. [cap-045][cap-047]
- **Apps híbridos** — Com ES6/Chrome/multi-core, nasceram apps desktop/mobile com casca nativa controlada por fina camada de HTML5/CSS3/JS: Electron (encapsula o Chrome), React Native (iOS/Android como site React). Spotify, Slack, VS Code. [cap-045]
- **Design responsivo e UX nasceram do smartphone** — Telas verticais de resolução variável + touch (sem teclado/mouse) exigiram layout que se adapta (tableless, espaçamento relativo, media queries) e uma nova disciplina de usabilidade (UX), discutida muito entre 2009–2015. [cap-045]
- **Bootstrap e design systems** — Pra parar de redesenhar botões do zero, surgiram frameworks CSS (Bootstrap) escritos em SASS/LESS; por isso muitos sites parecem do mesmo designer (hambúrguer, jumbotron, scrollspy). [cap-045]
- **IANA e portas conhecidas** — A IANA registra o uso das portas baixas (0–1023); por isso o navegador preenche 80/443 sozinho e você omite a porta na URL. São convenção, não obrigação técnica. [cap-051]
- **Centralização em HTTP(S) e walled gardens** — Antes se rodava o próprio servidor IRC/NNTP e plugava na rede distribuída; acumular tudo em HTTPS trouxe segurança pro comércio (TLS, cartão de crédito) mas criou jardins emparedados — restaram poucos serviços realmente distribuídos (BitTorrent, Bitcoin). [cap-051]
- **Proxy com whitelist/blacklist e túnel pessoal** — Empresa força proxy pra filtrar e logar navegação; dá pra contornar com túnel SSH servindo como proxy SOCKS5 pessoal (`ssh -D`), que sai por uma porta alta provavelmente aberta. Precede SSH havia o HTTP Tunnel. [cap-052]
- **Tempo compartilhado originou multiusuário e terminais** — Computador caro dos anos 60/70 (US$150k) exigia dividir tempo/memória entre usuários (time sharing, origem do UNIX); daí os terminais burros e a entrada de tecnologias de telecom no computador. [cap-048]
- **Latência é o custo da VPN** — Passar por máquinas intermediárias em túnel criptografado adiciona rota e processamento; conexão via VPN tem mais latência e menos velocidade. Irrelevante pra navegar, importa pra games; servidores próximos amenizam. [cap-053]
- **Cabos e Wi-Fi têm categorias/frequências** — Par trançado cancela interferência eletromagnética (fios de polaridade oposta); cat5e dá 1 Gbit, cat6/6a chegam a 10 Gbit. Wi-Fi 2.4 GHz vai mais longe/atravessa parede; 5 GHz é mais rápido mas cai com obstáculo. Quanto maior a frequência, mais banda e menos alcance (problema do 5G mmWave). [cap-048]
- **Cache e TTL de DNS** — A resposta do DNS traz TTL (expiração); o SO cacheia localmente e só repergunta quando expira. Pi-Hole é um DNS caseiro que cacheia e bloqueia domínios de propaganda/malware, e com DoH esconde o padrão de navegação até do provedor. [cap-051][cap-053]
- **Client-to-site vs site-to-site** — Túnel SSH de acesso remoto é client-to-site (um cliente alcança um serviço interno); já dois clientes atrás de redes privadas conversando como se fossem a mesma rede (ZeroTier) é site-to-site/gateway-to-gateway. [cap-053]
- **NAS é um PC Linux dedicado a blocos; tradeoffs de RAID** — Synology roda Linux Raid (mdadm) + ext4/Btrfs; SHR aproveita HDs de tamanhos diferentes e usa hot spare (religa e reconstrói na falha). Performance de um volume soma ~metade da velocidade de cada HD. Não rode outros apps num NAS de trabalho (risco de corromper dado). [cap-049][cap-053]
- **Erlang/BEAM, atores e supervisores; imagens Smalltalk** — Erlang (telecom) tem VM (BEAM), modelo de atores (objeto com estado trocando mensagens) e supervisores que gerenciam o ciclo de vida — feito pra ficar de pé eternamente e trocar código sem reiniciar; Elixir moderniza a sintaxe sobre o BEAM. Smalltalk guarda objetos vivos em "imagens" (banco de objetos) que sobrevivem ao desligamento da VM. [cap-047]

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Entendendo/escolhendo uma tecnologia web desconhecida | Pesquisar a história: que problema ela resolveu, se ainda resolve melhor, se já há substituto | Cada "letrinha" existe por um contexto; sem isso você engole sem entender | cap-045 |
| Escrevendo código que será mantido por humanos | Priorizar legibilidade; deixar a otimização pro compilador | "É trabalho do compilador melhorar pra ficar mais eficiente"; se a linguagem exige otimizar sempre, ela é defeituosa | cap-046 |
| App vai de uma máquina pra outra sem dependências dinâmicas | Compilar estático (estilo Go) | Binário gigante, mas copia e roda sem instalar libs | cap-046,cap-047 |
| Bug crítico numa lib usada por 10 programas | Preferir linkagem dinâmica | Recompila só a lib e os 10 pegam a correção; estático exige recompilar tudo | cap-046 |
| Rodar interpretador (PHP/Python/Ruby) em produção | Preferir Linux/Mac a Windows | Interpretadores nasceram no UNIX e só rodam bem lá; bindings nativos divergem no Windows | cap-046 |
| JVM/Erlang em produção | Subir 1 processo, deixar "parasitar" a máquina toda | São quase-SO com GC caro: ruins pra fork/boot rápido, ótimos de pé eternamente | cap-046,cap-047 |
| Atualizar dependências | Nunca subir tudo pra última versão de uma vez; subir controlado e testar | Atualização quebra código; mas versão velha acumula falha de segurança | cap-047 |
| Perdeu um pacote de áudio/vídeo em streaming | Usar UDP | Perder pacote ocasional não é problema; TCP (confiável) adiciona peso desnecessário | cap-050 |
| Internet caindo num download longo | Entender que é repetição de pacote, não corrupção | Em rede, pedir cópia é mais barato que corrigir | cap-049 |
| Dados importantes no disco com 2 HDs | RAID-1 (espelho); nunca RAID-0 pra dado que importa | RAID-0 perde tudo se um HD morre; RAID-1 sobrevive | cap-049,cap-053 |
| Dado só seu, sem a quem pedir cópia | Correção de erro (Reed-Solomon/ECC/scrubbing), não só checksum | Não há servidor pra reenviar; tem que recuperar localmente | cap-049 |
| Porta pra subir app em dev | Usar 1024–49151 (3000/4000/8080) | Abaixo de 1024 exige root/sudo; 49152+ são efêmeras de VoIP/games | cap-051 |
| Servidor em dev vs produção | Dev em localhost/127.0.0.1; produção bind em 0.0.0.0 | Loopback bloqueia acesso externo durante o desenvolvimento | cap-051 |
| Recebe qualquer entrada de usuário (URL/form) | Sanitizar sempre (remover/escapar especiais); mas só na fronteira | Entrada externa é "material radioativo"; sanitizar função interna é defensividade inútil | cap-052 |
| Privacidade de DNS | Trocar DNS do provedor por 1.1.1.1/8.8.8.8 ou Pi-Hole com DoH | DNS do provedor vai em texto aberto e registra sua navegação | cap-050,cap-053 |
| Acessar banco em Wi-Fi público (Starbucks/aeroporto) | Usar VPN | Qualquer um na rede aberta pode estar escutando os pacotes | cap-053 |
| Expor serviço caseiro (Plex/NAS) fora de casa sem abrir a rede | VPN site-to-site (ZeroTier/Tailscale) | Fura CGNAT com túnel criptografado sem expor a rede doméstica | cap-053 |
| Montar NAS confiável sem virar sysadmin | Solução madura (Synology) sobre componentes open source conhecidos | O teste de fogo é recuperar dados na pane, não funcionar da 1ª vez; valor ≠ preço inicial | cap-053 |
| App web lenta / quer escalar | Medir o gargalo e aliviar recurso, não trocar de stack | (reforço do tema, ver cap-061) | cap-045 |
| Dois serviços querem a mesma porta num mesmo host | Rodar em container/VM com rede bridge e mapear portas | Cada um ganha rede privada isolada e ouve a "porta 80" interna sem colidir | cap-053 |
| Interpretador bloqueado em threads | Suspeitar de binding a lib nativa não thread-safe | Binding não thread-safe trava o interpretador inteiro | cap-046 |
| Reiniciar interpretador a cada request custa caro | FastCGI (processo persistente isolado), não mod_php embutido | Mantém interpretador vivo e isola falha do servidor web | cap-046 |
| Casa com vários dispositivos cabeados compartilhando internet | Switch, nunca hub | Hub manda pacote pra todos (modo promíscuo lê tudo); switch isola | cap-050 |
| Escolher protocolo de VPN hoje | Wireguard (ou serviços sobre ele); evitar PPTP | PPTP é obsoleto/inseguro; Wireguard é leve e já vem na kernel | cap-053 |
| Transferir arquivo pra servidor de forma segura | SFTP (sobre SSH), não FTP | FTP (como Telnet) precede segurança moderna | cap-051 |
| Escolher cabo de rede na dúvida | Comprar cat6 (cat5e já serve 1 Gbit) | Aguenta até 10 Gbit; cat7/8 a maioria não precisa | cap-048 |
| Wi-Fi fraco longe do roteador | Entender 2.4 GHz (alcance) vs 5 GHz (velocidade) | Frequência maior = mais banda, menos alcance e pior com parede | cap-048 |
| Rodar outros apps num NAS de trabalho | Não rodar; dedicá-lo a arquivos | App com bug pode corromper dados; CPU deve servir blocos | cap-053 |
| Privacidade de navegação em casa | Pi-Hole como DNS local com DoH, cacheando e bloqueando domínios ruins | Esconde padrão de navegação e corta propaganda/malware na origem | cap-053 |

## Armadilhas e mitos

- "Plano de 300 mega é 300 megabytes/s" → é 300 megabits; divide por 8 → ~37 MB/s. Upload costuma ser metade. [cap-048]
- "Múltiplos de 1000 (kilo/mega)" → o certo em computação é potência de 2 (1024); os múltiplos de 1000 são marketing. [cap-048]
- "A menor unidade é o arquivo" → HD é dispositivo de blocos, não de arquivos; rede trafega pacotes/blocos. É por isso que RAID-0 dobra a velocidade e o streaming funciona pedaço a pedaço. [cap-049][cap-051]
- "Base64 é criptografia" → não; é só codificar binário em texto de ~7 bits (fica 133% maior). [cap-049]
- "A porta 8080/3000/80 tem significado especial" → portas são só números arbitrários numa tabela do SO; qualquer uma com algo escutando serve. [cap-045][cap-051]
- "127.0.0.1 e 0.0.0.0 são a mesma coisa" → só 127.0.0.1 == localhost (interno); 0.0.0.0 é "qualquer placa", acessível de fora. [cap-051]
- "Dois programas que compilam e rodam nos dois SOs rodam igual" → não; fork é barato no Linux e caro no Windows; coisas feitas pra Windows rodam melhor no Windows e vice-versa. [cap-046]
- "O IP que minha app recebe é o computador do usuário" → quase sempre é o IP do CGNAT do provedor ou de um proxy/VPN; o usuário está escondido atrás de NAT. [cap-050][cap-052]
- "IPv6 não pegou por falha técnica" → o inimigo do IPv6 foi o sucesso do IPv4: NAT/CGNAT adiaram o colapso e migrar virou "otimização prematura"; o crash de 2001 tirou o investimento. [cap-050]
- "O mínimo de código que faz a ideia funcionar está pronto pra produção" → é exatamente o que o amador faz; sem sanitizar entrada vira injeção de comando e controle total da máquina. Código de junior não vai pra produção sem review de sênior. [cap-052]
- "Firewall fechando entrada me protege" → se a saída (porta 80) está aberta, dá pra furar com túnel SSH iniciado de dentro; a conexão nascida internamente flui nos dois sentidos. [cap-052]
- "Minha RAM nunca corrompe bit" → bit flip existe (até raios cósmicos: caso dos 4096 votos, speedrun do Mario 64); datacenter usa ECC, seu PC caseiro escapa porque reinicia e dá refresh. [cap-049]
- "CD/Blu-ray dura pra sempre" → a camada de proteção é orgânica e degrada com O₂/umidade; nem Reed-Solomon salva se degradar muito (M-Disc usa camada inorgânica). [cap-049]
- "Defensividade = sanitizar em toda função" → errado; só as funções que recebem dado direto do usuário precisam. Atirar pra todo lado é amador; "use sniper". [cap-052]
- "VPN é só pra burlar região da Netflix" → esse é um uso; o correto também é navegar seguro em rede não-confiável e acessar servidor interno da empresa/casa. VPN adiciona latência (rota + cripto). [cap-052][cap-053]
- "Pré-compilar o interpretado (.pyc/.plx) acelera muito" → hoje HDs são rápidos o bastante; na prática tanto faz, só poupa o parse inicial. [cap-046]
- "Mais bits de redundância = sempre caro (45%)" → num bloco de 16 bits sim, mas como a checagem é procura binária, bloco maior precisa de proporção muito menor (256 bits → ~3%). Por isso estudar algoritmos importa. [cap-049]
- "JVM/máquina virtual é lenta por natureza" → a inicialização é cara (GC sofisticado, carrega muito), mas o JIT otimiza em runtime só o código quente, chegando perto do nativo. [cap-046]
- "Java é totalmente orientado a objetos" → o autor argumenta que é mais orientado a classes (a própria classe não é objeto); Smalltalk/Objective-C foram representações melhores de OO. [cap-047]
- "Go é moderno por ser de 2009" → pro autor é "um passo pra trás": tipos pobres, permite ponteiro nulo, sem traits/generics decentes; cresceu por softwares importantes (Docker), não por mérito de linguagem. [cap-047]
- "O servidor escolheu me mandar ao data center da Califórnia" → o DNS devolve IPs e faz round-robin/proximidade geográfica; você cai em servidores diferentes por aba, com cookie te mantendo "no mesmo lugar". [cap-050]
- "Streaming sempre existiu" → não; com modem de 56 kbit uma música mp3 levava ~24 minutos pra baixar. Streaming só virou possível quando a banda passou a baixar o conteúdo no tempo de reprodução. [cap-048]
- "Internet = Web" → Web (World Wide Web) é só a capacidade de hipertexto/HTML; internet é o conjunto de tecnologias (Ethernet, Wi-Fi, fibra, TCP/IP e todos os protocolos de aplicação). [cap-051]
- "Switch com '5 Gbit, 5 portas' dá 5 Gbit por porta" → às vezes é 5 Gbit no aparelho todo (1 Gbit/porta); leia as letras miúdas. [cap-053]

## Checklist para aplicar num sistema real
- [ ] Em qual camada (física/enlace/rede/transporte/aplicação) mora o problema que estou projetando ou depurando? A correção na camada certa não deve exigir mexer nas de cima. [cap-050][cap-051]
- [ ] Toda entrada do usuário (query string, header, form, upload) está sendo sanitizada na fronteira antes de virar comando, query ou caminho de arquivo? [cap-052]
- [ ] O servidor está em bind em 0.0.0.0 (produção) ou localhost (dev) conforme a intenção? A porta é adequada (não exige root à toa, não colide com efêmeras)? [cap-051]
- [ ] As dependências têm versões fixadas e um plano de atualização (nem tudo na última, nem tudo parado)? Alguma lib abandonada com falha conhecida? [cap-047]
- [ ] Linkagem/empacotamento: estático (portável, grande) ou dinâmico (menor, exige a lib certa presente) — a escolha combina com o deploy? [cap-046][cap-047]
- [ ] Concorrência: fork (isolado, caro no Windows) ou threads (baratas, exigem lock/thread-safety)? Há risco de deadlock ou corrupção de memória compartilhada? [cap-046]
- [ ] Depurando rede: rodei `ping`/`arp -a` (MAC na LAN), `dig` (DNS), `traceroute`/`tracert` (rota/hops), `ipconfig`/`route print` (IP, gateway, tabela de rotas)? [cap-050][cap-053]
- [ ] "Não conecta de fora": é CGNAT/NAT (precisa iniciar de dentro), firewall bloqueando a porta, ou bind só em loopback? [cap-050][cap-051][cap-052]
- [ ] Colisão de porta ao subir 2 serviços na mesma porta: usar rede bridge (container/VM) e mapear portas internas pra externas livres? [cap-053]
- [ ] Dados importantes: há redundância (RAID-1+, nunca RAID-0 pra o que importa), checksum/scrubbing e backup fora do disco? ECC onde a corrupção silenciosa importa? [cap-049][cap-053]
- [ ] Tráfego sensível vai criptografado (HTTPS/TLS) e o DNS também (DoH/DoT)? Em rede pública, há VPN? [cap-050][cap-053]
- [ ] O IP/geolocalização que a app registra pode ser de NAT/proxy/VPN — a lógica de negócio não está assumindo que é o usuário real? [cap-050][cap-052]
- [ ] Antes de trocar de stack por lentidão: medi o gargalo real (não troquei por palpite)? [cap-045]
- [ ] O serviço persistente (long-lived) é um loop de eventos saudável, ou é um short-lived sendo reiniciado a cada request (custo de boot/interpretador)? FastCGI/processo persistente resolve? [cap-046][cap-051]
- [ ] Interpretado: alguma dependência faz binding a lib nativa não thread-safe que trava a concorrência? A app roda igual entre Linux/Mac/Windows ou assumi portabilidade indevida? [cap-046]
- [ ] Arquitetura em camadas: cada camada ignora a de baixo (ex.: a app não precisa saber de TLS)? Troquei uma camada sem quebrar as de cima? [cap-051][cap-053]
- [ ] A entrada externa que vira comando de shell/SQL/caminho está escapada, não apenas "limpa por regex" frágil? Código de junior passou por review antes de produção? [cap-052]
- [ ] Tipos: a linguagem é estática/dinâmica/com inferência — o "molde" está certo pra não vazar nem estourar memória? [cap-047]
- [ ] Protocolo de aplicação: estou empilhando tudo em HTTP por hábito, quando outro protocolo/transporte (UDP, SFTP, WebSocket) caberia melhor no caso? [cap-051]
- [ ] Portas de serviço estão fechadas por firewall exceto as necessárias, e nenhum serviço de dev (porta 3000) ficou exposto publicamente sem querer? [cap-052]
- [ ] Acesso remoto a serviço interno usa VPN/túnel (site-to-site ou client-to-site) em vez de abrir a rede doméstica/corporativa pra internet? [cap-052][cap-053]
- [ ] Integridade: downloads/ISOs são conferidos por hash (sha256/md5)? Dado de longo prazo tem mídia/filesystem com correção (Reed-Solomon/ZFS/Btrfs) e scrubbing agendado? [cap-049]
- [ ] Protocolo de VPN é atual (Wireguard), não PPTP? Tráfego sensível em rede pública passa por túnel criptografado? [cap-053]
- [ ] Containers/VMs: estou ciente de que a rede deles é virtual (driver "mente")? O modo (host/bridge) e o mapeamento de porta batem com o que quero expor? [cap-053]
- [ ] Ao comprar hardware de rede (switch/placa/cabo), a especificação real cobre a banda pretendida de ponta a ponta (não só no "até X" de marketing)? [cap-053]

## Frases-âncora
> "A história do front-end é a própria história da Web e algumas coisas que você tem hoje só fazem sentido no contexto dos anos 90 ou dos anos 2000…" — cap-045

> "qualquer número de porta serve, contanto que exista alguma coisa "escutando" nessa porta." — cap-045

> "parta do princípio que seu código deve ser escrito primariamente pra que outro ser humano consiga entender e é trabalho do compilador melhorar pra ficar mais eficiente na máquina." — cap-046

> "Só porque dois programas compilam pra dois sistemas operacionais diferentes e rodam, não significa que rodam igual." — cap-046

> "os sistemas de telecomunicação mudaram do modelo de circuit switch pra packet switch e de sessões stateful pra stateless. É isso que permite a escalabilidade que temos agora." — cap-048

> "No caso de rede é mais barato pedir o pacote de novo do que tentar corrigir ele. Correção é mais importante quando você não tem pra quem pedir uma cópia original…" — cap-049

> "Em computação você não deve nunca confiar em ninguém, sob nenhuma hipótese, essa é a regra número um." — cap-050

> "tudo que vem do mundo externo de um usuário, por definição, é podre, sujo, e precisamos sanitizar, antes de usar." — cap-052

> "Se dá pra executar qualquer comando, eu já tenho controle sobre essa máquina inteira. Eu ownei essa máquina. É game over." — cap-052

> "Entender arquivos e entender redes é o mínimo do mínimo que um programador precisa saber." — cap-053

## Onde aprofundar
| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-045 | A História do Front-End para Iniciantes | https://www.akitaonrails.com/2019/02/13/akitando-39-a-historia-do-front-end-para-iniciantes-em-programacao-serie-comecando-aos-40/ | precisar situar HTML/CSS/CGI/AJAX/SPA/pipelines e entender por que cada ferramenta de front existe |
| cap-046 | Entendendo Back-End (Parte 1) | https://www.akitaonrails.com/2019/02/20/akitando-40-entendendo-back-end-para-iniciantes-em-programacao-parte-1-serie-comecando-aos-40/ | for decidir compilado×interpretado, linkagem estática×dinâmica, fork×thread, JVM/JIT/.NET |
| cap-047 | Entendendo Back-End (Parte 2) | https://www.akitaonrails.com/2019/02/27/akitando-41-entendendo-back-end-para-iniciantes-em-programacao-parte-2-serie-comecando-aos-40/ | lidar com gerenciamento de dependências, cliente-servidor, 2/3 camadas, RPC/COM/CORBA/J2EE |
| cap-048 | Introdução a Redes: Como Dados viram Ondas? (Parte 1) | https://www.akitaonrails.com/2022/07/01/akitando-121-entendendo-transferencia-de-sinais-digitais-introducao-a-redes-parte-1/ | precisar de ordens de grandeza de banda, modulação, multiplexação e circuit×packet switching |
| cap-049 | Detecção e Correção de Erros (Parte 2) | https://www.akitaonrails.com/2022/07/13/akitando-122-detecao-e-correcao-de-erros-introducao-a-redes-parte-2/ | for proteger dados: Hamming, Reed-Solomon, checksum, RAID, ECC, scrubbing, bit rot |
| cap-050 | Como sua Internet Funciona (Parte 3) | https://www.akitaonrails.com/2022/07/23/akitando-123-como-sua-internet-funciona-introducao-a-redes-parte-3/ | depurar IP/máscara/ARP/DHCP/DNS/NAT-CGNAT/IPv6 numa rede caseira ou corporativa |
| cap-051 | Sockets, Cliente, Servidor e a Web (Parte 4) | https://www.akitaonrails.com/2022/08/04/akitando-124-como-funciona-sockets-cliente-servidor-e-a-web-introducao-a-redes-parte-4/ | entender bind/listen/accept, portas, loopback, HTTP request/response e o loop de servidor |
| cap-052 | Burlando Proxies e Firewalls - SSH (Parte 5) | https://www.akitaonrails.com/2022/08/17/akitando-125-burlando-proxies-e-firewalls-introducao-a-redes-parte-5-ssh/ | tratar injeção/sanitização e entender firewalls, proxies e túneis SSH (forwarding) |
| cap-053 | Criando uma Rede Segura - VPN e NAS (Parte 6) | https://www.akitaonrails.com/2022/08/29/akitando-126-criando-uma-rede-segura-introducao-a-redes-parte-6-vpn-e-nas/ | montar VPN (TUN/TAP, tabela de rotas, ZeroTier/Wireguard) e armazenamento NAS/RAID seguro |
