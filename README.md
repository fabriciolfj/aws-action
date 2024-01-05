# aws-action

## pontos importantes
VPC (Virtual Private Cloud) é um recurso importante na AWS para criar redes virtuais privadas. Alguns conceitos:

## VPC

- Permite criar uma rede virtual privada, isolada e configurável dentro da AWS.
- Nela podemos criar sub-redes e determinar range de IP.
- Por padrão, as instâncias criadas em uma VPC não são acessíveis publicamente. 

**Sub-redes Públicas x Privadas**

- Sub-redes Públicas: tem uma rota e IP público, permitindo acesso direto à internet. 
- Sub-redes Privadas: só tem acesso interno, sem IP público direto.

**Tabela de rotas**

- Determina para onde o tráfego de rede será direcionado.
- Podemos definir rotas customizadas por sub-rede.
- A sub-rede pública precisa ter uma rota para internet.

**Relação**  

- Uma VPC pode conter uma sub-rede pública E privada
- As instâncias da sub-rede privada podem acessar a internet saindo pelo NAT da sub-rede pública.

Então o VPC permite criar redes totalmente personalizáveis e segregadas na nuvem.


Boa pergunta! Vou explicar melhor sobre NAT e o que quis dizer com "saindo pelo NAT":

## NAT (Network Address Translation)

O NAT é um serviço que permite que máquinas de uma rede interna privada (por exemplo, a sub-rede privada do VPC) se conectem à Internet, sem expor seus endereços IP privados.

Ele faz a "conversão de endereços", traduzindo o IP privado para um IP público, somente quando o tráfego precisar sair para a Internet.

**Saindo pelo NAT** 

O que quis dizer é que, como as instâncias da sub-rede privada não possuem IP público próprio, elas não conseguem se conectar direto na Internet.

Para dar acesso à Internet para essas máquinas, configuramos um NAT na sub-rede pública do VPC.

Assim o tráfego da sub-rede privada será direcionado para o NAT, que fará a tradução de IP, e então sairá para a Internet através do IP público do NAT.

Então elas acessam a Internet "saindo pelo NAT", através desse serviço de tradução de endereço que vive na sub-rede pública.

Espero ter conseguido explicar melhor essa relação! Fique à vontade para perguntar mais alguma coisa.

# Outros detalhes
### acl
-  Se você abrir a porta de entrada 80 em um NACL para sua sub-rede, talvez ainda não consiga se conectar via HTTP. Além disso, você precisa permitir portas efêmeras de saída, porque o servidor web aceita conexões na porta 80, mas usa uma porta efêmera para comunicação com o cliente. As portas efêmeras são selecionadas no intervalo que começa em 1024 e termina em 65535. Se você deseja fazer uma conexão HTTP de dentro de sua sub-rede, é necessário abrir a porta de saída 80 e as portas efêmeras de entrada também.

- Outra diferença entre as regras do grupo de segurança e as regras NACL é que você precisa definir a prioridade para as regras NACL. Um número de regra menor indica uma prioridade mais alta. Ao avaliar um NACL, a primeira regra que corresponde a um pacote é aplicada; todas as outras regras são ignoradas.

### conectando na internet pela rede privada
- crie um gateway NAT em uma sub-rede pública e crie uma rota de sua sub-rede privada para o gateway NAT. Dessa forma, você pode acessar a Internet a partir de sub-redes privadas, mas a Internet não pode acessar suas sub-redes privadas. Um gateway NAT é um serviço gerenciado fornecido pela AWS que lida com a tradução de endereços de rede. O tráfego da Internet da sua sub-rede privada acessará a Internet do endereço IP público do gateway NAT.

## diferença rede publica com privada
-  entre uma sub-rede pública e uma privada é que uma sub-rede privada não possui uma rota para o IGW.

## recuperação de maquinas ec2
 - via cloudwatch podemos configurar uma alarme que se ativo, atendendo a métrica, fará um recover na ec2 vinculada

