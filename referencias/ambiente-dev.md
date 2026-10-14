# Ambiente de desenvolvimento e conhecimentos básicos
> Destilado de: cap-008, cap-009, cap-010, cap-011, cap-012, cap-013 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).
> Datas: cap-008, cap-009, cap-011 e cap-012 são de 2019 (Ubuntu 18.04, Windows 10, WSL2 ainda em alfa); cap-010 é de 2021; cap-013 é de 2025 (hardware). Números de hardware e comandos de instalação envelhecem rápido; os princípios continuam valendo.

## Ideia central
Programar é dar ordens a uma máquina controlada por um sistema operacional. Quem não sabe onde estão as próprias ferramentas, como o shell acha um executável, que privilégio cada processo tem e como o hardware limita o desempenho programa no chute e vira supersticioso. Para o autor, tudo se reduz a sequências de bytes interpretadas por convenções (tabelas de caracteres, cabeçalhos, protocolos posicionais, tabelas de opcodes), e entender essas convenções é o que separa diagnóstico de mandinga. O ambiente de dev deve espelhar a produção (Linux), isolar versões por projeto e ser reproduzível; e como cada década troca a plataforma dominante, o investimento certo é em fundamentos e no processo de aprender, não na ferramenta da moda.

## Conceitos que todo sistema depende

### Bytes, formatos e codificação
- **Tudo é uma fita de bytes** — arquivo de texto, executável, JPEG, RAM e SSD são todos sequências de bits. "Texto" é só o conjunto de bytes que uma tabela (ASCII/UTF-8) traduz em glifos; `cat` apenas tenta essa tradução, e por isso um executável aparece como lixo na tela. `xxd` (ou `xxd -b`) mostra os bytes crus em hexadecimal ou em bits, com o offset à esquerda. [cap-010]
- **A extensão é convenção; o tipo está no cabeçalho** — SO e programas reconhecem o formato pelos primeiros bytes (magic number): ELF começa com `7f` seguido de `ELF` em ASCII, executável Windows (PE) começa com `MZ`, JPEG abre com `ff d8` e fecha com `ff d9` (com `JFIF`/`EXIF` logo no início). O comando `file` faz essa identificação. [cap-010]
- **Formato binário é protocolo posicional** — o cabeçalho define campos com posição de início, posição de fim e valores esperados (no ELF, o 5º byte diz se é 32 ou 64 bits; no exemplo do autor, metadados até 0x1000 e instruções dali em diante). Protocolo de rede é a mesma ideia: um "linguição de bytes" com campos em posições combinadas. [cap-010]
- **Programa e dados vivem na mesma fita** — instruções também são bytes, traduzidos por uma tabela opcode↔mnemônico própria de cada arquitetura (`f3 0f 1e fa` = `endbr64` em x86-64); `objdump` faz essa tradução. Um binário compilado fica amarrado à arquitetura da CPU e ao formato executável do SO. [cap-010]
- **Encoding é uma tabela, e existem várias** — ASCII é um padrão de 7 bits (128 símbolos) herdado dos telégrafos; línguas com milhares de ideogramas exigem codificações de vários bytes (EUC-JP, Shift-JIS, esta com ambiguidade quando mistura ASCII); Unicode tem UTF-8 (compatível com ASCII) e UTF-16/32. Mais bytes por caractere significa mais espaço, e busca, ordenação e similaridade ficam muito mais difíceis fora do ASCII. Texto com lixo no meio pode ser encoding errado, não corrupção. [cap-010]
- **Glifo igual não é byte igual (homógrafos)** — o "a" latino (0x61) e o "а" cirílico (U+0430, `d0 b0` em UTF-8) desenham igual. Isso já permitiu registrar domínios falsos idênticos a olho nu (hoje os registradores filtram) e é primo do typosquatting (`santader.com`). [cap-010]
- **Firmware vem antes do SO** — a BIOS/firmware faz o POST, identifica o hardware, procura o dispositivo de boot numa ordem definida (no MBR, os primeiros 512 bytes) e já traz uma tabela de caracteres para escrever na tela antes do SO. Uma atualização de firmware interrompida deixa a máquina sem o programa que inicia tudo. [cap-010]

