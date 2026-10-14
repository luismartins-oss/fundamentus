# Linux, sistema operacional e armazenamento

> Destilado de: cap-064, cap-065, cap-066, cap-067, cap-068, cap-069, cap-070, cap-071 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).
> Datas: WSL em 2019 (cap-066); armazenamento e sistemas de arquivos em 2021 (cap-068 a cap-071); boot, pacotes e setup Arch/WSL2 em 2022 (cap-064, cap-065, cap-067). Preços, velocidades, versões de Ubuntu/WSL e passos de instalação são retratos da época e vêm marcados como "datado". Os mecanismos (cadeia de boot, blocos, caches, journal, copy-on-write, dependências) continuam valendo.

## Ideia central
Para o autor, o sistema operacional é uma pilha de abstrações simples que dá para verificar uma a uma: o firmware acha uma partição, a kernel monta blocos como árvore de arquivos, o PID 1 sobe os serviços, e "tudo é arquivo" (`/proc`, `/dev`, imagens de disco). Quem entende essas camadas diagnostica a falha; quem não entende reinstala e coleciona superstições. Armazenamento é sempre blocos endereçados, caches voláteis e peças que vão falhar, então todo sistema precisa assumir falha de disco (redundância, backup, journal ou COW, checksums) e escolher de propósito entre espaço, velocidade e segurança. Dependência (biblioteca, pacote, versão) é o problema que acompanha o programador a vida inteira, e a saída é usar gerenciador de pacotes, fixar versões e instalar só o mínimo.

## Conceitos que todo sistema depende

### Boot, kernel e serviços
- **Cadeia de boot**: o firmware UEFI (que muita gente ainda chama de "BIOS") faz o POST e procura na tabela GPT a partição de sistema EFI (ESP, em FAT32, montada no Linux em `/boot/efi`). De lá carrega o bootloader (GRUB), que carrega a kernel (`vmlinuz`) e a imagem `initrd`. A kernel monta um initramfs em RAM e depois executa o PID 1, que lê `/etc/fstab`, monta a raiz e sobe os serviços até a tela de login. [cap-064, cap-069]
- **initramfs**: a kernel pode não ter, compilado dentro dela, o driver para chegar ao disco real (partição LUKS, RAID, LVM). Por isso descomprime um mini-Linux na RAM com esses drivers e só então monta a raiz. É também por isso que sempre sobra uma partição de boot pequena, não criptografada, legível pelo firmware. [cap-064]
- **"Linux" é só a kernel**: o binário `vmlinuz` é o Linux. Shell, gerenciador de pacotes e utilitários são userland (GNU), e o que diferencia as distros é o userland. Linux também não é um UNIX: é compatível em boa parte, mas é outro SO. A herança do UNIX vive nos BSDs e no Darwin do macOS. [cap-064, cap-066]
- **PID 1 e gerenciador de serviços**: o primeiro processo (systemd, OpenRC, runit; antigamente o `init` do SysV) sobe os daemons respeitando as dependências que cada um declara. No SysV, os scripts ficam em `/etc/init.d` e os links em `/etc/rcN.d`. O systemd troca isso por units e targets. [cap-064]
- **Runlevels e targets**: são conjuntos de serviços. 0 desliga, 1 é manutenção (um usuário), 3 é multiusuário em texto com rede, 5 é gráfico, 6 reinicia. No systemd o equivalente são `multi-user.target` e `graphical.target`, e o conceito é o mesmo do "Modo de Segurança" do Windows. [cap-064]
- **Desligamento gracioso**: `shutdown`/`halt` avisam os processos para encerrarem e fazerem flush dos caches de disco. Dar reset forçado com frequência acaba corrompendo arquivos e partições. [cap-064]
- **Kernel anterior como rede de segurança**: `vmlinuz` e `initrd.img` são links simbólicos para a versão real. Depois de um upgrade, a kernel antiga continua no menu do GRUB para bootar e consertar. [cap-064]
- **Ambiente de recuperação**: o Windows guarda um mini-Windows de diagnóstico numa partição escondida, o Mac tem o seu (Cmd+R) e, no Linux, o LiveCD/pendrive cumpre esse papel: boota, monta o disco instalado e permite consertá-lo. [cap-064, cap-065]
- **Boot como superfície de ataque**: o que estiver na ESP roda antes da kernel. Um rootkit que se injeta ali ganha privilégio de kernel e engana o antivírus. O Secure Boot assina kernel e drivers e se recusa a bootar binário modificado. Pendrive ou porta Thunderbolt/USB maliciosa também é um vetor de ataque. [cap-069]
- **Kernel monolítica × microkernel**: kernel e drivers rodam em Ring-0, os aplicativos em Ring-3. O microkernel é mais elegante, mas paga muita IPC entre os anéis. Puxar código para o Ring-0 (drivers de vídeo no NT 4, o HTTP.sys do IIS) ganha desempenho, mas faz um bug virar crash ou brecha no nível da kernel. [cap-066]

