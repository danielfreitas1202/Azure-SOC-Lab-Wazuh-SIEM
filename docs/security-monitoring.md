# Monitoramento de Segurança

## 1. Windows

O monitoramento do endpoint Windows foi configurado para coletar diferentes tipos de telemetria, permitindo acompanhar a execução de processos, a utilização do PowerShell e as alterações realizadas em arquivos monitorados. Os eventos são coletados pelo agente Wazuh e enviados ao servidor central para análise e visualização no dashboard.

### 1.1 Sysmon

**Objetivo**

O Sysmon (System Monitor), ferramenta da Microsoft Sysinternals, foi instalado no endpoint Windows para fornecer informações detalhadas sobre atividades do sistema operacional. Essa telemetria complementa os logs nativos do Windows, especialmente durante investigações que envolvem execução de processos e possíveis atividades suspeitas.

A criação de processos é particularmente útil para identificar quais programas foram executados, analisar suas relações com outros processos e investigar a execução de ferramentas potencialmente maliciosas.

**Configuração**

O Sysmon foi instalado no endpoint `SOC-WIN01`. Em seguida, o canal de eventos `Microsoft-Windows-Sysmon/Operational` foi adicionado à configuração do agente Wazuh para permitir a coleta dos registros gerados pela ferramenta.

Trecho adicionado à configuração do agente:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

**Testes realizados**

Foi realizada a verificação dos eventos gerados pelo Sysmon no próprio Windows, utilizando o PowerShell para consultar o canal de eventos e filtrar registros pelo ID 1, correspondente à criação de processos (*Process Create*).

Também foi verificada a chegada desses eventos ao Wazuh, utilizando filtros relacionados ao canal do Sysmon no dashboard.

**Resultado observado**

Os eventos de criação de processos foram encontrados no Windows e visualizados no Wazuh, confirmando que a coleta dessa telemetria está funcionando.

É importante diferenciar a coleta de eventos da geração de alertas: um evento pode ser armazenado e pesquisável no Wazuh sem necessariamente produzir um alerta de segurança. A classificação depende das regras aplicadas ao registro.

**Utilidade em uma investigação**

Os eventos do Sysmon podem ajudar a reconstruir a execução de programas, identificar processos envolvidos em uma atividade suspeita e estabelecer uma sequência temporal dos acontecimentos. Dependendo dos campos registrados, também podem fornecer informações sobre o processo responsável pela execução e o programa iniciado.

**Evidências**

<img width="1542" height="812" alt="image" src="https://github.com/user-attachments/assets/494759a7-56e5-44d8-aaf3-2d69f945218c" />


### 1.2 PowerShell Script Block Logging

**Objetivo**

O PowerShell é uma ferramenta administrativa legítima, mas também pode ser utilizado por invasores para executar comandos, automatizar tarefas maliciosas ou realizar atividades sem depender de arquivos executáveis tradicionais.

O *Script Block Logging* permite registrar o conteúdo de blocos de script processados pelo PowerShell. Esses registros podem ajudar a compreender quais comandos foram executados durante uma atividade investigada.

**Configuração**

O registro de blocos de script foi habilitado por meio do Registro do Windows, utilizando a seguinte configuração:

```powershell
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force

New-ItemProperty `
  -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
  -Name "EnableScriptBlockLogging" `
  -PropertyType DWORD `
  -Value 1 `
  -Force
```

O canal `Microsoft-Windows-PowerShell/Operational` também foi adicionado à configuração do agente Wazuh:

