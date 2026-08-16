## Voltages

| Group           | Variable     | Name                          | A / M  | Value | Comment                          |
|:--------------- |:------------ |:----------------------------- |:------ | -----:|:-------------------------------- |
| Core Voltages   | DRAM Voltage | DRAM Module Voltage           | Manual | 1.35V | When 1.38v, tWrWrSD can be set 4 |
| Core Voltages   | SoC Voltage  | System on Chip Voltage        | Manual | 1.10V |                                  |
| Fabric Voltages | CLDO VDDP    | Memory Module Signal Strength | Auto   |       |                                  |
| Fabric Voltages | VDDG CCD     | Core-to-I/O Die Transfer      | Auto   |       |                                  |
| Fabric Voltages | VDDG IOD     | MC-to-I/O Die Transfer        | Auto   |       |                                  |

## Core Signal Control

| Group               | Variable         | Name                         | A / M  |    Value | Comment                               |
|:------------------- |:---------------- |:---------------------------- |:------ | --------:|:------------------------------------- |
| Core Signal Control | ProcODT          | Processor On-Die Termination | Manual | 53.3 Ohm | 48 Ohm cause stressapptest miscompare |
| Core Signal Control | CR / CMD / Cmd2T | Command Rate 2T              | Manual |       1T |                                       |
| Core Signal Control | GDM              | Gear Down Mode               | Manual |  Enabled |                                       |
| Core Signal Control | PDE              | Power Down Enable            | Manual | Disabled |                                       |

## Primary Timings