## code deploy
```
aws s3 cp etherpad-lite-1.8.17.zip \
 s3://etherpad-codedeploy-artifactbucket-1o18siq5jec0t/etherpad-lite-1.8.17.zip



aws deploy create-deployment --application-name etherpad-codedeploy \
 --deployment-group-name etherpad-codedeploy \
 --revision "revisionType=S3,
 s3Location={bucket=etherpad-codedeploy-artifactbucket-1o18siq5jec0t,
 key=etherpad-lite-1.8.17.zip,bundleType=zip}"



$ aws deploy create-deployment --application-name etherpad-codedeploy \
 --deployment-group-name etherpad-codedeploy \
 --revision "revisionType=S3,
 s3Location={bucket=etherpad-codedeploy-artifactbucket-1o18siq5jec0t,
 key=etherpad-lite-1.8.18.zip,bundleType=zip}"


aws s3 rm --recursive s3://etherpad-codedeploy-artifactbucket-1o18siq5jec0t

aws cloudformation delete-stack --stack-name etherpad-codedeploy
```

## ECS
- cria-se o cluster (pode ter vários services)
- crio uma definição de tarefa
- posso ja utilizar a mesma para criar um serviço ou ao criar um serviço selecionar a definição de tarefa
- indicado que cada serviço seja para uma app
- veja o serviço como deployment e a tarefa como um pod
- caso queria adicionar um load balance, precisa criar um novo serviço através de uma task definitions
- dentro da task tenho a imagem no ecr e a porta configurada para o conteiner
- e esta posso adiciionar as variáveis de ambiente necessárias para minha aplicação


## Resumo para certificação architect associate

### iam
- dicas:
  - não use a conta root, apenas para criar outras contas
  - não compartilhe credenciais
  - crie grupos para atribuir permissões
  - crie politica de password forte
  - use MFA
  - crie e use roles para permissões a servços da aws
  - utilize access keys programatic para cli
  - para auditoria, temos relatório e access advisor
  - nunca compartilhe sua conta/credenciais
 
   
#### role
- quando criamos uma role, podemos dizer pra qual serviço ela será utilizada
- especificamos isso no atributo principal, por exemplo:
```
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Sid":"Statement1",
         "Effect":"Allow",
         "Principal":{
            "Service":"ec2.amazonaws.com"
         },
         "Action":"sts:AssumeRole"
      }
   ]
```

### Diferença entra host dedicado  e instâncias dedicadas
- instâncias dedicadas -> você possui sua instância em seu próprio hardware
- host -> você obtem acesso ao próprio servidor físico


### instancias spots
```
A diferença entre Spot Instances, Spot Fleets e Spot Requests na AWS é a seguinte:

**Spot Instances** 

- São instâncias EC2 individuais requisitadas diretamente. 
- Oferecem grandes descontos, mas podem ser interrompidas se o preço máximo definido for ultrapassado.

**Spot Requests**

- São requisições individuais de instâncias Spot. 
- Cada Spot Request cria uma Spot Instance se tiver capacidade.

**Spot Fleets**

- Permitem criar um grupo de Spot Instances e Spot Requests com diferentes configurações.
- Balanceia automaticamente a disponibilidade, performance e custo.
- Requisita Instances para atingir a capacidade definida.

**Resumo**

Spot Request é a requisição individual. 

Spot Instance é a instância em si provisionada.

Spot Fleet administra um grupo de Requests e Instances Spot.

O Fleet traz mais automação, balanceamento e confiabilidade no uso de Spot.
```

#### spot pools
```
O Spot pools é um recurso da AWS para Spot Instances que permite definir um pool de capacidade computacional spot com características específicas de instance type, SO e AZ.

Principais características:

- Permite criar um grupo de instâncias spot com atributos em comum
- Garante capacidade alvo dentro do pool
- O pool irá repor automaticamente capacidade se instâncias forem interrompidas
- Alternativa mais confiável que spots individuais 
- Tem políticas de allocation para distribuir instâncias
- Integra com auto scaling groups

O spot pool ajuda a tratar a intermitência das spots, fornecendo um agrupamento mais confiável e com reposição automática.

Dessa forma, aplicações que demandam alta disponibilidade podem se beneficiar de preços spot, contando com o aumento de confiabilidade e consistência do pool.

Então é uma excelente opção para provisionar e gerenciar grupos de spots de modo mais resiliente.
```

#### elastic ip
- criar um ip publico elastico e estático, onde podemos vincular a uma ec2


## Placement Groups
Placement Groups (Grupos de Posicionamento) na AWS são um recurso que permite influenciar como as instâncias do EC2 são posicionadas fisicamente nos datacenters da AWS.

Existem 3 estratégias de Placement Group:

**1. Cluster**

- Instâncias posicionadas o mais próximo possível dentro de uma zona de disponibilidade, ideal para cargas de trabalho de alta performance que precisam de baixa latência em rede para comunicação entre nós.
  - indicado para aplicações que precisam de baixa latência, pois estão dentro do mesmo rack
  - indicados para bigdatas

