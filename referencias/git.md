# Git e controle de versão
> Destilado de: cap-022, cap-023, cap-024, cap-025, cap-026 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).
> Datas: cap-023 (2008) e cap-024 (2009) usam sintaxe antiga (`checkout` para tudo, branch `master`, protocolo `git://`, diretório `.dotest/`), e o texto publicado perdeu traços de várias opções (`-hard`, `commit m`); cap-022 e cap-025 são de 2020; cap-026 é de 2023. Os equivalentes modernos aparecem marcados como complemento.

## Ideia central
O Git é um banco de dados de objetos endereçados pelo hash do próprio conteúdo, ligados num grafo acíclico (DAG) de commits; branch é só uma etiqueta e o merge de três vias usa o ancestral comum, por isso branch e merge são baratos, rápidos e funcionam offline. A função do repositório é guardar um registro íntegro e legível da história: reescreva à vontade o que só existe na sua máquina ou no seu branch, nunca o que outros já baixaram. Quase nada que entrou num commit se perde (as referências mudam, os objetos ficam até o garbage collector), mas o que nunca foi commitado não tem volta: commit cedo, remoto fora da máquina e backup em camadas. Git não é GitHub; o fluxo centralizado com pull requests é uma convenção conveniente, não um requisito.

## Conceitos que todo sistema depende

### Por que controle de versão distribuído
- **O problema original** — sem versionamento você sobrescreve o código de ontem, apaga o arquivo errado, enche o diretório de cópias e, numa pasta de rede compartilhada, um salva por cima do outro. Bloquear arquivos (o checkin/checkout exclusivo do SourceSafe e do ClearCase) não escala: alguém trava o arquivo e vai embora, e ainda assim não há histórico. [cap-022]
- **diff e patch** — `diff` encontra a maior subsequência comum (LCS) entre dois arquivos (feito de forma ingênua o problema explode; com o algoritmo certo fica em m×n) e gera um patch com hunks marcados com `-`/`+` e números de linha; `patch` aplica o remendo. Guardar e transmitir só a diferença economiza disco e banda, e foi assim (tarball no FTP + patches por e-mail) que o kernel do Linux foi desenvolvido nos anos 90. [cap-022]
- **tar e compressão** — tar ("tape archive") gruda vários arquivos um atrás do outro num só; compressão pode ser com perda (JPEG, MP3) ou sem perda (zip, gzip, cujo deflate combina Lempel-Ziv e Huffman). Os objetos do Git são gravados comprimidos com zlib/deflate. [cap-022]
- **Centralizado vs distribuído** — em CVS/SVN (cliente-servidor) commit significa enviar ao servidor, então sem rede não há commit, e o CVS ainda versionava arquivo por arquivo. Em DVCS (BitKeeper, Darcs, Mercurial, Bazaar, Git) cada clone é um repositório completo: commit local e offline, push quando houver rede. A novidade real do BitKeeper foi tornar branch, fork e principalmente merge operações triviais; o Git venceu os concorrentes por desempenho (é escrito em C) e integridade. [cap-022, cap-023]

