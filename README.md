# Atividade - AWS - Docker

Impantar a seguinte arquitetura:
![Arquitetura](https://user-images.githubusercontent.com/127341401/251820538-e412bb89-811c-4f1f-a8f9-b87715a1cc59.png)

1. Instalação e configuração do DOCKER ou CONTAINERD no
host EC2;

    Ponto adicional por utilizar a instalação via script de
Start Instance (user_data.sh)

2. Efetuar Deploy de uma aplicação Wordpress com:
  - container de aplicação
  - RDS database Mysql

3. configuração da utilização do serviço EFS AWS para estáticos
do container de aplicação Wordpress

4. configuração do serviço de Load Balancer AWS para a aplicação
Wordpress

Pontos de atenção:
- Não utilizar ip público para saída do serviços WP (Evitem
publicar o serviço WP via IP Público)
- Sugestão para o tráfego de internet sair pelo LB (Load
Balancer)
- Pastas públicas e estáticos do wordpress sugestão de utilizar
o EFS (Elastic File System)
- Fica a critério de cada integrante (ou dupla) usar Dockerfile
ou Dockercompose;
- Necessário demonstrar a aplicação wordpress funcionando
(tela de login)
- Aplicação Wordpress precisa estar rodando na porta 80 ou
8080;




## Serviço VPC
Acesse o Console da Amazon Web Services e entre na área de serviços VPC.

### Criando VPC
- Clique em "Criar VPC"
- Agora vamos configurar a VPC.
  - selecione "Somente VPC".
  - Dê um nome a esta VPC, pode ser "VPC-Projeto01".
  - Em "Bloco CIDR IPv4", selecione a opção "Entrada manual de CIDR IPv4"
  - Logo baixo, adicione o CIDR IPv4 desejado. Aqui usaremos "172.28.0.0/16".
  - Em "Bloco CIDR IPv6", mantenha selecionada a opção "Nenhum bloco IPv6".
  - Locação "Padrão".
  - Finalize e crie a VPC.

### Criando Sub-Redes
Ainda no serviço VPC, acesse no menu lateral esquerdo "Sub-redes".
- Clique em "Criar Sub-rede".
- Em "VPC", selecione a VPC criada anteriormente.
- Em "configurações de sub-rede", adicione 4 sub-redes(2 privadas e 2 públicas) com as configurações mostradas na tabela:

| Nome da sub-rede | Zona de disponibilidade | Bloco CIDR IPv4  | 
| ------------- | ------------- | ------------- |
| SN-Private01  | us-east-1a  | 172.28.0.0/24  | 
| SN-Private02  | us-east-1b | 172.28.3.0/24  |
| SN-Public01  | us-east-1a  | 172.28.1.0/24  |
| SN-Public02  | us-east-1b   | 172.28.2.0/24  |

- Finalize a criação clicando em "Criar sub-rede"
- No painel de todas as sub-redes, selecione as duas públicas(SN-Public), uma de cada vez, e vá em "ações" > "Editar configurações de sub-rede" e marque a opção "Habilitar endereço IPv4 público de atribuição automática" e finalize.

### Criando Gateway de internet
- No menu lateral esquerdo, clique em "Gateways da Internet" e depois em "Criar gateway da internet".
- Em "Tag de nome", defina um nome, neste caso usarei "IG-01"
- Finalize
- No painel dos gateways de internet, selecione o que acabamos de criar e vá em "Ações" > Associar à VPC > associe à VPC que criamos.

### Criando Nat Gateway
- No menu lateral esquerdo, clique em "gateways NAT" e depois em "Criar gateway NAT".
- Defina um nome. Aqui usarei "NatG-pedro"
- Selecione a sub-rede pública "SN-Public01".
- Em "Tipo de conectividade", mantenha selecionada a opção "Público"
- Clique em "Alocar IP elástico"
- Finalize

### Criando Tabelas de Rotas
- No menu lateral esquerdo, clieque em "Tabela de rotas" e depois em "Criar tabela de rotas"
- Crie 2 tabelas, uma para as sub-redes publicas (TR-Public) e outra para as sub-redes privadas (TR-Private).
- Após criar, associe cada uma nas suas respectivas sub-redes selecionando a tabela de rotas > "Editar associações de sub-rede" > selecione as respectivas sub-redes.
- Agora iremos configurar as rotas de cada tabela para permitir o tráfego na internet para cada sub-rede, a sub-rede pública com gateway de internet e a sub-rede privada com o gateway NAT:
  - sub-rede pública

    Selecione a tabela de rotas, no painel inferior, siga para rotas e selecione "Editar rotas". Após isso, adicione rotas com:

    Destino: ```0.0.0.0/0```

    Alvo: ```gateway da internet```


  - sub-rede privada
    
    Selecione a tabela de rotas, no painel inferior, siga para rotas e selecione "Editar rotas". Após isso, adicione rotas com:

    Destino: ```0.0.0.0/0```

    Alvo: ```NAT gateway ```

## Serviço EC2
Acesse o console da Amazon Web Services, e entre nos serviços de EC2

## Configurando Grupos de Segurança

Agora vamos configurar cinco grupos de segurança para diferentes componentes da 
infraestrutura: Bastion Host, Balanceador de Carga, Aplicação, EFS e RDS.

- Clique em "Criar grupo de seguarança e configure:

### Grupo de Segurança do Bastion Host
- Defina o nome do grupo, neste caso usarei "SG-BastioHost".
- Na descrição insira "Grupo de seguranca do bastion host para entrada via ssh apenas do meu IP"
- Selecione a VPC "VPC-Projeto01".
- Adicione a seguinte regra de entrada:

| Tipo | Intervalo de Portas | Protocolo | Tipo de Origem    |
|-------------------|--------------------|-----------|-----------|
| SSH | 22 | TCP       | MEU-IP  |


### Grupo de Segurança do Balanceador de Carga
- Defina o nome do grupo, neste caso usarei "SG-LoadBalancer".
- Na descrição insira "Grupo de seguranca do Load Balancer"
- Selecione a VPC "VPC-Projeto01".
- Adicione as seguintes regras de entrada:

| Tipo | Intervalo de Portas | Protocolo | Origem    |
|-------------------|--------------------|-----------|-----------|
| HTTP | 80 | TCP       | 0.0.0.0/0  |

### Grupo de Segurança da Aplicação
- Defina o nome do grupo, neste caso usarei "SG-App".
- Na descrição insira "Grupo de seguranca da aplicacao"
- Selecione a VPC "VPC-Projeto01".
- Adicione as seguintes regras de entrada:

| Tipo | Intervalo de Portas | Protocolo | Origem                        |
|-------|-------|-----------|-------------------------------|
| SSH | 22    | TCP       | IP PRIVADO DO BASTION HOST |
| HTTP | 80    | TCP       | SG DO LOAD BALANCER |
| NFS | 2049   | TCP       | SG DO EFS |

### Grupo de segurança do EFS
- Defina o nome do grupo, neste caso usarei "SG-EFS".
- Na descrição insira "Grupo de seguranca do EFS"
- Selecione a VPC "VPC-Projeto01".
- Adicione as seguintes regras de entrada:

| Tipo | Intervalo de Portas | Protocolo | Origem                        |
|-------|-------|-----------|-------------------------------|
| NFS | 2049    | TCP       | CIRDR IPv4 DA VPC |

### Grupo de segurança do RDS
- Defina o nome do grupo, neste caso usarei "SG-RDS".
- Na descrição insira "Grupo de seguranca do RDS"
- Selecione a VPC "VPC-Projeto01".
- Adicione as seguintes regras de entrada:

| Tipo | Intervalo de Portas | Protocolo | Origem         |
|-------|-------|-----------|-------------------------------|
| MYSQL/Aurora | 3306    | TCP       | CIRDR IPv4 DA VPC |

## Serviço RDS
No console da Amazon Web Services, acesse o serviço RDS.

### Criando RDS
- Clique em "Criar Banco de dados"
  - Método de criação: Padrão
  - Mecanismo: MySQL
  - Modelo: Nível gratuito
  - Nome: DB-01
  - senha: (de sua preferência)
  - VPC: VPC criada anteriormente
  - Acesso Público: Sim
  - Grupos de segurança da VPC existentes: "SG-RDS"
  - Finalize em "Criar banco de dados"


## Serviço EFS
### Criando EFS
O Amazon Elastic File System (Amazon EFS) oferece um sistema de arquivos simples, escalável e elástico para cargas de trabalho de uso geral para uso com Serviços de Nuvem AWS e recursos no local. 
- Na AWS, Acesse o serviço EFS(Elastic File System)
- Clique em "Criar sistema de arquivos"
- Defina o nome para o EFS(opcional), usarei "EFS-01"
- Selecione a VPC "VPC-Projeto01".
- Crie.

### Configurando EFS
- No painel dos sistemas de arquivos, selecione a EFS criada e clique em visualizar detalhes.
- Abaixo, procure por "Rede" e clique em "Gerenciar"
- Em cada zona de disponibilidade, selecione o grupo de segurança do EFS.


## Serviços EC2
Acesse o serviço de EC2

### Par de chaves
Antes de executar uma instância, devemos criar um par de chaves. 
- Na lateral esquerda, acesse "Pares de chaves".
- Clique em "Criar par de chaves".
- Dê um nome para o par de chaves, usarei "keySSHPedro".
- Tipo de par de chaves: ```RSA```
- Formato de arquivo de chave privada: ```.PEM```
- A chave será baixada em formato ```.PEM```, você deve salvá-la em um diretório/pasta segura.

### Criar instância Bastion Host
- Em instâncias, clique em "Executar instância" 
- Dê o nome para a instância, colocarei "BastionHost"
- Em "imagens de aplicação e de sistema operacional", selecione ```Amazon Linux 2```
- Tipo de instância: ```t2.micro```
- Par de chaves: ```keySSHPedro```
- Em "Configurações de rede", clique em "Editar" > em "Rede", selecione a VPC ```VPC-Projeto01``` > em "Sub-Rede", selecione a ```SN-Public01``` > Selecione o grupo de segurança do Bastion host ```SG-BastioHost``` 
- Finalize e execute a instância.

### Criar instância aplicação
- Em instâncias, clique em "Executar instância" 
- Dê o nome para a instância, colocarei "App"
- Em "imagens de aplicação e de sistema operacional", selecione ```Amazon Linux 2```
- Tipo de instância: ```t2.micro```
- Par de chaves: ```keySSHPedro```
- Em "Configurações de rede", clique em "Editar" > em "Rede", selecione a VPC ```VPC-Projeto01``` > em "Sub-Rede", selecione a ```SN-Private01``` > Selecione o grupo de segurança da aplicação ```SG-App```
- Em "Avançados", em "User Data", preencha com:
  ```
  #!/bin/bash
  sudo yum update -y
  sudo yum install nfs-utils -y
  sudo mkdir -p /mnt/efs
  sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-0a769358c96fbe63d.efs.us-east-1.amazonaws.com:/ /mnt/efs
  sudo yum install -y docker
  sudo systemctl start docker
  sudo systemctl enable docker
  sudo usermod -a -G docker ec2-user
  sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
  sudo chmod +x /usr/local/bin/docker-compose
  sudo echo "fs-0a769358c96fbe63d.efs.us-east-1.amazonaws.com:/    /mnt/efs         nfs    defaults          0   0 " >> /etc/fstab
  ```

- Finalize e execute a instância.


## Acessando a instância da aplicação
Pensando em uma maior segurança, foi utilizado um Bastion Host para acessar a aplicação. O bastion host é usado como ponto de entrada seguro para acessar e gerenciar servidores em uma rede, protegendo-os de acesso direto não autorizado. Então primeiro o acessaremos para, a partir dele, acessar a aplicação.

### Acessando Bastion Host
- No terminal da sua máquina, vá até o diretório que está seu Par de chaves.
- Digite ```ssh-add <nome_do_seu_ParDeChaves>``` para usar um agente ssh e não precisar copiá-lo para dentro da instância bastion host.
- Agora acesse a EC2 BastionHost digitando ```ssh -A -i <nome_do_seu_ParDeChaves> ec2-user@<Ip_Público_do_BastionHostEC2>```
- Agora você está dentro do Bastion

### Acessando a instância da aplicação 
- Agora iremos acessar a aplicação a partir deste ambiente digitando ```ssh ec2-user@<Ip_Privado_da_AplicaçãoEC2>```
- Pronto

### Criando arquivo docker compose na ec2 da aplicação
- Dentro da EC2 da aplicação, vá até o diretório do efs ```cd /mnt/efs``` 
- Para criar e configurar o docker-compose.yml digite ```sudo nano docker-compose.yml``` e adicione o seguinte código:

```version: '3'
services:
  wordpress:
    image: wordpress:latest
    ports:
      - "80:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: <ENDPOINT_DO_SEU_RDS> 
      WORDPRESS_DB_USER:  <SEU_USUARIO_DO_DB>
      WORDPRESS_DB_PASSWORD: <SUA_SENHA_DO_DB>
      WORDPRESS_DB_NAME: <NOME_INICIAL_DO_DB>
    volumes:
      - /mnt/efs/website:/var/www/html
```

- Agora para subir o container, digite ```docker-compose up ```

### Configurando wordpress
- Vá até o Load Balancer e copie seu "DNS Name" e pesquise no seu navegador, irá abrir a pagina do wordpress.
- Selecione o idioma desejado e se cadastre.
- Finalize


## Criando e configurando Balanceador de carga (Load Balancer)
No console da Amazon Web Services, acesse o serviços de EC2

### Grupo de destino (Target Grop) 
- No painel esquerdo, acesse "Grupos de destino"
- Clique em "Create target group" para criar grupo de destino e configure:
  -Tipo: ```instância```
  - Nome do grupo de destino: ```TG-APP```
  - VPC: ```VPC-Projeto01```
  - Clique em Configurações avançadas de integridade" e deixe códigos de sucesso: ```200``` 
  - Após criado, no painel dos grupos de destino, selecione o mesmo e cliquem em "Ações" > "Registrar destinos" > Selecione a instância da aplicação(App) > abaixo, clique em "incluir como pendente", e finalmente, Clique em "Registrar destinos pendentes".

### Balanceador de carga (Load balancer)
- No painel esquerdo, acesse "Balanceador de carga".
- Clique em "Criar balanceador de carga"
- Selecione "Application Load Balancer"
- Em "Configurações básicas":
  - Nome: ```ALB-APP```
  - Esquema: ```voltado pra internet```
  - Tipo de endereço IP: ```IPv4```
- Em "Mapeamente de rede":
  - VPC: ```VPC-Projeto01```
  - Selecione as duas AZs: us-east-1a (SN-Public01), us-east-1b(SN-PUBLIC02)

- Em "Grupos de segurança", selecione o grupo de segurança do Load Balancer (SG-LoadBalancer)
- Listeners:

| Protocolo | Portas | Ação padrão |
|-------|-------|-----------|
| HTTP | 80   | grupo de destino criado (TG-APP)      | 


## Serviço EC2
### Criando AMI da instância da aplicação
Acesse o painel das instâncias
- Selecione a instância da aplicação e vá em "Ações" > "Imagem e modelos" > "Criar imagem".
- Agora criaremos uma imagem da EC2:
Nome: ```AMI-APP```
Descrição: ```AMI da aplicação```
Não reinicializar: deixe desabilitado
Volumes de instâncias: Apenas aumente o "Tamanho" para 16
- Finalize a criação

### Criando modelo de execução
No painel esquerdo, acesse "Modelos de execução"
- CLique em "Criar modelo de execução"
- Defina um nome. Usarei "Modelo-01"
- Descrição: ```Modelo-01```
- Marque a opçao ```Fornecer orientação para me ajudar a configurar um modelo que eu possa usar com o Auto Scaling do EC2```
- Em "Imagens de aplicação e de sistema operacional (imagem de máquina da Amazon) - obrigatório", selecione "Minhas AMIs" > "De minha propriedade" e selecione a AMI criada anteriormente.
- Em "Tipo de instância", selecione ```t2.micro```
- Selecione o par de chaves criado.
- Em "Configurações de Rede":
  - Sub-rede: ```Não incluir no modelo de execução```
  - Grupo de segurança: ```Selecionar grupo de segurança existente``` > selecione o grupo de segurança da aplicação (```SG-App```)
- Em "Detalhes avançados" coloque o user data ```sudo yum update -y```
- Finalize em "Criar modelo de execução"

### Auto Scaling
No painel esquerdo, acesse "Grupos Auto Scaling"
- Clique em "Criar Grupos Auto Scaling"
- Defina um nome. Usarei "ASG-01"
- Selecione o modelo que foi criado (Modelo-01) e clique em "Próximo"
- Em "Rede":
  - VPC: selecione a VPC criada anteriormente
  - Zonas de disponibilidade e sub-redes: selecione as duas sub-redes privadas criadas anteriormente (SN-Private01, SN-Private02)
- "Próximo"
- Em "Balanceamento de carga", selecione "Anexar a um balanceador de carga existente"
- Selecione o grupo de destino criado (TG-APP)
- Em "Verificações de integridade", marque a opção "Ative as verificações de integridade do Elastic Load Balancing"
- "Próximo"
- Em "Tamanho do grupo":
  - Capacidade desejada: ```2```
  - Capacidade mínima: ```2```
  - Capacidade máxima: ```4```
- Em "Políticas de escalabilidade", marque a opção Política de dimensionamento com monitoramento do objetivo" e mude o "Valor de destino para "70"
- "Próximo"
- Finalize em "Criar grupo do Auto Scaling"





  
## Integrantes
- [Pedro Liu](https://github.com/PedroTxfl/Projeto_docker_wordpress) 
- [Bruno Marques](https://github.com/BrunoMarques1/Atividade_DOCKER/tree/main)
- [José Toniolo](https://github.com/vitortoniolo/PB-AtividadeDocker)