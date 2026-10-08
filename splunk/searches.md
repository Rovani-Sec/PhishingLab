# Buscas SPL utilizadas no laboratório

## 1. Objetivo

Este arquivo reúne as principais consultas utilizadas no Splunk durante a investigação do laboratório de phishing.

As buscas foram construídas para identificar eventos do Sysmon, localizar conexões com a infraestrutura de phishing e organizar os campos relevantes para análise.

---

## 2. Validar eventos recebidos do endpoint

```spl
index=main host="Windows10"
```

Objetivo:

- confirmar que o endpoint está enviando dados ao Splunk;
- validar o host utilizado nas pesquisas posteriores.

---

## 3. Listar fontes e sourcetypes

```spl
index=main host="Windows10"
| stats count by source sourcetype
| sort - count
```

Objetivo:

- verificar quais logs estão sendo ingeridos;
- confirmar a presença de `WinEventLog:Microsoft-Windows-Sysmon/Operational`.

---

## 4. Consultar eventos do Sysmon

```spl
index=main host="Windows10"
source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
```

Objetivo:

- restringir a investigação aos eventos do Sysmon.

---

## 5. Filtrar Sysmon Event ID 3

```spl
index=main host="Windows10"
source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
EventCode=3
```

Objetivo:

- localizar eventos de conexão de rede;
- identificar processos que estabeleceram comunicação.

---

## 6. Localizar conexões com o servidor de phishing

```spl
index=main host="Windows10"
source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
EventCode=3
DestinationIp="10.10.10.101"
```

Objetivo:

- filtrar comunicações do endpoint com a infraestrutura de phishing.

---

## 7. Exibir campos relevantes

```spl
index=main host="Windows10"
source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
EventCode=3
DestinationIp="10.10.10.101"
| table _time Image User SourceIp SourcePort DestinationIp DestinationPort Protocol
| sort _time
```

Objetivo:

- visualizar horário;
- processo responsável;
- usuário;
- IP e porta de origem;
- IP e porta de destino;
- protocolo.

Essa consulta permitiu diferenciar acessos ao Mailpit e ao GoPhish.

---

## 8. Identificar somente a landing page do GoPhish

```spl
index=main host="Windows10"
source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
EventCode=3
DestinationIp="10.10.10.101"
DestinationPort=80
| table _time Image User SourceIp SourcePort DestinationIp DestinationPort Protocol
| sort _time
```

Objetivo:

- identificar exclusivamente conexões HTTP com a landing page do GoPhish.

Resultado principal observado:

```text
Process: msedge.exe
Destination: 10.10.10.101:80
Protocol: TCP
```

---

## 9. Identificar acesso ao Mailpit

```spl
index=main host="Windows10"
source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
EventCode=3
DestinationIp="10.10.10.101"
DestinationPort=8025
| table _time Image User SourceIp SourcePort DestinationIp DestinationPort Protocol
| sort _time
```

Objetivo:

- separar o acesso à interface web do Mailpit da conexão com a landing page.

---

## 10. Resumo por porta e processo

```spl
index=main host="Windows10"
source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
DestinationIp="10.10.10.101"
| stats count by EventCode Image DestinationPort
| sort - count
```

Objetivo:

- resumir as comunicações relacionadas à infraestrutura do laboratório;
- identificar quais processos e portas aparecem com maior frequência.

---

## 11. Busca textual ampla pelo IP

```spl
index=main "10.10.10.101"
```

Objetivo:

- realizar uma busca inicial quando ainda não se conhece o nome exato dos campos ou `source`.

Essa consulta é útil durante triagens e troubleshooting.

---

## 12. Validar logs internos do Universal Forwarder

```spl
index=_internal host="Windows10"
```

Objetivo:

- confirmar comunicação entre o Universal Forwarder e o Splunk Enterprise;
- diferenciar problemas de forwarding de problemas específicos de coleta do Sysmon.

---

## 13. Fluxo de investigação com SPL

```text
index=main host="Windows10"
        ↓
identificar source/sourcetype
        ↓
Sysmon Operational
        ↓
EventCode=3
        ↓
DestinationIp=10.10.10.101
        ↓
DestinationPort=80 / 8025
        ↓
Processo + usuário + protocolo + timeline
```

---

## 14. Observação

As consultas deste arquivo representam as buscas utilizadas durante o laboratório e foram mantidas simples para facilitar reprodução, entendimento e adaptação para futuros cenários de SOC / Blue Team.
