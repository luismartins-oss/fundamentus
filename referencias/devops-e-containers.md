# DevOps, deploy e containers

> Destilado de: cap-072, cap-073, cap-074, cap-075, cap-076 (Compêndio de Programação, textos de Fabio Akita, CC BY-NC-SA 4.0).
> Datas: história de infra e DevOps em 2019 (cap-072, cap-073); Heroku e 12 fatores em 2022 (cap-074, cap-075); containers por dentro em 2023 (cap-076). Heroku (planos, tamanhos de dyno, preços), Docker Machine, "Kubernetes roda Docker" e comandos de ferramentas são datados. Os princípios (infra como código, build imutável, stateless, config no ambiente, descartabilidade, deploy contínuo, container = processo isolado pela kernel) continuam valendo sem Heroku.

## Ideia central
DevOps não é cargo, ferramenta nem metodologia isolada: é o resultado de tratar infraestrutura como software, com máquinas descartáveis recriadas por código, deploy a cada `git push` gerando build imutável e rollback em segundos. Container não é máquina virtual, e sim um processo nativo que a kernel Linux engana (cgroups limitam recursos, namespaces escondem o resto do sistema, capabilities cortam privilégios) e que compartilha a mesma kernel; VM é outra kernel sobre hardware falso. Os 12 fatores (dependências declaradas, config no ambiente, backing services, app stateless, logs em stdout, build/release/run separados) são o mínimo para escalar horizontalmente, mas não garantem escala: o gargalo costuma ser o banco e a arquitetura. A qualidade vem de testes, integração e deploy contínuos e da comunicação do time, não da ferramenta da moda: "você não é o Netflix".

## Conceitos que todo sistema depende

### Virtualização × containers
- **DevOps**: termo difícil de definir, e o autor o compara a "Agile". Não é título, ferramenta, metodologia nem cultura isoladamente, e se entende pelos problemas que resolve: provisionar, configurar, atualizar, escalar e recriar infra com confiabilidade e segurança, que são o mais importante em infra. [cap-072]
- **Hypervisor**: é o "supervisor de supervisores", um programa que responde às instruções privilegiadas mentindo para a kernel convidada, que acredita estar numa máquina real. O tipo 1 roda direto no hardware (vSphere, XenServer, Hyper-V); o tipo 2 roda sobre um SO (VirtualBox, VMware Workstation, Parallels, QEMU). VM tem o peso de dois SOs inteiros. [cap-072]
- **Virtualização completa, paravirtualização e VT-x/AMD-V**: na completa, a kernel convidada não sabe que está numa VM. Na paravirtualização (Xen), a kernel é modificada para cooperar com o hypervisor. As extensões de CPU de 2005–2006 (cerca de 10 instruções novas) puseram o processador no "conluio" e aproximaram a VM do desempenho nativo. [cap-072]
- **Linhagem do container**: chroot, FreeBSD jails (2000), Linux VServer (2001), zonas do Solaris (2004) e OpenVZ, os "process containers" do Google (2006, que viraram os cgroups), LXC (2008) e por fim Docker (2013). Em todos, o mecanismo é a kernel restringir e mentir para os processos sobre CPUs, RAM, processos visíveis e sistema de arquivos. [cap-072, cap-073]
- **VM × container (o TL;DR do autor)**: a VM roda uma kernel nova, até de outra arquitetura (x86 sobre ARM). O container reaproveita a mesma kernel, na mesma arquitetura. Por isso o processo do container aparece no `ps` do host, o `/boot` dentro dele está vazio e não dá para rodar Windows ou BSD dentro de um container Linux. [cap-076, cap-073]
- **cgroups**: mecanismo da kernel que agrupa processos numa árvore e limita CPU, memória e outros recursos. Todo processo já pertence a um cgroup (o systemd organiza em `system.slice`, `user.slice`, `init.scope`; `cat /proc/PID/cgroup` mostra qual), e tudo fica exposto em `/sys/fs/cgroup` (tipo `cgroup2`). Exemplo do autor: um slice com `CPUQuota=30%` e `systemd-run --slice=...` prende um loop infinito a 30% de CPU. Serve para que um processo não sature a máquina (DoS) e deixe a kernel sem fôlego. [cap-076]
- **namespaces**: dão ao processo uma visão própria de PIDs (ele vira o PID 1), mounts, rede, IPC, UTS (hostname), usuários e cgroups. `unshare --fork --pid --mount-proc` cria o namespace, `nsenter` entra num existente, e com `--user` o "root" de dentro é "nobody" fora. Com namespaces de rede, dois containers podem ouvir na porta 80 e ser expostos como 4001 e 4002, ou dividir uma rede virtual só entre eles. [cap-076, cap-073]
- **rootfs e userland**: uma distro sem kernel é só userland. Montar o userland do Ubuntu como raiz (chroot ou equivalente) faz o container "ser Ubuntu" sobre a kernel de um Fedora. Daí vêm as imagens enxutas (Alpine, CoreOS) e a regra de um único processo por container (um para o Postgres, outro para o Redis, outro para o NGINX). [cap-073, cap-076]
- **Linux capabilities**: o poder do root dividido em capacidades: `CAP_NET_BIND_SERVICE` (portas abaixo de 1024), `CAP_KILL`, `CAP_AUDIT_WRITE` e outras. Há os conjuntos efetivo, permitido, herdado e o bounding set, guardados como bitmask em `/proc/PID/status`. Ferramentas: `capsh`, `setcap`, `man capabilities`. Um container recebe só as capabilities que precisa. [cap-076]
- **seccomp, AppArmor, SELinux e Bubblewrap**: o seccomp filtra syscalls; AppArmor e SELinux impõem políticas de acesso a arquivos, dispositivos, rede e webcam; o Bubblewrap é um container mais simples, com namespaces sem privilégio. Juntos com o OSTree formam o Flatpak (e o Snap, que é parecido), "primos" do Docker para aplicativos de desktop. [cap-076]
- **Docker em Windows e Mac**: essas kernels não têm cgroups, namespaces nem union filesystem. Primeiro sobe uma VM Linux (Hyper-V/WSL2, Hypervisor.framework, VirtualBox) e os containers rodam dentro dela, com o cliente falando com o daemon pela rede interna. Por isso é sempre mais pesado que em Linux nativo. [cap-073, cap-076]