### Modelo interno
- **Banco de objetos endereçado por conteúdo** — `.git/objects` guarda blobs (só o conteúdo), trees (nomes, modos como `100644`/`755` e estrutura de diretórios) e commits (tree + commit pai + autor + data + mensagem). O ID é o SHA-1 de um cabeçalho com tipo e tamanho seguido do conteúdo, e o objeto é gravado comprimido; os 2 primeiros caracteres do hash viram subdiretório para não estourar o limite de arquivos por diretório. O mesmo hash é nome do objeto e verificação de integridade: mudou um bit, mudou o ID. [cap-022]
- **Plumbing e porcelain** — `hash-object`, `cat-file`, `update-index`, `write-tree` e a criação manual de commits (encanamento) fazem o que `add` e `commit` (porcelana) escondem. O formato é aberto e simples, por isso existem bibliotecas de Git em qualquer linguagem e serviços como GitHub e GitLab. [cap-022]
- **A working tree é um espelho** — os arquivos que você edita podem ser recriados a partir de `.git`; dá para apagar tudo menos `.git` e recuperar sem perguntar nada a servidor algum. Por isso trocar de branch no mesmo diretório é instantâneo, e há um único `.git` na raiz (contra `.cvs`/`.svn` em cada subpasta). [cap-022, cap-023]
- **Conteúdo primeiro, metadado depois** — o Git rastreia conteúdo, não arquivos: como o nome fica na tree e o conteúdo no blob, ele detecta renomeação por similaridade (`rename LICENSE => licenca.txt (94%)`) e mescla o caso "renomeei num branch, editei no outro", que no SVN/CVS dava conflito. Diretório vazio não entra no repositório (daí o `.gitkeep`). [cap-022, cap-023, cap-025]
- **Commit é snapshot, não patch** — o autor descreve o commit como "o patch desde o commit anterior" por didática, e o próprio cap-025 ajusta o quadro ao dizer que o Git grava o arquivo inteiro a cada modificação (por isso binário incha o repositório). (complemento, não está no compêndio: cada commit aponta para uma tree completa; diffs são calculados na hora e deltas só aparecem na compressão dos packfiles.) [cap-022, cap-025]
- **DAG e cadeia de hashes** — cada commit guarda o SHA-1 do pai; mudar qualquer commit antigo muda o hash dele e de todos os descendentes. Reescrever história, portanto, cria commits novos, e quem tinha os antigos fica divergente. [cap-022, cap-025]
- **Branch é etiqueta; HEAD é onde você está** — branch é um rótulo móvel apontando para um commit; HEAD é o commit atual; `HEAD~N` anda N commits para trás (com merges no caminho a notação fica menos intuitiva, estude na documentação). Um prefixo curto do SHA basta para identificar o commit. (complemento, não está no compêndio: `HEAD^2` é o segundo pai de um commit de merge, não "dois commits atrás" como diz o cap-023; para andar para trás use `~`.) [cap-023, cap-025]
- **Merge de três vias** — além das duas pontas, o Git consulta o ancestral comum (onde o branch nasceu) para saber quem mudou o quê e decidir sozinho na maioria dos casos; um merge de duas vias (CVS) para em cada linha divergente. O commit de merge vira o novo ancestral, então o próximo merge não reabre os mesmos conflitos (o SVN só ganhou merge tracking na 1.5, em 2008). Conflito real só quando os dois lados mudam a mesma linha; se não houve divergência, o merge é fast-forward. [cap-022, cap-023]
- **Rebase** — "rebobina" o seu branch até o ponto onde ele nasceu, aplica os commits novos do branch base e reaplica os seus por cima, deixando o histórico linear; os seus commits ganham hashes novos. Conflitos aparecem commit a commit e se resolvem editando, `git add` e `git rebase --continue` (ou `--skip`/`--abort`). [cap-022, cap-023]
- **Três estágios** — working tree (onde você edita) → stage/index (onde estaciona o que vai no commit) → commit. Só o que está no index entra; `git add -i` escolhe arquivos, `git add -p` escolhe hunks (com `s` para dividir um hunk), `git diff --staged` mostra o que vai no próximo commit. [cap-024, cap-025]
- **Remoto é só configuração** — um clone difere de um repositório criado localmente por algumas linhas em `.git/config` (`remote origin`) apontando para HTTPS, SSH ou até outro diretório. Dá para clonar de uma pasta, de um pendrive ou do notebook de um colega via SSH; `git clone --bare` cria um repositório sem working tree, próprio para servir de remoto. [cap-022, cap-023, cap-026]

