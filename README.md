# Enterprise MES to PLC Recipe Management Gateway

## 📌 Project Context & Overview
This repository contains the complete IT/OT vertical network integration mapping and data mapping block specifications for a pharmaceutical **Manufacturing Execution System (MES) to PLC Recipe Download Gateway**. 

In regulated life sciences operations, manually entering batch parameters (such as target fill volumes, cycle time limits, or critical sterilization temperatures) on localized HMI panels introduces massive human-error risks. This project establishes the design structure for a deterministic, automated download framework that delivers pristine master batch record parameters directly to the factory floor while enforcing strict validation handshakes to align with **21 CFR Part 11 electronic record regulations** and **GAMP 5 lifecycle practices**.

---

## 🏗️ System Architecture & Data Path Infrastructure
The integration establishes a secure data highway connecting Level 3 enterprise layers down to Level 1 and Level 0 plant floor control platforms (Purdue Model alignment):

```text
  [ LEVEL 3: ENTERPRISE MANUFACTURING EXECUTION SYSTEM / MES ]
                             │
                             ▼ (Dynamic JSON Master Recipe Payload Download)
  [ LEVEL 2: NETWORK CONNECTOR APPLIANCE / INTEGRATION GATEWAY ]
                             │
                             ▼ (EtherNet/IP & PROFINET Memory Bit Mapping)
  [ LEVEL 1: DETACHED PROGRAMMABLE AUTOMATION CONTROLLERS / PLC RACKS ]
    ┌────────────────────────┴────────────────────────┐
    ▼                                                 ▼
[ Filling Controller Rack ]                  [ Checkweigher Scale I/O ]
  - Target Vol Real Block                      - Weight Value Real Blocks
  - Sterile Temp Verification                  - Modbus TCP Registers
```

### Integrated Interlocking & Verification:
* **The Handshake Cycle:** The network gateway writes the parameters to the PLC registers and toggles `RECIPE_READY = 1`. The production line remains locked until the local PAC validates the memory bounds and responds with `RECIPE_VALID = 1`.
* **The Heartbeat Safety Circuit:** The system maintains a continuous watchdog heartbeat between the gateway and the PLC. If data visibility drops for more than 3.0 seconds, the line immediately initiates a controlled hold sequence to prevent out-of-spec production.

---

## 📁 Repository Directory Structure

```text
📁 pharma-recipe-gateway
├── 📁 docs
│   ├── 📝 Project3_MES_PLC_Gateway_Specification.md   <- Dynamic payloads, sequence paths, & alarm controls
│   └── 📊 Project3_Hardware_Communication_Interface.csv <- Network drivers, physical mediums, & scan times
└── 📄 README.md                                       <- Core portfolio project brief & system index
```

---

## 📑 Core Documentation Breakdown

### 1. Functional MES to PLC Specification
Located at [`docs/Project3_MES_PLC_Gateway_Specification.md`](./docs/Project3_MES_PLC_Gateway_Specification.md), this document details the dynamic data payload design matrices, the step-by-step sequence diagrams, and the communication interrupt matrices.

### 2. Hardware Communication Interface Map
Located at [`docs/Project3_Hardware_Communication_Interface.csv`](./docs/Project3_Hardware_Communication_Interface.csv), this data grid inventories the active field connectivity drivers, mapping out industrial media types (Shielded Cat6, Fiber Optics), memory block definitions (Modbus Holding Registers, Data Blocks), scan times, and threshold configurations.