### Imagens e runtimes
- **OCI e runC**: a Open Container Initiative (2015) padroniza o formato de imagem e o runtime. O runC executa um container a partir de um rootfs e de um `config.json` (gerado por `runc spec`): processo e argumentos, usuário, env, cwd, raiz read-only, hostname (UTS), mounts de `/proc`, `/dev`, `/dev/pts`, `/dev/shm`, `/dev/mqueue`, `/sys` e cgroup, a lista de namespaces, `maskedPaths`/`readonlyPaths`, capabilities, `rlimits` e `noNewPrivileges`. [cap-076, cap-073]
- **containerd × runC/Podman**: o autor diz que o Docker usa o containerd, que roda como daemon root (daí o `systemctl enable docker`), enquanto Podman e Kubernetes usam o runC. O Podman tem a mesma CLI e o mesmo Dockerfile (só o `FROM` leva o registro explícito, ex. `docker.io/debian`) e roda rootless. (complemento, não está no compêndio) Na prática o containerd também delega a execução ao runC. Desde a v1.24 (2022) o Kubernetes fala com containerd ou CRI-O via CRI, sem o Docker no meio, e o Docker também tem modo rootless. [cap-076]
- **Imagem = camadas copy-on-write**: uma imagem OCI é um pacote de camadas (`blobs` endereçados por hash, parecido com `.git/objects`) mais índice e configuração. Em execução, as camadas são unificadas por um union filesystem (overlay2). Cada instrução do Dockerfile gera uma camada, como um commit, e cada camada guarda só os blocos que mudaram. `docker export` mostra que no fundo é um rootfs num tar. [cap-076, cap-073]
- **O valor do Docker**: executar containers já é papel da kernel e do runC. O que o Docker trouxe foi o ecossistema de imagens (Docker Hub, com `latest` como uma espécie de branch `main`), a receita padronizada (Dockerfile) e a CLI que todo mundo copiou. [cap-076, cap-073]
- **Dockerfile**: receita declarativa, parente das "promessas" do CFEngine e dos cookbooks do Chef: `FROM` (partir de uma imagem existente, como um `git clone`), `ENV`, `RUN`, `COPY`, `WORKDIR`, `EXPOSE`, `CMD`. O `docker build` reaproveita do cache todas as camadas até a primeira linha alterada. [cap-073, cap-076]

