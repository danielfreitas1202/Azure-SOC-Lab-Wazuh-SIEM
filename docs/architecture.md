--Objetivo da arquitetura:
O principal objetivo é centralizar eventos de endpoints em uma plataforma SIEM

--Infraestrutura:
A infraestrutura usada no laboratório foi:
Grupo de recursos: SOC-Lab-RG
VNet: SOC-VNet
Subrede: 10.10.1.0/24
NSG(Network Security Group): SOC-NSG
Maquinas usadas:
SOC-LINUX01 sistema operacional: Ubuntu Linux; IP: 10.10.1.6; Função: Syslog, SSH e FIM
SOC-WIN01 sistema operacional: Windows Server; IP: 10.10.1.5; Função: Syslog, FIM
SOC-WAZUH sistema operacional: Ubuntu Linux; IP: 10.10.1.4; Função: Wazuh manager, Indexer e Dashboard

Fluxo: As maquinas SOC-WIN01 e SOC-LINUX01 são adicionadas como agentes do Wazuh, assim eles enviam eventos ao Manager, que os processa para consulta no Dashboard

Segurança: Cada maquina possui seus próprios IPs privados, o NSG esta configurado para permitir apenas a utilização de portas necessárias para o funcionamento da estrutura

Limitações: Dependência dos recursos do Azure, conectividade e orçamento. Atualmente conta usada para o laboratório possui a assinatura de estudante, o que limitar muito a quantidade de créditos disponível para manter a estrutura funcionando e até mesmo hospedada 


<img width="1060" height="1024" alt="mermaid-diagram" src="https://github.com/user-attachments/assets/e7762ed2-5010-4619-83a3-fff9bcd10b92" />