### "Tudo é arquivo", montagem e dispositivos
- **Pseudo-sistemas de arquivos**: `/proc` (um diretório por PID, ex. `/proc/PID/status`), `/dev` e `/sys` são pontos de montagem virtuais criados no boot. Nada ali existe no disco. Dá para escrever um `top` só lendo `/proc`. [cap-064]
- **Dispositivo de bloco**: é qualquer sequência de bits que dê para ler e gravar em blocos: HD, pendrive, arquivo `.img`/`.iso`/VHD ou até um destino na rede (Google Drive montado). Um arquivo pode ser formatado com `mkfs` e montado como se fosse um pendrive. [cap-064]
- **Montagem**: montar é dizer ao SO em que diretório o dispositivo aparece e qual sistema de arquivos interpreta os bits (`mount -t ext4 /dev/sda3 /mnt`). O `/etc/fstab` declara isso para o boot. No Windows o ponto de montagem é uma letra (`D:`) e o disco físico é `\\.\PHYSICALDRIVEn`. [cap-064, cap-065, cap-067]
- **Dispositivos especiais**: `/dev/null` engole saída; `/dev/zero` gera nulos sem fim (com `dd` serve para criar uma imagem vazia); `/dev/urandom` é o gerador pseudoaleatório que bibliotecas de criptografia usam. [cap-064]
- **Block device × character (raw) device**: o block device passa pelo cache da kernel (mais rápido, mas arrisca perder dados numa pane). O raw entrega byte a byte e sem espera, como teclado e mouse. Os BSDs tratam disco como raw para que a aplicação decida quanto risco de cache aceita. [cap-069]
- **Terminal é cliente**: o emulador de terminal se conecta a um servidor de terminal local. Fechar a janela não desliga o Linux. O LSP leva o mesmo modelo cliente-servidor para os editores: o servidor da linguagem analisa o código e o VS Code ou o Neovim só exibem. [cap-067]
- **Pipe de bytes × pipe de objetos**: no Unix, stdin/stdout são fluxos de bytes, e é preciso `awk`/`sed`/`grep` para parsear. O PowerShell passa objetos estruturados de um comando para o outro. [cap-066]

### Pacotes e dependências
- **Biblioteca compartilhada**: um executável depende de `.so` do sistema (`ldd` lista quais). Se uma delas some, o programa crasheia ao iniciar. [cap-065]
- **Anatomia de um `.deb`**: é um arquivo `ar` com `data.tar` (os arquivos que vão para `/usr`) e `control.tar` (descrição, `Depends` com versões mínimas, `md5sums` para checar integridade). RPM (Red Hat/Fedora/SUSE) e PKG (Arch) são equivalentes. [cap-065]
- **Duas camadas de gerenciador**: o `apt` lê as fontes em `/etc/apt/sources.list`, baixa o índice (`Packages.gz`) no `apt update`, resolve as dependências e chama o `dpkg`, que só instala ou remove um `.deb` específico. `dnf`/`zypper` fazem o mesmo sobre o `rpm`. No Slackware, `installpkg`/`removepkg` são meros scripts de shell. [cap-065]
- **Por que pacote e não `make install`**: o gerenciador mantém um banco do que foi instalado e o manifesto de cada pacote, então a remoção é limpa. Com tarball + `./configure && make && sudo make install`, sobram arquivos espalhados (ex.: libs em `/usr/local/lib`). [cap-065]
- **Distro como fábrica de binários**: compilar do fonte era necessário quando havia muitas arquiteturas de CPU. Hoje Red Hat e Canonical baixam o tarball, compilam, rodam os testes automatizados de cada pacote e publicam o binário, com rollback se algo quebrar. [cap-065]
- **Repositório de terceiros e assinatura**: software que não está na distro (o Docker no Ubuntu, por exemplo) exige instalar a chave GPG pública do fornecedor e adicionar a fonte. O gerenciador valida a assinatura para não instalar malware de um servidor falso. [cap-067]
- **Repositórios de receitas**: AUR (Arch) e SlackBuilds guardam receitas, não binários. `yay` e `sbopkg` automatizam clonar, montar o pacote, instalar e resolver dependências. [cap-065, cap-067]
- **Famílias de distro**: Debian/Ubuntu (DEB/apt); Red Hat/Fedora/SUSE (RPM/dnf, preferidas no mundo enterprise); Arch (pacman/AUR, wiki muito completa); Slackware/Gentoo, para avançados. Debian e as versões LTS ficam atrás nas versões em troca de estabilidade, e muito software só suporta a LTS. [cap-067]
- **Versões por projeto**: um dev mantém vários projetos em versões diferentes de linguagem. Um gerenciador de versões (asdf, com `.tool-versions` no projeto) evita que o upgrade global quebre um projeto antigo. [cap-067]