### Recuperação e limpeza
- **Reset move etiquetas; o reflog é a lixeira** — `reset` só reaponta o branch; os commits "apagados" continuam alcançáveis pelo `git reflog` (o histórico de para onde o HEAD andou) até o `git gc` coletá-los. Recupere logo depois do erro. (complemento, não está no compêndio: entradas do reflog expiram, por padrão 90 dias e 30 para commits inalcançáveis, e o reflog é local, não vem num clone.) [cap-024, cap-025]
- **O que nunca foi commitado não volta** — reflog e gc só valem para objetos que entraram no repositório; arquivo novo apagado antes do commit, ou mudança descartada com `reset --hard` ou `clean`, está perdido para o Git. [cap-024]
- **Binários e Git LFS** — não existe delta eficiente de binário, então cada versão de um PSD de 200 MB entra inteira e todo clone baixa todas (5 commits = 1 GB). O Git LFS grava no commit um ponteiro de texto e o binário em `.git/lfs` e no servidor, baixando só a versão necessária; vale apenas para commits feitos depois de ativado. Projeto que é quase só mídia não deve usar versionador de código: mídia tem fluxo em cascata e pede servidor de arquivos rápido. [cap-025]
- **Segredo commitado continua no histórico** — apagar a senha do arquivo não a remove dos commits antigos; é preciso reescrever a história (BFG), expirar o reflog, rodar `gc`, dar push e todo mundo clonar de novo. (complemento, não está no compêndio: trate a credencial como comprometida e rotacione-a; hoje o próprio projeto Git recomenda `git filter-repo` no lugar do `filter-branch`.) [cap-025]
- **Higiene do histórico é profissionalismo** — mensagens descritivas, um assunto por commit, nada de "fixing" no branch principal. Sujeira mergeada vira débito técnico, que cobra juros crescentes. [cap-025]

### Fluxos de trabalho
- **O GitHub-flow é uma simplificação centralizada** — repositório principal, forks ou branches e pull requests de volta resolvem a grande maioria dos projetos. O kernel do Linux usa uma malha: dezenas de repositórios (o do Linus só ele usa; distros e fabricantes têm os seus), discussão em mailing lists, patches enviados por e-mail com cópia para vários repositórios e o arquivo `MAINTAINERS` para achar o responsável, algo que o modelo fork→PR de um repositório só não suporta. [cap-022, cap-025]
- **Monorepo resolve problema de gigante** — Google (bilhões de linhas, mais de 80 TB, dezenas de milhares de engenheiros) e Twitter juntam tudo num repositório e vendorizam dependências para gerar deploys coerentes; o Go nasceu dessa necessidade. Para times de 10 a 20 pessoas isso não se aplica. [cap-025]
- **Git convive com sistemas legados** — o `git-svn` permite trabalhar em Git local num projeto SVN (`git svn clone`, `git svn rebase` para atualizar, `git svn dcommit` para publicar), e há pontes parecidas para Mercurial, Bazaar, Perforce e TFS. Também serve para migrar de vez preservando o histórico. [cap-022, cap-025, cap-026]

### Proteção de dados além do Git
- **Backup em camadas** — Git com commits frequentes e remoto; snapshots locais (BTRFS + Timeshift) para desfazer acidentes e upgrades quebrados; backup externo para quando o disco morre. Snapshot depende do mesmo SSD e não substitui backup fora dele; backup sincronizado sozinho não protege de ransomware, porque copia os arquivos já criptografados por cima — são as versões antigas (snapshots) que salvam. Segurança funciona em camadas: plano A, B e C. [cap-026]
- **Copy-on-write é "Git no sistema de arquivos"** — em ZFS/BTRFS um arquivo editado ganha blocos novos sem sobrescrever os antigos, o que permite snapshots instantâneos (só metadados). Em contrapartida, o espaço não volta sozinho: snapshots seguram arquivos que você "apagou". [cap-026]
- **Apagar remove o índice, não os blocos** — o sistema de arquivos só tira a entrada do mapa; os blocos continuam no disco até algo escrever por cima. Por isso, depois de um acidente: parar de escrever, desmontar, tirar uma imagem bit a bit com `dd` para outro disco e só então tentar recuperar. [cap-026]
- **O superbloco é o mapa** — os primeiros bytes da partição dizem onde está cada bloco; sem eles, os dados existem mas não se remontam. O ext4 guarda várias cópias do superbloco (liste com `dumpe2fs <dev> | grep -i superblock`, ou simule com `mke2fs -n`), e `fsck -b <cópia>` tenta restaurar; sem superbloco, PhotoRec/TestDisk recuperam pelo cabeçalho de cada arquivo, perdendo nomes e diretórios. [cap-026]
- **Journaling e checksum** — journaling registra a operação antes de reapontar blocos, para concluir ou desfazer depois de uma queda de energia; checksum detecta corrupção silenciosa, mas só corrige se houver cópia redundante (RAID) e alguém rodar o scrubbing periódico, que vem desligado por padrão em NAS. (complemento, não está no compêndio: NTFS e ext4 não fazem checksum dos dados de usuário e o APFS só dos metadados; checksum de dados por bloco é coisa de ZFS, BTRFS e ReFS.) [cap-026]