### Infra como código e nuvem (contexto histórico)
- **Hospedagem compartilhada e LAMP**: dezenas de sites num único Apache com virtual hosts (Linux, Apache, MySQL, PHP; painéis como cPanel; deploy por FTP). O mod_php era rápido, mas rodava no processo compartilhado do Apache, e um bug deixava um usuário chegar aos dados de outro. O FastCGI isolava por usuário ao custo de memória. É o dilema permanente entre desempenho e isolamento. [cap-072]
- **Telnet → SSH**: Telnet, FTP, SMTP e HTTP em texto aberto podem ser farejados na rede e foram trocados por SSH, SFTP, SMTP com TLS e HTTPS. O SSH também é túnel e canal de automação. [cap-072]
- **VPS e imagens**: VM alugada a partir de uma imagem pronta (snapshot do SO instalado), sem dor de cabeça com driver. Provisionar caiu de dias de ordem de serviço no data center para minutos com cartão de crédito. [cap-072, cap-073]
- **Gestão de configuração**: CFEngine (1993/1998, promessas declarativas), Puppet (2005) e Chef (2009) são cliente-servidor com agentes, bons para frotas grandes. Capistrano (2006), Fabric e Ansible (2015) não usam agente (o "agente" é o próprio servidor SSH), bons para poucas máquinas. Todos são infra como código: receitas versionáveis, repetíveis e compartilháveis. [cap-072]
- **AWS (2006) e IaaS**: EC2, S3, SQS e, logo depois, EBS (volumes de bloco anexáveis). A virada foi cobrar por uso (hora, MB) e controlar tudo por API, o que permitiu criar e destruir máquinas por programa e replicar em várias regiões. [cap-073]
- **Soft × hard**: o desenvolvedor quer mudar sempre; o sysadmin, depois de configurar, quer não mexer nunca mais. Infra programática permitiu ambientes de teste idênticos aos de produção e reduziu essa guerra. A burocracia ao estilo ITIL não aguentou o ritmo das startups. [cap-073]
- **IaaS, PaaS e SaaS**: a AWS inaugurou a IaaS, o Heroku a PaaS, e serviços como e-mail transacional, logs, Redis gerenciado e processamento de imagens são SaaS. Cloud privada (OpenStack, OpenShift, Cloud Foundry) atende quem precisa de perímetro fechado. [cap-073]
- **Máquina efêmera, estado em volume ou serviço**: a instância é tratada como volátil, e o que precisa persistir vai para um volume (EBS) ou um serviço gerenciado. Para crescer, troca-se a máquina e reanexa-se o volume. (complemento, não está no compêndio) Hoje a maioria das instâncias EC2 usa disco raiz EBS, que sobrevive a parar e reiniciar; efêmeros são o instance store e a instância terminada. O princípio continua. [cap-073]

