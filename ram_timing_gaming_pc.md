
## Hardware & Signal Configuration

| Group                 | Variable                  | Name                                             |  A / M | Value    | Comment |
|:--------------------- |:------------------------- |:------------------------------------------------ | ------:|:-------- |:------- |
| Core Signal Control   | ProcODT                   | Processor On-Die Termination                     | Manual | 48 Ohm   |         |
| Core Signal Control   | CR / CMD / Cmd2T          | Command Rate 2T                                  | Manual | 1T       |         |
| Core Signal Control   | GDM                       | Gear Down Mode                                   | Manual | Enabled  |         |
| Core Signal Control   | PDE                       | Power Down Enable                                | Manual | Disabled |         |
| Termination Impedance | RttNom                    | Nominal Termination Resistance                   |   Auto |          |         |
| Termination Impedance | RttWr                     | Write Termination Resistance                     |   Auto |          |         |
| Termination Impedance | RttPark                   | Park Termination Resistance                      |   Auto |          |         |
| Setup Times           | MemAddrCmdSetup           | Memory Address Command Setup Time                |   Auto |          |         |
| Setup Times           | MemCsOdtSetup             | Memory Chip Select On-Die Termination Setup Time |   Auto |          |         |
| Setup Times           | MemCkeSetup               | Memory Clock Enable Setup Time                   |   Auto |          |         |
| Drive Strength        | MemCadBusClkDrvStren      | Memory CAD Bus Clock Drive Strength              |   Auto |          |         |
| Drive Strength        | MemCadBusAddrCmdDrvStren  | Memory CAD Bus Address/Command Drive Strength    |   Auto |          |         |
| Drive Strength        | MemCadBusCsOdtDrvStren    | Memory CAD Bus Chip Select/ODT Drive Strength    |   Auto |          |         |
| Drive Strength        | MemCadBusCkeDrvStren      | Memory CAD Bus Clock Enable Drive Strength       |   Auto |          |         |
| Misc                  | Mem Over Clock Fail Count | Memory Overclock Fail Count                      |   Auto |          |         |

## Primary Timings

| Group       | Variable | Name                                           | A / M  | Value | Comment |
|:----------- |:-------- |:---------------------------------------------- |:------ | -----:|:------- |
| Base Timing | tCL      | CAS (Column Address Strobe) Latency            | Manual |    16 |         |
| Base Timing | tRCDRd   | RAS to CAS Delay - Read                        | Manual |    19 |         |
| Base Timing | tRCDWr   | RAS to CAS Delay - Write                       | Manual |     8 |         |
| Base Timing | tRP      | Row Precharge                                  | Manual |    12 |         |
| Base Timing | tRAS     | Active to Precharge Delay (Row Address Strobe) | Manual |    36 |         |

## Secondary Timings

| Group                         | Variable | Name                                  | A / M  | Value | Comment     |
|:----------------------------- |:-------- |:------------------------------------- |:------ | -----:|:----------- |
| Bank Cycle                    | tRC      | Bank Cycle Time (Row Cycle)           | Manual |    58 | 56 unstable |
| Row Activation (Holy Trinity) | tRRDS    | Row Active to Row Active Delay, Short | Manual |     4 |             |
| Row Activation (Holy Trinity) | tRRDL    | Row Active to Row Active Delay, Long  | Manual |     4 |             |
| Row Activation (Holy Trinity) | tFAW     | Four Activate Window                  | Manual |    16 |             |
| Write Latency & Recovery      | tWR      | Write Recovery Time                   | Manual |    12 |             |
| Write Latency & Recovery      | tCWL     | CAS Write Latency                     | Manual |    14 |             |
| Recovery Delays               | tRTP     | Read to Precharge                     | Manual |     6 |             |
| Write-to-Read Turnaround      | tWTRS    | Write to Read Delay, Short            | Manual |     4 |             |
| Write-to-Read Turnaround      | tWTRL    | Write to Read Delay, Long             | Manual |     8 |             |
| Refresh Cycle Time            | tRFC     | Refresh Cycle Time                    | Manual |   560 |             |
| Refresh Cycle Time            | tRFC2    | Refresh Cycle Time 2                  | Auto   |   486 |             |
| Refresh Cycle Time            | tRFC4    | Refresh Cycle Time 4                  | Auto   |   299 |             |

## Tertiary Timings

| Group            | Variable | Name                                            |  A / M | Value | Comment    |
|:---------------- |:-------- |:----------------------------------------------- | ------:|:----- |:---------- |
| Raw Turnaround   | tRdWr    | Read to Write Delay                             |   Auto | 18    |            |
| Raw Turnaround   | tWrRd    | Write to Read Delay                             |   Auto | 7     |            |
| Bank Group Sync  | tRDRDSCL | Read to Read Delay, Same Chip, Long             | Manual | 4     |            |
| Bank Group Sync  | tWRWRSCL | Write to Write Delay, Same Chip, Long           | Manual | 4     |            |
| Cross-Rank Read  | tRdRdSC  | Read to Read Delay, Same Chip                   |   Auto | 1     |            |
| Cross-Rank Read  | tRdRdSD  | Read to Read Delay, Same DIMM, Different Rank   | Manual | 4     |            |
| Cross-Rank Read  | tRdRdDD  | Read to Read Delay, Different DIMM              |   Auto | 4     |            |
| Cross-Rank Write | tWrWrSC  | Write to Write Delay, Same Chip                 |   Auto | 1     |            |
| Cross-Rank Write | tWrWrSD  | Write to Write Delay, Same DIMM, Different Rank | Manual | 6     | 5 unstable |
| Cross-Rank Write | tWrWrDD  | Write to Write Delay, Different DIMM            | Manual | 4     |            |
| Misc Signal      | tCKE     | Clock Enable                                    |   Auto | 1     |            |
| Misc Signal      | tTRCPAGE | Target Row Cycle Page                           |   Auto | 0     |            |