## Heurísticas de decisão
| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Começar qualquer projeto, mesmo pessoal ou sem servidor | `git init` e commits pequenos e frequentes desde o primeiro dia | Git independe de GitHub e protege contra apagar ou sobrescrever | cap-026 |
| Fim do dia com trabalho pela metade | Commit "feio" agora; ao terminar, `git commit --amend` com a mensagem boa | Commit feio é melhor que perder o arquivo | cap-026 |
| Trabalhando no seu branch | Commits à vontade; antes do merge ou PR, squash (`git rebase -i HEAD~N`, trocando `pick` por `squash`) e mensagens decentes | O histórico do branch principal é o registro do projeto | cap-025, cap-026 |
| Trazer novidades da base para o seu branch | Rebase da base no seu branch; levar o trabalho de volta com merge ("do master pro meu branch, rebase; do meu branch pro master, merge") | Conflitos são resolvidos no seu branch e o merge final sai limpo | cap-022, cap-023 |
| Commit já publicado ou recebido do remoto | Nunca reescrever (sem rebase, reset ou amend); desfazer com `git revert <sha>`, que cria um commit inverso | Quem já baixou fica com história divergente | cap-024, cap-026 |
| Desfazer commit local ainda não publicado, mantendo o conteúdo | `git reset --soft HEAD~N` e commitar de novo | As mudanças voltam para o stage | cap-024, cap-025 |
| Jogar fora tudo o que não foi commitado | `git reset --hard` (arquivos rastreados) + `git clean -d -f` (arquivos novos); é irreversível para o que não foi commitado | Reflog não guarda o que nunca entrou no repositório | cap-024 |
| Descartar a edição de um arquivo ou restaurar arquivo apagado | Nos textos: `git checkout <arquivo>`; hoje: `git restore <arquivo>` (complemento, não está no compêndio) | O conteúdo do último commit está em `.git` | cap-024, cap-026 |
| Tirar arquivo do stage | `git reset -- <arquivo>` (ou `git reset HEAD <arquivo>`); hoje também `git restore --staged <arquivo>` (complemento, não está no compêndio) | Desfaz o `add` sem perder a edição | cap-024, cap-025 |
| Criar branch, inclusive a partir de commit antigo | Nos textos: `git checkout -b nome [sha]`; hoje: `git switch -c nome [sha]` e `git switch nome` para trocar (complemento, não está no compêndio) | O branch nasce com o ponto de origem registrado | cap-023, cap-024 |
| Mensagem errada no último commit | `git commit --amend` na hora, antes do push | Corrigir no minuto em que notou custa 2 segundos | cap-024, cap-025 |
| Mudanças não relacionadas misturadas | Commits separados com `git add <arquivo>`, `git add -i` ou `git add -p`; conferir com `git status` e `git diff --staged` | Um commit deve conter uma coisa só | cap-023, cap-025 |
| Usar `git commit -a` ou `git add .` | Só sabendo o efeito: `-a` inclui modificados já rastreados mas não arquivos novos; `add .` pode levar logs e sujeira | Commits acidentais poluem o histórico | cap-023, cap-024 |
| Interromper trabalho pela metade para outra tarefa | `git stash`, depois `git stash apply` + `git stash clear` (texto de 2008); hoje prefira `git stash pop`, porque `stash clear` apaga todos os stashes (complemento, não está no compêndio) | Guarda o trabalho sem commit | cap-023 |
| Apagou commit ou branch por engano | Agir já: `git reflog`, achar o SHA e reapontar (`git merge <sha>` no cap-024); para criar branch de resgate use `git branch resgate <sha>` (complemento, não está no compêndio: o `git reflog -b` citado no cap-025 não existe) | O commit continua lá até o gc | cap-024, cap-025 |
| Reescrever histórico remoto (exceção) | Só em repositório controlado com time conhecido: avisar todos, ensaiar em repositório de teste, ter backup, `git push --force` e cada um faz `git pull --rebase`; hoje prefira `--force-with-lease` (complemento, não está no compêndio) | Reescrever não é o problema; fazer sem saber é | cap-024 |
| Arquivos binários grandes (PSD, vídeo, instaladores, PDFs) | Git LFS desde o início: `git lfs install` e `git lfs track "*.psd"` (gera `.gitattributes`) | Sem delta, cada versão incha todos os clones | cap-025 |
| Repositório já inchado ou segredo commitado | Parar o time, garantir que todos deram push, `git clone --mirror`, BFG (por nome ou por tamanho de blob), expirar reflog, `gc`, push e todos clonam de novo; rotacionar o segredo (complemento, não está no compêndio) | LFS novo não limpa o passado; histórico guarda a senha | cap-025 |
| GitHub fora do ar ou sem servidor | Remoto alternativo: `git clone --bare` num pendrive, ou servir o bare por HTTP (`git --bare update-server-info`, ativar o hook `post-update`, `python -m http.server`) via VPN ou ngrok | O código não depende do serviço; perde-se só a conveniência | cap-026 |
| Clonar seguindo tutorial antigo | Trocar URLs `git://` por HTTPS ou SSH (complemento, não está no compêndio: o GitHub desativou o `git://` não autenticado em 2022) | Protocolo antigo, sem autenticação nem criptografia | cap-023 |
| Empresa obriga SVN ou outro legado | `git-svn` localmente (`git svn clone`, `git svn rebase`, `git svn dcommit`) | Branches locais, commits pequenos e squash antes de publicar | cap-022, cap-025 |
| Projeto de cliente sendo freelancer | Pagar hospedagem privada (GitHub, GitLab) ou manter GitLab próprio | Não ter remoto é "economia porca" | cap-022, cap-026 |
| Pensar em monorepo ou no fluxo do kernel | Só com problema e escala equivalentes; o padrão é repositório principal + branches/forks + PR | Copiar gigante sem o problema dele é fanboyismo | cap-025 |
| Revisar pull request | Rejeitar o que está quebrado, sem testes, misturado ou com mensagens ruins | Depois de mergeado, ninguém arruma | cap-025 |
| Descobrir quem mudou uma linha e por quê | `git blame <arquivo>` e seguir o commit | Contexto antes de refatorar | cap-024 |
| Proteger os dados da máquina de dev | Git + snapshots (BTRFS/Timeshift, com autosnap antes de `apt upgrade`) + HD externo com o backup nativo do SO | Três níveis de segurança | cap-026 |
| Snapshots enchendo o disco | Tirar `/var/lib/docker`, `/var/lib/libvirt`, `/var/log` dos snapshots (o texto usa `chattr +C`); depois de apagar snapshots, rodar `btrfs filesystem balance start /` | Snapshots seguram imagens e logs apagados; o espaço não volta sozinho | cap-026 |
| Disco não monta ou não dá boot | Parar; não rodar `fdisk` nem `mkfs`; bootar por pendrive (live); imagem com `dd` para outro disco; `fsck -b` com superbloco reserva; PhotoRec/TestDisk; profissional se o dado for valioso; descartar o hardware depois | Qualquer escrita pode destruir os blocos restantes | cap-026 |
| Dúvida sobre um comando | `git help <comando>` e a documentação oficial (Pro Git, capítulo Git Internals) | A fonte primária explica o modelo, não só o atalho | cap-022, cap-024 |

