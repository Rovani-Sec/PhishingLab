# Investigação do Incidente de Phishing

## 1. Objetivo

Este documento registra a investigação de uma campanha de phishing simulada em ambiente controlado, correlacionando evidências de três fontes distintas:

- **GoPhish** — interação do usuário com a campanha;
- **Sysmon + Splunk** — telemetria do endpoint Windows;
- **Suricata** — telemetria e detecção na camada de rede.

O objetivo foi reconstruir a sequência do incidente e validar se a interação registrada pelo GoPhish também poderia ser identificada no endpoint e na rede.

---

## 2. Escopo do laboratório

| Componente | Identificação |
|---|---|
| Endpoint vítima | `Windows10` |
| IP do endpoint | `10.20.20.151` |
| Usuário de laboratório | `WINDOWS10\joao` |
| Usuário da campanha | `Joao Lab` |
| E-mail fictício | `usuario@bluecorp.local` |
| Phishing Server | `10.10.10.101` |
| GoPhish | HTTP `80` |
| Mailpit Web | TCP `8025` |
| Splunk Enterprise | `10.20.20.102` |
| Forwarding Splunk | TCP `9997` |

---

## 3. Resumo do incidente

Uma campanha de phishing foi criada no GoPhish e enviada para um usuário fictício do laboratório. O e-mail foi visualizado no Mailpit e o link foi acessado pelo endpoint Windows 10.

A investigação confirmou a atividade por meio de três perspectivas:

1. o **GoPhish** registrou o envio, a abertura do e-mail e o clique no link;
2. o **Sysmon Event ID 3**, centralizado no Splunk, registrou conexões do `msedge.exe` com o servidor `10.10.10.101`;
3. o **Suricata** detectou tráfego HTTP associado ao GoPhish e gerou o alerta `ET INFO Gophish X-Server`.

Com isso, o incidente foi classificado como **True Positive em laboratório controlado**.

---

## 4. Evidência da campanha — GoPhish

O GoPhish registrou a sequência completa da interação do usuário:

| Evento | Horário |
|---|---:|
| Campaign Created | 20:59:30 |
| Email Sent | 20:59:30 |
| Email Opened | 20:59:36 |
| Clicked Link | 20:59:37 |

A evidência demonstra que o e-mail foi entregue, aberto e que o link foi efetivamente clicado.

<img width="1877" height="948" alt="02-gopish-timeline" src="https://github.com/user-attachments/assets/01ab8b87-f08c-4816-a506-9a74e97a773e" />


---

## 5. Evidência no endpoint — Sysmon e Splunk

No endpoint `Windows10`, o Sysmon registrou conexões de rede por meio do **Event ID 3 — Network connection detected**.

A busca utilizada no Splunk para localizar conexões com a infraestrutura de phishing foi:

```spl
index=main host="Windows10"
source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
EventCode=3
DestinationIp="10.10.10.101"
| table _time Image User SourceIp SourcePort DestinationIp DestinationPort Protocol
| sort _time
```

A investigação identificou o processo:

```text
C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe
```

estabelecendo comunicação TCP com:

```text
10.10.10.101:8025
10.10.10.101:80
```

A porta `8025` corresponde ao acesso à interface web do **Mailpit**, enquanto a porta `80` corresponde ao acesso à **landing page do GoPhish**.

<img width="1533" height="575" alt="06-splunk-event-timeline" src="https://github.com/user-attachments/assets/d3feccf1-f24b-45d4-a82c-bf47171cb245" />


### Resultado observado

Foram identificadas conexões do `msedge.exe` com `10.10.10.101`, incluindo acessos à porta `80`, confirmando no endpoint a comunicação com o servidor de phishing.

---

## 6. Evidência de rede — Suricata

Na camada de rede, o Suricata registrou tráfego associado ao GoPhish e gerou o alerta:

```text
ET INFO Gophish X-Server
```

O alerta observado apresentou:

```text
Source:      10.10.10.101:80
Destination: 10.20.20.151:50288
Protocol:    TCP
Time:        20:59:38
```

A direção registrada representa a resposta HTTP enviada pelo servidor GoPhish ao endpoint vítima.

<img width="806" height="763" alt="07-suricata-gophish-alerta" src="https://github.com/user-attachments/assets/cda1c5c0-f299-4ea8-b78f-ae4df0aa6231" />


---

## 7. Correlação das evidências

A correlação entre as fontes permitiu reconstruir o fluxo do incidente:

```text
GoPhish
   │
   ├── Email Sent
   ├── Email Opened
   └── Clicked Link
          │
          ▼
Windows 10 / Microsoft Edge
          │
          ├── acesso ao Mailpit :8025
          └── acesso ao GoPhish :80
                    │
                    ▼
              Sysmon Event ID 3
                    │
                    ▼
                  Splunk

Em paralelo:

Windows 10 ───── HTTP/TCP ───── GoPhish
                    │
                    ▼
                 Suricata
                    │
                    ▼
        ET INFO Gophish X-Server
```

### Timeline consolidada

| Horário aproximado | Fonte | Evento |
|---|---|---|
| 20:59:30 | GoPhish | Campanha criada |
| 20:59:30 | GoPhish | E-mail enviado |
| 20:59:36 | GoPhish | E-mail aberto |
| 20:59:37 | GoPhish | Link clicado |
| ~20:59:27–20:59:28 | Sysmon / Splunk | `msedge.exe` acessa `10.10.10.101:8025` e `10.10.10.101:80` |
| 20:59:38 | Suricata | `ET INFO Gophish X-Server` |

> **Observação sobre timestamps:** foi observada uma pequena diferença de horário entre as fontes. Em uma investigação real, a sincronização de relógio/NTP entre os ativos deve ser validada antes de concluir a sequência temporal com precisão absoluta.

---

## 8. Classificação

| Campo | Resultado |
|---|---|
| Tipo de incidente | Phishing |
| Vetor | Link em e-mail |
| Resultado | Link acessado |
| Endpoint afetado | `Windows10` |
| Processo observado | `msedge.exe` |
| Infraestrutura de phishing | `10.10.10.101:80` |
| Evidência de endpoint | Sysmon Event ID 3 |
| Evidência de rede | Suricata — `ET INFO Gophish X-Server` |
| Classificação | True Positive — laboratório controlado |

---

## 9. Conclusão

A investigação confirmou uma interação bem-sucedida com a campanha simulada de phishing.

O GoPhish registrou o comportamento do usuário, o Sysmon forneceu evidência do processo e das conexões realizadas pelo endpoint, o Splunk permitiu centralizar e consultar essa telemetria, e o Suricata confirmou a comunicação na camada de rede.

A combinação dessas fontes permitiu reconstruir o incidente com maior confiança e demonstrou, na prática, um fluxo de investigação semelhante ao realizado por equipes de **SOC / Blue Team**.

A etapa seguinte do projeto é documentar separadamente a lógica de detecção, o mapeamento MITRE ATT&CK e as lições aprendidas durante a configuração e o troubleshooting do ambiente.