### Sistema operacional, shell e processos
- **Kernel não é distribuição** — Linux é o kernel (o binário que sobe a máquina, configura dispositivos e oferece serviços aos programas); a distro decide a seleção de aplicativos, a configuração, o gerenciador de pacotes e o ecossistema. No boot, `vmlinuz` (kernel mínimo) carrega e `initrd.img` sobe um RAM disk com os módulos; symlinks em `/` apontam para `/boot`, o que permite manter vários kernels e trocar só o link. [cap-009, cap-012]
- **Executar texto vs executar binário** — o kernel só executa um formato válido (ELF/PE) com permissão. Um script precisa de `chmod +x` e de shebang na primeira linha apontando o interpretador; sem shebang, quem interpreta é o shell corrente (por isso rodar `hello.c` dá erro de sintaxe do bash, não de C). C precisa ser compilado (`gcc hello.c -o hello`); o `./` manda executar do diretório atual em vez de procurar no PATH. [cap-010]
- **Shell é linguagem e REPL** — bash, zsh, fish e PowerShell são interpretadores com sintaxe, variáveis e `if`; consoles como o IRB ou o do Python são REPLs (read-eval-print loop) rodando por cima do shell. Um comando pode ser builtin do shell (`cd`, `alias`), binário no PATH ou script com permissão: saiba qual dos três está rodando. (complemento, não está no compêndio: `type <cmd>` diz se é builtin; `which` só olha o PATH, por isso responde `/usr/bin/echo` mesmo quando o bash usa o `echo` embutido.) [cap-010, cap-012]
- **PATH e variáveis de ambiente** — tudo que não é instrução do shell é procurado nos diretórios listados em `$PATH`; `$HOME` (e a expansão `~`), `$USER` e as demais aparecem com `set | less`. Ter várias versões de uma linguagem é, no fundo, manipular o PATH. [cap-009, cap-012]
- **Linguagem é um conjunto de executáveis** — interpretador, compilador, debugger e bibliotecas ficam em algum lugar do disco; o programador tem obrigação de saber onde estão e por que funcionam. [cap-009]
- **Dois níveis de privilégio e daemons** — root/administrador vs usuário comum. Programas que você abriu rodam com os seus privilégios e podem ser mortos a qualquer momento; daemons (serviços em background iniciados após o boot: rede, sistema de arquivos, bancos) rodam com mais privilégio e precisam de `sudo` para reiniciar. No Ubuntu de 2019, `service <nome> start|stop|status` chama os init scripts de `/etc/init.d`; com systemd é `systemctl start <nome>` (ordem invertida). [cap-009, cap-012]
- **Mapa do filesystem Unix** — `/home/<user>` é o seu espaço (não mexa fora dele sem saber para que serve); `/etc` configura o sistema e os dotfiles/`~/.config` configuram o usuário; `/bin` e `/sbin` têm binários do sistema (shells, `cp`, `fdisk`, `mkfs`); `/usr/bin`, `/usr/lib`, `/usr/include` e `/usr/src` têm programas, bibliotecas, headers e fontes instalados; `/var/log` guarda logs, `/var/run` PIDs, `/var/cache` caches; `/tmp` não sobrevive ao boot; `/opt` recebe software de fora; `/root` é o home do root. [cap-012]
- **Dispositivo não é ponto de montagem** — `/dev/sdb1` é o dispositivo cru; só depois de montado com um sistema de arquivos (ext4, NTFS, exFAT, HFS+) em `/media` ou `/mnt` dá para navegar. `/etc/fstab` declara as montagens e `df -h` lista o que está montado. [cap-012]
- **Processos são arquivos** — `ps aux` lista processos com PID; em `/proc/<PID>/` há pseudo-arquivos (`status`, `io`, `cgroup`) que se leem como texto. É a forma do SO expor processos sem API complicada. [cap-012]
- **Ocultos e dotfiles** — o que esconde um arquivo é só o nome começar com ponto; configurações de shell, editor e ferramentas vivem nesses dotfiles no home. [cap-012]
- **Streams, redirecionamento e pipes** — `>` liga a saída padrão a um arquivo sobrescrevendo, `>>` anexa, `|` liga o stdout de um comando ao stdin do próximo (`ps aux | grep bash | awk '{ print $2 }'`). `less` lê por stream e navega com atalhos do vi, `tail -f` fica pendurado no fim de um log em produção, `tee` grava e repassa, Ctrl-R busca no histórico. Bash combinado com AWK, Perl ou Python automatiza o sistema inteiro. [cap-012]
- **Nomes curtos têm razão histórica** — `cd`, `rm`, `chown` vêm de terminais remotos lentos e de quando cada byte custava caro; `man <cmd>` é o manual de cada um. [cap-012]