### Deploy no estilo Heroku (datado: 2022, o fluxo é o modelo)
- **`git push` como deploy**: `heroku create` adiciona um remote Git. O `git push heroku main` dispara o build pela buildpack, que detecta a linguagem e instala dependências (é o "Dockerfile" do Heroku), gera uma imagem e troca os dynos antigos por novos. O Heroku (2008) é anterior ao Docker (2013), que o imitou. [cap-074, cap-073]
- **Dyno e escala**: dyno é o container do Heroku. O de 512 MB da época obrigava a fazer aplicações enxutas. `heroku ps:scale web=N` sobe N instâncias, e `web=0` tira a app do ar. O autoscale serve sobretudo para economizar, desligando instâncias de madrugada. [cap-074, cap-073]
- **Capacidade "de padeiro"**: requisições por segundo ≈ processos simultâneos × (1 s ÷ latência). Com 4 processos e 100 ms por requisição, cerca de 40 req/s por dyno. É teórico, porque recursos compartilhados (o banco) bloqueiam. [cap-074]
- **Load balancer e proxy reverso**: recebe na porta 80 e distribui (round robin, `least_conn`) para instâncias em IPs e portas internos, escondendo dos clientes quantas instâncias existem e onde estão. Com várias instâncias, `localhost` deixa de valer. [cap-074, cap-076]
- **Procfile**: declara os tipos de processo (`web`, `worker`/fila) e o comando de cada um; é o `CMD` do Heroku. Virou formato universal; `foreman`, `node-foreman` e `goreman` sobem a app localmente como em produção. [cap-074]
- **Manifesto e lockfile**: as dependências ficam declaradas (`package.json`, `Gemfile`, `composer.json`, `pom.xml`/`build.gradle`, `requirements.txt`, `go.mod`, `Cargo.toml`), e o lock (`package-lock.json`, `composer.lock`) registra as versões exatas instaladas, nunca editado à mão. Dev e build instalam exatamente o mesmo conjunto. [cap-074, cap-075]
- **Configuração e segredos**: localmente, `.env` (no `.gitignore`) mais `.env.example` versionado. Em produção, variáveis reais (`heroku config:set`), com convenções como `DATABASE_URL`. Token em variável de ambiente é aceitável: se alguém lê o ambiente do container, tudo já está exposto. O erro grave é segredo no Git. [cap-074]
- **Addons e backing services**: Postgres, Papertrail (logs de todos os dynos num lugar pesquisável) e Rollbar (alerta de erro) entram por URL ou credencial em variável de ambiente. O tamanho do banco precisa ser escolhido com cuidado, porque migrar de plano não é trivial. [cap-074]
- **Container one-off**: `heroku run bash` e `heroku pg:psql` sobem um container novo com a mesma imagem, fora do balanceador, destruído ao sair. Serve para tarefas administrativas; o que se roda no banco é permanente. [cap-074, cap-075]
- **Releases e rollback**: cada push vira uma release numerada (`heroku releases`), e `heroku releases:rollback vN` volta para a última que funcionava em vez de consertar no sufoco. [cap-074]
- **Migrations**: scripts versionados que criam e alteram tabelas e índices, executados no deploy. O console manual do tutorial é só atalho didático. [cap-074]
- **Heroku hoje** (complemento, não está no compêndio): o plano gratuito e o `hobby-dev` acabaram em novembro de 2022. O fluxo descrito continua valendo em qualquer PaaS ou pipeline equivalente.

### Os 12 fatores (Adam Wiggins, Heroku, por volta de 2011), na leitura do autor [cap-075, cap-073]
- **1. Base de código**: tudo num repositório Git que o time inteiro acessa, com deploys frequentes, no mínimo para staging. [cap-075]
- **2. Dependências**: declaradas em manifesto e instaladas pelo mesmo comando em todo lugar; nunca depender de pacote instalado no SO da máquina. [cap-075]
- **3. Configuração**: no ambiente, separada por ambiente (dev, teste, produção). Teste usa banco de teste e não manda SMS de verdade; produção não expõe detalhes de erro nem carrega bibliotecas de debug. [cap-075]
- **4. Backing services**: banco, cache, fila, e-mail, tudo é recurso externo acessado por URL. Nada instalado junto da app (MySQL, Redis e RabbitMQ cada um no seu container); some o `localhost` hardcoded. [cap-075]
- **5. Build, release, run**: o build gera imagem imutável, a release é numerada, a execução troca imagens. Editar arquivo no servidor via SSH/FTP é proibido. [cap-075]
- **6. Processos stateless (share-nothing)**: nada que precise sobreviver a um deploy é gravado no sistema de arquivos local. Uploads vão para S3 ou similar, logs para um serviço; o container é efêmero e read-only depois do build. [cap-075]
- **7. Port binding**: o autor lê como "todo serviço exposto numa porta de rede", e por isso um cron preso a arquivo de configuração vira serviço externo acessado por HTTP. (complemento, não está no compêndio) No texto original, a app é autocontida e exporta seu serviço fazendo bind numa porta, sem depender de um servidor web injetado no ambiente. [cap-075]
- **8. Concorrência**: escalar horizontalmente adicionando processos e containers, no jeito Unix de fazer `fork`. Threads e I/O assíncrono esbarram no tamanho da máquina. [cap-075]
- **9. Descartabilidade**: subir e matar containers a qualquer momento (tratar SIGTERM, shutdown gracioso, rolling restart). Máquina invadida não se conserta: joga-se fora e recria-se automaticamente, sem perder um bit. [cap-075]
- **10. Paridade dev/prod**: ambientes de desenvolvimento, staging e produção o mais iguais possível, e deploy rápido para não se distanciarem. [cap-075]
- **11. Logs como fluxo**: escrever no STDOUT e deixar a plataforma coletar e enviar para um agregador. Nada de arquivo e rotação de log dentro do container. [cap-075]
- **12. Processos administrativos**: rodam num container isolado com a mesma imagem, fora do balanceador, sem disputar recursos com as requisições. [cap-075]

