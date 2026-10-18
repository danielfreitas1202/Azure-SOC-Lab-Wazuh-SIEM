# Implantação do Laboratório SOC

## 1. Visão geral

Este documento descreve as principais etapas utilizadas para implantar o laboratório SOC no Microsoft Azure, desde a criação dos recursos de infraestrutura até a instalação e validação dos agentes Wazuh.

A implantação foi dividida em etapas para manter os recursos organizados, estabelecer a conectividade privada entre as máquinas virtuais e centralizar a coleta de eventos em um único servidor.

### Tecnologias utilizadas

- Microsoft Azure
- Ubuntu Server
- Windows Server
- Wazuh 4.14.7
- Wazuh Manager, Indexer e Dashboard
- Wazuh Agent para Windows e Linux
- Virtual Network (VNet), sub-rede e Network Security Group (NSG)

## 2. Pré-requisitos

Para reproduzir a implantação, são necessários:

- Uma assinatura do Microsoft Azure com permissões para criar grupos de recursos, redes virtuais e máquinas virtuais.
- Conhecimentos básicos de administração Linux e Windows.
- Acesso administrativo às máquinas virtuais.
- Conectividade de rede entre os endpoints e o servidor Wazuh.
- Pacotes de instalação do Wazuh compatíveis com os sistemas operacionais utilizados.

Neste laboratório, a assinatura Azure for Students foi utilizada para limitar os custos. Como os endpoints não possuíam acesso direto à Internet, os pacotes dos agentes foram obtidos em um ambiente com conectividade e transferidos manualmente.

## 3. Criação do grupo de recursos

Foi criado o grupo de recursos `SOC-Lab-RG`, destinado a centralizar os recursos utilizados pelo laboratório.

A utilização de um grupo de recursos dedicado facilita a administração da infraestrutura, a identificação dos recursos relacionados ao projeto e o gerenciamento de sua utilização.

## 4. Configuração da rede virtual e da sub-rede

Dentro do grupo `SOC-Lab-RG`, foi criada a rede virtual `SOC-VNet`, com a sub-rede `SOC-Subnet`, utilizando o intervalo `10.10.1.0/24`.

A rede virtual fornece a conectividade privada necessária para que os agentes instalados nos endpoints possam se comunicar com o servidor Wazuh.

Os endereços privados foram distribuídos da seguinte maneira:

| Máquina virtual | IP privado |
|---|---|
| `SOC-WAZUH` | `10.10.1.4` |
| `SOC-WIN01` | `10.10.1.5` |
| `SOC-LINUX01` | `10.10.1.6` |

A utilização de uma rede dedicada mantém os componentes do laboratório organizados e permite controlar o tráfego entre eles por meio das regras de segurança de rede.

## 5. Configuração do Network Security Group

Foi criado o grupo de segurança de rede `SOC-NSG` para controlar o tráfego de entrada e saída das máquinas virtuais.

As regras foram configuradas de acordo com as necessidades de comunicação da infraestrutura. Entre os serviços relevantes para o laboratório estão:

| Porta | Protocolo | Finalidade |
|---|---|---|
| `1514` | TCP | Comunicação dos agentes Wazuh com o Manager. |
| `1515` | TCP | Registro e autenticação de novos agentes, conforme a configuração utilizada. |
| `443` | TCP | Acesso HTTPS ao Wazuh Dashboard. |
| `22` | TCP | Administração remota por SSH, quando necessária. |
| `3389` | TCP | Administração remota do Windows por RDP, quando necessária. |

As portas administrativas e de acesso ao dashboard devem ser restritas às origens autorizadas. As portas `1514` e `1515` devem permitir somente a comunicação necessária entre os componentes do laboratório, sem exposição direta à Internet.

A necessidade de cada regra deve ser revisada de acordo com o método de acesso e a configuração efetivamente utilizados. Não é necessário manter abertas portas que não estejam sendo utilizadas.

## 6. Criação da máquina virtual SOC-WAZUH