### Ambiente reproduzível
- **O pacote da distro guarda uma versão só** — e anda atrás de propósito, porque distros priorizam o que já foi testado. Linguagens que mudam pouco (C, C++, Java) podem vir do APT; as que mudam rápido (Ruby, Node, Go, Python) e projetos que exigem versões diferentes pedem um gerenciador de versões. [cap-012]
- **Gerenciador de versões universal (ASDF)** — framework com um plugin por linguagem que instala várias versões e escolhe uma global e uma local por projeto, interceptando o PATH com shims (`asdf reshim` recria o shim quando um executável instalado globalmente "some"). (complemento, não está no compêndio: a partir do asdf 0.16, de 2025, os comandos `asdf global`/`asdf local` deram lugar a `asdf set`.) [cap-012]
- **Pacotes assinados e cadeia de confiança** — pacotes oficiais vêm assinados pela chave privada da mantenedora (Canonical); repositório de terceiro exige instalar a chave pública dele, e o APT recusa pacote adulterado, a menos que você tenha instalado a chave do atacante. Por isso confira o domínio de onde vem a chave. `dpkg -L <pacote>` mostra o que um pacote instalou. (complemento, não está no compêndio: o `apt-key` usado no texto de 2019 foi depreciado; hoje a chave vai em `/etc/apt/keyrings` e o repositório a referencia com `signed-by=`.) [cap-012]
- **Três famílias de criptografia** — digest/hash (impressão digital de tamanho fixo; mudou 1 bit, muda tudo: serve para integridade), simétrica (a mesma senha cifra e decifra, como AES) e assimétrica (par de chaves; o que uma cifra só a outra abre; a privada nunca sai da máquina onde foi criada). [cap-012]
- **Chave SSH é a sua identidade** — gere `ed25519` com passphrase (`ssh-keygen -o -a 100 -t ed25519`), cadastre só a `.pub` em GitHub, nuvem e servidores, use `ssh-agent`/`ssh-add` para não redigitar a passphrase e faça backup de `~/.ssh` em lugar seguro. Se perder ou vazar, gere outra e troque a pública em todo lugar: quem tem a privada tem o seu acesso. [cap-012]
- **Identidade no Git** — `git config --global user.name` e `user.email`, ou por projeto em `.git/config` (e-mail pessoal vs e-mail do cliente). Detalhes de Git em `git.md`. [cap-012]
- **Container não é VM** — o Postgres rodando no Docker aparece no `ps aux` do host: é um processo nativo no mesmo kernel, e o isolamento vem de cgroups ("o SO mente pro processo"); por isso é leve. Dados ficam fora do container via volume (`-v $HOME/docker/volumes/postgres:/var/lib/postgresql/data`), e o Docker Compose sobe app e serviços que se enxergam. [cap-012]
- **Serviços de dev têm defaults inseguros** — Memcached, Redis e Mongo não pedem senha por padrão; no Postgres de dev, criar usuário e banco com o nome do usuário do sistema simplifica a conexão, mas em produção nunca se conecta como superuser. [cap-012]
- **Multiplexador de terminal** — o tmux (sucessor do GNU Screen) mantém sessões, janelas e painéis vivos no servidor; se a conexão SSH cair, `tmux attach` recupera tudo. Vim e tmux funcionam igual na máquina local e via SSH. [cap-012]
- **WSL2 vs macOS** — o WSL2 é um kernel Linux de verdade numa VM Hyper-V leve, que aloca recursos sob demanda: o Windows passa a rodar binários Linux. O macOS é UNIX, mas não é Linux, e não roda binário Linux direto. [cap-011]