### Orquestração
- **Docker Compose**: um YAML declara services (app, db, redis), portas, variáveis, volumes e `depends_on`. Cada service tem hostname próprio (`DATABASE_HOST=db`), e `docker compose up` sobe tudo para quem acabou de clonar. É feito para desenvolvimento e servidores pequenos; não escala, porque instâncias extras são `app_1`/`app_2` escritas à mão e ele não reconcilia o que já está rodando. (complemento, não está no compêndio) O `depends_on` simples só espera o container iniciar, não o serviço ficar pronto; use healthcheck com `condition: service_healthy` ou retry na app. [cap-076, cap-073]
- **Kubernetes**: infra elástica com pods (containers), nodes, VPCs isoladas (rede de web separada da de jobs), Ingress (balanceador), adição e remoção dinâmica de pods, integração com serviços gerenciados (RDS, Cloud SQL) e múltiplas zonas/federação. Em 2019 era o padrão de fato nos grandes provedores. O que o Heroku chama de dyno ele chama de pod, e o `kubectl` se inspira na CLI do Heroku. [cap-073, cap-075, cap-076]
- **Terraform**: declara a infraestrutura do provedor (VPC, subnets, internet gateway, rotas, EC2, security groups, RDS, ElastiCache, load balancer, target groups). O fluxo é `terraform init` (plugins), `plan` (revisar o que vai mudar) e `apply`. O que já existe não é recriado, só reconfigurado. É IaaS orquestrada por código; o Kubernetes cuida dos containers em cima dela. [cap-076]
- **OSTree**: um "Git de sistema de arquivos". Para atualizar 100 servidores, monta-se a imagem nova uma vez e cada servidor troca a imagem inteira, em vez de rodar `apt upgrade` em cada um e torcer para não dar pau. [cap-076]

## Heurísticas de decisão