A primeira máquina virtual criada foi a `SOC-WAZUH`, responsável por hospedar os componentes centrais da plataforma de monitoramento.

### Configuração

- **Sistema operacional:** Ubuntu Server 24.04 LTS.
- **Tamanho:** `Standard B2as_v2`.
- **Processadores virtuais:** 2 vCPUs.
- **Memória RAM:** 8 GiB.
- **Endereço IP privado:** `10.10.1.4`.
- **Função:** Wazuh Manager, Wazuh Indexer e Wazuh Dashboard.

A máquina foi conectada à `SOC-Subnet` da `SOC-VNet`. Um endereço IP público foi utilizado para permitir o acesso externo ao servidor durante a configuração do laboratório; esse acesso deve permanecer restrito às origens autorizadas.

O dimensionamento foi escolhido considerando os recursos necessários para executar os componentes da plataforma e as limitações de crédito da assinatura.

## 7. Instalação do Wazuh

Após acessar o servidor Ubuntu, foi preparado o ambiente e executado o instalador oficial do Wazuh.

Primeiro, os índices de pacotes do sistema foram atualizados:

```bash
sudo apt update
```

Em seguida, o instalador da versão 4.14 foi baixado:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
```

A instalação completa dos componentes foi iniciada com:

```bash
sudo bash ./wazuh-install.sh -a
```

A opção `-a` foi utilizada para instalar os componentes da plataforma em um único servidor, adequado ao escopo deste laboratório.

Após a instalação, os serviços foram verificados individualmente:

```bash
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-dashboard --no-pager
```

Essa verificação permite confirmar se os serviços estão ativos e identificar possíveis problemas durante a implantação.

Com os componentes instalados, o dashboard foi utilizado para acompanhar os eventos recebidos dos endpoints e administrar o ambiente de monitoramento.

## 8. Criação dos endpoints Windows e Linux

Após preparar o servidor Wazuh, foram criadas duas máquinas virtuais para representar os sistemas monitorados.

### 8.1 SOC-WIN01

A máquina `SOC-WIN01` foi criada com Windows Server e conectada à `SOC-Subnet` da `SOC-VNet`.

- **Sistema operacional:** Windows Server.
- **Endereço IP privado:** `10.10.1.5`.
- **Função:** endpoint monitorado pelo Wazuh.
- **Fontes de telemetria:** eventos nativos do Windows, Sysmon, PowerShell Script Block Logging e monitoramento de integridade de arquivos (FIM).

A máquina utiliza um endereço IP privado para a comunicação com o servidor Wazuh. O acesso administrativo é realizado por um método autorizado de acesso remoto, sem necessidade de expor o endpoint diretamente à Internet.

### 8.2 SOC-LINUX01

A máquina `SOC-LINUX01` foi criada com Ubuntu Server e conectada à mesma sub-rede.

- **Sistema operacional:** Ubuntu Server.
- **Endereço IP privado:** `10.10.1.6`.
- **Função:** endpoint Linux monitorado pelo Wazuh.
- **Fontes de telemetria:** logs de autenticação, registros do sistema e monitoramento de integridade de arquivos (FIM).

A máquina foi preparada para enviar seus registros ao servidor central por meio do agente Wazuh.

O dimensionamento dos endpoints deve ser consultado diretamente nas configurações das respectivas máquinas virtuais no Azure, pois pode ser diferente do dimensionamento utilizado no servidor central.

## 9. Instalação e registro dos agentes Wazuh

Os agentes permitem que os endpoints encaminhem os registros configurados ao Wazuh Manager. Como as máquinas Windows e Linux não possuíam acesso direto à Internet, os instaladores foram transferidos manualmente para os respectivos sistemas.

Os pacotes utilizados correspondiam à versão `4.14.7`.

### 9.1 Agente Windows

O instalador `wazuh-agent-4.14.7-1.msi` foi transferido para a máquina `SOC-WIN01`.

No PowerShell, a instalação foi realizada com o endereço privado do Manager e o nome do agente:

```powershell
msiexec.exe /i "C:\Users\socadmin\wazuh-agent-4.14.7-1.msi" /q WAZUH_MANAGER="10.10.1.4" WAZUH_AGENT_NAME="SOC-WIN01"
```

Em seguida, o serviço foi iniciado:

```powershell
Start-Service WazuhSvc
```

O estado do serviço foi verificado com:

```powershell
Get-Service WazuhSvc
```

A inicialização do serviço confirma que o agente está em execução localmente. A confirmação de que ele está conectado ao Manager deve ser feita também no dashboard do Wazuh.

### 9.2 Agente Linux

O pacote `wazuh-agent_4.14.7-1_amd64.deb` foi transferido para a máquina `SOC-LINUX01`.

A instalação foi realizada com as variáveis de ambiente que definem o servidor e o nome do agente:

```bash
sudo env WAZUH_MANAGER='10.10.1.4' WAZUH_AGENT_NAME='SOC-LINUX01' dpkg -i /home/socadmin/wazuh-agent_4.14.7-1_amd64.deb
```

Após a instalação, o serviço do agente foi iniciado:

```bash
sudo systemctl start wazuh-agent
```

O estado do serviço pode ser consultado com:

```bash
sudo systemctl status wazuh-agent --no-pager
```

**Observação:** no Linux, o nome do serviço é `wazuh-agent`. No Windows, o serviço utilizado neste laboratório é `WazuhSvc`.

## 10. Validação da comunicação

Após iniciar os serviços dos agentes, o dashboard do Wazuh foi acessado para verificar o estado dos endpoints registrados.

As máquinas `SOC-WIN01` e `SOC-LINUX01` foram identificadas como agentes ativos, confirmando que a comunicação com o servidor central estava funcionando.

A validação da conexão foi seguida por testes de coleta de eventos e monitoramento, incluindo registros do Windows, eventos do Sysmon, PowerShell Script Block Logging, logs de autenticação SSH e FIM nos dois sistemas operacionais.

A presença de um agente ativo confirma a comunicação com a plataforma, mas não garante, isoladamente, que todas as fontes de telemetria estejam configuradas corretamente. Por esse motivo, cada funcionalidade foi validada por meio de testes específicos e da consulta aos registros no dashboard.

<img width="1539" height="617" alt="Agentes Wazuh registrados no dashboard" src="https://github.com/user-attachments/assets/877273a6-e780-418d-8f1e-47f201845cc5" />

## 11. Segurança e controle de custos

A implantação considerou as limitações da assinatura Azure for Students e a necessidade de reduzir a exposição da infraestrutura.

As principais medidas adotadas foram:

- Utilização de uma rede virtual e endereços IP privados para os endpoints.
- Configuração de regras de segurança de rede para controlar a comunicação entre as máquinas.
- Restrição do acesso administrativo e do dashboard às origens autorizadas.
- Utilização do endereço privado do Manager para configurar os agentes.
- Escolha do dimensionamento das máquinas considerando os recursos disponíveis e o orçamento do laboratório.
- Utilização do recurso de desligamento automático e interrupção das máquinas quando não estão sendo utilizadas, conforme a configuração aplicada no Azure.

O desligamento das máquinas quando o laboratório não está em uso ajuda a reduzir o consumo de créditos. Entretanto, discos gerenciados, endereços IP e outros recursos podem continuar gerando custos mesmo quando as máquinas estão paradas. Por isso, é importante revisar os recursos e os custos na própria plataforma Azure.

## 12. Considerações finais

A implantação estabeleceu uma infraestrutura funcional para centralizar eventos de endpoints Windows e Linux no Wazuh.

A separação entre servidor central e máquinas monitoradas permitiu testar diferentes fontes de telemetria e compreender o fluxo de coleta, processamento e consulta de eventos.

As etapas descritas neste documento servem como referência para reproduzir a estrutura básica do laboratório. Configurações adicionais de monitoramento, regras personalizadas e detecção de vulnerabilidades devem ser documentadas separadamente, após sua implementação e validação.
estavam conectados oa Wazuh eu segui para os teste. 