### Discos, partições e redundância
- **Setor, cluster e sobra**: o disco não lê byte a byte. Lê setores (512 B ou 4 KB), que o sistema de arquivos agrupa em clusters. O último cluster de cada arquivo desperdiça a sobra, então o espaço ocupado é sempre maior que a soma dos tamanhos. Setor grande fragmenta menos e desperdiça mais; setor pequeno faz o contrário. [cap-068, cap-070]
- **CHS → LBA**: as coordenadas físicas cilindro/cabeça/setor viraram um endereço linear (LBA, padrão ATA-6 de 2002, 48 bits). O SO enxerga um array gigante de setores e o disco faz o "de-para" para a geometria física. [cap-068]
- **MBR × GPT**: o MBR ocupa o primeiro setor (assinatura `55AA`) e admite no máximo 4 partições e cerca de 2 TB (2^32 setores de 512 B). O GPT identifica partições e tipos com GUID de 128 bits, admite milhares de partições e tamanhos que o hardware ainda nem alcança, e mantém um MBR por compatibilidade. [cap-069, cap-070]
- **Partição é índice**: a tabela de partições funciona como o índice de um livro. Apagar uma partição ou um arquivo tira a entrada do índice, mas os bytes continuam no disco. Por isso dá para redimensionar sem perder dados (se o conteúdo couber) e para recuperar o que foi "apagado". [cap-069, cap-071]
- **Partição × volume**: a partição é uma divisão lógica de um dispositivo. O volume é uma abstração do SO que pode juntar partições de vários discos (stripe, RAID). O "tipo" da partição (EFI System, Linux swap) é só uma etiqueta; quem interpreta os bits é a formatação. [cap-065, cap-069]
- **Swap**: é disco usado como memória quando a RAM acaba, ordens de grandeza mais lento. Hoje costuma ser um arquivo, e serve de emergência. Se o sistema vive em swap, a resposta é mais RAM. [cap-065, cap-069]
- **Níveis de RAID**: o RAID-0 (stripe) espalha os blocos de cada arquivo pelos discos. Soma espaço e velocidade, mas perde tudo se um disco morrer. O RAID-1 (espelho) usa metade do espaço e sobrevive à perda de um disco. O RAID-10 combina os dois. O RAID-5 distribui paridade, perde mais ou menos um disco de capacidade e sobrevive à falha de um. Um ZFS com vários discos de paridade aguenta várias falhas ao mesmo tempo. [cap-068, cap-069]
- **Falha é rotina em escala**: tudo que tem peça móvel vai falhar. Num data center com milhares de discos, alguns quebram por dia. Discos de NAS/data center (IronWolf, WD Red) são feitos para ficar ligados 24×7. O SMART faz o disco avisar que vai morrer antes de quebrar de vez. [cap-068]
- **Clone raw**: copiar bloco a bloco (`dd`, clonadora) não depende de entender o sistema de arquivos e é o jeito seguro de salvar um disco que está morrendo. Copiar arquivo por arquivo exige montar o sistema de arquivos e é mais arriscado. [cap-068]
- **Cadeia de caches**: o SO grava em RAM, acumula, reordena e devolve "OK" ao programa antes de mandar para o disco. O disco tem o próprio cache. Um arquivo "salvo" pode não estar no meio físico quando a energia cai. Num desktop isso é raro; num servidor com milhões de escritas, é certo que acontece. [cap-068, cap-069]
- **Durabilidade explícita** (complemento, não está no compêndio): em código, o retorno de `write()` só garante que o dado chegou ao cache. Durabilidade se pede com `fsync`/`fdatasync` (ou com as garantias de commit do banco), e um sistema crítico precisa saber em que ponto o dado passa a estar de fato no disco.

### Dispositivos e desempenho (números de 2021, datados)
- **Ordens de grandeza**: HD mecânico de cerca de 50 a 300 MB/s, que cai para 2 ou 3 MB/s com milhares de arquivos pequenos. SSD SATA por volta de 500 MB/s, limitado pelo SATA-3. NVMe via PCIe de 2 a 3 GB/s. Trocar HD por SSD rende pelo menos 10× mais desempenho. [cap-069]
- **Muitos arquivos pequenos**: o custo está no número de operações (busca, metadado, comando), não no total de bytes. Um arquivo grande lê e copia ordens de grandeza mais rápido que meio milhão de arquivos pequenos com o mesmo total. [cap-068, cap-067]
- **Interface é gargalo**: USB 2.0 dá uns 60 MB/s; USB 3.2 Gen1, 5 Gb/s; Gen2x2, 20 Gb/s; Thunderbolt 3, 40 Gb/s; Ethernet de 1 Gb, uns 125 MB/s; 10 Gb para editar vídeo pesado direto da rede. O conector (Type-C) não diz o protocolo (USB 3.2 ou Thunderbolt). Converter bits para bytes: dividir por 8. [cap-069]
- **NAND e desgaste**: SLC, MLC, TLC e QLC guardam de 1 a 4 bits por célula. Mais bits significam mais capacidade, mais lentidão e menos ciclos de escrita (SLC, de 50 a 100 mil ciclos P/E; QLC, uns 1000). SSD também morre, só que não por pancada no motor. [cap-069]
- **DAS, NAS, SAN**: DAS é o armazenamento ligado direto (USB/Thunderbolt). NAS é um "PC com Linux e discos em RAID" na rede. SAN é uma agregação maior de storage em rede, base de Dropbox e Google Drive e de volumes de bloco na nuvem, como o EBS. [cap-069]