| Group       | Variable | Name                                           | A / M  | Value | Formula                                                         | Comment |
|:----------- |:-------- |:---------------------------------------------- |:------ | -----:|:--------------------------------------------------------------- |:------- |
| Base Timing | tCL      | CAS (Column Address Strobe) Latency            | Manual |    16 |                                                                 |         |
| Base Timing | tRCDRd   | RAS to CAS Delay - Read                        | Manual |    19 |                                                                 |         |
| Base Timing | tRCDWr   | RAS to CAS Delay - Write                       | Manual |     8 |                                                                 |         |
| Base Timing | tRP      | Row Precharge                                  | Manual |    12 |                                                                 |         |
| Base Timing | tRAS     | Active to Precharge Delay (Row Address Strobe) | Manual |    36 | [[ram_tuning#tRAS (Active to Precharge Delay)\|≥ tRCDRd + tRP]] |         |

## Secondary Timings

| Group               | Variable | Name                                  | A / M  | Value | Formula                                                     | Comment                    |
|:------------------- |:-------- |:------------------------------------- |:------ | -----:|:----------------------------------------------------------- |:-------------------------- |
| Write Latency       | tCWL     | CAS Write Latency                     | Manual |    14 | [[ram_tuning#tCWL (CAS Write Latency)\|≤ tCL]]              |                            |
| Holy Trinity        | tRRDS    | Row Active to Row Active Delay, Short | Manual |     4 |                                                             |                            |
| Holy Trinity        | tRRDL    | Row Active to Row Active Delay, Long  | Manual |     4 |                                                             |                            |
| Holy Trinity        | tFAW     | Four Activate Window                  | Manual |    16 | [[ram_tuning#tFAW (Four Activate Window)\|= 4 * tRRDS]]     |                            |
| Bank Cycle Times    | tRC      | Row Cycle Time                        | Manual |    58 | [[ram_tuning#tRC (Row Cycle Time)\|≥ tRP + tRAS]]           | 56 unstable                |
| Recovery Delay      | tWR      | Write Recovery Time                   | Manual |    12 |                                                             |                            |
| Recovery Delay      | tRTP     | Read to Precharge                     | Manual |     6 | [[ram_tuning#tRTP (Read to Precharge)\|≤ tWR / 2]]          |                            |
| Wr-to-Rd Turnaround | tWTRS    | Write to Read Delay, Short            | Manual |     4 |                                                             |                            |
| Wr-to-Rd Turnaround | tWTRL    | Write to Read Delay, Long             | Manual |     8 |                                                             |                            |
| Refresh Cycle Time  | tRFC     | Refresh Cycle Time                    | Manual |   560 | [[ram_tuning#tRFC (Refresh Cycle Time)\|=(ns)*(MT/s)/2000]] | Tune last (temp sensitive) |
| Refresh Cycle Time  | tRFC2    | Refresh Cycle Time 2                  | Auto   |   486 | [[ram_tuning#tRFC2 (Refresh Cycle Time 2)\|≤ 0.7 * tRFC]]   | BIOS trained               |
| Refresh Cycle Time  | tRFC4    | Refresh Cycle Time 4                  | Auto   |   299 | [[ram_tuning#tRFC4 (Refresh Cycle Time 4)\|≤ 0.5 * tRFC2]]  | BIOS trained               |

## Tertiary Timings

| Group                 | Variable | Name                                            |  A / M | Value | Formula                                                                     | Comment    |
|:--------------------- |:-------- |:----------------------------------------------- | ------:|:----- |:--------------------------------------------------------------------------- |:---------- |
| Same Bank Group Delay | tRdRdSCL | Read to Read Delay, Same Chip, Long             | Manual | 4     | [[ram_tuning#tRDRDSCL (Read to Read, Same Bank Group, Long)\|= tWRWRSCL]]   |            |
| Same Bank Group Delay | tWrWrSCL | Write to Write Delay, Same Chip, Long           | Manual | 4     | [[ram_tuning#tWRWRSCL (Write to Write, Same Bank Group, Long)\|= tRDRDSCL]] |            |
| Raw Turnaround        | tRdWr    | Read to Write Delay                             | Manual | 16    |                                                                             |            |
| Raw Turnaround        | tWrRd    | Write to Read Delay                             | Manual | 6     |                                                                             |            |
| Same Chip Delay       | tRdRdSC  | Read to Read Delay, Same Chip                   | Manual | 1     |                                                                             |            |
| Same Chip Delay       | tWrWrSC  | Write to Write Delay, Same Chip                 | Manual | 1     |                                                                             |            |
| Same DIMM Delay       | tRdRdSD  | Read to Read Delay, Same DIMM, Different Rank   | Manual | 4     |                                                                             |            |
| Same DIMM Delay       | tWrWrSD  | Write to Write Delay, Same DIMM, Different Rank | Manual | 6     |                                                                             | 5 unstable |
| Diff DIMM Delay       | tRdRdDD  | Read to Read Delay, Different DIMM              | Manual | 4     |                                                                             |            |
| Diff DIMM Delay       | tWrWrDD  | Write to Write Delay, Different DIMM            | Manual | 4     |                                                                             |            |

## Signal Integrity & Drive Strength

| Group                 | Variable                  | Name                                             | A / M | Value | Comment |
|:--------------------- |:------------------------- |:------------------------------------------------ | -----:|:----- |:------- |
| Misc Signal           | tCKE                      | Clock Enable                                     |  Auto | 1     |         |
| Misc Signal           | tTRCPAGE                  | Target Row Cycle Page                            |  Auto | 0     |         |
| Termination Impedance | RttNom                    | Nominal Termination Resistance                   |  Auto |       |         |
| Termination Impedance | RttWr                     | Write Termination Resistance                     |  Auto |       |         |
| Termination Impedance | RttPark                   | Park Termination Resistance                      |  Auto |       |         |
| Setup Times           | MemAddrCmdSetup           | Memory Address Command Setup Time                |  Auto |       |         |
| Setup Times           | MemCsOdtSetup             | Memory Chip Select On-Die Termination Setup Time |  Auto |       |         |
| Setup Times           | MemCkeSetup               | Memory Clock Enable Setup Time                   |  Auto |       |         |
| Drive Strength        | MemCadBusClkDrvStren      | Memory CAD Bus Clock Drive Strength              |  Auto |       |         |
| Drive Strength        | MemCadBusAddrCmdDrvStren  | Memory CAD Bus Address/Command Drive Strength    |  Auto |       |         |
| Drive Strength        | MemCadBusCsOdtDrvStren    | Memory CAD Bus Chip Select/ODT Drive Strength    |  Auto |       |         |
| Drive Strength        | MemCadBusCkeDrvStren      | Memory CAD Bus Clock Enable Drive Strength       |  Auto |       |         |
| Misc                  | Mem Over Clock Fail Count | Memory Overclock Fail Count                      |  Auto |       |         |