## Armadilhas e mitos
- Git é GitHub → Git é uma ferramenta local; GitHub é um serviço opcional que precisa do Git, não o contrário. [cap-022, cap-026]
- Sem internet não dá para commitar → isso valia para o SVN; no Git o commit é local e o push vem depois. [cap-022]
- O difícil é criar branch → criar sempre foi fácil; o difícil era mergear, e é isso que o merge de três vias resolveu. [cap-022]
- O Git faz merge mágico e nunca dá conflito → quando os dois lados mudam a mesma linha, alguém precisa decidir. [cap-023]
- Travar arquivos evita problemas → não escala, trava o time e não cria histórico. [cap-022]
- Commit é um diff guardado → é um snapshot apontando para uma tree completa; por isso binários incham. (complemento, não está no compêndio, com base no próprio cap-025) [cap-022, cap-025]
- `reset --hard` apagou o commit para sempre → o commit fica no reflog até o gc; só se perde o que nunca foi commitado. [cap-024, cap-025]
- É melhor commitar só quando estiver tudo pronto → um acidente apaga o dia; commit parcial agora, `--amend` ou squash depois. [cap-026]
- "Depois a gente arruma" → depois de mergeado ninguém arruma; vira débito técnico com juros. [cap-025]
- Apaguei a senha do arquivo, então está resolvido → o histórico guarda; é preciso reescrever e (complemento, não está no compêndio) rotacionar a credencial. [cap-025]
- Ativar LFS agora resolve o repositório pesado → LFS vale só para commits novos; o passado exige reescrever com BFG. [cap-025]
- "Se o Google usa monorepo, devemos usar" → você não é o Google; adote solução do tamanho do seu problema. [cap-025]
- O kernel não está no GitHub por birra → além de não confiar a cópia mestra a terceiros, o fluxo em malha com patches multi-destino não cabe no modelo fork→PR. [cap-025]
- `HEAD^2` significa dois commits atrás, como diz o tutorial de 2008 → significa o segundo pai de um merge; dois commits atrás é `HEAD~2`. (complemento, não está no compêndio) [cap-023]
- Snapshot é backup → snapshot mora no mesmo disco; se o SSD morre, só o backup externo salva. [cap-026]
- Backup externo protege de ransomware → o backup pode copiar por cima os arquivos já criptografados; é preciso versões antigas (snapshots, histórico). [cap-026]
- Apagar snapshots do BTRFS libera espaço na hora → segundo o autor, é preciso rodar `balance` para reclamar o espaço; e o `df` comum pode mostrar valores errados (use `btrfs filesystem df /`). [cap-026]
- `docker compose down` limpa tudo → containers parados seguram imagens, e snapshots podem guardar cópias; use `docker system prune` e `docker container ls -a`. (complemento, não está no compêndio: a opção correta é `docker compose down --rmi all`; o texto escreve `down rmi --all`.) [cap-026]
- `chattr +C` tira um diretório dos snapshots → (complemento, não está no compêndio) `+C` desliga copy-on-write e checksum só para arquivos novos; o jeito garantido de excluir um diretório dos snapshots é transformá-lo em subvolume separado. [cap-026]
- `rm -rf /` apaga a máquina toda → há muito tempo o `rm` exige `--no-preserve-root` (ou `/*`) para isso. [cap-026]
- Disco corrompido: é só rodar ferramentas até funcionar → cada escrita no disco danificado pode destruir o que restava; primeiro a imagem com `dd`. [cap-026]

