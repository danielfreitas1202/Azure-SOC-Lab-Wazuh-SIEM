# Regras Personalizadas do Wazuh

## Visão geral

Este documento descreve a criação e a validação de uma regra personalizada do Wazuh para identificar tentativas repetidas de autenticação SSH utilizando um nome de usuário inexistente.

O objetivo é demonstrar como as regras nativas do Wazuh podem ser complementadas por regras próprias, capazes de identificar padrões de eventos que podem indicar atividades suspeitas.

## Cenário de detecção

A regra utiliza eventos SSH que correspondem à detecção nativa do Wazuh para tentativas de autenticação com usuários inexistentes.

A regra personalizada foi configurada para gerar um alerta quando cinco eventos correspondentes forem identificados, dentro de uma janela de 120 segundos, a partir do mesmo endereço IP de origem.

| Parâmetro                            | Configuração                  |
| ------------------------------------ | ----------------------------- |
| Identificador da regra personalizada | `100100`                      |
| Nível da regra                       | `10`                          |
| Identificador da regra nativa        | `5710`                        |
| Frequência                           | 5 eventos correspondentes     |
| Janela de tempo                      | 10 segundos                  |
| Condição de correlação               | Mesmo endereço IP de origem   |
| Mapeamento MITRE ATT&CK              | T1110.001 — Password Guessing |

A detecção indica uma possível tentativa de força bruta ou adivinhação de senhas. O alerta não comprova que houve acesso não autorizado bem-sucedido.

## Configuração

A regra foi adicionada ao arquivo:

`/var/ossec/etc/rules/local_rules.xml`

A regra de exemplo já existente, identificada pelo ID `100001`, foi preservada. A nova regra foi adicionada dentro do grupo `local`:

```xml
<rule id="100100" level="10" frequency="5" timeframe="10">
  <if_matched_sid>5710</if_matched_sid>
  <same_source_ip />
  <description>Possible SSH brute-force attempt using a nonexistent user from the same source IP.</description>
  <mitre>
    <id>T1110.001</id>
  </mitre>
  <group>authentication_failed,ssh_brute_force,</group>
</rule>
```

A descrição da regra pode ser traduzida para:

> Possível tentativa de força bruta via SSH utilizando um usuário inexistente a partir do mesmo endereço IP de origem.

A regra referencia o ID nativo `5710`, que identifica tentativas de autenticação com um usuário inexistente. Os parâmetros de frequência e intervalo de tempo permitem correlacionar eventos repetidos, enquanto `same_source_ip` exige que eles tenham o mesmo IP de origem.

## Procedimento de validação

A configuração foi verificada utilizando o mecanismo de análise do Wazuh:

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

Após a validação da configuração, a regra personalizada foi testada e gerou o alerta esperado.

Para repetir o teste, recomenda-se utilizar tentativas de autenticação controladas no laboratório e confirmar se os eventos correspondem à regra nativa e às condições de correlação configuradas.

## Resultado da validação

**Status: testada com sucesso.**

A regra personalizada gerou o alerta esperado durante o teste. Uma captura de tela foi registrada como evidência da implementação.

### Evidências

Adicione a captura de tela do alerta personalizado. Sempre que possível, a imagem deve mostrar o identificador da regra, a descrição, a severidade e o endereço IP de origem.

<img width="1545" height="814" alt="image" src="https://github.com/user-attachments/assets/a02199cb-49bf-4b75-a9f4-dd98e6aa9c1a" />

## Considerações de segurança

- A regra deve ser utilizada em ambientes próprios ou nos quais exista autorização para monitoramento.
- Um alerta deve ser investigado considerando o contexto. Falhas repetidas de autenticação também podem ser causadas por serviços mal configurados ou atividades administrativas legítimas.
- A regra identifica tentativas com usuários inexistentes, mas não cobre todas as formas de força bruta SSH, como várias tentativas contra usuários válidos.
- A eficácia da detecção depende da coleta dos logs SSH, da decodificação correta dos eventos e de sua disponibilidade para o Wazuh Manager.
- Os identificadores das regras personalizadas devem ser únicos para evitar conflitos com outras regras.

## Melhorias futuras

Como evolução do laboratório, podem ser implementadas as seguintes melhorias:

- Criar uma regra específica para falhas repetidas contra usuários existentes.
- Correlacionar tentativas distribuídas em intervalos maiores para identificar ataques mais lentos.
- Detectar uma autenticação SSH bem-sucedida após múltiplas falhas.
- Documentar um procedimento de resposta para verificar a origem das tentativas e investigar o endpoint afetado.

## Conclusão

A regra personalizada de SSH foi implementada e testada com sucesso no laboratório SOC desenvolvido no Azure. A implementação demonstra como as detecções nativas do Wazuh podem ser estendidas por meio de regras próprias e correlação de eventos, gerando alertas de maior severidade para padrões suspeitos de autenticação.

Essa funcionalidade complementa a coleta de logs SSH, a detecção de vulnerabilidades, o monitoramento de endpoints e o monitoramento de integridade de arquivos já implementados no laboratório.