**2. Spread** 

- Instâncias espalhadas por diferentes racks(prateleiras) que podem estar em diferentes zonas de disponibilidade. Protege contra falhas simultâneas no hardware do rack.
  - indicado para aplicações que precisam de alta disponibilidade, com failover

**3. Partition**

- Esparrama instâncias em partições lógicas que refletem os grupos de particionamento do hardware subjacente, como diferentes prateleiras em racks separados e em diferentes azs.
  -  limita em 7 partições por az
  -  indicados para aplicações distribuidas, como cassandra, redis, banco de dados distribuídos

As vantagens dos Placement Groups:

- Permite influenciar a posição física das instâncias
- Menor latência em rede entre instâncias no mesmo grupo
- Isola falhas de hardware

As limitações:

- Nem todos os tipos de instância são compatíveis
- Uma instância só pode fazer parte de um Placement Group

Então resumindo, Placement Groups são úteis para obter desempenho, isolamento de falhas e controle sobre posicionamento de instâncias.

## ENI

ENI (Elastic Network Interface) é um recurso da AWS que permite criar interfaces de rede virtuais e gerenciá-las de forma independente das instâncias EC2.

Principais características:

- Uma ENI é uma interface de rede virtual que você pode criar, conectar e desconectar de instâncias EC2.

- É alocada privadamente a uma subnet em uma VPC (Virtual Private Cloud). 

- Tem um IP privado primário na subnet, e pode ter um ou mais IPs secundários.

- Pode ser conectada ou desacoplada de uma instância EC2 no mesmo AZ.

- Retém todos os seus atributos quando desconectada, podendo ser reconectada a outra instância.

- Suporta segurança e diversos tipos de tráfego de rede por meio de grupos de segurança e ACLs.

- Permite criar designs de rede mais flexíveis com balanceamento de carga e failover.

As ENIs são úteis quando você precisa:

- Reassociar rapidamente endereços IP privados a outra instância.
- Implementar configurações de rede complexas em instâncias.
- Usar configurações consistentes ao migrar instâncias para novos servidores.  

Então o Elastic Network Interface habilita cenários avançados de rede no EC2.


## Hibernação maquinas ec2
O hibernate para instâncias EC2 permite "suspender" e "retomar" a execução de uma instância. Isso diferencia do stop/start tradicional.

Algumas características:

- Ao hibernar, o estado da memória RAM é salvo em disco na própria instância.

- O estado da memória é restaurado quando a instância é retomada, junto com processos em execução.

- É muito mais rápido que desligar e religar a instância.

- A instância vai para o estado "stopped" e não incide cobranças de uso.

- Somente instâncias com root volume EBS podem ser hibernadas, instâncias com instance store não.

- Instâncias grandes (m4.16xlarge ou superior) não suportam hibernação.

- Tipos de instâncias mais antigos, como T2, também não suportam.

Então o hibernate permite suspender e depois quickly resumir instâncias, sem perder estado.

É ideal para ambientes de teste/dev onde máquinas não precisam ficar sempre ligadas, economizando custos. Também agiliza inicialização de grandes aplicações.


## EBS (elastic block store)
Amazon EBS (Elastic Block Store) é um serviço de armazenamento em bloco que pode ser anexado às instâncias EC2 como dispositivos de armazenamento.

Principais características:

- Armazenamento persistente e de alto desempenho que pode ser anexado às instâncias.
  -  instnâncias que estejam na mesmoa az do ebs
  -  IOPS (Input/Output Operations Per Second) é uma métrica que mede o desempenho de dispositivos de armazenamento, indicando quantas operações de entrada/saída o dispositivo pode processar por segundo. 
- Permiten criar sistemas de arquivo ou até usar como raw block devices.
- Os dados armazenados são replicados para alta disponibilidade e durabilidade.
- Vários tipos para casos de uso específicos: SSD para alto desempenho, HDD para data warehousing, magnéticos para custo e outros. 
- Integram com snapshots para fazer backups pontuais que podem ser restaurados em qualquer zona de disponibilidade.

Vantagens sobre armazenamento instance-store:

- Persistência independentemente do ciclo de vida da instância
- Flexibilidade de mudar tipos de instância mantendo o mesmo armazenamento
- Migração de dados entre zonas de disponibilidade