```xml
<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

**Testes realizados**

Foi verificada a habilitação da configuração no Registro do Windows. Em seguida, foram executados comandos de teste no PowerShell e consultados os eventos gerados no canal operacional.

Os registros com o ID 4104 foram identificados localmente e no dashboard do Wazuh.

**Resultado observado**

A configuração de registro estava habilitada, e os eventos de Script Block Logging foram encontrados no Windows e recebidos pelo Wazuh.

**Utilidade em uma investigação**

Esses registros podem ajudar a identificar comandos utilizados em atividades suspeitas, entender o comportamento de scripts e reconstruir etapas de uma execução. São especialmente relevantes em investigações que envolvem comandos administrativos, automação e possíveis abusos do PowerShell.

Entretanto, o registro não garante que toda atividade maliciosa será detectada. A interpretação depende do conteúdo registrado, das configurações de auditoria e das regras de análise utilizadas.

**Evidências**

<img width="1541" height="814" alt="image" src="https://github.com/user-attachments/assets/9721f963-446f-499d-ba28-f25ab69a9d39" />

### 1.3 File Integrity Monitoring (FIM)

**Objetivo**

O *File Integrity Monitoring* (FIM) foi configurado para identificar alterações em arquivos de um diretório específico do endpoint Windows. Esse monitoramento permite observar inclusões, modificações e exclusões de arquivos, contribuindo para a investigação de alterações inesperadas no sistema.

**Configuração**

Foi criado o diretório de testes `C:\SOC-Lab\FIM-Test` e adicionado o caminho à configuração de monitoramento de integridade do agente Wazuh, utilizando a opção `realtime="yes"`:

```xml
<directories realtime="yes">C:\SOC-Lab\FIM-Test</directories>
```

O trecho foi inserido no bloco `<syscheck>` já existente na configuração do agente.

**Testes realizados**

Foram realizados testes controlados no diretório monitorado, incluindo:

- Criação de um arquivo.
- Modificação do conteúdo de um arquivo existente.
- Exclusão de um arquivo.

**Resultado observado**

O Wazuh registrou eventos correspondentes à inclusão, à alteração de integridade e à exclusão de arquivos. Esses resultados confirmaram o funcionamento do FIM no diretório configurado.

**Utilidade em uma investigação**

O FIM pode ajudar a identificar alterações inesperadas em arquivos de configuração, scripts ou outros arquivos relevantes. Em um ambiente real, essas informações podem contribuir para investigar modificações não autorizadas, persistência de ameaças ou mudanças ocorridas durante um incidente.

O monitoramento está limitado aos diretórios configurados; portanto, alterações fora desses caminhos não são necessariamente detectadas por essa configuração.

**Evidências**

<img width="1544" height="625" alt="image" src="https://github.com/user-attachments/assets/8fe43205-b50b-4944-8a14-16759e43ea2a" />
<img width="771" height="769" alt="image" src="https://github.com/user-attachments/assets/a7f07045-250e-42ed-8feb-508bba3b3733" /> <img width="769" height="767" alt="image" src="https://github.com/user-attachments/assets/34f7b172-0279-45b6-a468-5665658d2b63" /> <img width="771" height="764" alt="image" src="https://github.com/user-attachments/assets/197c98c0-5d32-44d5-955b-b252ad9df7cf" />

## 2. Linux

No endpoint Linux, o monitoramento foi direcionado à integridade de arquivos, às tentativas de autenticação e aos registros gerais do sistema. Essa combinação permite observar mudanças em arquivos e investigar atividades relacionadas ao acesso à máquina e ao funcionamento do sistema operacional.

### 2.1 File Integrity Monitoring (FIM)

**Objetivo**

O FIM foi configurado no endpoint Linux para identificar alterações em arquivos de um diretório de testes. Assim como no Windows, a funcionalidade permite acompanhar mudanças que podem ser relevantes para a segurança do sistema.

**Configuração**

Foi utilizado o diretório `/opt/SOC-Lab/FIM-Test`, adicionado à configuração do agente Wazuh no bloco `<syscheck>`:

```xml
<directories realtime="yes">/opt/SOC-Lab/FIM-Test</directories>
```

**Testes realizados**

Foram criados, modificados e excluídos arquivos no diretório monitorado. Depois, os registros correspondentes foram consultados no dashboard do Wazuh.

**Resultado observado**

Foram observados eventos relacionados à criação, à alteração da integridade e à exclusão de arquivos, confirmando o funcionamento do FIM no caminho configurado.

**Utilidade em uma investigação**

O FIM pode ajudar a identificar mudanças inesperadas em scripts, arquivos de configuração e outros recursos monitorados. Quando correlacionados com logs de autenticação e de sistema, esses eventos podem ajudar a investigar quando uma alteração ocorreu e se ela coincidiu com outras atividades suspeitas.

**Evidências**

<img width="770" height="766" alt="image" src="https://github.com/user-attachments/assets/6eff486d-4e05-49ec-aaeb-70a25b83c863" />
<img width="772" height="768" alt="image" src="https://github.com/user-attachments/assets/cc58affe-9d0a-4223-8fce-69eeecc2cba2" />
<img width="772" height="765" alt="image" src="https://github.com/user-attachments/assets/dbf2a7de-dda1-4956-a3c6-1fe5db5d2fe4" />

### 2.2 Autenticação e SSH

**Objetivo**

Os registros de autenticação são importantes para investigar tentativas de acesso ao endpoint Linux, incluindo falhas de login, possíveis tentativas repetidas de adivinhar credenciais e autenticações bem-sucedidas.

O SSH é um mecanismo comum de administração remota de servidores Linux. Por isso, acompanhar seus registros ajuda a identificar atividades de acesso que mereçam investigação.

**Configuração**

O agente Wazuh foi configurado para coletar os registros de autenticação em `/var/log/auth.log`:

```xml
<localfile>
  <location>/var/log/auth.log</location>
  <log_format>syslog</log_format>
