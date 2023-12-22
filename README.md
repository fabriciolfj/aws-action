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