Portanto o EBS fornece armazenamento de alto desempenho e custo eficiente para usar com instâncias EC2. A persistência permite designs mais resilientes.


### Diferença entre iops e taxa de transfefência
Ótima pergunta! A principal diferença entre IOPS e taxa de transferência (throughput) para volumes de armazenamento é:

**IOPS -** 
- Mede o número de operações de entrada/saída por segundo, como gravações, leituras, exclusões, etc. 
- Metrica a capacidade de realizar um grande número de operações pequenas e aleatórias.
- Importante para bancos de dados OLTP, onde o desempenho depende mais de IOPS.

**Taxa de Transferência -**
- Mede o volume total de dados que podem ser lidos/gravados por segundo, normalmente em MB/s.
- Indicativo da capacidade de sustentar transferências maciças sequenciais de dados. 
- Mais relevante para cargas analíticas/Big Data como data warehouses.

Em resumo:

- **IOPS:** quantidades de operações I/O por segundo, leituras e gravações pequenas e aleatórias.

- **Throughput:** volume total de dados transferidos por segundo, fluxos sequenciais de dados.

Tipos de armazenamento podem ter IOPS alto e throughput baixo ou vice-versa. Importante dimensionar com base na carga de trabalho específica.

Entendido como as duas métricas complementares? Fique à vontade para perguntar mais!

### Ebs snapshots
- são backups do ebs
- podemos gerar o bkp de um ebs de uma az e restaurar em outra az
- podemos copiar os snapshots para outras az

#### Features do ebs snapshots
- ebs snapshot archive -> mais barato, leva mais tempo para ser restaurado
- ebs snapshot recycle -> funciona como a lixeira do windows, caso dele um ebs por engano, ele vai para lixeira e fica por um tempo configuravél
- fast snapshot restore -> o mais caro, restaura instantaneamente o backup para uso

## AMI (amazon machine image)
-  ami -> usado para customizar instancias ec2
-  criamos com base em uma ec2 ja sendo executada
-  posso comprar ami do marketplace

## tipos ebs
Os principais tipos de volumes EBS (Elastic Block Store) da AWS são:

**SSD gp3**
- Uso geral, balanceado entre preço e performance
- IOPS a partir de 3000 por volume
- Taxa de transferência de 125 MiB/s a 1000 MiB/s

**SSD io2** 
- Alto desempenho, sensível a latência  
- IOPS entre 100 e 64000 (nível de SSD)
- Taxa de transferência de 100 MiB/s a 1000 MiB/s

**SSD io1**
- Desempenho muito alto 
- IOPS entre 100 e 64000 
- Taxa de transferência máxima de 1000 MiB/s

**HDD gp2**
- Uso geral
- IOPS baseline de 100
- Taxas de transferência de 125 MiB/s
- Menor custo

**HDD st1** 
- Otimizadas para throughput, dados sequenciais
- Streaming de dados, data warehouses
- Taxa de transferência de 500 MiB/s
  
**HDD sc1**
- Taxas de transferência baixas
- Dados com acesso pouco frequente

## tipo ec2
A AWS oferece uma ampla gama de tipos de instâncias EC2 para atender a diferentes casos de uso e requisitos. Os principais são:

**Uso Geral**
- M5, M6 - balanceado entre CPU, memória e rede
- T3, T4 - recursos expansíveis na nuvem

**Computação Otimizada** 
- C5, C6 - alto desempenho CPU e rede
- C7 - custo otimizado e ARM

**Memória Otimizada**
- R5, R6, X2 - muito alta memória e armazenamento 

**Armazenamento Otimizado**
- D2, D3 - alto throughput, baixa latência
- H1 - throughput altíssimo sequencial

**GPU Compute**
- G4, P4 - alta capacidade GPU, machine learning
  
**Otimizado F1**
- FPGA, lógica programável

Algumas instâncias também possuem características especiais como zonas locais, confidencialidade de dados, tolerância a falhas e muito mais.

Recomenda-se escolher o tipo com base no perfil da aplicação, benchmark de desempenho e análise de custo. Monitoramento contínuo também é recomendado.

Fique à vontade para perguntar sobre qualquer tipo ou caso de uso específico!

## tipos ec2 x ebs
Exato! Para tirar proveito total do desempenho de volumes EBS de alto nível, é importante ter instâncias EC2 adequadas.

Alguns pontos de atenção nessa relação EBS x EC2:

- Instâncias têm limites de IOPS e throughput que conseguem sustentar. Um EBS muito rápido não adianta se a EC2 não der vazão.

- Utilizar instâncias com bons perfis de rede e armazenamento como M5, C5 ou R5 para renderizar alto desempenho de EBS.

- Dimensionar aInstância para ter memória e CPU suficientes para alimentar o armazenamento.

- Combinar instâncias otimizadas para EBS como D3/D3en com EBS Provisioned IOPS para latência muito baixa.

- Testar benchmark da aplicação e identificar gargalos (bottlenecks).

- Escalar horizontalmente com um auto scaling group pode dividir a carga por mais nós.

Então essa relaçãoEC2 x EBS precisa ser observada em conjunto - o desempenho final será determinado pelo elo mais fraco.

Monitorando métricas como filas de disco, IOPS consumida e latência temos visibilidade dessa relação.

## Ebs multi-attach
- permite anexar o voluem ebs a mais de uma ecs na mesma zona de disponibilidade
- se limita a 16 instâncias

## criptografica ebs
- quando criamos um ebs e marcamos ele para usar criptografia, todo o processo (encriptar/decriptar) e transparente pra nós
- utiliza chaves kms (aes-256)
- problema ocorre quando queremos encriptar um ebs que não foi encriptado
  - precisamos criar um snapshot do ebs
  - copiamos ele e marcamos para encriptar
  - criamos um volume a partir dele
 
## amazon efs (elastic file system)
- sistem de arquivos em rede, montando em instâncias ec2
- ele fica disponivel em multiaz
- ami tem que ser linux
- uso:
```
O EFS é ideal para armazenar arquivos grandes, como vídeos, imagens e arquivos de áudio. Ele pode lidar com arquivos de até 50 TB, tornando-o uma ótima opção para empresas com grandes conjuntos de dados.
* **Armazenamento de dados não estruturados:** O EFS também é uma boa opção para armazenar dados não estruturados, como logs, dados de eventos e dados de sensores. Esses tipos de dados não são armazenados em um formato específico, o que os torna difíceis de armazenar e gerenciar em sistemas de arquivos tradicionais. O EFS é projetado para armazenar dados não estruturados e pode facilmente lidar com esse tipo de dado.
* **Armazenamento de dados de aplicativos:** O EFS pode ser usado para armazenar dados de aplicativos, como bancos de dados, arquivos de configuração e arquivos de mídia. Isso pode ajudar a melhorar o desempenho dos aplicativos e facilitar o gerenciamento dos dados.
* **Backup e recuperação:** O EFS pode ser usado para fazer backup de dados de outros sistemas de arquivos ou dispositivos. Isso pode ajudar a proteger os dados no caso de perda de dados ou falha do sistema. O EFS também pode ser usado para recuperar dados perdidos ou corrompidos.
* **Compartilhamento de arquivos:** O EFS pode ser usado para compartilhar arquivos entre diferentes usuários ou grupos. Isso pode ajudar a melhorar a colaboração e a produtividade.
```

## diferenca entre ebs e efs
A principal diferença entre EBS (Elastic Block Store) e EFS (Elastic File System) na AWS é:

**EBS**
- Discos de bloco em blocos para EC2 (instâncias virtuais).
- Anexado a somente uma instância por vez (alguns tipos podem ter o recurso multi attach, mas não e multi az).
- Maior performance em leitura/gravação.
- Snapshot possível para backup.

**EFS** 
- Sistema de arquivos escalável para ser usado entre várias instâncias (multi az)
- Pode ser montado simultaneamente em milhares de instâncias EC2.
- Performance mais baixa que EBS.
- Mais indicado para cenários de acesso simultâneo.

## ecs instancia store
```
O ECS Instance Store é um tipo especial de armazenamento disponível para alguns tipos de instâncias do Amazon EC2 usadas em clusters do Amazon ECS (Elastic Container Service).

Principais características:

- Armazenamento efêmero anexado fisicamente à instância do EC2. 
- Fornece alta performance de I/O, com baixa latência.
- Os dados não são persistidos quando a instância para/falha.
- Útil para buffer, cache, escrita temporária.

Como é anexado fisicamente ao servidor, o ECS Instance Store permite performance de leitura/gravação muito alta em comparação a volumes EBS. Porém os seus dados só existem enquanto a instância está em execução.

Por esse motivo, ele é mais indicado para dados temporários e buffer/cache. Já os volumes EBS permitem persistência dos dados, são desanexados da vida da instância.

Então em resumo, o ECS Instance Store provê armazenamento temporário de alto desempenho específico para instâncias usadas em clusters do serviço ECS da AWS.
```