### Sistemas de arquivos
- **Para que serve um sistema de arquivos**: é o intérprete que traduz blocos em árvore de diretórios e arquivos. Troca endereço cru por nome e organização, ao custo de metadados. Cartucho de videogame não usa sistema de arquivos e acessa os dados direto pelo endereço. [cap-065, cap-070]
- **Metadado custa bytes**: nome, datas, permissões e ponteiros podem pesar mais que o conteúdo de arquivos minúsculos. Coisa pequena e numerosa vai melhor agregada num único arquivo ou array e acessada por offset, como numa ROM. [cap-070]
- **FAT**: é uma tabela de alocação de clusters, e FAT12, FAT16 e FAT32 diferem na largura do índice. O tamanho do cluster define o desperdício e o tamanho máximo da partição. O FAT32 limita cada arquivo a 4 GB. Não tem journal, nem permissões, nem checksums. [cap-070, cap-071]
- **Bit vector e bitmask**: o FAT guarda vários atributos (somente leitura, oculto, sistema, diretório, arquivo) num único byte, um bit para cada um. É a técnica de densidade que reaparece em protocolos de rede e nas capabilities. [cap-070]
- **Compatibilidade retroativa como estratégia**: o VFAT gravou os nomes longos em entradas extras e manteve o nome 8.3 (`C:\PROGRA~1` funciona até hoje). O ext3 adicionou journal sendo compatível com o ext2, e por isso foi adotado sem reformatar. [cap-070, cap-071]
- **B+Tree**: os sistemas de arquivos modernos (HFS, HPFS, NTFS, XFS, btrfs) usam árvores balanceadas largas e rasas. Os nós internos guardam chaves e metadados, e as folhas apontam para os blocos. A busca é rápida e a fragmentação, menor, em troca de algum espaço. [cap-070, cap-071]
- **inode, dentry, superblock e VFS**: o inode guarda os metadados (uid, gid, timestamps; `stat` mostra), o dentry guarda o nome e o superblock define o sistema de arquivos. A kernel fala com o VFS, uma interface (`inode_operations`) que cada sistema de arquivos implementa. Por isso o mesmo programa funciona sobre ext4 ou btrfs. [cap-071]
- **Journaling**: uma operação que parece única tem várias etapas. Apagar um arquivo, por exemplo, é tirar o dentry, devolver o inode e liberar os blocos. O journal (um log circular) registra a operação antes de executá-la. Depois de uma pane, o sistema refaz ou volta ao último estado consistente. Sem journal, resta o `fsck`/Scandisk percorrer a árvore inteira no boot. [cap-071]
- **Checksums**: certificam cada bloco com custo da ordem de 1% e detectam corrupção, o que guardar uma cópia inteira não faz bem. O NTFS tem; o HFS+ não; o ext4 só no journal; o ZFS em cada bloco. [cap-071]
- **Copy-on-write (COW)**: em vez de sobrescrever no lugar, o sistema grava os blocos novos, copia o pequeno array de ponteiros e só no fim troca o ponteiro. O original fica intacto se a luz cair, o que dispensa o journal tradicional e viabiliza snapshots, cópias instantâneas e deduplicação. Lembra commits de Git. A memória usa o mesmo princípio no `fork` de processos. [cap-071]
- **Snapshot**: copia só a estrutura, em segundos, e depois ocupa espaço apenas com os blocos alterados. Serve para voltar no tempo depois de uma atualização arriscada e para recuperar arquivos apagados. [cap-071]
- **ZFS e btrfs**: são sistemas de arquivos de nova geração que também gerenciam volumes (pool de discos). O ZFS é a referência para storage sério (checksum por bloco, autorrecuperação, não usa `fsck`), mas é difícil, e mal configurado perde dados. A licença CDDL é um problema para o mundo GPL, então para servidor de arquivos o autor indica FreeBSD. O btrfs é a alternativa nativa do Linux, já que o ext4 chegou ao limite de evolução. [cap-071]
- **APFS e NTFS**: o APFS (Apple, 2017) traz cópias baratas, snapshots, checksums, criptografia e redirect-on-write. O NTFS tem journal, permissões, checksums, ADS e snapshots via Volume Shadow Copy. Ambos são proprietários: gravar NTFS fora do Windows ainda é arriscado. [cap-071, cap-070]
- **Forks e ADS**: o resource fork do HFS e os Alternate Data Streams do NTFS são dados "escondidos" junto ao arquivo. Somem ao copiar para FAT32; o Mac cria `._arquivo` e `__MACOSX` para não perdê-los. [cap-070]
- **Criptografia de volume**: FileVault, BitLocker e LUKS. Com AES em hardware, a perda de desempenho é imperceptível. A senha de login não protege um disco removido e montado em outro computador. Perder a chave é perder os dados. [cap-071, cap-065]
- **Apagar não é destruir**: apagar remove o inode, mas o conteúdo continua no disco e pode ser recuperado por perícia. O safe delete sobrescreve várias vezes (é lento). Num disco criptografado, basta descartar a chave. [cap-071]
- **exFAT**: é o único sistema de arquivos universal (Windows, Mac, Linux). É simples, rápido e sem limites práticos de tamanho, mas não tem journal, COW nem garantia nenhuma. Serve para transporte (câmera, pendrive), não para guardar. [cap-071]
- **Compartilhamento em rede**: o NFS (aberto, da Sun) "tende a simplesmente funcionar"; o CIFS/SMB do mundo Windows, segundo o autor, "vira e mexe tem probleminhas". (complemento, não está no compêndio) O autor mistura os nomes: hoje o protocolo do Windows é o SMB 2/3, CIFS é o dialeto antigo SMB1 (desaconselhado) e Samba é a implementação open source. [cap-071]