## Checklist para aplicar num sistema real
- [ ] Todo projeto (incluindo scripts, infraestrutura e configs) está num repositório Git com pelo menos um remoto fora da máquina? [cap-026]
- [ ] Existe cópia dos dados fora do disco (remoto + backup externo) e existem versões antigas (snapshots/histórico) para recuperar de ransomware? [cap-026]
- [ ] O branch principal (`main`/`master`) está protegido contra reescrita e só recebe merge revisado? [cap-024, cap-025]
- [ ] A regra do time está escrita: rebase/squash/amend só em branch próprio; `git revert` no que é compartilhado? [cap-022, cap-024, cap-026]
- [ ] Os commits são atômicos (uma coisa por commit) e têm mensagens descritivas antes de chegar ao branch principal? [cap-025]
- [ ] Pull requests quebrados, sem testes ou misturados são rejeitados? [cap-025]
- [ ] Logs, artefatos de build, dependências e arquivos de ambiente estão fora do stage por padrão? (complemento, não está no compêndio: via `.gitignore`; o compêndio só alerta contra `git add .` levar logs e sujeira) [cap-023]
- [ ] Nenhum segredo existe no histórico? Se já existiu, houve reescrita com BFG/filter-repo e rotação da credencial? [cap-025]
- [ ] Binários grandes passam pelo Git LFS desde o primeiro commit? [cap-025]
- [ ] Integrações entre branches usam merge/rebase do Git (com ancestral comum), nunca cópia de diretórios ou patch manual? [cap-022]
- [ ] Antes de operação destrutiva (`reset --hard`, `clean`, force push, BFG) todos deram push, existe mirror/backup e o procedimento foi ensaiado num repositório de teste? [cap-024, cap-025]
- [ ] O time sabe recuperar commits pelo reflog e entende que isso vale só para o que foi commitado e só até o gc? [cap-024, cap-025]
- [ ] Existe plano B se o GitHub/GitLab cair (bare clone, remoto alternativo, servidor HTTP simples)? [cap-026]
- [ ] Monorepo ou fluxos exóticos são justificados por um problema real de escala, não por imitação? [cap-025]
- [ ] Tutoriais e scripts internos usam sintaxe atual (`switch`/`restore`, `main`, HTTPS/SSH em vez de `git://`, `--force-with-lease`)? (complemento, não está no compêndio) [cap-023, cap-024]
- [ ] A verificação de integridade confia nos hashes, e todos entendem que reescrever um commit muda o ID de todos os descendentes? [cap-022, cap-025]
- [ ] Dados fora do Git (NAS, servidores de arquivo) têm checksum por bloco, redundância e scrubbing agendado (vem desligado)? [cap-026]
- [ ] Diretórios de alta rotatividade (Docker, VMs, logs, caches) estão fora dos snapshots, e há rotina de limpeza e rebalanceamento? [cap-026]
- [ ] Existe procedimento escrito para disco que não monta: parar, imagem com `dd`, só então recuperar? [cap-026]
- [ ] Armazenamento usado para código e dados de trabalho é de marca confiável (nunca SSD genérico ou SD card)? [cap-026]