### Distribuições e sistemas operacionais
- **Famílias de distro** — Debian→Ubuntu→Mint/Elementary (DEB/APT); RedHat→Fedora (desktop) e CentOS (servidor), mais o SUSE (RPM; o Fedora usa DNF, não mais YUM); Arch→Manjaro (pacman, rolling release); Slackware (desde 1993, pacotes são tarballs e não há resolução de dependências); Gentoo (compila tudo localmente, Portage). (complemento, não está no compêndio: o CentOS Linux tradicional foi descontinuado; hoje existem CentOS Stream e derivados como Rocky e Alma.) [cap-011]
- **Estável vs bleeding edge** — Debian e Ubuntu LTS (cerca de 5 anos de suporte) priorizam o testado; versões intermediárias e o Arch trazem o novo com mais chance de quebrar. Rolling release evita os saltos entre versões major, mas atualiza o tempo todo. Tudo que é novo deve ser tratado como potencialmente instável. [cap-011, cap-012]
- **Schedulers de CPU e de I/O** — o Linux deixa trocar o escalonador de threads (CFS; MuQSS em máquina fraca) e o de disco (CFQ ou noop com SSD rápido; BFQ com HD lento) e ajustar o swappiness para evitar swap em disco lento. (complemento, não está no compêndio: o CFQ saiu no kernel 5.0, hoje as opções são mq-deadline, bfq, kyber e none, e o CFS deu lugar ao EEVDF no 6.6; casar o escalonador com o dispositivo continua sendo o princípio.) [cap-011]
- **Retrocompatibilidade tem custo** — o Windows ainda roda binários da era Windows 98 e por isso carrega legado e vetores inseguros (`System32`/`WOW64`, `Program Files (x86)`). A Apple corta o legado (fim do 32-bit no macOS) usando camadas de transição: Carbon, Rosetta (PowerPC→Intel) e fat/universal binaries dentro de bundles. [cap-011]
- **64 bits ajuda a segurança** — ASLR (aleatorizar onde o código é carregado) é fraco em 32 bits porque há poucos endereços e a força bruta acha; em 64 bits fica exponencialmente mais difícil. [cap-011]
- **Compositing na GPU** — desenhar janelas em camadas compostas pela GPU (Quartz no OS X em 2000; Compiz/KWin em 2005; Windows do Vista ao 8) libera a CPU e dá animação fluida. É o exemplo do autor de aceitar ser lento hoje apostando que o hardware alcança em uma ou duas gerações. [cap-011]

### Hardware que limita o software
- **Cores, threads e scheduler** — hyperthreading faz o SO ver 2 threads por core físico (16 cores aparecem como 32 threads), mas nunca dá para usar as 32 a 100%. Ter muitos apps abertos não significa um core por app: o scheduler move processos entre cores. [cap-013]
- **Clock base, boost e throttling** — o boost é um pico curto. Quase toda a energia consumida vira calor, e calor demais faz a CPU cair abaixo do clock base para se proteger (thermal throttle). Notebook é sempre limitado pela dissipação; undervolting troca um pouco de pico por desempenho sustentado. [cap-013]
- **O barramento é o teto** — CPU e chipset expõem um número fixo de lanes PCIe, e cada geração dobra a banda por lane (3.0 ≈ 985 MB/s, 4.0 ≈ 1,9 GB/s, 5.0 ≈ 3,9 GB/s). HD e SSD SATA saturam o mesmo SATA III (≈ 550 MB/s); NVMe usa PCIe x4 (3.0 ≈ 3,5 GB/s; 4.0 ≈ 6 a 7,5 GB/s). Peça mais rápida que o barramento é desperdício (GPU feita para PCIe 5.0 x16 num link x4). [cap-013]
- **Sequencial não é aleatório** — a velocidade anunciada é leitura sequencial de arquivos grandes; com arquivos pequenos e acesso aleatório ela cai drasticamente. A latência vai de milissegundos (HD de 7200 RPM, 150 a 200 MB/s) a microssegundos (NVMe). [cap-013]
- **Unidades de marketing** — MT/s e GT/s contam "transferências", não bytes; DDR transfere nas duas bordas do clock (DDR5-6000 = 3000 MHz). Armazenamento é vendido em base 10 e o Windows mede em base 2, por isso 1 TB aparece como uns 931 "GB"; RAM é vendida em base 2. (complemento, não está no compêndio: velocidade de rede é medida em bits e em base 10, então 1 Gbps ≈ 125 MB/s, diferente do que o texto afirma.) [cap-013]
- **Endereços de 48 bits** — CPUs x86-64 usam 48 bits de endereço porque não existe RAM que precise de 64; economiza transistores (aparece em `lscpu`). [cap-013]
- **RAM ECC e bit flips** — ECC detecta e corrige bits corrompidos e é obrigatória em servidor; em máquina de consumidor o raro bit flip vira tela azul e reboot. [cap-013]
- **Armazenamento barato mente** — SSD ou microSD genérico pode ser um adaptador de microSD ou anunciar capacidade que não tem e passar a sobrescrever o começo quando enche; modelos sem DRAM e com controlador ruim são lentos e morrem cedo. [cap-013]
- **Inspecione o hardware real** — `lscpu`, `sudo dmidecode -t baseboard` e `sudo dmidecode -t memory` mostram CPU, placa-mãe, tipo, velocidade e ECC da RAM. Não confie no nome comercial: a GPU "RTX 4060" de notebook é 30 a 50% mais lenta que a de desktop com o mesmo nome. [cap-013]
- **USB-C é conector, não protocolo** — a mesma porta pode carregar de USB 2.0 a USB4 ou Thunderbolt; cor da porta e formato não garantem velocidade, e nem todo USB-C é Thunderbolt. Leia a especificação. [cap-013]

