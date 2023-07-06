# Título do Projeto

Uma breve descrição sobre o que esse projeto faz e para quem ele é

## Criar VPC
Acesse o Console da Amazon Web Services e entre na área de serviços VPC.
- Clique em "Criar VPC"
- Agora vamos configurar a VPC.
  - selecione "Somente VPC".
  - Dê um nome a esta VPC, pode ser "VPC-Projeto-Docker".
  - Em "Bloco CIDR IPv4", selecione a opção "Entrada manual de CIDR IPv4"
  - Logo baixo, adicione o CIDR IPv4 desejado. Aqui usaremos "172.28.0.0/16".
  - Em "Bloco CIDR IPv6", mantenha selecionada a opção "Nenhum bloco IPv6".
  - Locação "Padrão".
  - Finalize e crie a VPC.
### Criar Sub-Redes
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

### Criar Gateway de internet
- No menu lateral esquerdo, clique em "Gateways da Internet" e depois em "Criar gateway da internet".
- Em "Tag de nome", defina um nome, neste caso usarei "IG-pedro"
- Finalize
- No painel dos gateways de internet, selecione o que acabamos de criar e vá em "Ações" > Associar à VPC > associe à VPC que criamos.
### Criar Nat Gateway
- No menu lateral esquerdo, clique em "gateways NAT" e depois em "Criar gateway NAT".
- Defina um nome. Aqui usarei "NatG-pedro"
- Selecione a sub-rede pública "SN-Public01".
- Em "Tipo de conectividade", mantenha selecionada a opção "Público"
- Clique em "Alocar IP elástico"
- Finalize
### Criar Tabelas de Rotas
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

## EC2
### Criar Grupos de segurança
#### Bastion
#### APP Docker - Wordpress
regra de entrada ssh apenas para o ip privado via bastion host ?



## Criar instância Bastion Host

## Criar instância APP Docker -wordpress ??

## criar e configurar efs ??

## sg ELB