### Rodar Linux sobre Windows (WSL), datado: 2019 e 2022
- **Compatibilidade de código-fonte × binária**: o subsistema POSIX do NT, o SFU, o Cygwin e o macOS exigem recompilar ("portar"). O WSL1 traduzia syscalls do Linux para as do NT (um Wine ao contrário). O WSL2 roda uma kernel Linux real numa VM leve do Hyper-V e tem 100% de compatibilidade binária. [cap-066]
- **Camada de tradução é alvo móvel**: Wine e WSL1 quebram a cada versão nova de qualquer um dos lados. A Microsoft abandonou a tradução e passou a rodar a kernel de verdade. [cap-066]
- **Virtualização completa × paravirtualização**: na completa, a kernel convidada não sabe que está numa VM. Na paravirtualização, a kernel é modificada para cooperar com o hypervisor. Uma VM comum perde desempenho ("20% ou mais", diz o autor) e reserva CPU e RAM de antemão. O WSL2 usa VMs utilitárias que pedem recursos sob demanda e bootam em segundos. [cap-066]
- **A fronteira de arquivos é rede**: `/mnt/c` (Windows visto do Linux) e `\\wsl$` (Linux visto do Windows) passam por um protocolo de arquivos em rede (9P). É uma ordem de grandeza mais lento, pior ainda com projetos de muitos arquivos pequenos. Dentro do disco virtual ext4 o desempenho é quase nativo. [cap-066, cap-067]
- **Disco virtual dinâmico**: num VHDX ou arquivo esparso, o SO convidado acredita ter o tamanho inteiro, e o arquivo cresce sob demanda. Montado com `wsl --mount`, o mesmo disco aparece em todas as distros do WSL porque todas compartilham uma única kernel. [cap-066, cap-067]
- **Portas e NAT**: o WSL2 mapeia automaticamente as portas da VM para o `localhost` do Windows. É conveniente, mas portas que conflitam entre os dois lados geram problemas difíceis de diagnosticar. [cap-066]
- **systemd e GPU no WSL** (datado): em 2019 e 2022 o WSL não bootava com systemd (sem `systemctl`, daemons subidos à mão) e não expunha GPU estável, e o autor recomendava Linux nativo para ML. (complemento, não está no compêndio) Versões atuais do WSL2 aceitam systemd via `/etc/wsl.conf` e computação em GPU. Confirme na documentação antes de concluir que algo não existe. [cap-066, cap-067]

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Linux não entra no modo gráfico depois de upgrade ou script | Segurar Shift no boot para ver o GRUB, `e` na entrada, acrescentar `3` à linha da kernel (modo texto) ou escolher a kernel anterior; consertar e reiniciar | Evita reinstalar; quase sempre o problema está nos serviços gráficos ou na kernel nova | cap-064 |
| Programa gráfico travou a interface | `Ctrl+Alt+F3`, logar em texto, `ps aux \| grep <prog>`, `kill -9 <PID>`, voltar ao gráfico | Destrava sem reset forçado | cap-064 |
| Serviço não subiu | `systemctl status <serviço>` e `dmesg`; `systemctl enable` para subir no boot e `start` para subir agora | O log do boot e o status mostram a causa | cap-064 |
| Desligar ou reiniciar servidor/estação | `shutdown -h now` / `telinit 0` (ou `reboot`), nunca o botão | Os processos fazem flush e fecham arquivos | cap-064 |
| Gravar imagem com `dd` ou formatar com `mkfs` | Confirmar com `lsblk` qual `/dev/sdX` é o alvo antes de executar | O `dd` não pergunta nada; errou o dispositivo, perdeu o disco | cap-064 |
| Instalar software | Usar sempre o gerenciador de pacotes; repositório de terceiros só com chave GPG; evitar `make install` solto | Desinstalação limpa, dependências resolvidas, integridade verificada | cap-065, cap-067 |
| Compilar do fonte ou seguir instrução de projeto | Ler o README e as mensagens na tela antes (ex.: `autoreconf` antes do `./configure`) | "90% dos erros de iniciantes" é não ler o que está escrito | cap-065 |
| Usar tutorial ou blog post | Conferir a data e se o passo ainda vale; entender o que cada comando faz | Software muda (a dependência do `jq` sumiu entre 2020 e 2022) | cap-065, cap-067 |
| Servidor ou imagem de produção | Instalar só o necessário | Todo software parado é bug de segurança esperando escalada para root | cap-065 |
| Comprar hardware ou periférico | Verificar se o fabricante lança drivers com frequência; desconfiar de notebook recém-lançado no Linux | Driver é o maior problema de qualquer SO | cap-065, cap-067 |
| Máquina de trabalho | Não usar builds Insider/beta; preferir LTS | Não há rollback; em 2022 o Docker parou de funcionar | cap-067 |
| Máquina lenta com pouco orçamento | Trocar HD por SSD primeiro, depois aumentar RAM | É o upgrade de maior ganho (pelo menos 10×) | cap-067, cap-069 |
| Rodar VM ou WSL2 | Mínimo de 4 núcleos e 8 GB; ideal 8 núcleos e 16 GB; abaixo disso, distro leve nativa | A VM divide recursos e, quando falta RAM, tudo vai para swap (datado: 2022) | cap-067 |
| Projetos no WSL2 | `git clone` dentro do sistema de arquivos Linux (VHDX); para separar do SO, VHDX próprio montado com `wsl --mount` | `/mnt/c` passa por 9P e é muito mais lento | cap-067 |
| Copiar ou mover milhares de arquivos | Empacotar (tar/zip) e transferir um único arquivo | O custo é por operação, não por byte | cap-067, cap-068 |
| Disco avisando falha (SMART) ou vários avisando ao mesmo tempo | Clonar em modo raw, bloco a bloco, para um disco igual ou maior; não tentar reconstruir o RAID degradado com vários discos ruins | O clone não depende do sistema de arquivos | cap-068 |
| Escolher arranjo de discos | RAID-0 só para dados reconstruíveis (cache); dado único em RAID-1/5/10 ou ZFS com paridade, mais backup | Num stripe, perder um disco é perder tudo | cap-068, cap-069 |
| Política de backup | Várias cópias em lugares diferentes (PC, notebook, NAS, nuvem); 2FA nas contas; nunca jogar fora os originais | Tudo deteriora; conta na nuvem é suspensa "por erro"; remasterizar exige o original | cap-068, cap-069 |
| Notebook ou dado sensível | Ativar LUKS/BitLocker/FileVault e guardar a chave com cuidado | Roubo; com AES em hardware o custo é imperceptível | cap-071, cap-065 |
| Escolher sistema de arquivos no Linux | Na dúvida, ext4 (todo o ferramental assume ele); quer snapshots/COW, btrfs ou ZFS; storage multidisco sério, ZFS (de preferência em FreeBSD) | Equilíbrio entre suporte, recursos e risco de licença | cap-065, cap-071 |
| Trocar arquivos entre SOs ou com câmera | exFAT; transferir logo para o armazenamento definitivo | Universal e rápido, sem garantia nenhuma | cap-071 |
| Ligar compressão no sistema de arquivos | Só se os dados forem muito comprimíveis (texto) e houver necessidade comprovada | Ganho pequeno em JPG/vídeo; trabalho e risco extras | cap-071 |
| Garantir que dado sensível desapareça | Criptografar desde o início; não guardar senha em arquivo texto | Apagar só tira o dado do índice | cap-071 |
| Vários projetos em versões diferentes de linguagem | asdf (ou equivalente) com versão declarada por projeto | Upgrade global quebra projeto antigo sem ninguém perceber | cap-067 |
| Banco e serviços no ambiente de dev | Docker Compose por projeto com a mesma versão da produção, em vez de instalar nativo | Desenvolver no Postgres 14 e rodar no 9 quebra em produção | cap-067 |
| Ferramentas de ecossistema Linux (Node, PHP, Python) no Windows | Rodar o original num Linux (WSL2/VM), não o port nativo para Windows | Todo port tem limitações | cap-066 |
| VHDX grande | Não deixar em pasta sincronizada pelo Dropbox/OneDrive | Cada alteração reenviaria o arquivo inteiro | cap-067 |
| Copiar chaves SSH por pendrive FAT | `chmod -R 600 ~/.ssh` no destino (ou transferir num tar, que preserva permissões) | O FAT não guarda permissões Unix | cap-067 |
| Docker no WSL "não conecta" ou dá "permission denied" | Habilitar a WSL Integration da distro no Docker Desktop; sair e entrar de novo após ser adicionado ao grupo `docker` | Grupo e integração só valem numa nova sessão (datado: 2022) | cap-067 |
| Primeiro acesso a uma distro que loga como root | Criar usuário comum com sudo; `sudoers` aberto para `ALL` só em máquina de dev | Não se trabalha logado como root | cap-067 |