Resumindo:

- EBS é disco rígido em blocos para uma instância EC2. 
- EFS é file system flexível e paralelo para múltiplas instâncias.

Enquanto EBS é privado para uma instância, EFS permite o compartilhamento de arquivos entre várias máquinas.


## Questões
```
Qual dos seguintes tipos de volume do EBS pode ser usado como volumes de inicialização ao criar instâncias do EC2?

Os tipos de volumes EBS que podem ser usados como volumes de inicialização para instâncias EC2 são:

- gp2 (SSD General Purpose): volume de SSD balanceado em relação a preço e performance. É o tipo mais utilizado para a maioria das cargas de trabalho.

- gp3 (SSD General Purpose): similar ao gp2, mas permite ajustar a performance de IOPS e throughput de modo independente.

- io1 (SSD Provisioned IOPS): SSD de alto desempenho para cargas de trabalho sensíveis à latência, como bancos de dados.

- io2 (SSD Provisioned IOPS): nova geração do io1 com melhor eficiência econômica.

Portanto, gp2, gp3, io1 e io2 são os tipos que suportam volumes de inicialização para instâncias EC2.

Já os volumes magnéticos padrão (st1 e sc1) e o volume de throughput elevado (st1), não podem ser usados para inicialização. Apenas para dados.

Então em resumo, os tipos SSD (gp2, gp3, io1 e io2) podem ser boot volumes, os tipos magnéticos/throughput não.

```

# Escalabilidade

## load balance
- um servidor ou servidores, que rediciona uma requisição a uma ec2 ou destino
- ele vai balanceando as requisições para não sobrecarregar um destino
- ele verifique a saude do destino, caso esteja respondendo, ele poderá receber a requisioção, ao contrário não.
- funcionamento com ec2:
  - crie as instancias
  - vincule a um target group
  - dentro do target, podemos colocar algumas regras, como rota, cabeçalho http, host
  - crie o load balance e vincule o target group
- Ao usar um Application Load Balancer para distribuir tráfego para suas instâncias EC2, o endereço IP do qual você receberá solicitações serão os endereços IP privados do ALB. Para obter o endereço IP do cliente, o ALB adiciona um cabeçalho adicional chamado "X-Forwarded-For" que contém o endereço IP do cliente.
- Os ALBs podem rotear o tráfego para diferentes grupos-alvo com base no caminho da URL, nome do host, cabeçalhos HTTP e strings de consulta.
### tipos load balance
```
Os principais tipos de load balancers na AWS para distribuir tráfego entre instâncias são:

**Application Load Balancer (ALB)**

- Balanceamento em nível de aplicação (HTTP/HTTPS). 
- Suporta paths e hosts de rotas para serviços.
- Tem recursos avançados como auth, SSL, logs.
- Indicado para aplicações web serviço-orientadas.

**Network Load Balancer (NLB)** 

- Balanceamento em nível de conexão TCP.
- Latência extremamente baixa e alto throughput .
- Usa para tráfego muito intenso não HTTP ou interconectar serviços.
- para o health check, suporta os protocolos http, tcp e https.

**Classic Load Balancer** 

- Balanceador legado com menos recursos. 
- Possui suporte a camadas Classic do EC2.

**Gateway Load Balancer**

- Novo, permite escalar para milhões de requests por segundo usando GENEVE.
- Para aplicações extremamente intensas e cluster de containers.
- aplica firewall, mais segurança e etc

Alguns fatores para decidir:

- Regra geral, prefira ALB ou NLB para novos serviços. 
- ALB para maioria dos casos de HTTP/HTTPS.
- NLB quando precisar reduzir latência ao máximo.
- Gateway para cargas inimaginavelmente grandes.
```