### Tempo, aprendizado e prática
- **Ciclos de uma década** — pela Lei de Bell, a cada uns 10 anos surge uma classe de computadores mais barata com nova plataforma de programação, rede e interface. dBase/Clipper (1985), VB/Delphi (1995) e Flash/PHP (2005) declinaram na década seguinte. A Lei de Moore "morreu" no teto de clock e virou multicore. [cap-008]
- **Os 20% que resolvem 80%** — use um roadmap como checklist (cobrir o básico dele leva de 3 a 5 anos), aprenda o mínimo útil de cada caixa e aprofunde sob demanda. Comece por Git/GitHub e por terminal, SSH e Linux. [cap-009]
- **Tutorial é laboratório** — os passos de curso sempre funcionam porque foram preparados para isso; projeto real vem quebrado, com instrução desatualizada e código alheio. Treine clonando projetos reais e fazendo-os rodar: o erro é o material de estudo. [cap-009]
- **Ambiente hostil ensina** — começar em máquina modesta obriga a otimizar; quem não sabe tirar o máximo do próprio computador não vai otimizar servidor em produção. [cap-012, cap-013]

## Heurísticas de decisão
| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Escolher o SO de desenvolvimento | Linux como primário (nativo, dual boot, VM ou WSL2); Mac só se for desenvolver para iOS, onde é obrigatório; Windows se depende de pacotes Microsoft ou Adobe | A produção (AWS, Azure, Google Cloud, Heroku) roda Linux; quanto mais perto dela, melhor | cap-011, cap-012 |
| Linguagem nascida no open source (Python, Ruby, Node, PHP) | Rodar em Linux, não na versão nativa de Windows | Dependências nativas não têm equivalente no Windows; nunca funciona 100% | cap-012 |
| Projeto .NET | .NET Core roda em Linux com VS Code (SQL Server também); .NET clássico prende ao Windows | É opção suportada desde a compra da Xamarin | cap-011 |
| Escolher distro | Uma madura, com anos de estrada e ecossistema grande (Ubuntu, Fedora, Mint, Manjaro) | Distro mantida por empresa pequena pode sumir; tutoriais miram as maiores | cap-011 |
| Aprender Linux de verdade | Instalar ao menos uma vez algo manual (Arch, com o ArchWiki) | Instalador de "próximo, próximo" não ensina nada | cap-009, cap-012 |
| Projetos exigem versões diferentes da mesma linguagem | Gerenciador de versões com versão global + local por projeto; C, C++ e Java podem vir da distro | O pacote da distro guarda uma versão só e anda atrás | cap-009, cap-012 |
| Bancos e serviços de desenvolvimento | Subir via Docker/Docker Compose com versão fixa e volume para os dados | Cada projeto pode exigir uma versão (Postgres 7 vs 10) sem poluir o host | cap-012 |
| Escolher banco de dados padrão | Postgres; Redis como cache; Mongo só para dados que podem ser perdidos (analytics, logs) | Mongo costuma ocupar o lugar onde Postgres era o certo | cap-012 |
| Serviço sem autenticação por padrão | Nunca expor na internet; em produção, usuário de banco sem superuser | Defaults de dev são abertos | cap-012 |
| Adicionar repositório de pacotes de terceiro | Instalar só a chave pública vinda do domínio oficial | A assinatura só protege se a chave for legítima | cap-012 |
| Algo quebrou | Ler a mensagem inteira, pesquisar o texto exato (Issues do GitHub, Stack Overflow), entender a causa antes de reiniciar | 99% dos erros já aconteceram com alguém; reboot cego é mandinga | cap-009, cap-012 |
| Mudou a configuração de um daemon | Reiniciar o serviço, não a máquina | Quem precisa reler a configuração é o serviço | cap-009, cap-012 |
| Tarefa demorada via SSH | Abrir o tmux antes de começar | A sessão sobrevive à queda da conexão | cap-012 |
| Ler arquivo grande ou log de produção | `less` para navegar, `tail -f` para acompanhar; evitar `cat` e `more` | `less` navega por stream sem despejar tudo | cap-012 |
| Descobrir o tipo de um arquivo | `file`, `xxd`, magic bytes; nunca só a extensão | Extensão é convenção | cap-010 |
| Texto aparece com lixo | Testar outros encodings (UTF-8, EUC-JP, Shift-JIS) antes de supor corrupção | Os bytes podem ser válidos em outra tabela | cap-010 |
| Validar domínio, usuário ou link exibido | Comparar por código de caractere, não pela aparência | Homógrafos enganam humanos | cap-010 |
| Aplicação web | Testar em mais de um motor (Firefox e Chrome/Chromium) | Quase tudo virou Chromium, mas os usuários usam os dois | cap-012 |
| Comprar máquina de dev | O melhor que o orçamento permitir, sem precisar de topo de linha; 2025: 16 GB de RAM no mínimo, 32 GB ideal, 64 GB se sobe muitos containers pesados (2019: 8 GB mínimo, 16 GB recomendado); prefira SSD | Preço acompanha qualidade, e máquina modesta ensina a otimizar | cap-011, cap-012, cap-013 |
| Notebook novo para Linux | Pesquisar compatibilidade (ArchWiki) e testar com pendrive de boot antes | Hardware fora do padrão consome dias | cap-011, cap-012 |
| Avaliar disco para uma carga | Olhar barramento (SATA vs PCIe, geração, lanes) e desempenho aleatório | O número anunciado é pico sequencial | cap-013 |
| Guardar dado importante | Marca reconhecida, item novo e lacrado, loja confiável; nunca genérico barato nem microSD | Genérico mente capacidade e sobrescreve dados | cap-013 |
| Comprar Xeon/EPYC usado | Só para uso muito específico (ex.: NAS com ZFS que precisa de cores e RAM) | PCIe 3.0, clock baixo, TDP alto, placa-mãe e RAM ECC caras | cap-013 |
| Notebook que esquenta e engasga | Undervolting (Throttlestop, intel-undervolt, amdctl) | Evita cair abaixo do clock base no throttle | cap-013 |
| Planejar carreira ou stack | Plano flexível; investir no processo de aprender, não na tecnologia como identidade | A plataforma dominante muda a cada década | cap-008 |