</localfile>
```

**Testes realizados**

Foram realizadas tentativas de autenticação SSH, incluindo tentativas com credenciais inválidas. Em seguida, os eventos correspondentes foram consultados no dashboard do Wazuh.

**Resultado observado**

Os registros relacionados às tentativas de autenticação SSH foram encontrados no Wazuh, confirmando a coleta dos eventos de autenticação disponíveis no endpoint.

**Utilidade em uma investigação**

Esses registros podem auxiliar na identificação de tentativas de acesso não autorizado, na análise de falhas de autenticação e na verificação de acessos bem-sucedidos. Quando há informações suficientes nos logs, também é possível comparar horários e endereços de origem para identificar padrões suspeitos.

Uma tentativa de login malsucedida, isoladamente, não comprova um ataque. A investigação deve considerar frequência, origem, contexto e outros eventos relacionados.

**Evidências**

<img width="1544" height="814" alt="image" src="https://github.com/user-attachments/assets/67e082fc-1c77-4744-ad53-3254de538374" />

### 2.3 Logs do sistema

**Objetivo**

Além dos registros de autenticação, os logs gerais do sistema fornecem informações sobre serviços, mensagens operacionais e possíveis falhas ocorridas no Linux. Esses registros ajudam a contextualizar eventos de segurança e a entender o estado do endpoint durante uma investigação.

**Configuração**

O agente Wazuh foi configurado para coletar o arquivo `/var/log/syslog`:

```xml
<localfile>
  <location>/var/log/syslog</location>
  <log_format>syslog</log_format>
</localfile>
```

Essa configuração complementa a coleta de `/var/log/auth.log`, permitindo acompanhar diferentes categorias de mensagens registradas pelo sistema.

**Testes realizados**

Foi verificada a presença de eventos do sistema no dashboard do Wazuh, além dos registros de autenticação já coletados.

**Resultado observado**

Os logs do sistema foram incluídos na coleta do agente e ficaram disponíveis para consulta no Wazuh.

**Utilidade em uma investigação**

Os logs gerais podem fornecer contexto para eventos de segurança, como falhas de serviços, reinicializações e mensagens operacionais relevantes. A análise conjunta com os registros de autenticação e os eventos do FIM pode ajudar a estabelecer uma sequência temporal dos acontecimentos.

A coleta desses registros não significa que todas as mensagens representam ameaças. É necessário interpretar cada evento de acordo com seu conteúdo e contexto.

**Evidências**

<img width="1545" height="811" alt="image" src="https://github.com/user-attachments/assets/38038f04-ddcd-4527-9610-7cbcc0e860ed" />

## 3. Considerações sobre o monitoramento

As funcionalidades implementadas neste laboratório demonstram como diferentes fontes de telemetria podem ser centralizadas no Wazuh para apoiar o monitoramento de segurança.

Cada fonte oferece uma perspectiva diferente:

- **Sysmon:** fornece informações detalhadas sobre atividades como criação de processos.
- **PowerShell Script Block Logging:** registra blocos de script processados pelo PowerShell.
- **FIM:** identifica alterações nos arquivos e diretórios configurados.
- **Logs de autenticação e SSH:** ajudam a investigar tentativas de acesso ao Linux.
- **Logs do sistema:** fornecem contexto operacional para a análise de eventos.

A combinação dessas fontes permite ampliar a visibilidade sobre as atividades dos endpoints. Entretanto, a coleta de telemetria não equivale, por si só, à detecção de todas as ameaças. A identificação de comportamentos suspeitos depende também das regras de detecção, da correlação entre eventos e da análise realizada pelo analista.

Este documento descreve as funcionalidades testadas no laboratório. Recursos ainda não instalados ou validados, como a coleta por meio do `auditd` no Linux e regras personalizadas de detecção, não são considerados implementados nesta etapa.