## Armadilhas e mitos
- "O PC tem BIOS" → hoje é firmware UEFI com tabela GPT; BIOS com MBR é o legado, limitado a 4 partições e 2 TB. [cap-064, cap-069]
- "Dispositivo é um hardware físico" → para o SO, dispositivo de bloco é qualquer sequência de bits: um arquivo de imagem é um "pendrive" sem hardware. [cap-064]
- "Instalar distro ensina Linux" → instaladores e LiveCDs escondem particionamento, formatação, montagem e boot; não ensinam nada. [cap-064, cap-065]
- "LiveCD só serve para instalar" → também é o ambiente para montar e recuperar um sistema que não boota. [cap-065]
- "Distro com tudo pré-instalado acaba com o problema de dependência" → não acaba (o Slackware precisou de SlackBuilds/sbopkg). E o custo de manter coisas instaladas não é CPU/RAM (serviço desabilitado não carrega), é segurança. [cap-065]
- "RAID-0 com dois discos perde só metade dos arquivos se um morrer" → perde tudo, porque o stripe divide blocos, não arquivos. [cap-068]
- "RAID substitui backup" (complemento, não está no compêndio) → RAID protege contra falha de disco, não contra apagar sem querer, corrupção lógica ou ransomware. Por isso o autor mantém cópias separadas além do RAID. [cap-068]
- "SSD não quebra porque não tem motor" → as células NAND têm ciclos de escrita limitados (QLC bem menos que SLC); backup continua obrigatório. [cap-069]
- "Recebi OK, então está gravado" → o OK sai do cache do SO ou do disco; uma queda de energia pode perder a última escrita. [cap-068, cap-069]
- "Apaguei, então sumiu" → partição e arquivo apagados só saem do índice; os bytes ficam no disco até serem sobrescritos. [cap-069, cap-071]
- "A senha de login protege os dados do notebook" → basta montar o disco em outra máquina; só a criptografia de volume protege. [cap-071]
- "Espaço ocupado = soma dos tamanhos" → há a sobra do último cluster e os metadados, que podem ser maiores que o conteúdo. [cap-068, cap-070]
- "Velocidade de disco é um número fixo" → depende do número de operações, do tamanho dos arquivos, da fragmentação e da interface. [cap-068, cap-069]
- "O tipo da partição muda alguma coisa" → é só etiqueta; quem define como os bits são lidos é a formatação. [cap-065]
- "Criptografia de disco deixa tudo lento" → com AES em hardware a perda é imperceptível; a única desculpa válida é não saber guardar a chave. [cap-071]
- "Compressão no sistema de arquivos é ganho grátis" → em dado já comprimido o ganho é mínimo, e soma trabalho e risco. [cap-071]
- "ZFS resolve tudo" → é o mais robusto, mas complexo; mal configurado, perde dados. [cap-071]
- "Journal e COW desperdiçam espaço" → é a troca deliberada de espaço (barato) por integridade e recuperação. [cap-071]
- "Espaço é caro, tem que economizar bytes" → era assim nos anos 80/90 (nomes 8.3, datas `YYMMDD`). Hoje vale gastar espaço com setor de 4 KB, journal, snapshots e partição de recuperação para ganhar velocidade e segurança. [cap-068, cap-070, cap-071]
- "No WSL2 dá para trabalhar no projeto em `/mnt/c`" → funciona, mas é ordens de grandeza mais lento por causa do protocolo de rede. [cap-067]
- "Microkernel é sempre melhor" → é mais elegante, mas o custo de IPC é alto; o autor considera que o sucesso do Linux mostrou que a kernel monolítica funciona melhor. [cap-066]
- "Quando não sei o que é, tento receitas até alguma funcionar" → é superstição. É preciso entender as camadas (app, SO, cache, sistema de arquivos, dispositivo, rede) para diagnosticar no lugar certo. [cap-069]

