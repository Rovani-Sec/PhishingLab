# Mapeamento MITRE ATT&CK

## 1. Objetivo

Este documento relaciona as atividades observadas no laboratório com técnicas do framework MITRE ATT&CK.

O mapeamento foi baseado somente nas ações efetivamente simuladas e observadas durante a campanha de phishing, evitando atribuir técnicas que não foram executadas no ambiente.

---

## 2. Resumo das técnicas

| Tática | Técnica | ID | Aplicação no laboratório |
|---|---|---|---|
| Initial Access | Phishing: Spearphishing Link | `T1566.002` | O usuário recebeu um e-mail contendo um link para a infraestrutura de phishing |
| Execution | User Execution: Malicious Link | `T1204.001` | O usuário abriu o e-mail e clicou no link |
| Command and Control / comunicação associada | Application Layer Protocol: Web Protocols | `T1071.001` | A interação gerou comunicação HTTP com o servidor `10.10.10.101:80` |

---

## 3. T1566.002 — Phishing: Spearphishing Link

### Descrição no contexto do laboratório

A campanha criada no GoPhish enviou ao usuário fictício um e-mail contendo um link direcionado ao servidor de phishing.

A timeline do GoPhish registrou:

```text
Email Sent
Email Opened
Clicked Link
```

### Evidência

- ferramenta: GoPhish;
- usuário: `Joao Lab`;
- e-mail fictício: `usuario@bluecorp.local`;
- infraestrutura: `10.10.10.101`;
- status final: `Clicked Link`.

<img width="1877" height="948" alt="02-gopish-timeline" src="https://github.com/user-attachments/assets/8d5f6fd8-4464-477f-b38b-a8a913b1bff4" />


### Mapeamento

```text
T1566.002 - Phishing: Spearphishing Link
```

Essa é a técnica principal do laboratório, pois o vetor inicial foi um link entregue ao usuário por e-mail.

---

## 4. T1204.001 — User Execution: Malicious Link

### Descrição no contexto do laboratório

A continuidade da atividade dependeu da ação do usuário ao abrir o e-mail e clicar no link.

Após o clique, o endpoint estabeleceu conexões com a infraestrutura de phishing.

O Sysmon Event ID 3 registrou o processo:

```text
C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe
```

comunicando-se com:

```text
10.10.10.101:80
```

### Evidência

```text
Process: msedge.exe
Destination IP: 10.10.10.101
Destination Port: 80
Protocol: TCP
```
<img width="1533" height="575" alt="06-splunk-event-timeline" src="https://github.com/user-attachments/assets/4869976f-6df5-46fd-86a7-aa37386aa3bf" />


### Mapeamento

```text
T1204.001 - User Execution: Malicious Link
```

A técnica representa a necessidade de interação do usuário para que o fluxo simulado de phishing avance.

---

## 5. T1071.001 — Application Layer Protocol: Web Protocols

### Descrição no contexto do laboratório

Após o clique, o Microsoft Edge realizou comunicação HTTP com o servidor GoPhish na porta `80`.

O tráfego foi observado tanto no endpoint quanto na rede.

No Splunk:

```text
msedge.exe → 10.10.10.101:80
```

No Suricata:

```text
ET INFO Gophish X-Server
10.10.10.101:80 → 10.20.20.151
```

### Evidência

<img width="806" height="763" alt="07-suricata-gophish-alerta" src="https://github.com/user-attachments/assets/6f34c808-571a-4637-bb23-8a13514f3bcf" />


### Mapeamento

```text
T1071.001 - Application Layer Protocol: Web Protocols
```

> Observação: neste laboratório, essa técnica é tratada como mapeamento relacionado à comunicação HTTP observada. O projeto não simulou uma infraestrutura completa de Command and Control nem execução de payload pós-clique.

---

## 6. Relação entre as técnicas

```text
T1566.002
Spearphishing Link
      │
      ▼
T1204.001
User Execution: Malicious Link
      │
      ▼
Microsoft Edge
      │
      ▼
HTTP / TCP 80
      │
      ▼
T1071.001
Web Protocols
```

---

## 7. Evidências por fonte

| Técnica | GoPhish | Sysmon / Splunk | Suricata |
|---|---|---|---|
| `T1566.002` | Email enviado e link registrado | — | — |
| `T1204.001` | `Clicked Link` | `msedge.exe` conecta ao servidor | comunicação observada |
| `T1071.001` | landing page HTTP | Event ID 3 para `10.10.10.101:80` | `ET INFO Gophish X-Server` |

---

## 8. Técnicas não simuladas

Este laboratório não executou etapas como:

- entrega ou execução de malware;
- persistência;
- privilege escalation;
- credential dumping;
- lateral movement;
- exfiltration;
- comando remoto pós-comprometimento.

Essas técnicas não foram incluídas no mapeamento para manter a documentação fiel às evidências efetivamente produzidas.

---

## 9. Conclusão

O laboratório demonstrou principalmente o fluxo de acesso inicial por phishing, interação do usuário e comunicação HTTP decorrente do clique.

O mapeamento MITRE ATT&CK permite relacionar a atividade técnica observada a comportamentos reconhecidos no mercado de segurança, auxiliando na documentação e na comunicação dos achados durante uma investigação SOC / Blue Team.
