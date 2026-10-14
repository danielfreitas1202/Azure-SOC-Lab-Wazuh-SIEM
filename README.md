# Azure SOC Lab — Wazuh SIEM

## Sobre o projeto

Este projeto documenta a construção de um laboratório pessoal de cibersegurança na Microsoft Azure, com o objetivo de estudar monitoramento de endpoints, coleta de logs, análise de eventos e detecção de atividades potencialmente suspeitas.

O ambiente utiliza o Wazuh como plataforma de SIEM/XDR, centralizando eventos de máquinas Windows e Linux para investigação por meio de um dashboard.

## Objetivos

- Construir uma infraestrutura de laboratório na Microsoft Azure.
- Instalar e configurar o Wazuh Manager, Indexer e Dashboard.
- Integrar endpoints Windows e Linux.
- Coletar e analisar eventos de segurança.
- Implementar File Integrity Monitoring (FIM).
- Integrar Sysmon e PowerShell Script Block Logging no Windows.
- Monitorar autenticações SSH e eventos de sistema no Linux.
- Explorar detecções, regras e gerenciamento de vulnerabilidades.

## Arquitetura

O laboratório é composto por três máquinas virtuais:

- **SOC-WAZUH:** servidor central Wazuh, responsável pelo processamento e visualização dos eventos.
- **SOC-WIN01:** endpoint Windows para testes de telemetria e monitoramento de segurança.
- **SOC-LINUX01:** endpoint Ubuntu Linux para testes de autenticação, logs do sistema e integridade de arquivos.

As máquinas se comunicam por uma rede virtual privada na Azure.

## Tecnologias utilizadas

- Microsoft Azure
- Ubuntu Server
- Windows Server
- Wazuh SIEM/XDR
- Sysmon
- PowerShell Script Block Logging
- File Integrity Monitoring (FIM)
- Logs de autenticação SSH

## Testes realizados

- Verificação da comunicação entre agentes e servidor Wazuh.
- Coleta de eventos de segurança do Windows.
- Coleta de eventos Sysmon.
- Coleta de eventos do PowerShell.
- Detecção de criação, alteração e exclusão de arquivos.
- Coleta de logs de autenticação SSH no Linux.

Os resultados e as evidências de cada teste são documentados neste repositório.

## Aprendizados

O desenvolvimento do laboratório permitiu praticar a configuração de uma infraestrutura em nuvem, integração de endpoints com um SIEM, coleta de telemetria e validação de eventos de segurança.

Também foram documentados problemas de configuração, limitações do ambiente e procedimentos utilizados para solucioná-los.

## Limitações e próximos passos

As funcionalidades ainda não implementadas serão identificadas como pendentes. Entre os próximos passos estão a exploração de regras personalizadas de detecção e o aprofundamento do gerenciamento de vulnerabilidades.

## Aviso de segurança

Este projeto é destinado a estudos e testes em ambiente controlado. Informações sensíveis da infraestrutura, credenciais, chaves privadas e outros segredos não são publicados neste repositório.
