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
