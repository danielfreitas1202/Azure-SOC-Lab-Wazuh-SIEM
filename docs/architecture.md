# Arquitetura do Laboratório SOC

## 1. Objetivo da arquitetura

O objetivo deste laboratório é implementar uma infraestrutura de monitoramento de segurança baseada no Wazuh, centralizando os eventos gerados por endpoints Windows e Linux em uma plataforma SIEM (*Security Information and Event Management*).

A infraestrutura foi implantada no Microsoft Azure e organizada em uma rede virtual privada, permitindo a comunicação entre os endpoints monitorados e o servidor responsável pela coleta, análise e visualização dos eventos.

O projeto tem como finalidade simular parte da rotina de um analista SOC (*Security Operations Center*), explorando a coleta de logs, o monitoramento de integridade de arquivos e a investigação de eventos de segurança.

## 2. Infraestrutura utilizada

A infraestrutura foi organizada em um grupo de recursos dedicado ao laboratório, com uma rede virtual e três máquinas virtuais.

### 2.1 Recursos do Azure

| Recurso | Nome ou configuração | Finalidade |
|---|---|---|
| Grupo de recursos | `SOC-Lab-RG` | Centralizar e organizar os recursos do laboratório. |
| Rede virtual (VNet) | `SOC-VNet` | Fornecer conectividade privada entre as máquinas virtuais. |
| Sub-rede | `SOC-Subnet` — `10.10.1.0/24` | Agrupar as máquinas virtuais na rede do laboratório. |
| Grupo de segurança de rede (NSG) | `SOC-NSG` | Controlar o tráfego de entrada e saída por meio de regras de segurança. |

### 2.2 Máquinas virtuais

| Máquina virtual | Sistema operacional | IP privado | Função |
|---|---|---|---|
| `SOC-WAZUH` | Ubuntu Server | `10.10.1.4` | Executar o Wazuh Manager, Indexer e Dashboard. |
| `SOC-WIN01` | Windows Server | `10.10.1.5` | Atuar como endpoint monitorado, com coleta de eventos do Windows, Sysmon, PowerShell e FIM. |
| `SOC-LINUX01` | Ubuntu Server | `10.10.1.6` | Atuar como endpoint monitorado, com coleta de logs do sistema, autenticação SSH e FIM. |

O servidor `SOC-WAZUH` concentra os componentes da plataforma, enquanto `SOC-WIN01` e `SOC-LINUX01` representam os sistemas monitorados. Essa separação permite analisar eventos de diferentes sistemas operacionais em um ponto central.

## 3. Diagrama da arquitetura

O diagrama abaixo representa a organização lógica dos recursos e o fluxo de comunicação entre os endpoints e a plataforma de monitoramento.

<img width="1060" height="1024" alt="Diagrama da arquitetura do laboratório SOC" src="https://github.com/user-attachments/assets/e7762ed2-5010-4619-83a3-fff9bcd10b92" />

## 4. Fluxo de coleta e análise de eventos

O funcionamento da infraestrutura pode ser dividido nas seguintes etapas:

1. **Geração de eventos:** os endpoints Windows e Linux geram registros relacionados às suas atividades, como autenticações, execução de processos e alterações em arquivos monitorados.
2. **Coleta pelos agentes:** os agentes Wazuh instalados nos endpoints coletam os registros configurados e encaminham os dados ao servidor.
3. **Processamento no Manager:** o Wazuh Manager recebe os eventos e aplica os mecanismos de análise e as regras de detecção disponíveis.
4. **Indexação:** os dados processados são encaminhados para o Wazuh Indexer, responsável por armazenar e indexar os registros para consulta.
5. **Visualização:** o Wazuh Dashboard permite pesquisar eventos, acompanhar alertas e analisar as informações coletadas.

Esse fluxo centraliza os registros dos endpoints e facilita a investigação de atividades que, se analisadas separadamente em cada máquina, poderiam exigir mais tempo e esforço.

## 5. Segurança da infraestrutura

A arquitetura utiliza endereços IP privados para a comunicação interna entre as máquinas virtuais. O NSG controla o tráfego de rede de acordo com as portas e origens necessárias ao funcionamento do laboratório.

As principais medidas de segurança consideradas são:

- **Segmentação lógica:** utilização de uma rede virtual dedicada ao laboratório.
- **Controle de tráfego:** restrição das portas de comunicação aos serviços necessários.
- **Comunicação dos agentes:** utilização do endereço privado do servidor Wazuh (`10.10.1.4`) para o envio dos eventos.
- **Acesso administrativo:** utilização de mecanismos de acesso remoto apropriados para administrar as máquinas virtuais.
- **Proteção do dashboard:** restrição do acesso externo às origens autorizadas, evitando a exposição desnecessária da interface de gerenciamento.

A presença de um IP público no servidor, caso utilizado para acessar o dashboard, não deve significar acesso irrestrito. As regras de entrada precisam limitar o acesso às origens autorizadas, e as portas de comunicação dos agentes não devem ser expostas diretamente à Internet.

## 6. Limitações e considerações

O laboratório depende da disponibilidade dos recursos do Microsoft Azure, da conectividade entre as máquinas virtuais e dos créditos disponíveis na assinatura utilizada.

A assinatura Azure for Students possui um orçamento limitado, o que influencia o dimensionamento das máquinas, o tempo de execução e a possibilidade de manter todos os recursos ativos continuamente.

Além disso, algumas atividades de configuração e atualização dos endpoints são limitadas pela ausência de acesso direto à Internet nas máquinas monitoradas. Quando necessário, os instaladores foram obtidos em outro ambiente e transferidos manualmente para os endpoints.

Essas limitações fazem parte do contexto do projeto e demonstram a importância de considerar custos, conectividade, manutenção e segurança durante o planejamento de uma infraestrutura de monitoramento.

## 7. Possíveis evoluções

Como próximas etapas, o laboratório pode ser ampliado com:

- Validação e configuração do monitoramento de vulnerabilidades.
- Criação e teste de regras personalizadas de detecção.
- Ampliação das fontes de telemetria coletadas.