### sitcky sessions
```
Sticky sessions no Elastic Load Balancer (ELB) da AWS são uma forma de vinculação de sessão entre um cliente e um servidor específico atrás de um balanceador de carga. Aqui estão alguns detalhes importantes sobre sticky sessions no ELB da AWS:

- Elas permitem que requests de um cliente sejam roteados para o mesmo servidor behind o ELB para toda a duração da sessão. Isso é útil para aplicações web que armazenam informações de sessão no servidor.

- As sticky sessions são habilitadas definindo o atributo de stickiness no listener do ELB. Você pode habilitar o stickiness baseado em cookies ou na porta da origem.

- Ao usar stickiness baseada em cookies, o ELB insere um cookie de sessão especial que vincula subsequentes requests do mesmo cliente ao mesmo servidor.

- Com a stickiness baseada na porta da origem, o ELB gera uma porta aleatória para cada servidor e mapeia requests de um cliente para essa porta para garantir eles vão para o mesmo servidor.

- As sticky sessions são úteis para estado de sessão, mas podem causar desequilíbrio de carga se um servidor receber significativamente mais tráfego. É importante dimensionar adequadamente.

- Recomenda-se habilitar sticky sessions somente quando necessário, pois elas limitam a capacidade de escalabilidade e balanceamento de carga do ELB.
```

### cross zone load balance
```
O cross-zone load balancing é um recurso importante dos Elastic Load Balancers (ELBs) na AWS. Aqui estão alguns detalhes-chave:

- Por padrão, o ELB distribui o tráfego de entrada igualmente entre as zonas de disponibilidade que estão habilitadas. Isso pode causar desequilíbrio se você tiver mais instâncias do seu auto scaling group em algumas zonas.

- Habilitar o cross-zone load balancing faz com que o ELB distribua o tráfego uniformemente entre todas as instâncias registradas, independentemente da zona de disponibilidade.

- Isso é muito útil para equalizar a carga em seus recursos em todas as zonas disponíveis. Elimina o problema de "zonas quentes", onde algumas zonas recebem muito mais tráfego do que outras.

- É altamente recomendado habilitar o cross-zone load balancing para aplicações de missão crítica que exigem alta disponibilidade e tolerância a falhas.

- Há uma pequena cobrança adicional por tráfego entre zonas com o cross-zone load balancing habilitado. Mas na maioria dos casos o benefício supera esse custo extra.

- Não há desvantagem significativa em habilitá-lo. A latência entre zonas é normalmente muito baixa para causar problemas.

Em resumo, habilitar o cross-zone load balancing ajuda a igualar a distribuição de carga, evita zonas quentes e melhorar a disponibilidade geral. É considerada uma best practice para a maioria das implantações.
```
- para alb é aplicada por default ao cross zone
- ja para nlb e glb, não é ativa, e caso ative, tem uma cobrança adicional

### sni
```
SNI (Server Name Indication) é uma extensão de segurança do protocolo TLS (Transport Layer Security) que permite que vários certificados SSL sejam servidos do mesmo endereço IP e porta.

Aqui estão algumas informações importantes sobre SNI:

- Sem SNI, um servidor só pode hospedar um certificado SSL por endereço IP e porta. Isso porque o certificado é negociado no início da "handshake" TLS, antes que o hostname seja conhecido.

- SNI resolve esse problema enviando o hostname que o cliente está tentando acessar no início do processo de "handshake", permitindo que o servidor apresente o certificado SSL correto para esse domínio.

- Isso permite hospedar vários sites com SSL offload no mesmo endereço IP e mesmo load balancer, como é comum em implantações de larga escala na cloud.

- O SNI é amplamente adotado e suportado pela maioria dos navegadores modernos e bibliotecas de software. No entanto, alguns clientes antigos não suportam.

- Se um cliente não suportar SNI, a conexão irá falhar ou será apresentado um certificado padrão. Portanto o SNI ainda requer algum planejamento.
```

### handshake
```
Em resumo, o SNI é crucial para escalar implantações de terminação TLS que precisam suportar vários certificados SSL no mesmo endereço IP. É uma extensão importante para segurança e desempenho na cloud.

Handshake, em português "aperto de mãos", é um termo usado em redes de computadores e segurança que se refere ao processo inicial de estabelecimento de uma conexão entre duas entidades. 

Alguns exemplos de handshake em diferentes protocolos e tecnologias:

- TCP handshake: É o processo de estabelecer uma conexão TCP entre dois dispositivos. Consiste em uma troca de três mensagens (SYN, SYN-ACK, ACK) para sincronizar os números de sequência e permitir a comunicação.

- SSL/TLS handshake: É o processo pelo qual um cliente e servidor negociam os detalhes de criptografia e estabelecem uma conexão segura. Envolve várias etapas como negociação de versão do protocolo, seleção de conjunto de criptografia, autenticação do servidor e/ou cliente com certificados digitais, etc.

- WebSocket handshake: É o processo pelo qual o protocolo WebSocket atualiza uma conexão HTTP comum para uma conexão WebSocket, permitindo comunicação bidirecional em tempo real.

Em resumo, o termo handshake refere-se a esse processo inicial crucial de duas entidades se reconhecerem e sincronizarem para estabelecer um canal seguro de comunicação antes de efetivamente trocarem dados. Conforme a tecnologia, diferentes passos são necessários para garantir uma conexão confiável.
```

