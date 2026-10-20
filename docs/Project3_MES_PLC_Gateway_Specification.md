# Functional Design Specification: Enterprise MES to PLC Recipe Management Gateway

## 1. Engineering Objective
To define the network integration mapping, data payload parameters, and handshake mechanisms required to vertically integrate a corporate Manufacturing Execution System (MES) with an automated machine controller layer over an industrial EtherNet/IP network. This system ensures electronic batch compliance without manual parameter input hazards.

## 2. Dynamic Data Payload Layout (JSON Master Matrix)
When a supervisor releases a batch at the enterprise level, the MES compiles a structural configuration message. The physical network gateway maps this payload directly into specific tracking memory addresses in the PAC data blocks:

| Parameter Field | Target PLC Register Tag | Data Type | Engineering Limits | GxP Critical Control Variable |
| :--- | :--- | :--- | :--- | :--- |
| Dynamic Recipe ID | RECIPE_ID | INT | 1000 to 9999 | No (Administrative Verification) |
| Target Fill Volume | CFG_Target_Vol_mL | REAL | 0.50 to 5.00 mL | Yes (Critical Quality Attribute) |
| Autoclave Temperature | CFG_Sterile_Temp_C | REAL | 121.0 to 124.0°C | Yes (Critical Process Parameter) |
| Purge Time Delay | CFG_N2_Purge_Sec | REAL | 1.5 to 5.0 Sec | Yes (Oxidation Prevention) |
| Rejection Threshold | CFG_Weight_Tol_Pct | REAL | 1.0 to 3.0 % | Yes (Defect Isolation Limits) |

## 3. Vertically Integrated Handshake Routine
To guarantee that the PLC does not start processing a batch using old or unverified data registers, the download gateway follows a deterministic sequence:

```text
 [ MES Layer ]                          [ Gateway Layer ]                      [ PLC Layer ]
      │                                         │                                    │
      ├─────── 1. Download Recipe Payload ─────►│                                    │
      │                                         ├──────── 2. Set RECIPE_READY ──────►│
      │                                         │                                    │ (Verifies data
      │                                         │◄─────── 3. Set RECIPE_VALID ───────┤  limits locally)
      │                                         │                                    │
      │                                         ├──────── 4. Set BATCH_START ───────►│ (Conveyors activate)
```

1. **MES Download:** The MES transfers the parameter payload block down to the network gateway appliance.
2. **Gateway Signaling:** The gateway writes the data values to the PLC registers and sets the boolean bit `RECIPE_READY = 1`.
3. **PLC Local Verification:** The PLC checks that all received values sit safely within hardcoded physical limits. If verified, the PLC sets `RECIPE_VALID = 1`.
4. **Batch Release:** The gateway verifies the validity flag, clears `RECIPE_READY = 0`, and fires `BATCH_START = 1` to open the physical line valves.

## 4. Communication Interrupt & Loss Matrix

| Alarm Code | Signal Trigger Condition | Operational Hazard | Machine Fail-Safe Action |
| :--- | :--- | :--- | :--- |
| ALM-301 | Heartbeat pulse bit missing >3.0 seconds | Loss of active supervisory data visibility | Hold current cycle step; lock out downline indexing wheels. |
| ALM-302 | Out-of-bounds register write attempt | Corrupt recipe footprint download | Refuse block; write error flag back to gateway; raise HMI warning. |