| Situação | O que fazer | Por quê | Fonte |
|---|---|---|---|
| Projeto novo, sem sysadmin dedicado | Começar numa PaaS (no texto, o Heroku) e comparar o custo da pessoa de infra, não o preço por hora da máquina | Infra feita por quem nunca fez "é sempre um desastre"; o custo real é o plantão de devops | cap-073, cap-075 |
| Vontade de ir direto para Kubernetes ou ferramenta de gigante | Só com dados de uso que justifiquem; começar simples | "Você não é o Netflix"; Mesos para 3 máquinas é risco sem retorno | cap-072, cap-075 |
| Alguém argumenta "o Twitter/Google usa" | Descartar o argumento e decidir pelo seu contexto | Prática de unicórnio só faz sentido para ele | cap-072 |
| Precisa de e-mail, Redis, cache, upload/processamento de imagem | Contratar SaaS em vez de montar e operar | Operar com redundância e backup é outro emprego | cap-073 |
| Escolher plataforma | Preferir a que usa padrões abertos e componentes comuns (Postgres) à que exige programar para ela (App Engine, Firebase) | Evita ficar preso ao fornecedor | cap-075 |
| Máquina invadida ou com defeito inexplicável | Destruir e recriar por automação; não consertar à mão | Máquina comprometida não se recupera; o que é manual sempre esquece algo | cap-072, cap-075 |
| Configurar uma VPS simples × frota heterogênea | Uma ou poucas: ferramenta agentless (Ansible). Dezenas de máquinas com papéis diferentes: receitas cliente-servidor (Chef/Puppet) | Custo de montar servidor e agentes × escala | cap-072 |
| Publicar nova versão | Build imutável a cada push, releases mantidas para rollback; nunca FTP nem edição no servidor | Reprodutível e reversível | cap-074, cap-075 |
| Deploy quebrou produção | Rollback para a última release boa, depois investigar | Consertar na tentativa e erro no sufoco piora tudo | cap-074 |
| App passou do limite de memória do container | Procurar vazamento e código porco antes de aumentar a máquina; desenvolver sob restrição parecida | "Código mal feito roda na máquina do desenvolvedor, mas quebra em produção" | cap-074, cap-073 |
| App grande demais para um container | Cortar gordura ou separar em mais de uma aplicação; microsserviço só com motivo assim | Microsserviço sem motivo é custo | cap-073 |
| Senhas, tokens, chaves | Variáveis de ambiente/config da plataforma; `.env` no `.gitignore`, `.env.example` versionado; nenhum segredo no Git, nem em `.tf`/compose de produção | Repositório é lido por todo mundo e fica para sempre no histórico | cap-074 |
| Upload de usuário ou arquivo gerado | Armazenamento externo (S3, Blob, GCS) | O container é descartado a cada deploy | cap-075 |
| Logs | STDOUT, coleta pela plataforma, agregador pesquisável e alerta de erro | Arquivo de log some com o container; biblioteca de log que faz demais vira brecha (Log4j) | cap-075, cap-074 |
| Tarefa agendada | Serviço externo acessível por rede (ou processo próprio), não crontab dentro do container web | Containers não têm estado local e há várias réplicas | cap-075 |
| Query de emergência ou tarefa administrativa | Container one-off com a mesma imagem; revisar duas vezes antes de rodar no banco | É permanente e fica fora do balanceador | cap-074, cap-075 |
| Escalar web horizontalmente | Antes, dimensionar o banco: conexões máximas × instâncias × pool; tirar leitura do banco com cache (Redis) e réplicas de leitura, sabendo que replicar também tem custo | Mais dynos só travam esperando conexão | cap-074, cap-075 |
| Sistema legado que não aguenta pico (reservas, loteria) | Fila ou sala de espera na frente | O que não se controla precisa ser protegido do pico | cap-074 |
| Organizar o fluxo do time | Todo PR com teste (sem teste é rejeitado); `pull`/`rebase` do `main` pelo menos uma vez por dia; CI a cada push; deploy automático para staging com tudo verde; feature flags para trabalho longo; teste de regressão para cada bug | Quanto mais demora para integrar, mais conflito e bug; `main` "tem que ser sagrado" | cap-075 |
| Escrever Dockerfile | Base com versão fixa/LTS (não `latest`); copiar o manifesto e instalar dependências antes de copiar o resto; instalar e limpar no mesmo `RUN`; pôr no fim o que muda mais; `.dockerignore` com `node_modules` e `*.log` | Camadas menores e build que aproveita o cache | cap-076 |
| Montar volume em container | Mapear só o diretório necessário; nunca `-v /:/mnt`; preferir runtime rootless (Podman) | Com daemon root, o volume amplo dá root no host (edita `/etc/passwd`) | cap-076 |
| Ambiente de dev com vários serviços | `docker-compose.yml` no repositório, versões iguais às de produção, mantido atualizado | Quem entra no time sobe tudo com um comando, sem configuração velha | cap-076 |
| Produção na nuvem | Terraform para a infraestrutura e Kubernetes para os containers; nada clicado na interface web | Declarativo, revisável (`plan`) e repetível | cap-076 |
| Isolar um app gráfico (GIMP, LibreOffice, Spotify) | Flatpak/Snap, não Docker | Feitos para apps com janela, arquivos, USB e dbus | cap-076 |
| Um processo pesado ameaça a máquina | Colocar num cgroup (slice do systemd com `CPUQuota` e limite de memória) | Deixa folga para a kernel se recuperar | cap-076 |
| Atualizar uma frota de servidores | Gerar a imagem nova e trocar (OSTree/imagem), não `apt upgrade` em cada nó | Atualização atômica e igual em todos | cap-076 |
| Operar infraestrutura | Paranoia: checar e rechecar, redundância, backup do backup, testar a restauração, monitorar; cuidado com script enviado a muitos servidores | Uma letra errada num script mandado por SSH apagou o Bash de dezenas de máquinas | cap-073 |
| Medir desempenho com Docker no Windows/Mac | Não comparar com Linux nativo e lembrar da VM no meio | Sempre mais pesado, sem exceção | cap-076 |

