# aws-action

## pontos importantes
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
- veja o serviço como deployment e a tarefa como um pode
- caso queria adicionar um load balance, precisa criar um novo serviço através de uma task definitions