## Checklist para aplicar num sistema real
- [ ] Consigo descrever a cadeia de boot do alvo (UEFI → ESP → bootloader → kernel/initramfs → PID 1 → serviços) e sei em que ponto cada falha aparece?
- [ ] O initramfs tem os drivers para chegar à raiz (LUKS, RAID, LVM)? Existe kernel anterior ou mídia de recuperação para voltar?
- [ ] Os serviços sobem pelo gerenciador (units do systemd, com dependências declaradas) e não por scripts soltos? O desligamento é gracioso?
- [ ] Tudo foi instalado por gerenciador de pacotes ou repositório assinado? O servidor tem só o mínimo necessário?
- [ ] As versões de linguagem e de serviços (banco, cache) estão fixadas por projeto e batem com a produção?
- [ ] O dado crítico tem cópias em lugares diferentes? Existe dado "original" que alguém pretende descartar?
- [ ] O arranjo de discos sobrevive à falha de um disco? Algum RAID-0 guarda dado que não dá para reconstruir?
- [ ] Há monitoramento de SMART e alerta de disco degradado?
- [ ] Onde uma escrita crítica deixa de estar só em cache? Há flush explícito, bateria ou nobreak? (o flush explícito é complemento, não está no compêndio)
- [ ] O sistema de arquivos tem journal ou COW? Tem checksums? Há snapshot antes de cada mudança arriscada?
- [ ] Volumes de notebooks e de dados sensíveis estão criptografados, com a chave guardada em local seguro?
- [ ] A carga envolve milhares de arquivos pequenos? Dá para agregar em menos arquivos ou empacotar a transferência?
- [ ] O gargalo é a interface (SATA, USB, Ethernet de 1 Gb)? Fiz a conta em bytes (dividindo por 8)?
- [ ] A durabilidade do SSD (SLC/TLC/QLC) combina com o volume de escrita?
- [ ] Algum limite de formato vai morder: 4 GB por arquivo no FAT32, 2 TB ou 4 partições no MBR?
- [ ] Os dados de transporte (exFAT, pendrive) são copiados para o armazenamento definitivo logo depois?
- [ ] Permissões e metadados (ADS, resource fork, chmod) sobrevivem à cópia entre sistemas de arquivos diferentes?
- [ ] No WSL2/VM, o código fica dentro do sistema de arquivos Linux? As portas expostas conflitam com as do host?
- [ ] O tutorial ou receita que estou seguindo é de quando, e ainda vale para esta versão?
- [ ] Ao investigar uma falha, sei em qual camada estou olhando (app, SO, cache, sistema de arquivos, dispositivo, rede) e tenho evidência, não palpite?

