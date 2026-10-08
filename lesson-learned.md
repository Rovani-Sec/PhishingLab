# Lições Aprendidas

## 1. Visão geral

Este laboratório mostrou que uma investigação de phishing não depende de uma única ferramenta. A principal aprendizagem foi correlacionar contexto de campanha, telemetria de endpoint e visibilidade de rede para reconstruir a atividade com maior confiança.

As três fontes principais utilizadas foram:

- **GoPhish** para registrar envio, abertura e clique;
- **Sysmon + Splunk** para observar a atividade no endpoint;
- **Suricata** para validar a comunicação na camada de rede.

---

## 2. Correlação entre fontes é mais importante que um único alerta

O evento `Clicked Link` do GoPhish isoladamente mostra que houve interação com a campanha, mas não explica o que ocorreu no endpoint.

A confirmação ficou mais forte quando foi possível observar:

```text
GoPhish: Clicked Link
        ↓
Sysmon Event ID 3
        ↓
msedge.exe → 10.10.10.101:80
        ↓
Suricata: ET INFO Gophish X-Server
```

Essa correlação mostrou, na prática, como diferentes fontes podem complementar uma investigação SOC.

---

## 3. Troubleshooting do Splunk Universal Forwarder

Um dos principais problemas encontrados foi a ausência dos eventos do Sysmon no Splunk.

Inicialmente, o Universal Forwarder estava ativo e o servidor Splunk estava configurado em:

```text
10.20.20.102:9997
```

Mesmo assim, os eventos do Sysmon não apareciam nas buscas.

Durante o troubleshooting foram verificados:

- status do serviço `SplunkForwarder`;
- comunicação TCP com a porta `9997`;
- configuração efetiva do `inputs.conf` usando `btool`;
- existência do índice utilizado;
- logs internos do `splunkd.log`;
- permissões da conta do serviço.

---

## 4. Permissão para leitura do canal Sysmon

O serviço estava sendo executado como:

```text
NT SERVICE\SplunkForwarder
```

O canal:

```text
Microsoft-Windows-Sysmon/Operational
```

estava ativo e continha eventos, porém a conta do serviço não possuía inicialmente a permissão necessária para leitura.

A configuração do canal mostrou acesso para o grupo:

```text
S-1-5-32-573
```

que corresponde ao grupo local:

```text
Leitores de log de eventos
```

A solução foi adicionar:

```text
NT SERVICE\SplunkForwarder
```

a esse grupo e reiniciar o serviço.

Após essa alteração, os eventos do Sysmon passaram a ser ingeridos pelo Splunk.

### Aprendizado

Antes de concluir que uma falha está no SIEM ou na configuração da coleta, é importante validar também as permissões da conta de serviço sobre a fonte de dados.

---

## 5. Diagnóstico por camadas

O troubleshooting ficou mais simples quando o problema foi dividido em etapas:

```text
Sysmon gera o evento?
        ↓
Universal Forwarder lê o evento?
        ↓
Forwarder conecta ao Splunk?
        ↓
Splunk aceita dados na porta 9997?
        ↓
Evento é indexado?
        ↓
Campo pode ser consultado?
```

Essa abordagem evitou alterações aleatórias e permitiu identificar exatamente onde estava a falha.

---

## 6. Validação do receiver do Splunk

Em determinado momento, o `splunkd.log` apresentou mensagens de timeout e bloqueio de saída para:

```text
10.20.20.102:9997
```

Foram então validados:

- processo do Splunk Enterprise;
- porta `9997` em estado `LISTEN`;
- teste com `Test-NetConnection`;
- status do destino com `list forward-server`.

Isso reforçou a necessidade de testar tanto a aplicação quanto a conectividade de rede.

---

## 7. Diferença de timestamps

Foi observada uma pequena diferença entre os horários exibidos pelo GoPhish e os eventos consultados no Splunk.

Em um ambiente real, antes de reconstruir uma timeline com precisão, seria necessário verificar:

- sincronização NTP;
- timezone dos servidores;
- timezone do endpoint;
- forma como cada ferramenta armazena e apresenta timestamps;
- uso de UTC ou horário local.

### Aprendizado

Correlação temporal não deve depender apenas de horários visualmente próximos. Sincronização de tempo faz parte da qualidade da telemetria.

---

## 8. Separação entre tráfego do Mailpit e do GoPhish

O Splunk mostrou conexões do `msedge.exe` para o mesmo servidor em portas diferentes:

```text
10.10.10.101:8025
10.10.10.101:80
```

Isso permitiu separar:

- `8025` — interface web do Mailpit;
- `80` — landing page do GoPhish.

### Aprendizado

O IP sozinho não é suficiente para entender uma comunicação. Processo, porta, protocolo, direção e contexto precisam ser analisados em conjunto.

---

## 9. Valor do Sysmon Event ID 3

O Sysmon Event ID 3 foi fundamental para identificar:

- processo que iniciou a conexão;
- usuário associado;
- IP de origem;
- IP e porta de destino;
- protocolo utilizado.

A evidência permitiu relacionar o clique da campanha diretamente ao processo `msedge.exe` no endpoint.

---

## 10. Visibilidade complementar do Suricata

O Suricata forneceu uma confirmação independente da camada de endpoint.

O alerta:

```text
ET INFO Gophish X-Server
```

confirmou tráfego HTTP associado à infraestrutura do GoPhish.

Isso demonstrou a importância de combinar:

```text
Endpoint telemetry + Network telemetry
```

em vez de depender apenas de uma fonte.

---

## 11. Melhorias futuras

O laboratório pode ser evoluído com:

- sincronização NTP entre todas as VMs;
- índice dedicado para dados de endpoint;
- Splunk Add-on for Microsoft Windows;
- dashboards específicos para Sysmon;
- regras de correlação e alertas automáticos;
- ingestão de logs do Suricata no Splunk;
- eventos DNS e HTTP adicionais;
- criação de playbook de resposta;
- simulação de contenção e erradicação;
- documentação de falso positivo e critérios de severidade.

---

## 12. Conclusão

O principal aprendizado do projeto foi compreender o fluxo completo entre geração de atividade, coleta de telemetria, ingestão no SIEM, validação em rede e investigação.

Além da parte técnica de phishing, o laboratório exigiu troubleshooting real de permissões, serviços, conectividade, coleta e indexação, aproximando o exercício de atividades comuns em um ambiente de SOC / Blue Team.
