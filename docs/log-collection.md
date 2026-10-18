# Coleta e Centralização de Logs

## 1. Visão geral

A coleta e a centralização de logs são etapas fundamentais para o monitoramento de segurança em um ambiente de Security Operations Center (SOC). Neste laboratório, o Wazuh foi utilizado para centralizar eventos provenientes de endpoints Windows e Linux, permitindo consultar registros do sistema, atividades de autenticação e eventos relevantes para investigações de segurança.

A infraestrutura utiliza agentes Wazuh instalados nas máquinas monitoradas, responsáveis por coletar os eventos locais e encaminhá-los ao servidor central. O processamento desses dados permite que os eventos sejam consultados no Wazuh Dashboard e, quando correspondem às condições de regras de detecção, sejam apresentados como alertas.

## 2. Coleta de logs no Windows

O endpoint Windows, identificado como `SOC-WIN01`, foi configurado para fornecer diferentes fontes de telemetria ao Wazuh. Além dos registros nativos do sistema operacional, foram habilitados canais adicionais para ampliar a visibilidade sobre a execução de processos e atividades realizadas por meio do PowerShell.

### 2.1. Logs nativos do Windows

Foram utilizados os seguintes canais de eventos do Windows:

- **Security:** registra eventos relacionados à segurança e auditoria, incluindo tentativas de autenticação, atividades de contas e alterações em grupos de usuários, conforme as políticas de auditoria habilitadas.
- **System:** contém eventos relacionados ao funcionamento do sistema operacional, serviços, drivers e outros componentes.
- **Application:** reúne eventos produzidos por aplicativos e componentes de software.

Esses registros permitem investigar atividades como falhas de autenticação, alterações em contas de usuários e eventos operacionais do sistema.

A coleta foi validada por meio do Wazuh Dashboard, onde foram observados eventos de auditoria, falhas de login, criação e alteração de contas e outros registros produzidos pelo Windows.

### 2.2. PowerShell Script Block Logging

Para ampliar a visibilidade sobre atividades realizadas por meio do PowerShell, foi habilitado o recurso **Script Block Logging** no endpoint Windows.

Esse recurso registra o conteúdo dos blocos de script processados pelo PowerShell no canal:

`Microsoft-Windows-PowerShell/Operational`

A configuração foi realizada por meio da política de registro do Windows, habilitando a propriedade `EnableScriptBlockLogging`.

O agente Wazuh foi configurado para coletar esse canal utilizando o formato `eventchannel`, conforme o trecho abaixo:

```xml
<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

A implementação foi validada por meio da execução de comandos de teste e da consulta aos eventos de Script Block Logging, incluindo o evento de ID `4104`. Os registros foram identificados tanto no log local do Windows quanto no Wazuh Dashboard.

Essa telemetria pode auxiliar na investigação de comandos executados, scripts utilizados e atividades potencialmente suspeitas realizadas por meio do PowerShell. A interpretação dos eventos depende do contexto, das regras de detecção e das informações disponíveis.

### 2.3. Coleta de eventos do Sysmon

O **System Monitor (Sysmon)** foi instalado no endpoint `SOC-WIN01` para fornecer informações adicionais sobre atividades do sistema operacional, especialmente a criação de processos.

O Sysmon registra seus eventos no canal:

`Microsoft-Windows-Sysmon/Operational`

Para permitir a coleta centralizada, foi adicionada a seguinte configuração ao arquivo `ossec.conf` do agente:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

A validação foi realizada por meio da consulta aos eventos locais do Sysmon e da aplicação de filtros específicos no Wazuh Dashboard. Foram identificados eventos produzidos pelo Sysmon, incluindo registros relacionados à criação de processos.

O evento de ID `1` corresponde à criação de um processo e pode fornecer informações como o caminho do executável, a linha de comando e o processo pai, de acordo com os dados registrados.

É importante destacar que a presença de um evento Sysmon não implica necessariamente a geração de um alerta de segurança. A coleta do evento e sua classificação pelas regras do Wazuh são etapas distintas.

<img width="1542" height="812" alt="image" src="https://github.com/user-attachments/assets/afc6b05d-9ad9-4f20-91a5-baefe4df171d" />

## 3. Coleta de logs no Linux

O endpoint Linux, identificado como `SOC-LINUX01`, utiliza Ubuntu Server e possui o agente Wazuh instalado e registrado no servidor central.

A coleta foi configurada para monitorar arquivos de log locais relacionados à autenticação e ao funcionamento geral do sistema operacional.

### 3.1. Logs de autenticação — `/var/log/auth.log`

O arquivo `/var/log/auth.log` foi configurado como fonte de eventos relacionados à autenticação e ao controle de acesso.

Entre os registros relevantes para o monitoramento de segurança estão:

- Tentativas de autenticação SSH bem-sucedidas ou malsucedidas;
- Tentativas de acesso com credenciais inválidas;
- Eventos relacionados ao uso de `sudo`;
- Outras atividades de autenticação registradas pelos serviços do sistema.

A coleta desse arquivo permite acompanhar tentativas de acesso ao endpoint e investigar possíveis atividades suspeitas relacionadas à autenticação.

A validação foi realizada por meio de testes de SSH e comandos que geram registros de autenticação. Os eventos correspondentes foram consultados no Wazuh Dashboard, confirmando a comunicação entre o agente Linux e o servidor central.

A disponibilidade e o conteúdo desses registros dependem dos serviços de autenticação e da configuração de logging da distribuição utilizada.

<img width="1543" height="810" alt="image" src="https://github.com/user-attachments/assets/c4d30588-9ace-44b9-a078-248aefd3da7d" />

### 3.2. Logs gerais do sistema — `/var/log/syslog`

O arquivo `/var/log/syslog` foi adicionado à configuração do agente para coletar mensagens gerais produzidas pelo sistema operacional e por serviços que registram seus eventos nesse arquivo.

Esses registros complementam os logs de autenticação, permitindo consultar informações operacionais que podem contribuir para a análise de incidentes e a identificação de comportamentos inesperados.

A configuração do agente utiliza o formato `syslog`, conforme o exemplo:

```xml
<localfile>
  <location>/var/log/auth.log</location>
  <log_format>syslog</log_format>
