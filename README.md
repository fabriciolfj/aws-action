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

**2. Spread** 

- Instâncias espalhadas por diferentes racks(prateleiras) dentro de uma zona de disponibilidade. Protege contra falhas simultâneas no hardware do rack.

**3. Partition**

- Esparrama instâncias em partições lógicas que refletem os grupos de particionamento do hardware subjacente, como diferentes prateleiras em racks separados.

As vantagens dos Placement Groups:

- Permite influenciar a posição física das instâncias
- Menor latência em rede entre instâncias no mesmo grupo
- Isola falhas de hardware

As limitações:

- Nem todos os tipos de instância são compatíveis
- Uma instância só pode fazer parte de um Placement Group

Então resumindo, Placement Groups são úteis para obter desempenho, isolamento de falhas e controle sobre posicionamento de instâncias.