## Armadilhas e mitos
- "DevOps é um cargo, uma ferramenta ou um ritual" → não é nenhuma dessas coisas isoladamente, assim como Agile não é sprint. [cap-072]
- "Docker é tipo VirtualBox / container é VM leve" → não há hypervisor nem outra kernel: é um processo nativo que a kernel isola e engana. [cap-073, cap-076]
- "Container tem sistema operacional próprio" → tem só userland; compartilha a kernel do host e só roda distros Linux. [cap-073, cap-076]
- "Container é leve em qualquer SO" → só em Linux nativo; Windows e Mac sobem uma VM Linux antes. [cap-076]
- "Root dentro do container é inofensivo" → com daemon root e volume amplo, vira root na máquina real. [cap-076]
- "Sem Docker não há container" → quem faz o isolamento é a kernel (cgroups, namespaces) via runC; o Docker é ecossistema, formato e CLI. [cap-076]
- "Mais `RUN` deixa o Dockerfile organizado" → cada `RUN` é uma camada; apagar arquivos numa camada separada não diminui a imagem. [cap-076]
- "Docker Compose serve para produção grande" → não escala nem reconcilia estado; é ferramenta de dev e de servidor pequeno. [cap-076]
- "O Heroku copiou o Docker" → é o contrário: buildpacks e dynos (2008) vieram antes do Docker (2013). [cap-074, cap-073]
- "Ansible é a grande novidade" → é evolução do Fabric, que era clone do Capistrano (2006). [cap-072]
- "Colocar autoscale e mais dynos faz escalar" → o banco (conexões, CPU, replicação) estrangula; escalabilidade depende mais da arquitetura e do programador que da infraestrutura. [cap-074, cap-075]
- "Seguir os 12 fatores torna a app escalável" → são necessários, não suficientes. [cap-075]
- "Heroku é caro comparado ao EC2" → a conta certa inclui o profissional de devops que você não precisa ter. [cap-075, cap-073]
- "Deploy automático para produção é perigoso" → só com gerência ruim; o volume de bugs reflete a comunicação do time. [cap-075]
- "Feature grande justifica branch de semanas" → use feature flag e integre continuamente; adiar a integração multiplica bugs e conflitos. [cap-075]
- "Gravar upload num diretório da app funciona" → funciona até o próximo deploy, quando tudo some. [cap-075]
- "Está funcionando, então não mexe" → software nunca está pronto; sistemas congelados viram "ilhas radioativas" cercadas de gambiarras. [cap-074]
- "Vamos deixar preparado para o caso de um dia precisar" → software desnecessário é buraco de segurança esperando ser descoberto (a lição do Log4j). [cap-075]
- "Copiar o exemplo de Terraform/compose do tutorial" → os exemplos didáticos do cap-076 usam senha literal e `latest`; num projeto real, fixe versões e tire os segredos dos arquivos, como o próprio autor manda em cap-074 e cap-076. [cap-076, cap-074]
- "Ferramenta de devops de tutorial é eterna" → Docker Machine "ninguém usa mais" e os planos do Heroku mudaram; o fluxo (push → build → release → rollback) é o que fica. [cap-076, cap-074]

## Checklist para aplicar num sistema real
- [ ] O ambiente inteiro pode ser destruído e recriado por script, em minutos, sem perder dados?
- [ ] Todo código e toda configuração de infra (Dockerfile, compose, Terraform, manifests) estão no Git? Alguém edita coisa direto no servidor?
- [ ] O build gera artefato ou imagem imutável, com releases numeradas e rollback testado?
- [ ] As dependências estão declaradas, com lockfile commitado? A app assume algum pacote do SO?
- [ ] Configuração e segredos vêm do ambiente? `.env` ignorado, `.env.example` presente, zero segredo no repositório e no histórico?
- [ ] Banco, cache, fila e e-mail são acessados por URL configurável, sem `localhost` hardcoded?
- [ ] A app é stateless: nenhum upload, sessão ou log gravado no disco local do container?
- [ ] Os logs vão para STDOUT, são agregados, pesquisáveis e geram alerta de erro?
- [ ] O processo trata SIGTERM e encerra graciosamente? Dá para reiniciar instâncias uma a uma sem tirar o serviço do ar?
- [ ] Jobs agendados e tarefas administrativas rodam como processos separados ou one-off, com a mesma imagem?
- [ ] Fiz a conta de capacidade (processos × 1/latência) e medi? Quantas conexões o banco aceita frente a instâncias × pool?
- [ ] Há balanceador na frente das instâncias, e cache ou réplicas de leitura onde o banco é o gargalo?
- [ ] Sistemas que não aguentam pico têm fila ou sala de espera?
- [ ] Todo PR tem teste? A CI roda a cada push, o `main` está sempre verde e o deploy para staging é automático? Trabalho longo fica atrás de feature flag?
- [ ] As imagens partem de versão fixa/LTS, com camadas enxutas, `.dockerignore` e sem `latest` em produção?
- [ ] Os containers rodam com o mínimo de privilégio (rootless quando possível, volumes mínimos, capabilities mínimas)?
- [ ] Há limites de CPU e memória (cgroups) para que um serviço não derrube os vizinhos?
- [ ] Um dev novo sobe o ambiente com `git clone` e um comando, com as mesmas versões da produção?
- [ ] A infra da nuvem é declarada (Terraform) e cada mudança passa por `plan` revisado antes do `apply`?
- [ ] A plataforma escolhida prende o código (API proprietária) ou permite sair?
- [ ] O tamanho da infra é proporcional ao problema (não é Kubernetes para três máquinas)?
- [ ] O acesso administrativo é só por canal criptografado (SSH), com 2FA nas contas de plataforma?
- [ ] Os backups são testados por restauração e há monitoramento contínuo?