## Armadilhas e mitos
- "Binário" e "texto" são categorias opostas → todo arquivo é binário; texto é o binário que uma tabela de caracteres traduz, e "binário" no uso popular é só o executável. [cap-010]
- A extensão define o tipo → define só a convenção; o SO olha cabeçalho e permissão (no Windows basta terminar em `.exe`/`.bat`; no Linux precisa de `chmod +x`). [cap-010]
- "Deu erro de C ao executar `hello.c`" → quem tentou interpretar o arquivo foi o bash; código C precisa ser compilado antes. [cap-010]
- Docker é máquina virtual → é processo nativo no mesmo kernel, isolado por cgroups. [cap-012]
- "Reinicia que volta" → superstição; descubra o que precisa ser reiniciado e por quê. [cap-009]
- Seguir o tutorial sem erro é saber → tutorial é laboratório controlado; a vida real é tentativa e erro com código dos outros. [cap-009]
- Começar por distro fácil é o melhor jeito de aprender → ela esconde tudo; sofrer num Arch é o que ensina. [cap-009]
- Dev de verdade precisa de Mac ou de máquina topo de linha → Mac só é obrigatório para iOS; uma máquina modesta basta e ensina mais. [cap-011, cap-012]
- Rodar Python/Node/PHP nativo no Windows é equivalente → bibliotecas com dependências nativas de Linux falham ou se comportam diferente. [cap-012]
- As opiniões fortes de devs experientes sobre Windows, Mac e Linux são atemporais → refletem épocas (o sofrimento com o XP, a era de ouro do Mac até os iPads, o Linux sem compositing). [cap-011]
- Linguagem boa se impõe sozinha → a adoção vem de um killer app (Objective-C só explodiu com o iPhone e a App Store). [cap-011]
- 32 threads no gerenciador são 32 núcleos de trabalho → é hyperthreading; nem todas rodam a 100%. [cap-013]
- O clock de boost é a velocidade da CPU → é um pico curto; o sustentado depende da dissipação de calor. [cap-013]
- "DDR5-6000 = 6000 MB/s" e "meu SSD de 1 TB veio com 931 GB" → transferências não são bytes; base 10 vs base 2. [cap-013]
- Todo SSD é parecido e o barato serve → genéricos mentem capacidade, corrompem dados e morrem cedo. [cap-013]
- Porta azul ou USB-C garantem velocidade → USB-C é só o conector; nem todo USB-C é Thunderbolt. [cap-013]
- Hardware de servidor usado é pechincha → é mais lento por core, gasta mais energia e exige placa e RAM específicas. [cap-013]
- "Estou atrasado" → talvez para a década passada; uma nova já começou. [cap-008]
- O texto indica `rm -f foo` para apagar um diretório → `rm -f` não remove diretório; use `rmdir` (vazio) ou `rm -r` (com conteúdo), conferindo o caminho antes. (complemento, não está no compêndio) [cap-012]
- Pôr o usuário no grupo `docker` (passo do tutorial) é só conveniência → o grupo `docker` equivale a acesso de root na máquina; não faça isso em servidor compartilhado. (complemento, não está no compêndio) [cap-012]

