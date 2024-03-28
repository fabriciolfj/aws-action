# aws-action
(exame https://awscertificationpractice.benchprep.com/app/aws-certified-solutions-architect-associate-official-practice-question-set-saa-c03?locale=pt-br#exams/details/165791)
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
Máquinas Spot são instâncias EC2 que aproveitam a capacidade ociosa da AWS para conseguir economias significativas em comparação com as instâncias sob demanda. Alguns pontos importantes:

- São instâncias que podem ser interrompidas pela AWS com um aviso de 2 minutos se a capacidade for necessária para outras cargas de trabalho.

- Como utilizam capacidade excedente, são disponibilizados com descontos de até 90% comparado ao preço sob demanda.

- Úteis para cargas de trabalho flexíveis, que podem tolerar interrupções, como processamento em lotes, análises, transcodificação de mídia etc.

- Não são adequadas para aplicações com estado importante ou missão crítica.

- É possível definir o preço máximo por hora que você está disposto a pagar. Se o preço do Spot no momento ultrapassar o seu preço máximo, a instância é encerrada.

- Disponíveis em todas as famílias de instâncias e tipos de AWS.

Em resumo, Spot Instances permitem economia em troca da flexibilidade na dispobilidade de capacidade, sendo úteis para workloads tolerantes a falhas.

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
```
 Golden AMI (Amazon Machine Image) refere-se a uma imagem de máquina da AWS (AMI) que é tratada como o padrão ouro ou autoritativo dentro de um ambiente. Algumas características importantes das Golden AMIs:

- São AMIs base/customizadas que foram otimizadas e hardened previamente com configurações de segurança, sistemas operacionais, pacotes e aplicações necessárias.

- Agem como uma imagem base consistente para permitir deployments rápidos e confiáveis de instâncias idênticas.

- São imutáveis e versões novas são criadas a partir de builds automatizados, não sendo modificadas depois da criação.

- Permitem escalar rapidamente a infraestrutura para atender demandas, pois usam a mesma AMI base confiável.

- Facilitam o gerenciamento de atualizações e versões, já que mudanças podem ser introduzidas criando novas AMIs douradas.

- Seguem práticas recomendadas de segurança e conformidade desde a fase de construção.

- Reduzem erros e inconsistências entre ambientes, pois todos usam a mesma linha de base.

Então as Golden AMIs são cruciais para ambientes consistentes, escaláveis e seguros em nuvem. Elas trazem velocidade e confiança para deployments.

Golden AMI é uma imagem que contém todo o seu software instalado e configurado para que futuras instâncias do EC2 possam inicializar rapidamente a partir dessa AMI
```

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

 O desempenho básico de E/S para armazenamento SSD de uso geral é de 3 IOPS para cada GiB. Para 334 GiB de armazenamento, o desempenho básico seria de 1.002 IOPS. Além disso, o armazenamento SSD de uso geral é mais econômico do que o armazenamento de IOPS provisionado.

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
- O Network Load Balancer tem um endereço IP estático por AZ e você pode anexar um endereço IP elástico a ele. Application Load Balancers e Classic Load Balancers têm um nome DNS estático.

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
Em resumo, o SNI é crucial para escalar implantações de terminação TLS que precisam suportar vários certificados SSL no mesmo endereço IP. É uma extensão importante para segurança e desempenho na cloud.
```

### handshake
```
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
```
O Auto Scaling Group e o Target Group são dois conceitos diferentes, mas relacionados, no contexto do gerenciamento de recursos computacionais na nuvem, especificamente no serviço Amazon Elastic Load Balancing (ELB) e Amazon Elastic Compute Cloud (EC2) da AWS.

Auto Scaling Group (Grupo de Auto Scaling):
- É um componente do serviço Auto Scaling da AWS, que é responsável por escalar automaticamente os recursos de computação (instâncias EC2) com base em métricas definidas, como a utilização de CPU, tráfego de rede ou métricas personalizadas.
- Permite que você defina regras para adicionar ou remover instâncias EC2 de acordo com a demanda, garantindo que haja recursos suficientes para atender às cargas de trabalho.
- O Auto Scaling Group gerencia o ciclo de vida das instâncias EC2, criando novas instâncias quando necessário e terminando instâncias ociosas.

Target Group (Grupo de Destino):
- É um componente do serviço Elastic Load Balancing (ELB) da AWS, responsável por receber e encaminhar o tráfego de entrada para as instâncias EC2 registradas.
- Um Target Group é configurado com um protocolo (HTTP, HTTPS, TCP, etc.) e uma porta de destino, para os quais o balanceador de carga encaminhará o tráfego.
- As instâncias EC2 são registradas no Target Group, e o balanceador de carga distribui o tráfego entre essas instâncias de acordo com o algoritmo de balanceamento de carga configurado.

A correlação entre o Auto Scaling Group e o Target Group é que, geralmente, as instâncias EC2 gerenciadas pelo Auto Scaling Group são registradas no Target Group do Elastic Load Balancing. Dessa forma, quando novas instâncias EC2 são criadas pelo Auto Scaling Group para lidar com o aumento da demanda, elas são automaticamente registradas no Target Group, tornando-se disponíveis para receber tráfego do balanceador de carga.

Por outro lado, quando o Auto Scaling Group termina instâncias EC2 devido à diminuição da demanda, essas instâncias são automaticamente desregistradas do Target Group correspondente.

Essa integração entre o Auto Scaling Group e o Target Group permite que sua arquitetura de aplicação seja escalável e tolerante a falhas, garantindo que haja sempre recursos suficientes para atender à demanda, e que o tráfego seja distribuído de maneira eficiente entre as instâncias disponíveis.
```

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

# RDS
## read replicas e multiza
```
Read Replicas e Multi-AZ são recursos importantes do Amazon RDS (serviço de banco de dados gerenciado) para aumentar a disponibilidade e escalabilidade:


Read Replicas:

- Réplicas somente leitura da sua instância de banco de dados RDS.

- Permitem escalar horizontalmente a capacidade de leitura distribuindo carga.

- As operações de leitura podem ser isoladas nas réplicas. A escrita continua apenas no master.

- Réplicas estão disponíveis dentro da mesma região. (podemos colocar em região diferente e habilitar o multi-az nela)


Multi-AZ:  
- nao precisamos mudar a cadeia de conexão

- O RDS cria um standby da instância do banco em outra zona de disponibilidade da mesma região.

- Para proteger contra falhas na zona ou infraestrutura, promovendo alta disponibilidade. 

- Em caso de falha ou perda do banco principal, o RDS faz failover para o standby com endereço IP diferente, porém transparente para aplicação.

- Não requer mudanças na lógica do aplicativo para usar a nova instância primária.

Em resumo, o Multi-AZ é para alta disponibilidade enquanto as Read Replicas são para aumentar o desempenho de leitura escalando horizontalmente. Os dois podem ser usados juntos para ter ambos benefícios.

Em relação a replicação de dados:
 - assincrona em replicas
 - sincrona em multi-az
```

## rds backup
- backups manuais não expliram, ja os automatizados sim
- caso queria guardar os dados de um banco de dados, faça bkp dele e apague-o, pois o bkp e mais barato o armazenamento.
- a restauração de um bkp, criará um novo banco de dados
- tem um recurso de clonagem para banco de dados aurora, que não precisamos para a base e ele gera um novo pra nós

## rds/aurora security
- podemos criptografar os nosso banco de dados nos volumes
- se eu não encriptar o master, as réplcias também não serão encriptadas
- rds não temos acesso via ssh, pois são serviços gerenciados, a não ser um rds customizavél

## rds proxy
```
O RDS Proxy é um recurso do Amazon RDS que provê um proxy SQL transparente e altamente disponível entre aplicações e bancos de dados. Os principais benefícios são:

- Agregação de conexões de aplicativos para o banco de dados, aumentando utilização de recursos.

- Conexões são multiplexadas através do proxy, reduzindo overhead de IO entre app e banco.

- O proxy é altamente disponível, com failover automático entre zonas. Remove ponto único de falha.

- Facilita administração e security, por exemplo limitando conexões às subnets de aplicação e integrando com IAM Authentication. 

- Suporta split horizon DNS, direcionando leitura para réplicas e escrita para primário transparentemente.

- Permite upgrade sem impactos removendo conexões diretas aos bancos.

Em resumo, o RDS Proxy melhora escalabilidade, performance, disponibilidade e segurança entre aplicações e bancos de dados na AWS. 

Ele abstrai e gerencia conexões de forma inteligente, além de habilitar routings avançados. Tudo mantendo compatibilidade com frameworks e linguagens existentes.


RDS Proxy ajuda a reduzir o estresse e sobrecarga de conexões no banco de dados de várias maneiras:

- Agregação de Conexões - Múltiplas conexões do aplicativo são consolidadas pelo proxy, reduzindo a carga de conexões que chega ao RDS.

- Reuso de Conexões - Uma pool de conexões é mantida no proxy ao invés de reconnects constantes ao RDS. Isso reduz sobrecarga de establichment de conexões.

- Pooling de Transações - Sessões de banco de dados são reutilizadas pelo proxy para múltiplas queries de aplicativo, sem precisar finalizar e reabrir sessões a todo momento.

- Limitador de Taxa - As taxas de solicitações e conexões podem ser limitadas para aliviar sobrecarga repentina no RDS.

- Monitoramento Embutido - O proxy tem visibilidade de queries problemáticas que podem ser otimizadas.

- Failover Automático - Em caso de problemas no RDS, o proxy faz retry transparente e encaminha para uma instância íntegra.

Então sim, ele atua como um escudo inteligente de acesso ao banco, encarregando-se de gerenciar e consolidar conexões de forma mais eficiente.
```

## aurora
```
O Amazon Aurora é um serviço de banco de dados relacional gerenciado e proprietário da AWS com foco em desempenho e alta disponibilidade. Alguns aspectos importantes:

- Compatível como MySQL e PostgreSQL, permitindo migração de cargas de trabalho existentes.

- Entrega até 5x melhor performance que o MySQL tradicional e 3x mais rápido que o PostgreSQL. Escala até 64TB de dados.

- Arquitetura altamente paralela com replicação contínua para redundância e tolerância a falhas. 

- O armazenamento é separado do computo, permitindo escala independente. O volume de armazenamento cresce e encolhe dinamicamente.

- Disponibiliza replicação sem esforço entre zonas de disponibilidade e automatic Failover em menos de 30 segundos em caso de perda na zona principal.

- Backup contínuo e restauração pontual são feitos sem impactar as operações do banco com a capacidade de fazer snapshots em segundos. 

- Opções de memória, armazenamento e nós computo que podem ser trocados online conforme a necessidade da carga de trabalho.

Em resumo, o Aurora entrega alta performance, disponibilidade e tolerância a falha focado em workloads críticas que exigem banco de dados robusto e confiável. Além de ser escalável na AWS.


Amazon Aurora permite a criação de réplicas de leitura (Read Replicas) para escalar horizontalmente e separar o tráfego de leitura do de escrita.

Alguns pontos importantes sobre as Réplicas de Leitura no Aurora:

- Até 15 réplicas podem ser criadas a partir da instância principal de banco de dados.

- As operações de gravação e mutação de dados ocorrem somente no banco de dados primário (writer instance).

- Já as consultas de dados, relatórios e outras queries somente-leitura podem ser direcionadas para uma das read replicas.

- Isso permite escalar a camada de leitura distribuindo a carga entre as várias instâncias de réplica.

- As replicações para as read replicas são feitas de forma assíncrona, rápida e automatizada.

- Em caso de falha no banco de dados primário, o Aurora faz failover promovendo automaticamente uma das read replicas como a nova instância de escrita primária.

Dessa forma o Aurora aproveita o melhor da replicação síncrona e assíncrona para escalar enquanto mantém alta disponibilidade. As Read Replicas são uma parte fundamental desse design distribuído.


Quando você cria réplicas de leitura (Read Replicas) no Amazon Aurora, o próprio serviço de banco de dados gerencia o roteamento das conexões para as várias instâncias em segundo plano.

Você não precisa se preocupar com os detalhes como endereços IP ou configurações específicas de cada read replica. Basta interagir com o endpoint do Writer (banco primário) que todas as queries somente leitura são automaticamente enviadas para uma das instâncias disponíveis de réplica.

Essa é uma grande vantagem do Aurora sobre gerenciar replicas por conta própria. Você tem:

- Criação e exclusão automatizada das replicas conforme a necessidade
- Failover e eleição de nova instância primária é transparente
- Balanceamento de carga entre as instâncias de réplica é feito builtin  
- Aplicação se conecta sempre ao mesmo endpoint lógico

Ou seja, tudo é abstraído e gerenciado pelo Aurora como um serviço. Você só dimensiona o número desejado de Read Replicas para aumentar a capacidade de leitura, sem precisar gerenciar cada réplica individualmente.

Isso simplifica muito a escalabilidade e também reduz o overhead operacional.
```

### aurora serverless e aurora global
```
**Serveless no Aurora**

O Aurora Serverless é uma opção de provisionamento do Aurora que permite que você execute cargas de trabalho sem ter que gerenciar a infraestrutura do banco de dados. Com o Aurora Serverless, você só paga pelo que usa, e o serviço dimensiona automaticamente para atender às suas necessidades.

**Benefícios do Aurora Serverless**

* **Sem gerenciamento de infraestrutura:** Com o Aurora Serverless, você não precisa se preocupar em provisionar, dimensionar ou gerenciar a infraestrutura do banco de dados. O serviço faz tudo isso para você.

* **Dimensionamento automático:** O Aurora Serverless dimensiona automaticamente para atender às suas necessidades. Isso significa que você não precisa se preocupar em ficar sem recursos ou ter que dimensionar manualmente o banco de dados.

* **Pagamento por uso:** Você só paga pelo que usa com o Aurora Serverless. Isso significa que você não precisa se preocupar em pagar por recursos que não está usando.

**Quando Usar o Aurora Serverless**

O Aurora Serverless é uma boa opção para aplicativos que:

* São sensíveis à latência
* São altamente imprevisíveis
* Têm picos de tráfego
* Requerem alta disponibilidade
* Não requerem controle preciso sobre a infraestrutura do banco de dados

**Conclusão**

O Aurora Serverless é uma opção de provisionamento do Aurora que oferece vários benefícios, incluindo sem gerenciamento de infraestrutura, dimensionamento automático e pagamento por uso. O Aurora Serverless é uma boa opção para aplicativos que são sensíveis à latência, altamente imprevisíveis, têm picos de tráfego, requerem alta disponibilidade e não requerem controle preciso sobre a infraestrutura do banco de dados.

e o aurora global **Aurora Global: Conceito Geral**

O Aurora Global é um recurso do Amazon Aurora que permite que você tenha um único cluster de banco de dados que abrange várias regiões da AWS. Isso significa que você pode ter dados replicados em várias regiões, o que proporciona alta disponibilidade e resiliência.

**Benefícios do Aurora Global**

* **Alta disponibilidade:** Com o Aurora Global, seus dados são replicados em várias regiões, o que garante que eles estejam sempre disponíveis, mesmo que uma região fique indisponível.

* **Resiliência:** O Aurora Global também é resiliente a falhas de hardware e software. Se um nó do banco de dados falhar, o serviço automaticamente redirecionará o tráfego para outro nó.

* **Escalabilidade global:** O Aurora Global permite que você dimensione facilmente seu banco de dados para atender às suas necessidades globais. Você pode adicionar ou remover regiões conforme necessário.

**Quando Usar o Aurora Global**

O Aurora Global é uma boa opção para aplicativos que:

* São globais em escopo
* Requerem alta disponibilidade e resiliência
* Precisam escalar facilmente para atender às necessidades globais

**Conclusão**

O Aurora Global é um recurso do Amazon Aurora que permite que você tenha um único cluster de banco de dados que abrange várias regiões da AWS. O Aurora Global oferece alta disponibilidade, resiliência e escalabilidade global. O Aurora Global é uma boa opção para aplicativos que são globais em escopo, requerem alta disponibilidade e resiliência e precisam escalar facilmente para atender às necessidades globais.
```

# Elastic cache
```
O Amazon ElastiCache é um serviço web da AWS para implementar caches de alto desempenho e disponibilidade na nuvem. Os principais destaques são:

- Permite criar clusters compatíveis com Redis ou Memcached, os dois principais sistemas de cache na memória.

- Os nós do cluster ficam na VPC para segurança e baixa latência com aplicações AWS.

- Alta disponibilidade com replicação automática e failover gerenciados.

- Provê opções otimizadas tanto para desempenho (instâncias espeficas para cache), quanto custo eficiente (baseadas em HW commodities).

- Integra com serviços como Auto Scaling Groups e Node Groups para provisionamento simplificado.

- Fácil monitoramento de métricas do cache como uso de memória, conexões, misses, etc. 

- Suporta backups automatizados também.

Em suma, o ElastiCache facilita implementar soluções avançadas de cache na memória que superam desafios operacionais de disponibilidade, durabilidade, escala e monitoramento. Provendo substrato de cache rápido e robusto para aplicações críticas.

As principais diferenças entre Redis e Memcached são:

TIPO DE ARMAZENAMENTO:

- Redis: Armazena estruturas de dados na memória (strings, hashes, lists, sets, etc). Permite operações avançadas nessas estruturas.

- Memcached: Armazena dados simples na forma chave-valor. Sem abstrações de dados complexas.

PERSISTÊNCIA:

- Redis: Dados podem ser persistentes em disco para durabilidade se necessário.

- Memcached: Cache estritamente na memória RAM. Os dados não persistem a reinícios ou falhas.


RECURSOS:

- Redis: Mais recursos como suporte a transações, pub/sub, Lua scripting, maior flexibilidade.

- Memcached: Mais simples, apenas focado em cache na memória de alto desempenho.


Em resumo, o Redis provê um conjunto de dados e recursos mais rico e com mais opções de persistência, enquanto o Memcached foca exclusivamente em cache na RAM de alta velocidade.

Iam é suportado apenas para redis (alem do iam para usar o recurso, redis tambem pode usar usuário e senha, como mais uma camada de segurança, e alem do ssl)
Memcache usa apena user/password

```


# ROTA 53
```
 O Amazon Route 53 é o serviço de DNS escalável e de alta disponibilidade oferecido pela AWS. Alguns pontos importantes:

- Permite gerenciar e fazer consultas de nomes de domínio de forma altamente confiável e de baixa latência.

- Serve autoritativamente bilhões de consultas de DNS por dia para domínios registrados na Route 53.

- Integra-se facilmente com outros serviços AWS como registros alias para balanceadores de carga e instâncias EC2.

- Oferece encaminhamento geográfico de tráfego por meio do DNS, latência mínima e failover entre regiões. 

- Disponibiliza monitoramento de integridade de endpoints por meio de health checks. Se um endpoint falhar os checks, ele é automaticamente removido.

- Tem recursos avançados como split horizon DNS para separar tráfego de leitura e escrita.

- Cobrança é baseada no uso, permitindo escalar para qualquer volume de tráfego sem custos fixos.

Dessa forma o Route 53 fornece serviços DNS altamente escaláveis, confiáveis e de baixa latência críticos para aplicações na AWS.


Registros de DNS:
O Route 53 suporta todos os principais tipos de registros DNS como A, AAAA, CNAME, MX, TXT, etc. Isso mapeia nomes de domínio para endereços IP, outros nomes (aliases) ou informações de roteamento de email/tráfego.

Encaminhamento Geográfico de Tráfego:
Permite que o tráfego de usuários seja encaminhado para endpoints na região AWS mais próxima. Isso reduz latência e melhora performance percebida pelo usuário final ao ser direcionado para infra mais perto geograficamente.

Health Checks:
Os health checks fazem requisições periódicas dos endpoints e apenas encaminham tráfego se obtiverem respostas sucesso. Isso remove automaticamente da rotação instâncias não íntegras. Vários protocolos suportados (HTTP, TCP, HTTPS com sni).

DNS Privativo:
Permite criar zonas DNS customizadas apenas acessíveis internamente pela sua VPC. Útil para nomes de recursos internos não exposer publicamente. Integra com DHCP para fácil uso dos nomes em instâncias EC2.

Monitoramento:
O Route53 fornece métricas detalhadas sobre consultas de DNS como latência, requests por segundo, erros etc. Integra com CloudWatch para construir dashboards e alarms.

Hosted Zones: 

- Uma hosted zone é uma coleção de registros DNS para um domínio específico que você gerencia no Route53. Por exemplo, uma hosted zone para meuapp.com.

- Ela contém os registros DNS que definem como tratar requests para nomes de subdomínios como www.meuapp.com ou api.meuapp.com.

- Você pode ter várias hosted zones para múltiplos domínios gerenciados no Route53.

Registros CNAME:

- Um registro CNAME mapeia um nome DNS para outro nome DNS, não diretamente para um IP. 

- Úteis para apontar subdomínios para outros hosts. Por exemplo: 
api.meuapp.com CNAME meuapp.azurewebsites.com

- Também usados para mapear domínios apex (meuapp.com) para recursos Route53 como Load Balancers.

Registros A e AAAA:

- Registram A mapeiam um nome de host para endereços IPv4.
- Registros AAAA mapeiam um nome de host para endereços IPv6.
- Usados para mapear nomes como www.meuapp.com para endereços IP finais.

Record
Recordes (registros) de DNS são as unidades fundamentais de informação armazenadas em um domínio no sistema de nomes de domínio (DNS). Cada registro DNS mapeia um nome de domínio específico para algum tipo de dado, como endereço IP, outro nome de domínio ou informações de roteamento de e-mail.

Alguns dos principais tipos de registros DNS:

- A: Mapeia um nome de domínio para endereço IPv4 (ex: 192.168.1.1)

- AAAA: Mapeia um nome de domínio para endereço IPv6 

- CNAME: Mapeia um nome de domínio para outro nome de domínio

- MX: Especifica qual servidor de e-mail é responsável por receber e-mails destinados a esse domínio

- TXT: Permite associar texto arbitrário com o nome do domínio

- NS: Especifica quais servidores de nomes são autoritativos para aquele domínio

Cada registro possui um tipo (A, CNAME etc) além de outras informações como nome do domínio mapeado e qualquer dado associado, como endereços IP. 

O conjunto de todos os registros é o que permite que requisições de usuários ou aplicativos sejam roteadas para o destino certo com base no nome solicitado.

TTL
O TTL (Time To Live) em um registro DNS na AWS Route53 determina por quanto tempo uma resposta DNS pode ser cacheada pelos resolvers e quando deverá ser atualizada.

Alguns pontos importantes sobre o TTL do Route53:

- Valores baixos (por exemplo 60 segundos) fazem o resolver DNS reconsultar frequentemente as mudanças nos registros. Útil quando os IPs são alterados constantemente.

- Valores mais altos (1 dia ou mais) reduzem carga no servidor DNS pois permitem que os resolvers cachem por mais tempo antes de precisar reconsultar.

- Valores altos funcionam bem para servidores estáticos ou mudanças pouco frequentes. Evita consultas desnecessárias.

- O padrão do Route53 é 48 horas, um balanceamento entre estabilidade e capacidade de atualizar mudanças em dias.

- O TTL define o compromisso entre consistência e eficiência do cache do DNS. Valores muito longos impedem atualizações rápidas.

Portanto o TTL ajuda a controlar quanto os clientes DNS podem cache localmente as respostas do Route53 antes de precisar obter atualizações mais recentes.


CNAME vs Alias
 A principal diferença entre registros CNAME e Alias no Amazon Route 53 é:

CNAME:

- Mapeia um nome DNS para outro nome DNS (não diretamente para endereço IP).
- Requer lookups DNS adicionais para resolver o nome de destino antes de chegar nos IPs.
- Funciona entre zonas DNS e provedores DNS.
- Não reflete automaticamente mudanças nos IPs dos recursos de destino.
- nao funciona para dominio raiz, ex: fabricio.net (tem que te algum antes app.fabricio.net)
- podemos definir um ttl (tempo de vida do cache no cliente, para não fazer muitos lookups dns, que acarreta em custos)

Alias:

- Mapeia diretamente de um nome DNS para um recurso AWS (ELB, CloudFront, etc). 
- Encaixa perfeitamente serviços AWS, sem lookups extras.
- Resolve sempre para endereços IP atuais dos recursos referenciados.
- Alterações nos recursos (ex: IP do ELB) são automaticamente refletidas.
- Restringe-se a mapeamentos dentro da AWS.
- funciona para domínios raiz e não raiz.
- record alias é sempre do tipo A/AAAA para aws resorces (IPV4, IPV6)
- ttl
```
Cada registro DNS possui um TTL (Time To Live) que ordena aos clientes por quanto tempo armazenar esses valores em cache e 
não sobrecarregar o DNS Resolver com solicitações DNS.
O valor TTL deve ser definido para atingir um equilíbrio entre quanto tempo
o valor deve ser armazenado em cache e quantas solicitações devem ir para o Resolvedor DNS.
````
- nao podemos colocar um alias para um ec2 dns
- alias funciona para:
  - elastic load balancer
  - amazon cloudfront
  - aws api gwt
  - elastic beanstalk
  - s3 websites
  - vpc interface endpoints
  - global acclerator
  - route53 record

Em resumo, os registros Alias provêm integração mais simples e eficiente com recursos AWS, enquanto CNAMEs são compatíveis entre DNSs mas requerem mais processamento.


Politica de roteamento (routing policy)

Roteamento Simples:
Um único registro DNS que roteia para um recurso.
Não há checagem de integridade ou failover.
Use quando tiver um único recurso/destino.

Roteamento com Falha:
Cria registros de DNS primário e secundário.
Faz health check do primário, e em caso de falha roteia para o secundário.
Permite definir failover automático entre recursos.

Roteamento Baseado em Latência:
Retorna o recurso/região de menor latência para o usuário com base na localização geográfica.
Otimiza tempo de resposta ao direcionar usuários para infraestrutura próxima.

Roteamento Ponderado:
Define pesos (%) para vários recursos/registros de destino.
O tráfego é distribuído proporcionalmente de acordo com os pesos.
Útil para balanceamento de carga e consistência de capacidade.

Roteamento de Latência Múltipla:
É uma extensão do roteamento de latência, permitindo especificar múltiplas regiões AWS como destino.
As consultas DNS são respondidas com todas as opções, para que o cliente selecione a latência mínima.

Geoproximidade de Tráfego (Geolocation):
Direciona os usuários para destinos específicos com base nas geolocalizações de onde partem as requisições.
Permite personalizar comportamento com base no país ou continente de origem.
exemplo: para o Brasil vem ser direcionado para esse ip

Respostas DNS Multivaloradas (Multi Value):
Retorna até 8 registros DNS aleatoriamente embaralhados em cada resposta.
Útil para distribuir carga entre vários destinos.
Não substitui o alb, ideia é efetuar o balanceamento do lado do cliente, pois ele recebe todos os ips vinculados ao dns

Geoproximity Routing Policy
Coloco pesos nas regiões, região com maior peso atrai mais usuarios, como se expandi-se essa região para englobar mais usuários:
exemplo: um pais 2 regiões, direta e esquedar, se ambas com peço 50, os usuários deste pais dicaria metade em uma az e outra em outra az

Routeamento baseado em ip
Direciona para o dns com base no block de ip do cliente. Exemplo:
 example.com na maquina 1, block 200.0.0.0, cliente com essa faixa vai para essa maquina
 example.com na maquina 2, block 233.0.0.0, cliente com essa faixa vai para essa maquina

Route53 health check
 O Amazon Route 53 oferece várias opções de health checks que podem ser configuradas para monitorar a disponibilidade e funcionalidade de endpoints. Aqui estão as principais opções em detalhe:
- HTTP/HTTPS: verifica se um endpoint HTTP ou HTTPS está respondendo códigos de resposta HTTP esperados (como 200 OK). Permite configurar string para check na resposta.
- TCP: verifica se uma porta TCP está aberta em um endpoint. Útil para check de disponibilidade de banco de dados, aplicações sem interface web, etc.
- HTTP(S) proxy: similar ao HTTP/HTTPS normal, mas passa o check por um proxy AWS. Útil quando o endpoint está em uma VPC privada.
- CloudWatch alarm: ao invés de fazer checks próprios, monitora alarmes do CloudWatch como métrica de disponibilidade.
- Ping path: envia pings ICMP a endpoints para verificar apenas disponibilidade básica de rede.
- Calculated/composite: combina resultados de outros checks para criar uma lógica customizada de disponibilidade.
Cada tipo de check permite configurar threshold de falhas, frequência de checks, falha para outras regiões AWS, e integração com sistemas de monitoring.
Alguns recursos avançados incluem monitoramento de certificados SSL, validação de strings em requests/responses, autenticação em checks HTTP, entre outros.
Os health checks do Route53 são uma maneira eficiente de monitorar endpoints críticos e configurar failovers automatizados.
````

# S3
- utilizados para guardar dados, videos e etc
- podemos colocar um arquivo de até 5 tera
- acima de 5 tera usamos o multipart
- para ter acesso ao bucket, temos que vincular policies a role ou iam user.
- outra conta pode acessar nosso bucket, isso é chamado de acesso cruzado
- embora o nome do bucket é global, ele é vinculado a região em que foi criado
- alem das policies (restrições vinculadas direta no nosso bucket), temos também as acls, um segundo mecanismo de segurança para nosso bucket.
  - site para gerar policies https://awspolicygen.s3.amazonaws.com/policygen.html
```
 Há algumas diferenças importantes entre colocar políticas de acesso diretamente em um bucket S3 versus em um usuário IAM ou função:

- Políticas no bucket (resource-based) são específicas e limitadas apenas aquele bucket. Já políticas IAM podem dar acesso ampla a vários recursos.

- Políticas em bucket sobrescrevem políticas IAM, ou seja, um deny no bucket vai sobrescrever qualquer allow do IAM do usuário para aquele recurso específico.

- Se você negar acesso direto no bucket, o usuário não conseguirá acessar aquele objeto mesmo que tenha permissão via IAM. O deny sempre sobrescreve o allow.

- Políticas em bucket não podem ter condições complexas ou sofisticadas como as de IAM.

- Administrar no bucket facilita isolar acessos por ambientes ou tipo de dados. Via IAM fica mais ampla.

Então em resumo, se você colocar um deny explícito no bucket, o usuário não conseguirá acessar mesmo que seu IAM permita. 
Estrategicamente políticas em bucket e IAM se complementam para dar os acessos necessários e também restringir o que for preciso.
```
### Versionamento no s3
- posso ativar o versionamento no s3, nesse caso, ao realizar o upload de um arquivo da mesma chave (mesmo nome), o existente será versionado e o novo asumirá
- para excluir a versão correnet, somente selecionar e excluir 
- para excluir o arquivo inicial, ele ficará marcado para delete, para restaurar so tentar excluí-lo
- lembrando que os arquivos ja existentes no bucket, caso seja marcado o versionamento depois, ficaram com a version null.


### replicação no s3 (precisamos do versionamento habilitado nos buckets de origem e destino)
- temos dois tipos de replicação no s3:
  - CRR -> replicação em diferentes regiões, uso -> para compliance, baixa latencia
  - SRR -> replicação na mesma região, uso -> replicar dados para ambientes diferentes como prod e qa
- a replicação ocorre para novos objetos, após sua ativação
- para objetos existentes, antes da ativação da replicação, temos o recurso batch replication
- O não encadeamento de replicação de buckets no S3 significa que quando você configura a replicação de um bucket origem para um bucket de destino,
  esse bucket de destino não continuará replicando os objetos para um outro bucket.
- podemos tambem mandar arquivo marcado para deleção na réplica (por default isso e desmarcado), lembrando que essa marcacao e para o primeiro arquivo (não tenho mais versoes)
- ao deletar uma versão especifica, e uma delecao permanente

### politicas do clico de vida
```
 As políticas de ciclo de vida são recursos do Amazon S3 que permitem definir ações automáticas para transpôr, arquivar ou expirar objetos após um determinado período ou evento.

Alguns pontos importantes:

- Permite definir regras baseadas no tempo decorrido desde a última modificação, criação ou acesso ao objeto

- podemos definir tempo de expiracao ou arquivos que tiveram seu upload interrompido (inacabado), para serem excluidos

- As ações suportadas incluem mover entre camadas de acesso, arquivo em camadas Glacier e remoção permanente 

- Podem ser usadas para automatizar descomissionamento de dados antigos, mover entre tiers de custo-benefício ou limpeza de objetos temporários

- As regras são definidas no bucket, podendo se aplicar a todos objetos ou utilizar filtros por prefixo ou tags para subgroups específicos

- Ajuda a governar dados para conformidade com políticas de retenção, além de gerenciar custos e necessidades funcionais

- Integra-se com todas classes de armazenamento S3 como Standard, IA, Glacier e Intelligent Tiering

Alguns casos de uso são: mover logs após X meses para Glacier, deletar dados processados após Y dias, transicionar para camadas IA após Z dias sem uso.

As regras permitem lidar com todo ciclo de vida dos dados de modo granular e automatizado.
```

### classes de armazenamento do s3
```
 As classes de armazenamento do Amazon S3 definem como os dados são armazenados e permitem balancear custo, disponibilidade e desempenho:

S3 Standard:

- Armazenamento de uso geral padrão, balanceia custo e alta disponibilidade.

- Projetado para sustentar a perda de 2 instâncias simultâneas sem perda de disponibilidade e durabilidade de dados de 99,999999999%.

- Replica os dados em múltiplas instâncias em uma região. 

- Ideal para backups, big data, aplicações web, arquivos e mobile.

S3 Standard - Infrequent Access: 

- Para dados acessados com menor frequência, mas requer rápido acesso quando necessário.

- Preço de armazenamento mais baixo que o Standard, mas custo de acesso maior.

- Disponibilidade e durabilidade de dados iguais ao S3 Standard.

S3 One Zone-Infrequent Access:

- Versão de menor custo do S3 Standard-IA. 

- Replica os dados em uma única zona de disponibilidade (AZ) apenas.

- Menor disponibilidade e durabilidade comparado ao Standard (99,5%). 

- 20% mais barato que o Standard-IA.

S3 Glacier e S3 Glacier Deep Archive:

- Armazenamento de objetos para dados muito pouco acessados (arquivamento).

- Preços muito baixos de armazenamento, porém custo alto para acesso rápido aos dados.

- Diferentes políticas temporais para acesso aos dados (de minutos a horas).
  - amazon s3 glacier instant retrieval -> recupera o dado em milisegundos, tempo minimo de armazenamento deve ser de 90 dias
  - amazon glacier flexible retrieval -> pode recuperar o dado de 1 a 5 minuots, 3 a 5 horas ou de 5 a 12 horas (gratuito este),  tempo minimo de armazenamento deve ser de 90 dias
  - amazon glacier deep archive -> 12 a 48 horas (menor custo), tempo minímo de armazenamento deve ser 180 dias

- Ideal para backups, arquivamento e conformidade.


S3 Intelligent-Tiering

- Armazenamento de objetos ideal para dados com padrões de acesso desconhecidos ou imprevisíveis. 

- Move automaticamente objetos entre duas camadas acesso frequente e acesso pouco frequente com base em padrões de acesso.

- Balanceia otimização de custos com desempenho e disponibilidade.

- Não precisa definir ou gerenciar as camadas manualmente.

- Oferece alta disponibilidade e durabilidade igual ao S3 Standard (99,999999999%).

- Mais caro que o S3 Standard-IA, porém não cobra pelo monitoramento e movimentação entre tiers.
Ele acompanha métricas de acesso aos objetos como quantidade e frequência.
Com base em regras pré-definidas e limites quantitativos, ele decide se deve mover objetos entre as camadas.
Por exemplo, objetos com X acessos por mês são movidos para camada frequente. Objetos com menos que Y acessos por mês são movidos para camada infrequente.
Não há auto-aprendizado ou machine learning verdadeiro fazendo essas decisões. São regras pré-programadas.
Porém ainda sim consegue otimizar custos de armazenamento de forma automática conforme o uso.

```

### Ciclo de vida
```
S3 Standard:

Tempo de acesso: milissegundos
Armazenamento máximo: sem limite
Ciclo de vida: pode transicionar objetos para camadas de acesso infrequente após N dias sem uso.

S3 Standard-IA:
Tempo de acesso: milissegundos
Armazenamento máximo: sem limite
Ciclo de vida: pode mover para Glacier para arquivamento após X meses.

S3 One Zone-IA:
Tempo de acesso: milissegundos
Armazenamento máx.: sem limite
Ciclo de vida: pode deletar objetos permanententemente após período.

S3 Glacier Instant Retrieval:
Tempo acesso: milissegundos
Armazenamento máximo: 180 dias
Ciclo de vida: pode mover para Deep Archive após período

S3 Glacier Flexible Retrieval:
Tempo de acesso: minutos
Armazenamento máximo: sem limite
Ciclo de vida: pode aplicar políticas de lifecycle

S3 Glacier Deep Archive:
Tempo de acesso: horas
Armazenamento máximo: décadas
Mais barato que todas classes Glacier
Políticas de lifecycle configuráveis
```

### Resumo classes s3
```
A classe S3 Standard provê armazenamento de propósito geral com alta performance de milissegundos e disponibilidade,
porém custo mais elevado. Ideal para cenários como hospedagem de sites, apps, processamento frequente de dados e big data analytics.

O S3 Intelligent Tiering gerencia automaticamente objetos entre camadas de alto e baixo acesso, com custo intermediário 
e mesmo desempenho do Standard. Útil para repositórios com utilização variável, como documentos corporativos. 

O S3 Standard Infrequent Access (IA) e o S3 Standard One Zone IA oferecem camadas de menor custo para dados raramente acessados,
com alta disponibilidade e latência de milissegundos. Perfeitos para backups secundários, dados médicos antigos e streaming de vídeo sob demanda.

Para arquivamento de longo prazo com custo muito reduzido por TB armazenado porém latência alta na recuperação,
o Glacier Instant Retrieval possui restauração rápida embora armazene por no máximo 180 dias. Já o Glacier Flexible Retrieval 
e Glacier Deep Archive podem armazenar a baixíssimo custo por décadas, sendo ideais para backups regulatórios e telemetria histórica offine.

```


### s3 request pays
- o dono do bucket paga pelo armazenamento, e quanto alguem faz download do arquivo, paga-se também pela transferência na rede
- podemos delegar o custo da transferência da rede ao solicitante do arquivo no bucket, desde que ele esteja autenticado na aws
- ideal para compartilhar arquivos grandes entre contas

### s3 event notifications
- quando criamos uma replica, realizamos um upload de um arquivo ou removemos um arquivo, podemos notificar esses eventos
- com esses eventos podemos notifcar por exemplo em um sns, sqs, lambda e etc.
- para que isso ocorra, precisa-se de policies no recurso que será notificado e não uma role no s3, por exemplo: para nodificar um sqs, lá precisa de uma policy que permita (allow) ser notificado (SendMessage)
- quem faz o meio de campo entre s3 eventos aos recursos é o amazon eventBridge
- por ele podemos aplicar regras (rules), como qual recurso será notificado, qual tipo de arquivo será envolvido ou tamanho do mesmo e etc.

### s3 performance
- podemos obter no maximo 3.500 put/copy/post/delete ou 5.500 get/heade requisições por segundo, por prefixo em seu bucket (prefixo é tudo entre o bucket e o arquivo, ex: buckettest/pasta/sub/file, past/sub é o prefixo)
- multi-part upload -> recomendo para arquivos acima de 100mb, obrigatório apra acima de 5 gb, o arquivo será dividido e subirá as partes em paralelo
- transfer acceleration -> para aumentar a transferencia de dados de um bucket de uma região para outra, é compatível com multi-part upload
  -  mandamos o arquivo para uma borda e esta envia para o destino usando a rede interna da aws e não a publica (ex: arquivo no s3 manda para a edge location via net publica,  edge location manda para japao via rede privada da aws)
- leitura de arquivos de forma eficiente (s2 byte range fetches), onde pega um range de byte especifico do seu arquivo
  - pode ser usado para aumentar a velocidade do download 

### s3 select & glacier select
- podemos recuperar dados usando sql
- podemos filtrar linhas e colunas
- menos trafego na transferência, custo menor de cpu, pois estamos filtrando o que queremos

### s3 batch operations
- tratar varios objetos como um só
- como: modificar vários objetos de uma so vez, aplicar acls em vários objetos, restaurar vários objetos da camada glacier, gerar um notificação para um volume de objetos
- o exemplo mais comum seria, encriptar todos os arquivos não encriptados, ex: temos o inventario, efetuamos uma consulta (sql) para obter os objetos não encriptados, com o resultado os objetos não encriptados, mandamos uma bath operations para encriptar eles

### s3 criptografia
- temos 2 tipos de criptografia
  - sse (do lado do servidor, server-side encryption)
    - usando criptografia default do s3 (sse-s3)
      - nunca teremos acesso a essa chave 
      - tipo é aes-256
      - devemos fornecer o header no upload "x-amz-server-side-encryption":"256", que o objeto será encriptado
    - usando chaves kms aws (key management service, sse-kms)
      - mais controle e auditoria usando o cloudtrail
      - devemos fornecedor o header "x-amz-server-side-encryption":"aws:kms", no upload do arquivo, onde busca a chave no kms e encriptará o mesmo, em seguida armazenará no bucket
      - para consultar o objeto, precisamos também consultar a chave para decriptar, o que não acontece no caso so sse-s3
      - isso torna uma limitação, pois devemos chamar a api do kms e depois chamar o s3, assim decriptar o objeto, gerando um aumento no tempo da requisição
      - alem da quota por segundo no serviço kms, no entanto podemos aumentar via console.
      - no entanto tem um mecanismo chamado "bucket-key", quando habilitado diminiu a chamada ao kms
    - chave customiado fornecida pelo usuario (customer provider keys sse-c, possivel via cli e nao via console)
      - a aws não armazena essa chave
      - o usuario deve realizar upload na requisição https da chave e a aws utilizará para encriptar o arquivo
      - e em seguida armazena-lo no bucket
      - para decriptar, segue o mesmo processo
  - cliente (cient-side encryption), encriptamos um objeto e realizados upload deste
    - cliente gerencia a chave de criptografia
    - o mesmo encripta o objeto e realiza upload do mesmo.
- Caso tenha um objeto encriptado, por exemplo sse-s3, e mude o tipo de encriptação, para sse-kms, gerará uma nova versão o objeto com a nova criptografia
- opcoes de criptografia possíveis via console:
  - sse-s3
  - sse-kms
  - dsse-kms
- encriptografia em transito
  - no momento da chamada usando ssl/tls
  - s3 expõe 2 endpotins, um para encriptar e outro não
  - usa-se sse-c
  - para forçar e criptografia em transico (o usuario conseguir sempre realizar requisicao https), podemos anexar uma policy com uma condition, ex:
```
"Condition" : {
  "Bool" : {
      "aws:SecureTransport":"true"
  
  }

}
```
- a criptografia padrão (sse-s3) há habilitada por padrão, mas podemos forçar o uso de outra tipo de criptografia via policy, visto que esta é avaliada antes de subir ou realizar 
  upload do arquivo.
- ex:
```

sse-kms
{
    "Version": "2012-10-17",    
    "Statement": [
        {
            "Sid": "EnforceSSEKMS",
            "Effect": "Deny",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::mybucket/*",
            "Condition": {
                "Null": {
                    "s3:x-amz-server-side-encryption": "aws:kms"
                }
            }
        }
    ]
}

sse-c
{
  "Version":"2012-10-17",
  "Id": "PutObjPolicy",
  "Statement": [
    {
      "Sid": "DenyInsecureCiphers",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::mybucket/*",
      "Condition": {
        "StringNotEquals": {"s3:x-amz-server-side-encryption-customer-algorithm": "AES256"}
      }
    } 
  ]  
}
```
### se3 cors

```
 CORS (Compartilhamento de Recursos de Origem Cruzada) é um mecanismo que define como um recurso (por exemplo, uma fonte JavaScript) em um domínio pode interagir com outro recurso em um diferente domínio.

Alguns pontos importantes:

- Origem Cruzada se refere a requisições HTTP de um domínio diferente daquele que serviu o conteúdo original.

- Por padrão, requisições CORS são bloqueadas por razões de segurança, a menos que habilitadas via headers.

- Os principais headers relacionados ao CORS são:
    - Access-Control-Allow-Origin: Define quais origens podem acessar o recurso.
    - Access-Control-Allow-Methods: Métodos permitidos na requisição.
    - Access-Control-Allow-Headers: Headers permitidos na requisição.
    - Access-Control-Max-Age: Quanto tempo o browser pode cachear essa permissão de acesso.

- No S3, o CORS precisa ser explicitamente habilitado em um bucket para permitir que aplicações em outros domínios acessem os objetos.

Então o CORS provê uma forma segura dos navegadores permitirem requests de origem cruzada entre diferentes domínios. É essencial para muitos casos de uso em aplicações web modernas.
```
- cenário de uso:
  - browser faz uma requisicao ao bucket A
  - a pagina carrega do bucket A, precisa de uma imagem no bucket B
  - o browser precisa entao informar a origem no cabeçalho "Access-Control-Allow-Origin:http://bucketA" para pegar a imagem no bucket B
  - se o cors estiver habilitado, e permitir requisão dessa origem, funcionária
  - se o cors estiver desabilitado *, funcionária
  - caso contrário não

### s3 exclusao mfa
- para habilitar o mfa no upload de arquivo, devemos fazer via cli
- o mesmo ocorre para delete, precisa ser via cli com mfa ativado

### s3 access log
- quando queremos monitorar tudo o que ocorre em um bucket, gera-se logs que são enviados para outro bucket
- nunca podemos usar o bucket monitorado para gravar os logs, pois entrará em um loop infinito
- obs os buckets devem estar na mesma regiao

### se urls pre assinadas (pre signed urls)
- podemos gerar urls temporarias, afim de disponibilizar a outros usuarios para: efetuar donwload de algum arquivo ou upload
- a url e temporaria e nosso objeto no bucket pode permanecer privado
- a url corresponde a um objeto dentro do bucket

###  s3 glacier vault lock
- a ideia é escrever uma vez e ler muitas vzes
- ou seja, evitar motificacao de arquivos no glacier
- para isso precismos criar uma policy no nosso glacier (vault lock policy)
- para usar o object lock, o versionamento deve estar habilitado
  - podemos criar policiy para o objeto em específico e não para o bucket inteiro
  - segue a mesma ideia, escreva uma vez, leia várias
  - temos alguns modos de retencao
    - compliance: as versões não podem ser sobrescritas por todos os usuarios, incluindo o root e tambem as configuracoes de modo por ex, não podem ser modificadas.
    - governance: a maioria dos usuários não pode anular ou excluir uma versao do objeto ou alterar suas configurações de bloqueio, mas usuarios adm com permissoes (iam) podem
  - periodo de retencao: podemos aplicar os modos por um perido fixo
  - legal hold: protege qualquer object por um periodo de retencao indefinido, somente usuarios com a role s3:putObjetLegalHold que podem remove-los

### s3 access points
- facilita a gestao do bucket, ou cada ponto de acesso terá seu dns
- podemos anexar a cada ponto uma policy

### s3 lambda
- colocar código para modificar e processar dados, conforme eles são recebidos ou recuperados do bucket
- exemplo de uso: xml armazenado no bucket, quero recuperar o mesmo no formato json
  - crio um access point, bate no lambda, que pega o o objeto, transforma em json, retorna para o access point que por sua vez, o solicitante o recebe 


# Aws cloud front
- CDN - content delivery network

```
CDN (Content Delivery Network) é uma rede distribuída de servidores que entrega conteúdo estático (como imagens, vídeos, arquivos CSS e JavaScript) aos 
usuários com base na sua localização geográfica. A ideia principal de um CDN é fornecer conteúdo de forma mais rápida e eficiente aos usuários finais.

No contexto da AWS (Amazon Web Services), a Amazon disponibiliza um serviço de CDN chamado Amazon CloudFront. O CloudFront é um serviço de entrega de 
conteúdo rápida, segura e programável que utiliza uma rede global de data centers, conhecidos como "Edge Locations".

Quando você hospeda seu conteúdo estático no Amazon CloudFront, a AWS distribui automaticamente e armazena esse conteúdo em vários data centers
estrategicamente localizados em diferentes regiões ao redor do mundo. Dessa forma, quando um usuário solicita um arquivo (como uma imagem ou um script JavaScript),
 o CloudFront entrega esse conteúdo a partir da "Edge Location" mais próxima do usuário, reduzindo a latência e aumentando a velocidade de carregamento.

O CloudFront também oferece recursos adicionais, como:

- Gerenciamento de cache e TTL (Time to Live) para controlar quanto tempo o conteúdo é armazenado em cache nas "Edge Locations".
- Integração com outras soluções da AWS, como o Amazon S3, Amazon ELB, Amazon EC2, entre outras.
- Suporte a HTTPS e recursos de segurança, como firewalls de aplicação web (WAF) e criptografia.
- Relatórios e logs detalhados sobre o tráfego de conteúdo.

Em resumo, o CDN da AWS (Amazon CloudFront) é um serviço que permite armazenar e distribuir conteúdo estático de forma global,
minimizando a latência e proporcionando uma experiência mais rápida para os usuários finais.
```
- origens:
  - s2 bucket
  - custom http
    - alb
    - ec2
    - s3 website static
    - qualquer http backend
- funcionamento:
  - cliente solicita o recurso no cloud front edge location
  - verifica-se se o recurso esta no cache do cloud front edge location
  - caso não esteja, vá ate a origem e retorne-o (coloca no cache e envia ao cliente)
  - na segunda requisição, caso o recurso ja esteja no cache, já é retornado, sem necessidade de ir ate a origem


## Cludfront vs s3 cross region replication
```
A principal diferença entre o Amazon CloudFront e a Replicação Cross-Region do Amazon S3 é o propósito e a forma como eles entregam o conteúdo.

Amazon CloudFront:
- É um serviço de CDN (Content Delivery Network) que distribui conteúdo estático (como imagens, vídeos, arquivos CSS, JavaScript) em cache através de várias "Edge Locations" ao redor do mundo.
- O objetivo principal é oferecer entrega de conteúdo com baixa latência e alta velocidade para os usuários finais, independentemente da localização geográfica.
- O CloudFront atua como uma camada intermediária entre o armazenamento de origem (como o S3) e os usuários finais, armazenando o conteúdo em cache mais próximo dos usuários.

Replicação Cross-Region do Amazon S3:
- É um recurso do Amazon S3 que replica automaticamente os objetos de um bucket de origem para um ou mais buckets de destino em diferentes regiões da AWS.
- O objetivo é fornecer resiliência, disponibilidade e acesso rápido aos dados em várias regiões, caso ocorra uma interrupção em uma determinada região.
- A replicação é feita diretamente entre os buckets do S3 nas diferentes regiões, sem envolver uma CDN.

Em resumo, o CloudFront é projetado para melhorar a velocidade de entrega de conteúdo para os usuários finais, enquanto a replicação cross-region do S3 é projetada para garantir a disponibilidade e resiliência dos dados em várias regiões.

Embora sejam recursos diferentes, eles podem ser usados em conjunto. Por exemplo, você pode configurar um bucket S3 como origem para o CloudFront e usar a replicação cross-region para manter cópias dos dados em várias regiões, garantindo alta disponibilidade e entrega rápida de conteúdo simultaneamente.
```

### access control cloudfront
```
O Access Control (Controle de Acesso) no Amazon CloudFront refere-se a um conjunto de recursos e configurações que permitem controlar quem pode acessar o conteúdo armazenado e distribuído pelo CloudFront.

Alguns dos principais recursos de controle de acesso do CloudFront incluem:

1. **Restrições de Acesso**: Você pode configurar restrições de acesso para restringir acesso a seu conteúdo com base em uma lista de controle de acesso (ACL), endereços IP ou assinaturas de URL seguras.

2. **Assinaturas de URL Seguras**: Permite que você distribua conteúdo privado adicionando assinaturas criptográficas a URLs do CloudFront. Essas assinaturas determinam quem pode acessar seu conteúdo e por quanto tempo.

3. **Origem de Acesso**: Você pode controlar quais origens (como buckets do S3 ou servidores web) podem fornecer conteúdo para o CloudFront.

4. **Campos Personalizados**: Permite adicionar cabeçalhos personalizados às solicitações e respostas do CloudFront, para casos de uso como controle de acesso baseado em cabeçalhos.

5. **Gerar Políticas**: Você pode gerar políticas personalizadas de controle de acesso para aplicar restrições mais complexas.

6. **Integração com AWS WAF** (Web Application Firewall): Permite proteger seu conteúdo contra ameaças comuns na Web, como SQL injection e scripts intersites.

Essas configurações ajudam a garantir que apenas usuários autorizados possam acessar seu conteúdo distribuído pelo CloudFront, protegendo recursos privados ou confidenciais. É essencial configurar corretamente esses controles para evitar acesso não autorizado ao seu conteúdo.
```

### preco cloudfront
- dependi da regiao aonde aplicará o edge location
- quando maior a transferencia, menor será o valor
- temos 3 classes
  -  todas as regiones - melhor performance
  - class 200, maioria das regiões, excluindo as mais caras
  - class 100, incluindo apenas as mais baratas

### invalidando cache no cloudfront
```
Existem duas principais maneiras de invalidar o cache no Amazon CloudFront:

1. **Invalidação de Objetos**:
   - Esse método permite invalidar (remover) objetos específicos do cache nas "Edge Locations" do CloudFront.
   - Você pode especificar caminhos de objeto individuais ou usar curingas para invalidar grupos de objetos de uma só vez.
   - É útil quando você atualiza um arquivo específico e deseja forçar os usuários a receber a nova versão.
   - Pode ser feito através da console do CloudFront, da API ou de ferramentas de CLI.

2. **Redefinição de Cache**:
   - Essa opção remove todos os objetos em cache em todas as "Edge Locations" do CloudFront.
   - É útil quando você faz alterações significativas em seu site ou aplicativo e deseja garantir que todos os usuários recebam a versão mais recente do conteúdo.
   - É uma operação mais agressiva e pode levar mais tempo para ser concluída em comparação com a invalidação de objetos.
   - Também pode ser feita através da console, API ou CLI.

Ao invalidar ou redefinir o cache, o CloudFront deve obter a versão mais recente dos objetos solicitados da origem configurada (por exemplo, um bucket S3 ou servidor web) antes de enviá-los para o cliente.

É importante notar que a invalidação e redefinição de cache podem gerar custos adicionais no CloudFront, dependendo da quantidade de dados invalidados. Portanto, é recomendado invalidar apenas o conteúdo necessário e com moderação.

Além disso, você também pode configurar cabeçalhos de Cache-Control ou Expires adequados na origem para determinar por quanto tempo os objetos devem ser armazenados em cache nas "Edge Locations" do CloudFront.
```

# Aws global accelerator
```
O AWS Global Accelerator é um serviço que melhora a disponibilidade e o desempenho das aplicações que usam o protocolo IP sobre a Internet. Ele aproveita a infraestrutura global da AWS para otimizar o roteamento e a entrega de tráfego através da rede.

O Global Accelerator funciona da seguinte maneira:

1. **Endereços IP Estáticos Globais**: Quando você configura o Global Accelerator, ele fornece dois endereços IP estáticos globais que atuam como pontos de entrada para o tráfego da sua aplicação.

2. **Pontos de Presença Globais**: A AWS possui Pontos de Presença Globais (Global Points of Presence - GPPs) espalhados pelo mundo. Esses pontos de presença são servidores da AWS que estão próximos a concentrações significativas de usuários.

3. **Roteamento Otimizado**: Quando um cliente acessa sua aplicação pelo endereço IP estático global, o Global Accelerator encaminha o tráfego através do ponto de presença mais próximo do cliente. Em seguida, o tráfego é roteado através da rede global da AWS para o recurso de destino (como um Elastic Load Balancing, EC2 ou outros serviços).

4. **Failover Automático**: Se alguma parte da rede ou do recurso de destino falhar, o Global Accelerator automaticamente redireciona o tráfego para caminhos alternativos saudáveis, mantendo a alta disponibilidade da aplicação.

5. **Otimização de Desempenho**: O Global Accelerator usa técnicas de otimização de rede, como o protocolo de transporte otimizado e o roteamento inteligente, para melhorar o desempenho e a estabilidade da conexão.

Alguns benefícios do AWS Global Accelerator incluem:

- **Melhor Desempenho**: Reduz a latência e o tempo de resposta para carregamento de aplicações, transferência de dados e chamadas de API.
- **Alta Disponibilidade**: Failover automático e caminhos de rede redundantes garantem que a aplicação permaneça disponível em caso de falhas.
- **Endereços IP Estáticos**: Facilita a configuração de DNS e evita problemas com endereços IP flutuantes.
- **Segurança**: O tráfego é roteado pela rede global privada da AWS, oferecendo mais segurança.

O Global Accelerator é especialmente útil para aplicações que exigem baixa latência e alta disponibilidade em escala global, como jogos online, streaming de mídia, provedores de VoIP e outros serviços de Internet.
```

# Aws snowball
```
O AWS Snowball é um serviço de transferência de dados físicos oferecido pela Amazon Web Services (AWS). É projetado para permitir o transporte seguro de grandes quantidades de dados entre os locais do cliente e o armazenamento na nuvem AWS.

O AWS Snowball consiste em dispositivos de armazenamento físico, chamados de "Snowballs", que são enviados pelo correio para os clientes. Esses dispositivos são projetados para transferir dados de maneira segura, rápida e econômica, especialmente quando a transferência pela Internet é impraticável devido a limitações de largura de banda, altos custos ou necessidades de conformidade.

Cada Snowball é um dispositivo resistente e seguro, com capacidade de armazenamento que varia de 50 TB a 80 TB. Os clientes podem transferir seus dados para o Snowball usando uma conexão de alta velocidade, como uma interface de 10 Gb ou 40 Gb. Depois de transferir os dados, eles devolvem o Snowball para a AWS, onde os dados são importados automaticamente para serviços de armazenamento de dados, como o Amazon S3 ou o Amazon EFS.

O AWS Snowball oferece várias opções para atender às necessidades de transferência de dados dos clientes:

1. **AWS Snowball Edge**: Uma opção de computação na borda que permite executar análises e processamento de dados no próprio dispositivo antes de transferir os dados para a nuvem.

2. **AWS Snowball Edge Compute Optimized**: Otimizado para análises avançadas na borda, com recursos de computação poderosos.

3. **AWS Snowmobile**: Um serviço para transferências de dados em uma escala exabyte, envolvendo um contêiner seguro de 45 pés que pode transportar até 100 PB de dados.

O AWS Snowball utiliza criptografia de dados e várias camadas de segurança física e lógica para proteger os dados dos clientes durante todo o processo de transferência. Ele também oferece recursos de rastreamento e monitoramento para acompanhar o progresso e o status da transferência de dados.

O AWS Snowball é amplamente utilizado em setores como mídia e entretenimento, saúde, finanças e governo, onde grandes quantidades de dados precisam ser transferidas de forma segura e confiável.
```

## Enviando dados para um s3 glacier
```
Não é possível enviar dados diretamente do AWS Snowball para o Amazon S3 Glacier. O processo correto é:

1. Enviar os dados do Snowball para um bucket padrão do Amazon S3.

2. Configurar uma Política de Ciclo de Vida (Lifecycle Policy) nesse bucket S3 para mover os objetos para o S3 Glacier após um determinado período de tempo.

A razão para isso é que o Glacier não aceita uploads diretos. Ele é projetado para ser uma solução de arquivamento de baixo custo, que recebe dados do Amazon S3 através de transições automáticas definidas pelas Políticas de Ciclo de Vida.

Portanto, os passos corretos são:

1. Ao criar um trabalho de importação no Snowball, selecione um bucket padrão do Amazon S3 como destino.

2. Transferir os dados para o Snowball e devolvê-lo à AWS.

3. Os dados serão importados para o bucket S3 selecionado.

4. Criar uma Política de Ciclo de Vida nesse bucket, especificando que os objetos devem ser movidos para o Glacier após um determinado período de tempo (por exemplo, 30 dias).

Com essa abordagem, os dados são enviados primeiro para o Amazon S3 e, em seguida, transicionados automaticamente para o Glacier, de acordo com a Política de Ciclo de Vida configurada. Isso garante que os dados sejam transferidos corretamente para o serviço de arquivamento de baixo custo do Glacier.
```

# Aws fsx
``` 
O Amazon FSx (File System for AWS) é um serviço totalmente gerenciado que fornece sistemas de arquivos compartilhados para cargas de trabalho corporativas. O FSx permite criar e configurar sistemas de arquivos rapidamente, dimensionar conforme necessário e pagar apenas pelo recurso consumido.

Certo, vou explicar os 4 principais tipos de sistema de arquivos oferecidos pelo Amazon FSx:

1. **Amazon FSx for Windows File Server**:
   - Sistema de arquivos totalmente gerenciado, compatível com o Windows nativo, baseado no protocolo SMB (Server Message Block).
   - Suporta todos os recursos e metadados do Windows, como permissões NTFS, cotas de disco, shadow copies e replicação DFS.
   - Ideal para cargas de trabalho corporativas, como home directories, compartilhamento de arquivos, aplicativos baseados em servidor, análises e processamento de dados.

2. **Amazon FSx for Lustre**:
   - Sistema de arquivos de alto desempenho, altamente paralelo e compatível com POSIX, baseado no sistema de arquivos Lustre de código aberto.
   - Projetado para cargas de trabalho de computação de alto desempenho (HPC), como simulações numéricas, análise de big data, computação de machine learning e processamento de mídia.
   - Suporta streaming de dados paralelo de alto rendimento com baixa latência e alta taxa de transferência.

3. **Amazon FSx for NetApp ONTAP**:
   - Sistema de arquivos gerenciado baseado no ONTAP da NetApp, fornecendo armazenamento em bloco e sistema de arquivos.
   - Destinado a cargas de trabalho empresariais que exigem recursos avançados de gerenciamento de dados e armazenamento, como snapshots, replicação, clonagem de dados e muito mais.
   - Suporta tanto protocolos ISCSI, NFS (Network File System) quanto SMB (Server Message Block).

4. **Amazon FSx for OpenZFS**:
   - Sistema de arquivos de alto desempenho, altamente duradouro e distribuído, baseado no OpenZFS de código aberto.
   - Projetado para cargas de trabalho que exigem alto desempenho, durabilidade de dados e recursos avançados de gerenciamento de dados, como snapshots, compactação de dados e recuperação automática de dados corrompidos.
   - Suporta protocolos NFS e SMB e é ideal para aplicativos empresariais, big data, análises e cargas de trabalho de arquivos.

Esses são os quatro principais tipos de sistemas de arquivos oferecidos pelo Amazon FSx, cada um projetado para atender a diferentes requisitos e cargas de trabalho específicas. A escolha dependerá das necessidades de desempenho, compatibilidade, recursos de gerenciamento de dados e protocolos de arquivo suportados para a carga de trabalho em questão.
```

## tipos aws fsx
```
O Amazon FSx suporta dois tipos principais de sistemas de arquivos: scratch file system e persistent file system.

1. **Scratch File System**:
   - O scratch file system é um sistema de arquivos temporário otimizado para cargas de trabalho de computação de alto desempenho (HPC) que geram grandes quantidades de dados temporários.
   - Ele é implantado em uma única instância do Amazon EC2 e fornece armazenamento de dados efêmero para essa instância.
   - Os dados armazenados no scratch file system são perdidos quando a instância do EC2 é interrompida ou encerrada.
   - O scratch file system é uma opção econômica e de alto desempenho para processamento de dados temporários, como simulações numéricas, renderização de mídia e análise de dados.

2. **Persistent File System**:
   - O persistent file system é um sistema de arquivos duradouro projetado para armazenar dados de longo prazo.
   - Ele é implantado como um recurso separado na AWS e pode ser acessado por várias instâncias do EC2 simultaneamente.
   - Os dados armazenados no persistent file system são mantidos mesmo quando as instâncias do EC2 são interrompidas ou encerradas.
   - O persistent file system oferece durabilidade de dados, alta disponibilidade, backup automático e recursos de replicação (az) para resiliência e recuperação de desastres.
   - É adequado para cargas de trabalho que exigem armazenamento persistente, como home directories, compartilhamento de arquivos, análises e processamento de dados de longo prazo.

A escolha entre o scratch file system e o persistent file system depende da natureza da carga de trabalho e dos requisitos de durabilidade de dados. O scratch file system é uma opção adequada para processamento temporário e de alto desempenho, enquanto o persistent file system é preferível para cargas de trabalho que exigem armazenamento duradouro e recursos avançados de gerenciamento de dados.

É importante considerar cuidadosamente as necessidades da carga de trabalho ao escolher entre esses dois tipos de sistemas de arquivos no Amazon FSx.
```

# Aws storage gateway
- é uma ponte dos seus dados no local e dos dados da sua nuvem
- casos de uso:
  - recuperação de desastres
  - backup 
  - tipos de armazenamentos
  - uso onprimeses como cache e os dados persistentes na nuvem

## tipos aws storage gateway
```
O AWS Storage Gateway é um serviço que fornece uma interface de armazenamento na nuvem para integração com ambientes locais. 
Ele permite que os clientes aproveitem o armazenamento em nuvem, como o Amazon S3, Amazon EFS ou Amazon FSx, ao mesmo tempo em que mantêm 
parte dos dados localmente para acesso com baixa latência. O Storage Gateway oferece quatro tipos principais de configuração:

1. **S3 File Gateway (Gateway de Arquivos)**: Permite que os clientes armazenem e recuperem dados em arquivos através do protocolo 
NFS (Network File System) ou SMB (Server Message Block). Os dados são armazenados no Amazon S3 e cacheados localmente no gateway para acesso rápido (os mais recentes).
Não funciona com o s3 glacier
Se utilizar o SMB protocol, para integracao com o active directory, os usuarios podem se autenticar para acessar o diretorio no bucket s3

2. **Volume Gateway (Gateway de Volume)**: O Volume Gateway oferece dois métodos para acessar o armazenamento em blocos na nuvem:
Volume em disco (Stored Volumes): Nesse modo, você armazena seus dados localmente e faz backups assíncronos de volumes inteiros para a Amazon S3.
Volume em cache (Cached Volumes): Nesse modo, seus dados são armazenados primariamente na AWS e apenas os dados frequentemente acessados são mantidos em cache localmente. Isso minimiza a latência de acesso aos dados.
É útil para cargas de trabalho que requerem acesso a blocos de armazenamento, como bancos de dados ou aplicativos que precisam de um sistema de arquivos local rápido.

3. **Tape Gateway (Gateway de Fita)**:  permite que você substitua as infraestruturas de bibliotecas de fita física por um gateway virtual que simula bibliotecas de fita para backups e arquivamento.
Ele é ideal para cargas de trabalho que dependem de bibliotecas de fita para backups de longo prazo e requisitos de conformidade.
O Tape Gateway oferece suporte a protocolos de fita virtuais, como VTL (Virtual Tape Library), para que aplicativos existentes possam ser configurados para usar a fita virtual sem modificações.
 
4. **FSx File Gateway (Gateway de Arquivos FSx)**: 
Integração com Amazon FSx: O FSx File Gateway facilita a integração entre seus sistemas de arquivos locais e os sistemas de arquivos Amazon FSx. Ele permite que você acesse e
 gerencie arquivos no Amazon FSx como se estivessem localmente montados em seu ambiente.
oferece acesso de baixa latência a arquivos do FSx for Windows File Server on-premises, além de outros benefícios, como:
Fácil de usar: O File Gateway é fácil de configurar e gerenciar, com uma interface intuitiva e documentação abrangente.
Altamente disponível: O File Gateway oferece alta disponibilidade, garantindo que seus arquivos estejam sempre acessíveis.
Escalável: O File Gateway é escalável, podendo ser dimensionado para atender às suas necessidades de armazenamento e desempenho.
No entanto, é importante considerar algumas diferenças entre o Direct Connect e o File Gateway:
Conectividade: O Direct Connect estabelece uma conexão dedicada e privada entre sua rede local e a AWS, enquanto o File Gateway utiliza a internet pública para comunicação.
Funcionalidade: O Direct Connect é apenas um serviço de conectividade, enquanto o File Gateway oferece funcionalidades adicionais, como cache local e otimização de acesso a arquivos.
Custo: O Direct Connect pode ser mais caro que o File Gateway, especialmente se você precisar de uma grande largura de banda.
 
Recursos de Gateway de Arquivos: O FSx File Gateway oferece funcionalidades semelhantes ao File 
Gateway padrão do AWS Storage Gateway, permitindo que você acesse arquivos em sistemas de arquivos Amazon FSx 
por meio de protocolos de rede padrão, como NFS (Network File System) ou SMB (Server Message Block).

Replicação de Dados: Você pode configurar a replicação de dados entre seus sistemas de arquivos locais e os 
sistemas de arquivos Amazon FSx, garantindo que seus dados estejam sincronizados e protegidos.

Suporte a Casos de Uso Diversos: O FSx File Gateway é útil para uma variedade de casos de uso, incluindo migração de dados para a nuvem,
 recuperação de desastres, arquivamento e compartilhamento de arquivos entre locais.

Integração com Infraestrutura Existente: Como parte do AWS Storage Gateway, o FSx File Gateway é projetado para integrar-se facilmente com a infraestrutura existente e aplicativos, permitindo uma migração suave para a nuvem.


Cada tipo de gateway é projetado para atender a diferentes casos de uso e requisitos. O File Gateway e o FSx File Gateway 
são ideais para compartilhamento de arquivos, enquanto o Volume Gateway é adequado para cargas de trabalho de armazenamento em bloco. 
O Tape Gateway é uma solução para backup na nuvem e arquivamento de dados.

Ao escolher o tipo de gateway, é importante considerar os requisitos de acesso local, protocolos suportados, integração com aplicativos
 existentes e a necessidade de armazenamento em cache local para garantir o desempenho ideal para a carga de trabalho específica.
```
## hardware appliance
- caso não tenha um servidor virtual, pode solicitar um servidor virtual on-primeses na aws
- e configura-loa como um file gateway

# Diferença entre aws storage gateway com o fsx
```
Tanto o Amazon FSx e o AWS Storage Gateway são serviços da AWS relacionados a armazenamento de dados, mas eles possuem finalidades e funcionalidades distintas. Vejamos as principais diferenças:

FSx (File System Service):

Foco: Armazenamento de arquivos de alto desempenho e escalável na nuvem.
Funcionalidade: Projetado para oferecer acesso rápido a arquivos por aplicativos e usuários.
Serviços: FSx for Windows File Server, FSx for Lustre, FSx for NetApp ONTAP.
Casos de uso: Compartilhamento de arquivos corporativos, armazenamento de dados para aplicativos, backups, análise de dados, HPC.
Vantagens: Alto desempenho, escalabilidade, fácil de usar, integração com outros serviços AWS.
Storage Gateway:

Foco: Ponte entre armazenamento on-premises e armazenamento em nuvem AWS (S3, Glacier).
Funcionalidade: Sincroniza dados on-premises com a nuvem, oferecendo acesso de baixa latência e backup seguro.
Tipos de Gateway: Cache Gateway, File Gateway, Tape Gateway.
Casos de uso: Backup e arquivamento de dados, compartilhamento de arquivos entre locais, hospedagem de sites estáticos.
Vantagens: Armazenamento escalável e econômico, backup e recuperação de desastres, integração com aplicativos existentes.
```

# Aws transfer family
- para usar o s3 ou efs como um protocolo ftp (ftp, ftps, sftp)
- para transferir dados via ftp

# Aws datasync
```
O AWS DataSync é um serviço online seguro que automatiza e acelera a transferência de dados entre serviços de armazenamento on-premises e da AWS. Ele também pode ser usado para transferir dados entre os serviços de armazenamento da AWS.

Funcionalidades do AWS DataSync:

Transferência de dados segura e confiável: O DataSync usa criptografia em repouso e em trânsito para garantir a segurança dos seus dados. Ele também oferece recursos de monitoramento e relatórios para que você possa acompanhar o progresso das suas transferências de dados.
Automação de tarefas: O DataSync permite automatizar a transferência de dados, incluindo agendamento de transferências, configuração de filtros de arquivos e definição de políticas de retry.
Suporte para diversos serviços de armazenamento: O DataSync oferece suporte a uma ampla gama de serviços de armazenamento on-premises e da AWS, incluindo Amazon S3, Amazon EBS, Amazon FSx, Azure Blob Storage, Google Cloud Storage, NFS e SMB.
Fácil de usar: O DataSync pode ser facilmente configurado e gerenciado usando o console de gerenciamento da AWS ou a API.
Casos de uso do AWS DataSync:

Migração de dados para a nuvem: O DataSync pode ser usado para migrar seus dados de armazenamento on-premises para a AWS de forma rápida e segura.
Replicação de dados entre locais: O DataSync pode ser usado para replicar seus dados entre diferentes locais, on-premises e na nuvem.
Sincronização de dados entre diferentes serviços de armazenamento: O DataSync pode ser usado para sincronizar seus dados entre diferentes serviços de armazenamento, on-premises e na nuvem.
Benefícios do AWS DataSync:

Agilidade: O DataSync acelera a transferência de dados entre diferentes serviços de armazenamento.
Segurança: O DataSync garante a segurança dos seus dados com criptografia em repouso e em trânsito.
Eficiência: O DataSync automatiza a transferência de dados e reduz o tempo e o esforço necessário para gerenciar suas transferências de dados.
Escalabilidade: O DataSync pode ser dimensionado para atender às suas necessidades de transferência de dados.
```

## Baixa capacidade de rede para usa do datasync
```

Se você não tiver capacidade de transferência suficiente na rede para usar o AWS DataSync, existem algumas alternativas que você pode considerar:

1. Otimizar a transferência de dados:

Reduza o tamanho dos dados: Utilize técnicas de compactação para reduzir o tamanho dos dados que você precisa transferir.
Transfira dados em horários de menor uso: Transfira seus dados durante períodos de menor uso da rede para evitar congestionamento.
Ajuste as configurações do DataSync: Ajuste as configurações do DataSync para otimizar o uso da largura de banda.
2. Use um serviço de transferência de dados offline:

AWS Snowball: O AWS Snowball é um dispositivo de armazenamento físico que você pode usar para transferir dados para a AWS offline.
AWS Snowcone (ja vem com o agente de sincronização pré instalado): O AWS Snowcone é um dispositivo de armazenamento portátil menor que o Snowball, ideal para transferir volumes menores de dados.
3. Use uma solução de armazenamento em cache local:

AWS Storage Gateway: O AWS Storage Gateway é um serviço de armazenamento em cache local que pode ser usado para armazenar dados em cache na nuvem e acessá-los localmente.
4. Aumente a capacidade da sua rede:

Atualize seu hardware de rede: Considere atualizar seu hardware de rede para aumentar a capacidade de transferência.
Aumente sua largura de banda: Entre em contato com seu provedor de serviços de internet para aumentar a largura de banda da sua conexão.
5. Use uma combinação de soluções:

Combine o DataSync com outras soluções: Você pode combinar o DataSync com outras soluções, como o AWS Snowball ou o AWS Storage Gateway, para atender às suas necessidades de transferência de dados.
A melhor solução para você dependerá de suas necessidades específicas e das suas restrições de rede.
```

# Mensageria

## sqs
- envio de mensagens no máximo 256kilobyte
- podemos dimensionar nossas instâncias com base no cloudwatch metric queue length ApproximateNumberOfMessages, ou seja, chegando a um certo volume de msgs
- aumentar nossas instâncias. (enviará um alarme e acionar o auto-scaling group por exemplo)

### tempo de visibilidade
- padrão e de 30 segundos
- o tempo começa a ser contado a partir do recebimento da mensagem pelo consumidor
- ele tem esse tempo para retornar que processou com sucesso ou não a mensagem
- caso não retorne nesse tempo, ela será disponibilizada para outro consumidor
- caso o consumidor identifique que precise de mais tempo, poderá chamar a api ChangeMessageVisibility, para impedir que ela fique disponível novamente e obter mais tempo

### long polling
- para diminuir as chamadas a fila sqs
- quando a fila estiver vazia ficamos aguardando um tempo, ate que apareça uma mensagem
- melhora a latência

### fila fifo
- primeira que entra, primeira que sai
- garante a ordenação, entrega extamente uma vez (remove mensagens duplicadas)
- tem um limite de throughput de 300msg/s em lote ou 3000/msg s 

### auto-scaling com sqs
- podemos aumentar o dimensionamento com das ec2 por ex, com base na quantidade de filas pendentes pra consumo
- fazemos isso definindo um alarme do CloudWatch - queue length (ApproximateNumberOfMessages)
- que irá triggar o scale

### opções de uso para sqs
- utilizar como buffer, ex: em vezes de mandar gravar direto na base de dados, dependendo do volume de requisições, colocamos em uma fila e em seguida o consumidor
- efetuará a inserção

# SNS
```
O Amazon Simple Notification Service (Amazon SNS) é um serviço de mensagens totalmente gerenciado fornecido pela Amazon Web Services (AWS). 
Ele permite que você envie notificações de uma fonte para uma ou mais destinações de forma assíncrona, escalável e altamente disponível.

Funcionamento do Amazon SNS:
1. **Tópicos**: No SNS, você cria um tópico, que é um ponto de acesso lógico para enviar notificações. 
Um tópico é identificado por um nome ou um ARN (Amazon Resource Name).

2. **Publicadores**: Os publicadores (aplicativos, serviços ou dispositivos) enviam notificações para um tópico do SNS. 
Essas notificações podem ser mensagens de texto ou dados binários, como arquivos ou dados de sensor.

3. **Assinantes**: Os assinantes são as destinações que recebem as notificações enviadas para um tópico. 
Eles podem ser endpoints de serviços AWS (como Amazon SQS, AWS Lambda ou Amazon Kinesis), endereços de email ou aplicativos móveis.

4. **Assinaturas**: Para receber notificações de um tópico, os assinantes devem se inscrever nele. 
Isso é feito criando uma assinatura, que define o protocolo de entrega (HTTP/HTTPS, email, SMS, etc.) e o endpoint de destino.

5. **Entrega de mensagens**: Quando um publicador envia uma notificação para um tópico, 
o SNS replica e entrega a mensagem para todos os assinantes inscritos nesse tópico, de acordo com seus protocolos de entrega configurados.

6. **Confirmação de entrega**: O SNS tentará entregar a mensagem até um número configurável de vezes, 
caso a entrega inicial falhe. Ele também pode ser configurado para armazenar mensagens não entregues em uma fila SQS.

7. **Políticas de acesso**: O SNS permite controlar o acesso aos tópicos e assinaturas através de políticas baseadas em identidade, 
recursos e condições.

O Amazon SNS é amplamente utilizado em cenários como:

- Notificações de aplicativos móveis
- Monitoramento e alerta de sistemas
- Comunicação entre serviços distribuídos
- Entrega de mensagens de email e SMS
- Integração com outros serviços AWS (como Lambda, S3, CloudWatch, etc.)

O SNS é altamente escalável, confiável e oferece baixa latência na entrega de mensagens. Ele se
 integra perfeitamente com outros serviços AWS e é uma solução popular para requisitos de mensagens assíncronas em arquiteturas na nuvem.
 
 obs: sns pode configurar um tipico fifo
```
### sns fan-out sqs
- varias filas sqs, se inscrevam em um topic, quando o sns produzir uma mensagem ao mesmo, todas elas receberam a mensagem
- e consequentemente todos os consumidores destas filas
- podemos filtrar mensagens com base na politica, ou seja, o sns recebe a mensagem e direciona para um inscrito(sqs queue por ex), com base nessa filtragem


# kinesis
```
O Amazon Kinesis é um serviço de streaming de dados fornecido pela Amazon Web Services (AWS). 
Ele permite coletar, processar e analisar dados em tempo real de diversas fontes, como aplicativos móveis, dispositivos IoT, 
websites e sistemas de infraestrutura.

O Kinesis é composto por três serviços principais:

1. **Kinesis Data Streams**:
   - Permite ingestão e processamento de grandes fluxos de dados em tempo real.
   - Os dados são armazenados temporariamente em shards (partições) e podem ser consumidos por várias aplicações simultaneamente.
   - Útil para casos de uso como análise de logs, métricas, dados de IoT, eventos de cliques, etc.

2. **Kinesis Data Firehose**:
   - auto administrador, serveless
   - pago pelos dados que passam pelo firehose
   - Serviço para capturar, transformar e carregar dados de streaming em destinos como Amazon S3, Amazon Redshift, Amazon opensearch  e 
   serviços de análise de terceiros.
   - Simplifica o processo de entrega contínua de dados para análises próximas ao tempo real.

3. **Kinesis Data Analytics**:
   - Permite executar análises SQL ou aplicativos Java em fluxos de dados Kinesis em tempo real.
   - Útil para casos de uso como detecção de anomalias, enriquecimento de dados, filtragem e amostragem de dados, monitoramento de métricas, etc.

Benefícios do Amazon Kinesis:

- **Processamento de streaming em tempo real**: Capacidade de processar e reagir a dados à medida que eles são gerados, em vez de aguardar o 
processamento por lotes.
- **Escalabilidade**: Pode lidar com qualquer volume de dados de streaming, escalando automaticamente para atender às demandas.
- **Integração com outros serviços AWS**: Integra-se facilmente com serviços como AWS Lambda, Amazon EC2, Amazon S3, Amazon Redshift, AWS IoT e outros.
- **Durabilidade**: Os dados são armazenados temporariamente em shards, permitindo a repetição de leitura em caso de falha.
- **Análise em tempo real**: Permite executar análises complexas em dados de streaming em tempo real usando SQL ou Java.
- para deixar ordenado, usamos o particion key para envio dos dados

O Amazon Kinesis é amplamente utilizado em casos de uso como monitoramento de aplicativos, análise de logs, processamento de dados de IoT,
 análise de dados de redes sociais, detecção de fraudes, monitoramento de métricas, entre outros cenários que exigem processamento
  e análise de dados em tempo real.
```

### kinesis data stream
- pode reter as mensagens de 1 a 365 dias
- desta forma podemos reprocessar os dados
- uma vez inserido os dados no kinesis, eles não podemo ser excluidos
- provisioned mode:
  - colocamos um numero determinado de shards
  - 1 mg por segundo
  - ou 2 mg por consumidor
  - pagamos por shard provisionado
- on-demand mode:
  - gerencia a capacidade
  - por default provisiona 4 mg ou 4000 records por segundo
   escalona automáticamente baseado nos ultimos 30 dias
  - pagamos por hora ou entrada por gb
- é implantado por região
- autorização com base no iam
- encripta usando https e rest usando kms


## kinesis data stream vs firehose
```
A escolha entre Kinesis Data Streams e Kinesis Data Firehose depende dos requisitos específicos do seu caso de uso. 
Aqui estão algumas diretrizes sobre quando usar cada um:

**Kinesis Data Streams**:
- Quando você precisa processar dados de streaming em tempo real usando aplicativos de consumo personalizados.
- Quando você precisa analisar, transformar ou enriquecer os dados antes de armazená-los.
- Quando você precisa de retenção de dados por um período mais longo (até 7 dias) para reprocessamento ou análise posterior.
- Quando você precisa de várias aplicações consumindo os mesmos dados simultaneamente.
- Quando você precisa controlar e dimensionar os recursos de consumo separadamente dos recursos de produção.

**Kinesis Data Firehose**:
- Quando você deseja simplesmente capturar e enviar dados de streaming diretamente para destinos de armazenamento, como Amazon S3, Amazon Redshift, etc.
- Quando você não precisa realizar processamento personalizado nos dados antes de armazená-los.
- Quando você não precisa de retenção de dados por períodos longos, apenas entrega contínua para armazenamento.
- Quando você não precisa de várias aplicações consumindo os mesmos dados simultaneamente.
- Quando você deseja um serviço totalmente gerenciado para entrega de dados de streaming, sem se preocupar com o dimensionamento de recursos ou gerenciamento de consumidores.

Em resumo, use o Kinesis Data Streams quando você precisa de processamento em tempo real, retenção de dados, consumidores múltiplos e controle granular sobre os recursos de consumo. Use o Kinesis Data Firehose quando você deseja simplesmente capturar e enviar dados de streaming diretamente para destinos de armazenamento de maneira totalmente gerenciada.

No entanto, esses serviços não são mutuamente exclusivos. Em alguns casos, você pode usar ambos em conjunto, 
onde o Data Firehose é usado para enviar dados brutos para armazenamento, enquanto o Data Streams
 é usado para processamento em tempo real e análise desses mesmos dados.
```

### kinesis x sns x sqs
```
Certamente! O Amazon SQS (Simple Queue Service), o Amazon SNS (Simple Notification Service) e o 
Amazon Kinesis são todos serviços de mensageria da AWS, mas com propósitos e características distintas. Aqui está uma explicação das diferenças entre eles:

**Amazon SQS (Simple Queue Service)**:

- É um serviço de filas de mensagens distribuídas que permite desacoplar e dimensionar componentes de aplicativos.
- As mensagens são armazenadas em filas e os componentes de aplicativos podem enviar e receber mensagens de forma assíncrona.
- Garante a entrega de mensagens pelo menos uma vez e mantém as mensagens armazenadas em filas por um período configurável.
- É adequado para processar cargas de trabalho assíncronas, desacoplar componentes de aplicativos e integrar sistemas distribuídos.
- É usado para cenários como processamento de tarefas em segundo plano, buffers de mensagens e comunicação assíncrona entre componentes.

**Amazon SNS (Simple Notification Service)**:

- É um serviço de publicação/assinatura (pub/sub) para entrega de mensagens e notificações.
- Os publicadores enviam mensagens para um tópico SNS, e os assinantes recebem as mensagens desse tópico.
- Suporta vários protocolos de entrega, como HTTP/HTTPS, email, SMS, AWS Lambda, Amazon SQS e dispositivos móveis.
- É adequado para enviar notificações em tempo real para vários destinatários, integrar componentes distribuídos e criar fluxos de
 trabalho orientados a eventos.
- É usado para cenários como notificações de aplicativos, alertas de monitoramento, comunicação entre microsserviços e entrega de
 mensagens para dispositivos móveis.

**Amazon Kinesis**:

- É um serviço de streaming de dados que permite coletar, processar e analisar fluxos de dados em tempo real.
- É composto por três serviços principais: Kinesis Data Streams, Kinesis Data Firehose e Kinesis Data Analytics.
- Kinesis Data Streams permite ingerir e processar grandes fluxos de dados em tempo real usando aplicativos de consumo personalizados.
- Kinesis Data Firehose permite capturar e enviar dados de streaming diretamente para destinos de armazenamento, como Amazon S3 e Amazon Redshift.
- Kinesis Data Analytics permite executar análises SQL ou aplicativos Java em fluxos de dados Kinesis em tempo real.
- É adequado para casos de uso como análise de logs, métricas, dados de IoT, eventos de cliques, detecção de fraudes e
 processamento de streaming em tempo real.

Em resumo, o Amazon SQS é usado para enfileirar mensagens e desacoplar componentes de aplicativos, o Amazon SNS é usado para
 enviar notificações e mensagens para vários destinatários, e o Amazon Kinesis é usado para coletar, processar e analisar fluxos de dados em tempo real. A escolha entre eles depende dos requisitos específicos do seu caso de uso, como entrega garantida de mensagens, processamento em tempo real ou envio de notificações.
```

# Ecs
- gerenciamento de containers da aws
- podemos subir containers usando ec2 ou fargate (serverless)
- ecs e dividio em:
  - cluster, que pode ter vários services
  - service, que possui uma task, mas podemos ter varias instancias dela
  - task e a definição do nosso microservice

## Role perfil ec2 vs role tasks ecs
- role de perfil, aonde podemos colocar as politicas mais genéricas, utilizadas por todas as tarefas, como"enviar logs, baixar imagem do ecr
- role de task aonde encontra-se as politicas mais específicas, que o microservice envolvido utilizara, como: salvar informções no dynamodb, s3 e etc

## autoscaling
- podemos aumentar o número de tasks com base em métricas, onde esta será o gatilho, quando adingido o percentual configurado, para 
- aumentar ou diminuir
- as métricas ficariam no cloudwatch  alarm

## integração com o event bridge
- podemos configurar o amazon eventbridge pra criar uma task dentro do ecs quando:
  - colocamos um objeto no bucket s3 
  - quando colocamos um agendamento no evento para criar uma task
  - quando nossos serviços estão recebendo muita mensagem de uma fila sqs, e o event brige trigga para aumentar o número de tasks (instâncias)

## ECR 
- repositório de imagens da aws
- é protegido pelo iam
- precisamos de roles, no caso para outros serviços, com as politicas certas, caso queria utiliza-lo

# eks
- gerenciador kubernetes da aws
- kubernetes e um orquestrador de containers, similar ao ecs
- ele é agnóstico a nuvem (outros provedores oferecem), ou seja, caso tenha um k8s e queria usar aws, é facil migrar
- k8s e de código aberto
- caso queria utilizar um storage, devemos definir o manifesto StorageClass, onde funciona com eks é:
  - abs
  - efs (funciona com farget)
  - fsx for lustre
  - fsx for netApp ONTAP

# AWS RUN
- forma mais facil de colocar app na aws usando container
- usa as boas pŕaticas
- interface facil para quem não conhece bem aws


# Serverless
- serviços nos quais não precisamos gerenciar ou provicionar servidores

## Beneficios lambda
- pagamos por execução
- integra com outros serviços da aws
- monitoramento pelo cloudwatch
- posso incrementar ram e cpu
- dimensionado automaticamente


## limits lambda
- memoria alocação até 10gb
- maximo execução 15 min
- variaveis de ambiente no máximo 4jb
- capacidade disco (funcion container) 10gb
- execução concorrente 1000
- para deploy, o arquivo zipado máximo de 50mb
- depois de descompactado, não pode passar 250mb
- utiliza directory temp para carregar outros arquivos no startup
- quando lançamos um lambda, ele e vinculado a uma vpc da aws, e não tem acesso aos recursos dentro do da nossa vpc
  - outro ponto é o uso de rds proxy para acessar a base de dados (o lambda),
    - lambda se conecta ao proxy
    - dessa forma tem melhor escalabilidade no pool de conexões, em vez de conectar direto ao rds
    - possui failover time e preserva as conexões
    - força o uso de iam authentication e secrets manager

### lambda vpc e vpc com seus recursos
```
Quando você cria uma função Lambda dentro de uma VPC (Virtual Private Cloud) da AWS, ela é executada dentro dessa VPC específica e isolada. Para que essa função Lambda possa acessar recursos (como bancos de dados, servidores, etc.) dentro da sua própria VPC, você precisa configurar corretamente a conectividade entre as duas VPCs.

Existem duas principais opções para permitir que a função Lambda acesse recursos em sua VPC:

1. **VPC Peering**:
   - O VPC Peering permite criar uma conexão de rede entre duas VPCs, permitindo que os recursos em cada VPC se comuniquem entre si como se estivessem na mesma rede.
   - Você precisa configurar um VPC Peering entre a VPC da AWS onde a função Lambda está sendo executada e sua própria VPC.
   - Depois de configurar o VPC Peering, você precisará adicionar regras de entrada às Security Groups dos recursos em sua VPC para permitir o acesso da VPC da AWS.

2. **AWS PrivateLink**:
   - O PrivateLink permite acessar serviços AWS de forma privada, sem precisar de uma conexão de internet ou atravessar a rede pública da AWS.
   - Você pode criar um endpoint de VPC para a função Lambda dentro de sua VPC, permitindo que sua função acesse recursos em sua VPC de forma segura e privada.
   - É necessário configurar uma política de recursos para permitir que a função Lambda acesse os recursos em sua VPC através do endpoint de VPC.

Após configurar a conectividade de rede usando uma dessas opções, você precisará fornecer à sua função Lambda as permissões necessárias para acessar os recursos em sua VPC, como bancos de dados, servidores, etc.

Aqui estão alguns passos gerais para configurar o acesso:

1. Identifique os recursos em sua VPC que a função Lambda precisa acessar (por exemplo, o endpoint de um banco de dados).
2. Configure a conectividade de rede entre a VPC da AWS e sua própria VPC usando VPC Peering ou PrivateLink.
3. Adicione regras de entrada às Security Groups dos recursos em sua VPC para permitir o acesso da VPC da AWS.
4. Forneça as permissões necessárias à função Lambda para acessar os recursos em sua VPC (por exemplo, através de políticas de IAM).
5. Configure a função Lambda com as informações de conexão necessárias (como endpoints, credenciais, etc.) para se comunicar com os recursos em sua VPC.

É importante lembrar que a configuração adequada da conectividade de rede e das permissões é crucial para garantir que a função Lambda possa acessar seus recursos de forma segura e eficiente.
```

### lambda rds
```
Conectar uma função Lambda ao Amazon RDS Proxy em vez de se conectar diretamente ao Amazon RDS oferece alguns benefícios importantes em termos de segurança, escalabilidade e gerenciamento de conexões. Aqui está como funciona e os principais benefícios:

**Como funciona**:

1. O RDS Proxy atua como um proxy de banco de dados, posicionando-se entre a aplicação (no caso, a função Lambda) e o banco de dados RDS.
2. A função Lambda estabelece uma conexão com o endpoint do RDS Proxy em vez de se conectar diretamente ao endpoint do RDS.
3. O RDS Proxy gerencia um pool de conexões de banco de dados e reutiliza conexões existentes sempre que possível.
4. Quando uma nova conexão é necessária, o RDS Proxy estabelece uma conexão com o banco de dados RDS e a encaminha para a função Lambda.

**Benefícios**:

1. **Segurança aprimorada**: Ao se conectar através do RDS Proxy, a função Lambda não precisa ter acesso direto ao endpoint do RDS, reduzindo a superfície de ataque.

2. **Controle de acesso refinado**: O RDS Proxy permite controlar com mais granularidade quais funções Lambda ou aplicações podem acessar o banco de dados, usando credenciais de banco de dados específicas.

3. **Gerenciamento de conexões aprimorado**: O RDS Proxy gerencia um pool de conexões de banco de dados, reutilizando conexões existentes quando possível e abrindo novas conexões somente quando necessário. Isso ajuda a evitar problemas de esgotamento de conexões, especialmente em cargas de trabalho com picos.

4. **Escalabilidade aprimorada**: O RDS Proxy pode lidar com um grande número de conexões simultâneas e solicitar automaticamente novos recursos de computação para acomodar a demanda adicional.

5. **Failover e failover automático**: O RDS Proxy pode detectar falhas no banco de dados primário e alternar automaticamente para uma instância de banco de dados em espera, sem exigir alterações nas aplicações do cliente.

6. **Monitoramento e logs aprimorados**: O RDS Proxy fornece métricas e logs detalhados sobre o uso de conexões, desempenho e outras informações úteis para monitoramento e solução de problemas.

Em resumo, conectar uma função Lambda ao RDS Proxy em vez de se conectar diretamente ao RDS oferece benefícios significativos em termos de segurança, gerenciamento de conexões, escalabilidade, failover automático e monitoramento. Embora adicione uma camada adicional, o RDS Proxy pode ajudar a simplificar o gerenciamento de conexões de banco de dados e fornecer uma experiência mais robusta e escalonável para aplicações sem servidor, como as funções Lambda.
```

## lambda snapstart
- aumentar a performance 10x para lambda feita em java
  - ele ja deixa o lambda pre iniciado 

## lambda@edge vs cloudfront functions
```
Certamente! O AWS Lambda@Edge e as CloudFront Functions são recursos da AWS que permitem executar código personalizado em resposta a eventos que ocorrem na edge (borda) da rede de entrega de conteúdo (CDN) da AWS, a CloudFront.

**AWS Lambda@Edge**:

O Lambda@Edge permite executar funções Lambda em resposta a eventos CloudFront, como a requisição de um objeto, a visualização de um objeto ou a resposta de um objeto. Isso permite personalizar o comportamento da CDN em tempo de execução.

Alguns casos de uso comuns para o Lambda@Edge incluem:

1. **Manipulação de Conteúdo**: Modificar o conteúdo servido pela CloudFront, como inserir/remover cabeçalhos HTTP, reescrever URLs, otimizar imagens, entre outros.
2. **Segurança Aprimorada**: Implementar controles de segurança adicionais, como proteção contra bots, filtragem de solicitações maliciosas, autenticação e autorização personalizadas.
3. **Roteamento Inteligente**: Redirecionar solicitações com base em condições personalizadas, como dispositivo, localização geográfica ou outros fatores.
4. **Monitoramento e Logs**: Registrar e monitorar solicitações e respostas da CloudFront para análise e depuração.

**CloudFront Functions**:

As CloudFront Functions são um recurso mais recente e simplificado em comparação com o Lambda@Edge. Elas permitem executar código leve em tempo de execução durante o processamento de solicitações na borda da CloudFront, sem a necessidade de provisionar e gerenciar funções Lambda separadas.

As CloudFront Functions são mais adequadas para casos de uso simples, como:

1. **Manipulação de Cabeçalhos HTTP**: Adicionar, remover ou modificar cabeçalhos HTTP nas solicitações e respostas.
2. **Reescrita de URLs**: Redirecionar solicitações com base em padrões de URL.
3. **Filtragem de Solicitações**: Bloquear ou permitir solicitações com base em critérios simples, como cabeçalhos, parâmetros de consulta ou a presença de cookies.

As CloudFront Functions são escritas em um subconjunto de JavaScript e executadas diretamente na edge da CloudFront, tornando-as mais leves e rápidas do que as funções Lambda@Edge.

Em resumo, tanto o Lambda@Edge quanto as CloudFront Functions permitem personalizar o comportamento da CDN da AWS com código personalizado. O Lambda@Edge é mais poderoso e flexível, permitindo a execução de lógica mais complexa, enquanto as CloudFront Functions são mais simples e adequadas para casos de uso mais básicos.

A escolha entre os dois depende das necessidades específicas do seu caso de uso, considerando a complexidade do código necessário, os requisitos de desempenho e a facilidade de manutenção.
```

# Dynamodb
- o banco de dados ja existe, criamos apenas tabelas
- a tabela tem uma primary key, que pode ser composta por chave + range
- podemos ter tambem indexes locais, onde a chave é igual a chave da primary key, mas um range diferente
- podemos ter indexes secundarios, onde c chave primary e o range, são diferentes da primary key.
- temos os modes de capacidade de Reqd/write
  - provisioned mode: determinamos quanto de escrita e leitura por segundo, ideal quanto temos uma previsibilidade e economia de custos.
  - on-demand mode: não precisamos nos planejar, ideal quando temos uma variação grande de escrita/leitura, pagamos por leitura/gravação

## DAX dynamodb accelerator
- cache gerenciado, para dynamodb
- ideal quando temos muitas leituras, afim de mitigar o tempo de resposta
- por padrão o ttl é de 5 min
- latencia de segundos
- não precisamos mudar nada na nossa app
- 

## dynamodbDB stream processing
```
O DynamoDB Streams é um recurso do Amazon DynamoDB que captura um feed de log sequencial de todas as modificações (criação, atualização e exclusão) feitas em uma tabela do DynamoDB. Ele permite que você capture essas alterações na tabela e replique os dados para outro sistema, como um sistema de processamento de dados ou um data lake, de forma assíncrona e em tempo real.

O DynamoDB Streams funciona da seguinte maneira:

1. **Habilitação de Streams**: Ao criar ou atualizar uma tabela do DynamoDB, você pode habilitar o DynamoDB Streams para essa tabela. Você também pode habilitar ou desabilitar os Streams em tabelas existentes a qualquer momento.

2. **Captura de Alterações**: Sempre que um item é adicionado, modificado ou removido da tabela habilitada para Streams, o DynamoDB captura essas alterações e as grava em um log de Streams associado à tabela.

3. **Estrutura do Registro de Streams**: Cada registro de Streams contém informações sobre a operação realizada (inserção, modificação ou remoção), o código de partição e a chave de classificação do item, bem como a imagem completa do item antes e depois da operação.

4. **Consumo de Streams**: Você pode consumir os registros de Streams usando serviços da AWS, como o AWS Lambda, o Amazon Kinesis, ou seu próprio aplicativo personalizado. O consumo de Streams pode ser feito de forma síncrona (processando registros individuais à medida que chegam) ou assíncrona (processando lotes de registros em intervalos regulares).

5. **Processamento Adicional**: Depois de consumir os registros de Streams, você pode executar tarefas adicionais, como armazenar os dados em um data lake, realizar análises em tempo real, disparar notificações ou atualizar outros sistemas.

Os casos de uso comuns para o DynamoDB Streams incluem:

1. **Replicação de Dados**: Replicar dados do DynamoDB para outros sistemas de armazenamento ou processamento de dados, como o Amazon Redshift, o Amazon Elasticsearch Service ou o Amazon S3.

2. **Processamento de Eventos**: Disparar ações ou fluxos de trabalho com base em alterações nos dados do DynamoDB, como enviar notificações, atualizar caches ou executar lógica de negócios.

3. **Análise de Dados**: Processar os dados de alterações do DynamoDB em tempo real para fins de análise, como análise de tendências, detecção de fraudes ou geração de relatórios.

4. **Integração de Sistemas**: Manter outros sistemas atualizados com as alterações no DynamoDB, permitindo a sincronização de dados entre sistemas diferentes.

O DynamoDB Streams oferece uma maneira eficiente e escalável de capturar e processar alterações de dados em tempo real, sem a necessidade de sondagem periódica ou rastreamento manual de alterações. Ele é especialmente útil em cenários onde é necessário replicar dados, reagir a eventos ou executar processamento de dados em tempo real com base nas alterações no DynamoDB.
```

## tabelas globais
```
O DynamoDB Global Tables é um recurso que permite replicar uma tabela do DynamoDB em várias regiões da AWS de forma transparente, proporcionando uma experiência de replicação multinacional completa, gerenciada e altamente disponível. Isso permite que você execute aplicativos globalmente distribuídos com latência de leitura e gravação baixa para os usuários finais.

Aqui estão alguns detalhes importantes sobre as Tabelas Globais do DynamoDB:

1. **Replicação entre regiões**: As Tabelas Globais replicam automaticamente os dados entre as regiões selecionadas, mantendo-os sincronizados. Você pode especificar quais regiões deseja que sua tabela seja replicada.

2. **Tabela totalmente operacional**: Cada réplica de tabela em uma região diferente é uma tabela do DynamoDB totalmente operacional. Você pode executar leituras e gravações em qualquer réplica da tabela.

3. **Consistência eventual**: As Tabelas Globais fornecem consistência eventual entre as replicações regionais. Isso significa que, após uma atualização em uma região, há um atraso antes que a alteração seja visível em outras regiões.

4. **Chaves de tabela idênticas**: Todas as réplicas de uma Tabela Global compartilham o mesmo esquema de chave de tabela. No entanto, você pode definir outras configurações, como configurações de capacidade de leitura/gravação e índices secundários, de forma independente para cada réplica.

5. **Failover transparente**: Se uma região falhar, o DynamoDB poderá redirecionar automaticamente o tráfego para outra região replicada, garantindo alta disponibilidade.

6. **Compatibilidade com outros recursos do DynamoDB**: As Tabelas Globais são compatíveis com outros recursos do DynamoDB, como Streams, Acelerador DAX, Backup e Restauração, Rastreamento de Objetos e Encriptação.

Os casos de uso comuns para as Tabelas Globais incluem:

1. **Aplicativos globais**: Para aplicativos que precisam ser implantados globalmente, as Tabelas Globais permitem que os usuários de diferentes regiões acessem os dados com baixa latência.

2. **Alta disponibilidade**: Com réplicas em várias regiões, as Tabelas Globais fornecem alta disponibilidade e tolerância a falhas regionais.

3. **Conformidade local**: Para atender a requisitos regulatórios ou de conformidade, os dados podem ser mantidos dentro de uma região específica.

4. **Replicação de dados**: As Tabelas Globais podem ser usadas para replicar dados entre regiões para fins de backup, recuperação de desastres ou análise.

Embora as Tabelas Globais forneçam capacidades poderosas, é importante observar que elas podem incorrer em custos adicionais, uma vez que você está mantendo várias réplicas de seus dados em diferentes regiões. No entanto, para aplicativos globais ou com requisitos de alta disponibilidade, as Tabelas Globais do DynamoDB podem ser uma solução valiosa.
```
# api gateway aws
````
O Amazon API Gateway é um serviço gerenciado pela AWS que permite criar, publicar, manter, monitorar e proteger APIs RESTful e WebSocket na nuvem. Ele atua como uma "porta de entrada" para aplicativos, permitindo que eles acessem dados, funções de negócios ou recursos de back-end, como:

- Serviços AWS (Lambda, DynamoDB, entre outros)
- Servidores HTTP/HTTPS em nuvem privada ou local
- Serviços web públicos

Aqui estão algumas das principais características e benefícios do API Gateway:

1. **Proxy Reverso**: Atua como um proxy reverso, recebendo solicitações de API dos clientes, as encaminha para os serviços de back-end apropriados e retorna as respostas.

2. **Mapeamento e Transformação de Dados**: Permite mapear solicitações e respostas entre formatos diferentes, como JSON para XML ou vice-versa.

3. **Controle de Acesso**: Fornece mecanismos de autenticação e autorização, como chaves de API, autenticação do AWS IAM, autenticação baseada em token, etc.

4. **Cached de Resposta**: Pode armazenar em cache respostas de back-end para melhorar o desempenho.

5. **Monitoramento e Logs**: Oferece logs detalhados e métricas para monitoramento e rastreamento de chamadas de API.

6. **SDK Geração e Documentação Swagger**: Gera SDKs cliente e documentação Swagger para simplificar a integração com seus aplicativos cliente.

7. **Versionamento de API**: Permite implantar novos estágios ou versões de uma API sem interrupções.

8. **WebSocket APIs**: Suporta APIs WebSocket para aplicativos em tempo real, além das APIs RESTful.

9. **Integração Sem Servidor**: Integra-se perfeitamente com o AWS Lambda e outros serviços AWS sem servidor.

Usando o API Gateway, você pode criar uma camada lógica consistente e escalável para expor serviços back-end através de APIs bem definidas, gerenciar o tráfego de API, aplicar políticas de segurança e transformar dados, entre outros benefícios. É uma parte fundamental da arquitetura de microsserviços e APIs na AWS.

Certamente! Os três tipos de gateways AWS edge-optimized, private e regional são:

1. **Edge-Optimized Gateways**:
   - Esses gateways são projetados para lidar com transferências de dados em larga escala.
   - Eles são implantados em locais estratégicos em todo o mundo, próximos aos recursos da Amazon CloudFront, para minimizar a latência.
   - Esses gateways são ideais para cenários que envolvem transferência de grandes quantidades de dados, como backups, recuperação de desastres e transferências periódicas de dados.

2. **Private Gateways**:
   - Os private gateways fornecem uma conexão segura entre sua rede local (data center ou escritório) e uma Virtual Private Cloud (VPC) na AWS.
   - Eles utilizam conexões de rede privada dedicadas, como AWS Direct Connect ou VPN.
   - Esses gateways permitem que você acesse recursos em sua VPC de forma privada e segura, sem precisar rotear o tráfego pela Internet pública.
   - Private gateways são recomendados quando você precisa de largura de banda consistente, latência baixa e transferências de dados seguras entre sua rede local e a AWS.

3. **Regional Gateways**:
   - Os regional gateways são implantados em todas as regiões da AWS.
   - Eles fornecem conexão entre uma VPC e buckets do Amazon S3 na mesma região.
   - Essas conexões são realizadas através do backbone de rede da AWS, que é mais rápido e confiável do que a Internet pública.
   - Os regional gateways são ideais para transferências de dados frequentes entre recursos em uma VPC e buckets do S3 na mesma região.
   - Eles oferecem melhor desempenho, segurança e custo reduzido em comparação com transferências pela Internet pública.

Em resumo, os edge-optimized gateways são projetados para transferências de dados em larga escala com baixa latência, os private gateways fornecem conexões seguras entre sua rede local e a AWS, e os regional gateways são otimizados para transferências de dados eficientes entre recursos em uma VPC e buckets do S3 na mesma região.
````

## Amazon Cognito:
```
O Amazon Cognito é um serviço de gerenciamento de identidade que ajuda a adicionar recursos de autenticação, autorização e gerenciamento de usuários às suas aplicações. Ele simplifica o processo de integração de autenticação em seus aplicativos. O Cognito oferece duas principais funcionalidades:

User Pools: Permite gerenciar um diretório de usuários e fornecer recursos de autenticação, como inscrição, login, recuperação de senha, autenticação de dois fatores (2FA) e federação de identidades sociais (Facebook, Google, etc.).
Identity Pools: Permite obter credenciais de acesso temporárias para autenticar usuários e acessar recursos da AWS, como o Amazon S3 ou o DynamoDB. As Identity Pools suportam diferentes provedores de identidade, como User Pools, provedores de identidade social ou identidades de contas da AWS.
federado : para acessar diretamente recursos da aws, como s3
```


# Detalhes no exame
```
Você tem um site estático hospedado em um bucket S3. Você criou uma distribuição do CloudFront que aponta para seu bucket S3 para atender melhor às suas solicitações e melhorar o desempenho. Depois de um tempo, você percebeu que os usuários ainda podem acessar seu site diretamente do bucket S3. Você deseja forçar os usuários a acessar o site somente por meio do CloudFront. Como você conseguiria isso?


Para forçar os usuários a acessarem o site estático somente através do CloudFront e não diretamente pelo bucket S3, você pode seguir estes passos:

Configurar o acesso público do bucket S3 para "Bloquear todo o acesso público":
Acesse as propriedades do bucket S3 que hospeda o site estático.
Na seção "Permissões", em "Bloquear configuração de acesso público (configuração de bucket)", clique em "Editar".
Marque todas as opções para bloquear o acesso público.
Salve as mudanças.
Criar uma Política de Bucket para permitir acesso somente ao CloudFront:
Abra o console do AWS S3 e navegue até o bucket que hospeda o site estático.
Acesse a seção "Permissões" e clique em "Política do bucket".
Substitua a política existente pela seguinte (substitua [ACCOUNT_ID] pelo seu ID de conta AWS e [DISTRIBUTION_ID] pelo ID da sua distribuição CloudFront):
================================================================================================================================================


O AWS WAF pode detectar a presença de código SQL que provavelmente seja mal-intencionado (conhecido como injeção de SQL). Ele também pode detectar a presença de um script que provavelmente seja mal-intencionado (conhecido como script cross-site).

Para obter mais informações sobre o AWS WAF, consulte AWS WAF.

Essa solução atende ao requisito de um RTO de 5 minutos. As instâncias são executadas com baixa capacidade e podem ser dimensionadas em minutos.
=========
Uma empresa está projetando uma arquitetura de recuperação de desastres (DR) para um aplicativo importante na AWS. A empresa determinou que o RTO é de 5 minutos com uma capacidade mínima de instância para dar suporte ao aplicativo no site de DR da AWS. A empresa precisa minimizar os custos da arquitetura de DR.

Qual estratégia de DR atenderá a esses requisitos?

Para obter mais informações sobre standby passivo, consulte Plan for Disaster Recovery (DR) (Plano de recuperação de desastres (DR)).
=========
Uma empresa de mídia está projetando uma nova solução para renderização gráfica. A aplicação exige até 400 GB de armazenamento para dados temporários que são descartados depois que os quadros são renderizados. Ela exige aproximadamente 40.000 IOPS aleatórias para realizar a renderização.

Qual é a opção de armazenamento MAIS econômica para essa aplicação de renderização?
uma instancia da amazon ec2 otimizada para armazenamento de instâncias
==========
Uma empresa está desenvolvendo uma aplicação de bate-papo que será implantada na AWS. A aplicação armazena as mensagens usando um modelo de dados de chave/valor. Grupos de usuários geralmente leem as mensagens várias vezes. Um arquiteto de soluções precisa selecionar uma solução de banco de dados que seja escalada para uma alta taxa de leituras e entregue mensagens com latência de microssegundos.

Qual solução de banco de dados atenderá a esses requisitos?

dynamodb com acelerador dax
==========
Uma empresa precisa procurar detalhes de configuração sobre como foi iniciada uma instância do Amazon EC2 baseada em Linux.

Qual comando um arquiteto de soluções deve executar na instância do EC2 para coletar os metadados do sistema?


A única maneira de recuperar metadados da instância é usar o endereço local do link, que é 169.254.169.254.
==========
Uma aplicação de relatórios é executada em instâncias do Amazon EC2 atrás de um Application Load Balancer. As instâncias são executadas em um grupo do Amazon EC2 Auto Scaling em várias zonas de disponibilidade. Para relatórios complexos, a aplicação pode levar até 15 minutos para responder a uma solicitação. Um arquiteto de soluções está preocupado com o fato de os usuários receberem erros HTTP 5xx se uma solicitação de relatório estiver em andamento durante um evento de redução.

O que o arquiteto de soluções deve fazer para garantir que as solicitações do usuário sejam concluídas antes que as instâncias sejam encerradas?

Aumentar o tempo limite de atraso de cancelamento de registro, para o grupo de destino das instâncias para mais de 900 segundos.
Por padrão, o Elastic Load Balancing aguarda 300 segundos antes da conclusão do processo de cancelamento do registro, o que pode ajudar a concluir as solicitações em andamento para o destino. Para alterar a quantidade de tempo que o Elastic Load Balancing aguarda, atualize o valor de atraso do cancelamento do registro.

Para obter mais informações sobre o atraso do cancelamento de registro, consulte Atraso do cancelamento de registro.
```