</localfile>

<localfile>
  <location>/var/log/syslog</location>
  <log_format>syslog</log_format>
</localfile>
```

Após a configuração, o serviço `wazuh-agent` foi reiniciado para aplicar as alterações. A coleta foi validada por meio da consulta aos registros locais e da observação dos eventos correspondentes no Dashboard.

A existência e o conteúdo desses arquivos podem variar conforme a distribuição Linux e o sistema de logging instalado. Neste laboratório, a configuração foi aplicada ao ambiente Ubuntu utilizado no endpoint.

<img width="1541" height="812" alt="image" src="https://github.com/user-attachments/assets/44d90b6a-362a-497f-9f9f-7ef216fcbeef" />

## 4. Fluxo de processamento dos eventos

O fluxo de coleta e centralização de logs do laboratório pode ser dividido nas seguintes etapas:

1. **Geração do evento:** uma atividade ocorre no endpoint, como uma tentativa de autenticação, a execução de um processo ou uma alteração em arquivo monitorado.
2. **Registro local:** o sistema operacional ou a ferramenta de monitoramento registra o evento em um canal de eventos ou arquivo de log.
3. **Coleta pelo agente:** o agente Wazuh monitora as fontes configuradas e encaminha os dados ao servidor central.
4. **Processamento pelo manager:** o Wazuh Manager recebe os eventos e aplica os mecanismos de análise, decodificação e avaliação de regras.
5. **Classificação:** quando um evento corresponde às condições de uma regra, o Wazuh pode gerar um alerta. Eventos coletados também podem não resultar em alertas.
6. **Consulta no Dashboard:** os dados disponíveis podem ser pesquisados e analisados por meio do Wazuh Dashboard, de acordo com o processamento realizado e as configurações de armazenamento e indexação.

Esse fluxo permite centralizar diferentes fontes de telemetria e consultar eventos de múltiplos sistemas em uma única plataforma, facilitando a investigação de atividades potencialmente suspeitas.

## 5. Resultados e validação

A implementação foi validada por meio de testes práticos nos dois endpoints.

No Windows, foram observados eventos dos canais nativos do sistema, registros do PowerShell Script Block Logging e eventos do Sysmon. Também foram realizados testes de File Integrity Monitoring, documentados separadamente na seção de monitoramento de segurança.

No Linux, foram observados registros de autenticação e eventos gerais do sistema, além de eventos de SSH e alterações em arquivos monitorados pelo FIM.

Os testes demonstraram que os agentes estavam encaminhando eventos ao servidor Wazuh e que as informações relevantes podiam ser consultadas no Dashboard.

A implementação representa uma base para atividades posteriores de investigação, desenvolvimento de regras personalizadas e análise de vulnerabilidades. A presença de eventos no Dashboard, isoladamente, não significa que todas as atividades potencialmente maliciosas serão detectadas; a capacidade de detecção depende das fontes de telemetria, das regras habilitadas e da validação de cada cenário.