## Checklist para aplicar num sistema real
- [ ] O ambiente de desenvolvimento roda o mesmo SO e a mesma arquitetura da produção (Linux)? Se não, há WSL2, VM ou container cobrindo a diferença? [cap-011, cap-012]
- [ ] As versões de linguagem estão fixadas por projeto num arquivo versionado do gerenciador de versões, e o setup não depende do que está instalado globalmente? [cap-009, cap-012]
- [ ] Bancos, caches e filas sobem por Docker Compose com versão fixa e volume de dados declarado? [cap-012]
- [ ] Serviços sem autenticação por padrão (Redis, Memcached, Mongo) estão fora do alcance da internet? O usuário de banco de produção não é superuser? [cap-012]
- [ ] Repositórios de pacote de terceiros usam chave obtida do domínio oficial? [cap-012]
- [ ] Scripts executáveis têm shebang e permissão de execução? Sabe-se de onde vem cada comando (builtin, PATH, script)? [cap-010, cap-012]
- [ ] Arquivos recebidos são identificados pelo conteúdo (magic bytes) e não só pela extensão? [cap-010]
- [ ] O encoding é declarado e tratado explicitamente (UTF-8), sem assumir 1 byte = 1 caractere? [cap-010]
- [ ] Identificadores exibidos ao usuário (domínios, nomes, links) são comparados por código, considerando homógrafos? [cap-010]
- [ ] Logs vão para lugar previsível (`/var/log` ou stdout) e podem ser inspecionados com `less` e `tail -f`? [cap-012]
- [ ] Daemons são gerenciados por `service`/`systemctl`, e o runbook diz qual serviço reiniciar em vez de "reiniciar a máquina"? [cap-009, cap-012]
- [ ] Chaves SSH são `ed25519` com passphrase, a privada nunca saiu da máquina, há backup e há plano de rotação caso vaze? [cap-012]
- [ ] `user.name` e `user.email` do Git estão corretos para o contexto (pessoal vs cliente)? [cap-012]
- [ ] Tarefas longas em servidor remoto rodam dentro de tmux? [cap-012]
- [ ] O gargalo de I/O foi avaliado pelo barramento e pelo padrão de acesso (aleatório vs sequencial), não pelo pico do datasheet? [cap-013]
- [ ] Capacidades e velocidades foram comparadas na mesma unidade (base 2 vs base 10, bits vs bytes, transfers vs bytes)? [cap-013]
- [ ] O hardware real foi conferido (`lscpu`, `dmidecode`, `lsblk`, `df -h`) antes de tirar conclusões de desempenho? [cap-012, cap-013]
- [ ] Dados importantes estão em armazenamento de marca confiável, nunca em microSD ou SSD genérico? [cap-013]
- [ ] Notebooks e mini-PCs foram avaliados para carga sustentada (throttling), não só pelo boost? [cap-013]
- [ ] As instruções de setup do README foram testadas num ambiente limpo, fora do "laboratório" de quem escreveu? [cap-009]
- [ ] A escolha de tecnologia considera que ela pode estar em declínio em cerca de 10 anos, e a dependência dela está isolada? [cap-008]

