# OAI Environment Setup Notes

This repository contains installation notes, configuration files, and verification guides for building an OpenAirInterface (OAI) 5G environment.

The documentation covers:

- Ubuntu 22.04 Virtual Machine setup
- OpenAirInterface (OAI) environment installation
- OAI gNB (CU/DU Split) with OAI NR-UE (ZMQ RF Simulator)
- OCUDU gNB deployment with OAI NR-UE
- Configuration files (CU / DU / UE)
- Successful log files
- Environment verification
- Troubleshooting

---

# Repository Structure

```
.
├── Ubuntu_VM_setup_note.md
├── README.md
│
├── oai_gnb_oai_ue
│   ├── config
│   │   ├── cu.conf
│   │   ├── du.conf
│   │   └── oai_ue.conf
│   ├── log
│   │   └── ue.log
│   ├── installation_note_zh.md
│   └── installation_note_en.md
│
└── ocudu_gnb_oai_ue
    ├── config
    │   └── ocudu_gnb.yaml
    ├── log
    │   ├── gnb.log
    │   └── ue.log
    ├── installation_note_zh.md
    └── installation_note_en.md
```

---

# Documentation

## Ubuntu VM Setup

The Ubuntu VM setup guide describes how to install and configure an Ubuntu 22.04 virtual machine for the OAI environment.

- `Ubuntu_VM_setup_note.md`

---

## OAI gNB + OAI NR-UE

The latest installation guide is available in both Chinese and English.

- **Chinese:** `oai_gnb_oai_ue/installation_note_zh.md`
- **English:** `oai_gnb_oai_ue/installation_note_en.md`

This guide includes:

- OAI build environment
- CU / DU / UE configuration
- RF Simulator setup
- Verification checklist
- Troubleshooting
- Configuration files
- Successful log files

---

## OCUDU gNB + OAI NR-UE

The deployment guide for OCUDU gNB with OAI NR-UE is also available in both Chinese and English.

- **Chinese:** `ocudu_gnb_oai_ue/installation_note_zh.md`
- **English:** `ocudu_gnb_oai_ue/installation_note_en.md`

This guide includes:

- OCUDU build environment
- gNB configuration
- OAI NR-UE configuration
- ZMQ RF Simulator setup
- Open5GS connection
- Verification checklist
- Troubleshooting
- Configuration files
- Successful log files

---

# Notes

This repository is intended for laboratory research and educational purposes.

Sensitive information such as VPN credentials, SSH passwords, IMSI, OP/OPc, private keys, IP addresses, and subscriber information has been removed or replaced with placeholders before publication.
