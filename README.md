# Título do Projeto

Uma breve descrição sobre o que esse projeto faz e para quem ele é

## Serviço VPC
### Criando VPC
Acesse o Console da Amazon Web Services e entre na área de serviços VPC.
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
| SN-Public01  | us-east-1b   | 172.28.2.0/24  |

- Finalize a criação clicando em "Criar sub-rede"
- No painel de todas as sub-redes, selecione as duas públicas(SN-Public), uma de cada vez, e vá em "ações" > "Editar configurações de sub-rede" e marque a opção "Habilitar endereço IPv4 público de atribuição automática" e finalize.

### Criando Gateway de internet
- No menu lateral esquerdo, clique em "Gateways da Internet" e depois em "Criar gateway da internet".
- Em "Tag de nome", defina um nome, neste caso usarei "IG-pedro"
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

## Configurando Grupos de Segurança

Agora vamos configurar quatro grupos de segurança para diferentes componentes da 
infraestrutura: Bastion Host, Balanceador de Carga, Aplicação e EFS.

- Clique em "Criar grupo de seguarança e configure:

### Grupo de Segurança do Bastion Host
- Defina o nome do grupo, neste caso usarei "SG-BastioHost".
- Na descrição insira "Grupo de segurança do bastion host para entrada via ssh apenas do meu IP"
- Selecione a VPC "VPC-Projeto01".
- Adicione a seguinte regra de entrada:

| Tipo | Intervalo de Portas | Protocolo | Origem    |
|-------------------|--------------------|-----------|-----------|
| TCP Personalizado | 22 | TCP       | "MEU-IP"  |

- Certifique-se de substituir "MEU-IP" pelo endereço IP correto para permitir o acesso ao Bastion Host. 

### Grupo de Segurança do Balanceador de Carga
- Defina o nome do grupo, neste caso usarei "SG-LoadBalancer".
- Na descrição insira "Grupo de seguranca do Load Balancer"
- Selecione a VPC "VPC-Projeto01".
- Adicione as seguintes regras de entrada:

| Tipo | Intervalo de Portas | Protocolo | Origem    |
|-------------------|--------------------|-----------|-----------|
| HTTP | 80 | TCP       | 0.0.0.0/0  |
| HTTPS | 443 | TCP       | 0.0.0.0/0  |

### Grupo de Segurança da Aplicação
- Defina o nome do grupo, neste caso usarei "SG-App".
- Na descrição insira "Grupo de seguranca da aplicacao"
- Selecione a VPC "VPC-Projeto01".
- Adicione as seguintes regras de entrada:

| Tipo | Intervalo de Portas | Protocolo | Origem                        |
|-------|-------|-----------|-------------------------------|
| SSH | 22    | TCP       | Grupo de Segurança do Bastion Host |
| HTTP | 80    | TCP       | IP privado do Load-Balancer |
| HTTPS | 443    | TCP       | IP privado do Load-Balancer |


### Grupo de seguraça do EFS
- Defina o nome do grupo, neste caso usarei "SG-EFS".
- Na descrição insira "Grupo de seguranca do EFS"
- Selecione a VPC "VPC-Projeto01".
- Adicione as seguintes regras de entrada:

| Tipo | Intervalo de Portas | Protocolo | Origem                        |
|-------|-------|-----------|-------------------------------|
| NFS | 2049    | TCP       | Grupo de segurança da Aplicação |


## Par de chaves
Antes de executar uma instância, devemos criar um par de chaves. 
- Na lateral esquerda, acesse "Pares de chaves".
- Clique em "Criar par de chaves".
- Dê um nome para o par de chaves, usarei "keySSHPedro".
- Tipo de par de chaves: ```RSA```
- Formato de arquivo de chave privada: ```.PEM```
- A chave será baixada em formato ```.PEM```, você deve salvá-la em um diretório/pasta segura.

#### APP Docker - Wordpress
regra de entrada ssh apenas para o ip privado via bastion host ?



## Criar instância Bastion Host

## Criar instância APP Docker -wordpress ??

## criar e configurar efs ??

## sg ELB