## Frases-âncora
> "Toda startup inovadora de ontem se torna o enterprise de amanhã. Entenda essa verdade." — cap-008

> "Se você não conhece o sistema operacional, sua programação sempre vai ter muito achismo e chutes. Você acaba se tornando supersticioso e em vez de resolver os problemas procura alguma mandinga." — cap-009

> "Tudo é binário. O que popularmente chamamos de “binário” é um tipo específico de binário, os executáveis." — cap-010

> "Se você acha que entende tudo de Unicode, você não entende Unicode." — cap-010

> "Prefira escolher distros que já tem alguns anos de estrada." — cap-011

> "por isso quanto mais perto do ambiente final de produção você puder estar, sempre vai ser melhor." — cap-012

> "Metade dos erros que os programadores iniciantes travam são trivialmente resolvíveis só lendo o que aparece na tela." — cap-012

> "Vou repetir: o melhor PC é aquele que você pode comprar!" — cap-013

## Onde aprofundar
| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-008 | [Akitando] #37 - A Dimensão do TEMPO para Iniciantes em Programação \| Série "Começando aos 40" | https://www.akitaonrails.com/2019/01/30/akitando-37-a-dimensao-do-tempo-para-iniciantes-em-programacao-serie-comecando-aos-40/ | for justificar uma escolha de tecnologia de longo prazo ou precisar de perspectiva histórica sobre ciclos de plataforma (Lei de Bell). |
| cap-009 | [Akitando] #38 - Conhecimentos Básicos para Iniciantes em Programação \| Série "Começando aos 40" | https://www.akitaonrails.com/2019/02/06/akitando-38-conhecimentos-basicos-para-iniciantes-em-programacao-serie-comecando-aos-40/ | for montar um plano de estudo ou onboarding: Git, terminal, SSH, Linux, PATH, root vs usuário, daemons. |
| cap-010 | [Akitando] #91 - Qual a REAL diferença entre Arquivos Binário e Texto?? | https://www.akitaonrails.com/2021/01/20/akitando-91-qual-a-real-diferenca-entre-arquivos-binario-e-texto/ | o problema envolver formato de arquivo, encoding, magic bytes, ELF/PE, shebang, shell vs REPL ou homógrafos. |
| cap-011 | [Akitando] #56 - Falando um pouco de MAC, LINUX e WINDOWS \| Qual eu devo escolher? | https://www.akitaonrails.com/2019/07/24/akitando-56-falando-um-pouco-de-mac-linux-e-windows-qual-eu-devo-escolher/ | precisar escolher SO ou distro, entender WSL2, schedulers, retrocompatibilidade binária ou a história das transições da Apple. |
| cap-012 | [Akitando] #54 - O Guia DEFINITIVO de UBUNTU para Devs Iniciantes | https://www.akitaonrails.com/2019/07/12/akitando-54-o-guia-definitivo-de-ubuntu-para-devs-iniciantes/ | for montar uma máquina Linux de dev do zero: filesystem, pipes, ASDF, Postgres/Redis, Docker, chaves SSH, tmux e Vim (comandos de 2019). |
| cap-013 | Coisas que "Iniciantes" Ignoram sobre PCs | https://www.akitaonrails.com/2025/09/14/coisas-que-iniciantes-ignoram-sobre-pcs/ | o gargalo puder ser hardware (CPU, PCIe, RAM, NVMe vs SATA, USB/Thunderbolt, térmica) ou for preciso especificar uma máquina. |