### connection draining
```
Connection draining é uma função importante provida pelos Elastic Load Balancers (ELBs) da Amazon Web Services (AWS) para graciosamente remover instâncias do serviço.

Quando você remove uma instância do seu Auto Scaling Group, o ELB pára de enviar novas conexões para essa instância, mas as conexões existentes podem ser interrompidas abruptamente.

O connection draining resolve isso garantindo que as requisições existentes terminem gracefully antes de remover a instância de serviço. Aqui estão mais detalhes:

- Você define por quanto tempo o ELB deve drenar as conexões existentes antes de completar a remoção da instância. Isso é configurado em segundos.

- Durante esse período de draining, o ELB não envia novas requisições para a instância, mas permite que requisições existentes sejam concluídas normalmente.

- Isso evita a interrupção abrupta de downloads, transações ou outros trabalhos em andamento que poderiam afetar negativamente o usuário.

- Quando o período definido expira, então o ELB para de enviar até mesmo as conexões existentes para aquela instância.

Habilitar o connection draining é considerado uma boa prática ao criar Auto Scaling groups com Elastic Load Balancers para garantir graceful shutdowns.

```

### asg (auto scaling group)
- uma forma de escalonar horizontalmente nossas ec2
- definimos um número minimo, deseja e máximo de instancias
- o load balance utiliza as ec2 criadas pelo asg
- usamos as trigger do cloudwatch (alarmes), para saber quando aumentar as instancias por exemplo
- etapas para criação:
  - crie um template
  - crie o asg, vinculando a um target group
  - vincule o tg no seu alb
 
### politicas de asg
```
Existem vários tipos de políticas de escalabilidade automática na AWS que podemos usar com Auto Scaling groups:

Target Tracking Scaling:
- Escala automaticamente para manter uma métrica específica em um determinado valor alvo. Por exemplo, manter a CPU em 40%.
- Não requer configuração complexa de alarmes.
- Responde continuamente a flutuações na métrica.

Simple/Step Scaling:  
- Escala com base em alarmes do CloudWatch associados a métricas. 
- Permite adicionar ou remover uma quantidade fixa de capacidade.

Scheduled Actions:
- Escala a capacidade de acordo com uma agenda recorrente, antecipando mudanças de tráfego. 
- Útil para flutuações previsíveis.

Predictive Scaling:
- Usa machine learning para prever o tráfego futuro com base em padrões históricos.
- Inicia o scale antes que a métrica exceda o limite para manter desempenho.

Em resumo, podemos combinar vários tipos de políticas de escala para obter controle refinado sobre o scale up/down automático em resposta tanto ao tráfego ao vivo quanto a padrões históricos. Isso ajuda a manter a estabilidade do aplicativo e otimizar custos.
```

### scaling cooldowns
```
Scaling Cooldowns são um importante recurso de Auto Scaling na AWS para controlar a frequência das atividades de escalabilidade (scale up/down).

Alguns pontos-chave sobre Scaling Cooldowns:

- Um default de 300 segundos é definido no Auto Scaling Group, mas você pode configurar explicitamente o valor.

- O cooldown aplica uma pausa após a conclusão de uma atividade de escalabilidade antes da próxima. 

- Isso dá tempo para que as métricas se estabilizem antes de tomar mais ações.

- Se uma alta demanda ainda persistir após o cooldown, um novo scale out pode ser acionado.

- Cooldown evita "thrashing" - criando/excluindo instâncias muito rapidamente o que pode perturbar aplicativos.

- Valores comuns variam de poucos minutos para cargas de trabalho com tráfego volátil ou até horas para backends mais estáveis.

- Também se aplica para diminuir a escala quando a demanda cai.

Definir well-tuned scaling cooldowns, junto com outras políticas balanceadas permite estabilidade, performance e custo ideal no Auto Scaling!

Então em resumo, os scaling cooldowns adicionam atraso intencional, mas são cruciais para controlar a taxa de mudança.
```
