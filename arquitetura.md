## Arquitetura do Projeto

```text
                         ┌─────────────────────────┐
                         │   Phishing Server       │
                         │     Ubuntu Server       │
                         │     10.10.10.101        │
                         │                         │
                         │  GoPhish     TCP/80     │
                         │  Mailpit     TCP/8025   │
                         │              SMTP/1025  │
                         └────────────┬────────────┘
                                      │
                                      │ HTTP / SMTP
                                      │
                         ┌────────────▼────────────┐
                         │        pfSense          │
                         │                         │
                         │       Suricata          │
                         │     IDS / Network       │
                         │      Monitoring         │
                         └────────────┬────────────┘
                                      │
                            BLUE TEAM NETWORK
                              10.20.20.0/24
                                      │
                    ┌─────────────────┴─────────────────┐
                    │                                   │
          ┌─────────▼─────────┐              ┌──────────▼─────────┐
          │    Windows 10     │              │ Splunk Enterprise  │
          │   Victim Host     │              │    SIEM Server     │
          │   10.20.20.151    │              │    10.20.20.102    │
          │                   │              │                    │
          │ Microsoft Edge    │   TCP/9997   │ Search & Analysis  │
          │ Sysmon            ├─────────────►│                    │
          │ Splunk Forwarder  │              └────────────────────┘
          └───────────────────┘
```

---

### Fluxo do laboratório

1. O usuário acessa o e-mail de teste pelo Mailpit.
2. O clique no link direciona o navegador ao servidor GoPhish.
3. O Sysmon registra as conexões de rede no endpoint.
4. O Splunk Universal Forwarder envia os eventos para o Splunk Enterprise.
5. O Suricata monitora o tráfego entre as redes e gera alertas de rede.
6. As evidências do GoPhish, Sysmon/Splunk e Suricata são correlacionadas durante a investigação.
