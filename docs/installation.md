# Implantação do laboratório

## 1. Pré-requisitos
Assinatura do Azure 
## 2. Criação do grupo de recursos
Criação de um RG especifico para a estrutura do laboratório, nomeado SOC-Lab-RG, para criar um ambiente separado de possíveis outros projetos futuros e manter a organização de tudo dentro do RG
## 3. Configuração da rede virtual e subnet
Houve a criação de uma VNet (SOC-VNet), dentro do RG SOC-Lab-RG, para ter uma rede privada virtual e ouve a criação de uma subrede (SOC-Subnet) para fazer a conexão das maquinas virtuais usadas no laboratório 
## 4. Configuração do Network Security Group
O NSG foi criado dentro do RG SOC-Lab-RG e configurado com permissões mínimas de regras de segurança de entrada e saída para o funcionamento do laboratório e conexão com as maquinas
## 5. Criação da VM SOC-WAZUH
A primeira VM criada foi a SOC-WAZUH, criada no SOC-Lab-RG e conectada a SOC-Subnet da SOC-VNet possuindo tando um IP publico quanto um privado, ele usa Ubuntu Linux como sistema operacional e tem o intuito de ser o servidor responsável para rodar o Wazuh. esta usando: Tamanho: Standard B2as v2, vCPUs: 2, RAM: 8 GiB, Threads por núcleo: 2. As demais configurações foram mantidas com o padrão sugerido pelo Azure 
## 6. Instalação do Wazuh
Ao conectar com a VM SOC-WAZUH, rodei o comando "sudo apt update" para atualizar todos os programas e pacotes instalados no SO; em seguida baixei o instalador do Wazuh com "curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh" e iniciei a instalação com "sudo bash ./wazuh-install.sh -a". Com a instalação concluída e verifiquei se os programas estavam sendo executados "sudo systemctl status wazuh-manager --no-pager", "sudo systemctl status wazuh-indexer --no-pager" e "sudo systemctl status wazuh-dashboard --no-pager".
## 7. Criação das VMs Windows e Linux
A VM SOC-WIN01 foi criada dentro do SOC-Lab-RG e conectada a SOC-Subnet da SOC-VNet possuindo apenas um IP privado, ele usa Windows Server como sistema operacional e tem o intuito de ser o um dos agentes Wazuh que terá suas atividades monitoradas pelo Wazuh. esta usando: Tamanho: Standard B2as v2, vCPUs: 2, RAM: 8 GiB, Threads por núcleo: 2. As demais configurações foram mantidas com o padrão sugerido pelo Azure
A VM SOC-LINUX01 foi criada dentro do SOC-Lab-RG e conectada a SOC-Subnet da SOC-VNet possuindo apenas um IP privado, ele usa Ubuntu Linux como sistema operacional e tem o intuito de ser o um dos agentes Wazuh que terá suas atividades monitoradas pelo Wazuh. esta usando: Tamanho: Standard B2as v2, vCPUs: 2, RAM: 8 GiB, Threads por núcleo: 2. As demais configurações foram mantidas com o padrão sugerido pelo Azure
## 8. Instalação e registro dos agentes
Acesso o dashboard do Wazuh através do IP público pela minha maquina pessoal. Dentro do dashboard cliquei em Deploy new agent e preenchi as informações pedidas para cada VM (SOC-WIN01 e SOC-LINUX01) e baixei seus respectivos arquivos de instalação, enviei ambos para a SOC-WAZUH para envia-los para suas restivas VMs pois elas não estão concitadas a internet.
Em cada uma das VMs com seus respectivos arquivos de instalação executei o comando "msiexec.exe /i "C:\Users\socadmin\wazuh-agent-4.14.7-1.msi" /q WAZUH_MANAGER="10.10.1.4" WAZUH_AGENT_NAME="SOC-WIN01"" no Powershell do SOG-WIN01 para instalar o agente, em seguida o executei com "Start-Service wazuhsvc" e verifiquei se estava rodando com "Get-Service WazuhSvc"; No SOC-LINUX01 executei "sudo env WAZUH_MANAGER='10.10.1.4' WAZUH_AGENT_NAME='SOC-LINUX01' dpkg -i /home/socadmin/wazuh-agent_4.14.7-1_amd64.deb", executei com "sudo systemctl start wazuhsvc".
<img width="1539" height="617" alt="image" src="https://github.com/user-attachments/assets/877273a6-e780-418d-8f1e-47f201845cc5" />

## 9. Validação da comunicação
Após a execução do agente, retorno ao dashboard para verificar se o Wazuh esta com os agente conectados e funcionando. Após ter a confirmação que ambos os agentes estavam conectados oa Wazuh eu segui para os teste. 
## 10. Segurança e controle de custos
Além de configurar o MSG com o menor número de permissões possíveis para conseguir realizar a conexão entre as máquinas virtuais, foi usado métodos diferentes. As máquinas Linux usavam chave SSH, enquanto o Windows Server utilizava sistema de login e senha. Sobre os custos, foi escolhido o menor tamanho disponível para baratear o custo das máquinas, pois a assinatura utilizada possuía poucos créditos.
