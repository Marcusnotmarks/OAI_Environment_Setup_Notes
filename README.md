# OAI Environment Setup Notes

This repository contains the installation notes and configuration guides for building an OpenAirInterface (OAI) environment.

The documentation covers:

- Ubuntu 22.04 Virtual Machine setup
- OpenAirInterface (OAI) environment installation
- OAI gNB (CU/DU Split) with OAI NR-UE RF Simulator
- Configuration files (CU / DU / UE)
- Common troubleshooting
- Environment verification

---

## Repository Structure

```
.
├── Ubuntu_VM_setup_note.md
├── README.md
├── oai_gnb_oai_ue
│   ├── config
│   ├── log
│   ├── installation_note.md           (Chinese)
│   └── installation_note_1.1_en.md    (English)
└── ocudu_gnb_oai_ue
    └── installation_note.md
```

---

## Documentation

### Ubuntu VM Setup

The Ubuntu VM setup guide describes how to install and configure an Ubuntu 22.04 virtual machine for the OAI environment.

- `Ubuntu_VM_setup_note.md`

### OAI gNB + NR-UE Setup

Version **1.1** is the latest installation guide.

Available languages:

- **Chinese:** `oai_gnb_oai_ue/installation_note.md`
- **English:** `oai_gnb_oai_ue/installation_note_1.1_en.md`

This guide includes:

- OAI build environment
- CU / DU / UE configuration
- RF Simulator setup
- Verification checklist
- Troubleshooting
- Security notes

### OCUDU

Documentation for the O-CU/O-DU deployment is located in:

- `ocudu_gnb_oai_ue/installation_note.md`

---

## Notes

This repository is intended for laboratory research and educational purposes.

Sensitive information such as VPN credentials, SSH passwords, IMSI, OP/OPc, and private keys has been removed or replaced with placeholders.