## Frases-âncora
> "Ouça isso com muita atenção: você não é o Netflix, tá muito longe." — cap-072

> "Não tente fazer infra sozinho se nunca fez isso antes, é sempre um desastre." — cap-073

> "Escalabilidade nunca é automática, você precisa estar preparado pra isso." — cap-074

> "Satisfazer os 12 fatores não torna nenhuma aplicação automaticamente escalável, mas não satisfazer eles quase garante que não vai escalar de jeito nenhum." — cap-075

> "Um sistema robusto precisa ser descartável." — cap-075

> "Lembrem-se, a qualidade do software é diretamente proporcional à qualidade da comunicação da equipe." — cap-075

> "uma máquina virtual roda um novo kernel, possivelmente numa outra arquitetura, como x86 em cima de ARM, já um container reaproveita o mesmo kernel, necessariamente na mesma arquitetura de hardware." — cap-076

> "Container, na prática é outro nome dado a um processo em execução com menos acessos que um processo normal, mas no fundo é só isso: a execução de um processo." — cap-076

## Onde aprofundar

| Capítulo | Título | URL original | Leia quando... |
|---|---|---|---|
| cap-072 | [Akitando] #47 - Entendendo "Devops" para Iniciantes em Programação (Parte 1) \| Série "Começando aos 40" (10/04/2019) | https://www.akitaonrails.com/2019/04/10/akitando-47-entendendo-devops-para-iniciantes-em-programacao-parte-1-serie-comecando-aos-40/ | Precisar distinguir hypervisor tipo 1/2, virtualização completa × paravirtualização × container; entender a origem de jails/zonas e de CFEngine, Puppet, Chef, Capistrano e Ansible |
| cap-073 | [Akitando] #48 - Entendendo "Devops" para Iniciantes em Programação (Parte 2) \| Série "Começando aos 40" (17/04/2019) | https://www.akitaonrails.com/2019/04/17/akitando-48-entendendo-devops-para-iniciantes-em-programacao-parte-2-serie-comecando-aos-40/ | Entender o que AWS, IaaS/PaaS/SaaS, Heroku, cgroups/LXC, Docker e Kubernetes mudaram e por que "um processo por container" |
| cap-074 | [Akitando] #112 - Subindo Aplicações Web em Produção \| Aprendendo HEROKU (10/01/2022) | https://www.akitaonrails.com/2022/01/10/akitando-112-subindo-aplicacoes-web-em-producao-aprendendo-heroku/ | Querer o fluxo de deploy ideal passo a passo: push → build → dynos, Procfile, lockfile, env vars, addons, one-off, releases e rollback (plataforma datada) |
| cap-075 | [Akitando] #113 - A Forma Ideal de Projetos Web \| Os 12 Fatores (29/01/2022) | https://www.akitaonrails.com/2022/01/29/akitando-113-a-forma-ideal-de-projetos-web-os-12-fatores/ | Revisar uma app contra os 12 fatores, o caso de gargalo de conexões no Postgres e as 4 regras de testes, CI e deploy contínuo |
| cap-076 | [Akitando #139] - Entendendo Como Containers Funcionam (02/03/2023) | https://www.akitaonrails.com/2023/03/02/akitando-139-entendendo-como-containers-funcionam/ | Precisar ver cgroups, namespaces (`unshare`), runC/`config.json`, camadas OCI, riscos do daemon root, capabilities, Dockerfile, Compose com NGINX, Kubernetes e Terraform na prática |