## Frases-âncora
> "Pra ele só interessa: eu consigo ler e gravar blocos de bits? Então tá, pra mim é um drive." — cap-064

> "E resolução de dependências é um problema recorrente que vai acompanhar sua vida inteira de programador." — cap-065

> "Eu recomendo sempre só ter instalado o que realmente precisa usar." — cap-065

> "Em vez de tentar fazer de tudo, melhor fazer menos mas fazer direito." — cap-066

> "Eu disse que vai distribuir blocos e não arquivos." — cap-068

> "Você precisa entender o que está usando e não confiar numa abstração." — cap-069

> "A pior coisa que um programador pode fazer é acreditar em superstições porque não entende como as coisas funcionam." — cap-069

> "Se você não sabe que precisa de compressão então não precisa de compressão." — cap-071

## Onde aprofundar

| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-064 | [Akitando] #127 - Como Funciona o Boot de um Linux? \| O que tem num LiveCD? (12/09/2022) | https://www.akitaonrails.com/2022/09/12/akitando-127-como-funciona-o-boot-de-um-linux-o-que-tem-num-livecd/ | Precisar diagnosticar máquina que não boota, entender UEFI/GRUB/initramfs, SysV × systemd, `/proc` e `/dev`, ou montar imagens de disco |
| cap-065 | [Akitando] #128 - Entendendo Pacotes com Slackware \| Deb, Apt, Tarbals (25/09/2022) | https://www.akitaonrails.com/2022/09/25/akitando-128-entendendo-pacotes-com-slackware-deb-apt-tarbals/ | Precisar entender o que tem num pacote, como apt/dpkg resolvem dependências, ou compilar do tarball |
| cap-066 | [Akitando] #60 - Entendendo WSL 2 \| E uma curta história sobre Windows NT (04/09/2019) | https://www.akitaonrails.com/2019/09/04/akitando-60-entendendo-wsl-2-e-uma-curta-historia-sobre-windows-nt/ | Comparar compatibilidade de código × binária, tradução de syscalls × VM, microkernel × kernel monolítica (detalhes de instalação datados) |
| cap-067 | [Akitando] #114 - O Melhor Setup Dev com Arch e WSL2 (15/02/2022) | https://www.akitaonrails.com/2022/02/15/akitando-114-o-melhor-setup-dev-com-arch-e-wsl2/ | Montar ambiente de dev em WSL2/Arch, entender o desempenho de `/mnt/c` × VHDX, `wsl --mount`, asdf e Docker Desktop |
| cap-068 | [Akitando] #99 - Quebrei 3 HDs: Entendendo Armazenamento (09/06/2021) | https://www.akitaonrails.com/2021/06/09/akitando-99-quebrei-3-hds-entendendo-armazenamento/ | Decidir RAID, entender por que o RAID-0 é perigoso, clone raw, CHS→LBA, setores e caches de escrita |
| cap-069 | [Akitando #101] Tudo que Você Queria Saber Sobre Dispositivos de Armazenamento (06/07/2021) | https://www.akitaonrails.com/2021/07/06/akitando-101-tudo-que-voce-queria-saber-sobre-dispositivos-de-armazenamento/ | Precisar de MBR × GPT, ESP e Secure Boot, partição × volume, RAID 1/10/5, DAS/NAS/SAN, USB/NVMe/Ethernet, desgaste de SSD, block × raw device |
| cap-070 | [Akitando] #102 - Entendendo Sistemas de Arquivos: FAT (20/07/2021) | https://www.akitaonrails.com/2021/07/20/akitando-102-entendendo-sistemas-de-arquivos-fat/ | Entender o custo de metadados, clusters, limites do FAT, bitvectors, nomes longos e forks/ADS |
| cap-071 | [Akitando] #103 - Todos os Sistemas de Arquivos: de FAT a ZFS (03/08/2021) | https://www.akitaonrails.com/2021/08/03/akitando-103-todos-os-sistemas-de-arquivos-de-fat-a-zfs/ | Escolher entre ext4, btrfs, ZFS, APFS, NTFS e exFAT; entender inode/VFS, journaling, checksums, copy-on-write, snapshots, criptografia e safe delete |