## Frases-âncora
> "O repositório git é um banco de dados de objetos." — cap-022

> "Mas o problema nunca foi criar branches, o problema sempre foi mergear os branches." — cap-022

> "Commits que você já enviou ao repositório remoto, ou commits que recebeu de lá, nunca devem ser modificados." — cap-024

> "Reescrever o histórico no Git não é problemático, fazer sem saber o que está fazendo, é." — cap-024

> "Commits devem ter só uma "coisa" de cada vez." — cap-025

> "Fanboyismo é o oposto de engenharia. Engenharia é encontrar a solução do tamanho certo pro problema certo em mãos, dentro das restrições do seu contexto." — cap-025

> "Git é uma ferramenta, GitHub é um serviço. GitHub precisa de Git, mas Git não precisa de GitHub." — cap-026

> "Melhor um commit feio agora do que ter um acidente e perder o arquivo." — cap-026

## Onde aprofundar
| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-022 | [Akitando] #70 - Entendendo GIT \| (não é um tutorial!) | https://www.akitaonrails.com/2020/02/05/akitando-70-entendendo-git-nao-e-um-tutorial/ | precisar explicar o modelo interno (blobs, trees, commits, SHA-1, DAG), merge de três vias, rebase vs merge ou por que DVCS venceu CVS/SVN. |
| cap-023 | Micro Tutorial de Git | https://www.akitaonrails.com/2008/04/02/micro-tutorial-de-git/ | quiser um passo a passo do fluxo diário (branch, stash, rebase com conflito, merge com renomeação, clone de clone); traduza os comandos para a sintaxe atual. |
| cap-024 | Dicas de Git | https://www.akitaonrails.com/2009/07/05/dicas-de-git/ | precisar desfazer algo (checkout, reset soft/hard, clean, revert), recuperar commit com reflog, forçar push com segurança, usar blame ou amend; sintaxe de 2009. |
| cap-025 | [Akitando] #71 - Usando Git Direito \| Limpando seus Commits! | https://www.akitaonrails.com/2020/02/12/akitando-71-usando-git-direito-limpando-seus-commits/ | for limpar histórico (amend, squash, add -p), lidar com binários (LFS), remover segredos/arquivos grandes (BFG), usar git-svn ou discutir monorepo e o fluxo do kernel. |
| cap-026 | [Akitando] #146 - Protegendo e Recuperando Dados Perdidos - Git, Backup, BTRFS | https://www.akitaonrails.com/2023/10/19/akitando-146-protegendo-e-recuperando-dados-perdidos-git-backup-btrfs/ | for desenhar a estratégia de proteção de dados (Git + snapshots + backup), servir Git sem GitHub, recuperar partição/superbloco ou entender copy-on-write, checksums e ransomware. |
