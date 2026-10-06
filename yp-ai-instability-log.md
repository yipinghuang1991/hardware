# yp-ai（AI Endpoint PC）硬體不穩事件總表

> 建檔 2026-10-06。來源：`journalctl` 全 boot 掃描（journal 涵蓋 2026-09-10 起）+ `rasdaemon`（10-06 起持續收集）。
> 用途：每次異常重開後對照——是否已知形態、頻率有無變化。
> 註：本 repo 其餘調校文件（ram_tuning / ram_timing_gaming_pc / cpu_tuning_gaming_pc）皆為 **yp-gaming** 用；本機（yp-ai）**無任何超頻/調校紀錄**。

## TL;DR

- yp-ai 自 2026-09-10 以來：**17 次「MC5 sync flood」致命重啟（截至 10-06 11:19）+ 2 次「BP_SYS_RST_L reset pin」重啟 + 7 次存活的記憶體域 deferred MCE**。
- 成因層級＝**平台硬體**（CPU/SoC fabric・時序/供電邊際）。軟體無法防止；日誌無法定位到元件，定案需換件測試。
- 分布＝「風暴期」而非均勻：9/17–19（6 死）、9/30（8 死）、10/6（3 死＋1 pin，11:19 仍在發）；其餘時間乾淨、可重載連跑數天。
- 已知排除：特定 RAM 套件 ✗、kernel 版本 ✗、特定推論引擎 ✗、PCIe/AER ✗、**本機超頻 ✗**（無調校；RAM 跑 JEDEC 2400）。

## 本機硬體（pc_rigs 對照）

- MSI MEG X570 UNIFY｜5950X 16C32T｜2× R9700 32G（顯示卡 `0000:2f` boot_vga=1、運算卡 `0000:32`）
- Kingston KF3600C18D4/32GX 4×32G dual-rank @ **2400 MT/s（JEDEC，無 XMP）**
- PSU：FSP Dagger PM MIT SFX-L **1200W ATX3**｜散熱：Thermalright Phantom Spirit 120｜機殼：Lian Li PC-A55B
- （`pc_rigs.md` 的 RAM 欄位寫 G.Skill 4000×4，與現裝不符＝舊資訊）

## 致命事件簽名（每次相同）

- `MC5_STATUS[-|UE|MiscV|AddrV|PCC|TCC|SyndV] 0xbea0000000000108`
- `Syndrome 0x000000004d000000`、`Execution Unit Ext. Error Code: 0`、`cache level: RESV, tx: GEN, mem-tx: GEN`
- CPU 編號每次不同（3/5/7/8/12/21/22…）→ 系統性、非單核
- 下次開機的 `Previous system reset reason [0x08000800]: an uncorrected error caused a data fabric sync flood event`
- 機制：不可修正錯誤（UE）→ PCC（context corrupt）→ 平台 sync flood → 秒級重置（OS 來不及寫任何關機序列）
- 位址欄兩型：多數集中 `0x01ffffff…` 區；變體為使用者空間型 `0x00007f95…`（9/30 首發）與 `0x00007f31…`（10-06 11:19）

## 事件時間線

- **09-14**：2× deferred MCE（MC10 10:51 / MC14 18:21，banks reserved，存活）
- **09-17 19:48**：flood #1（前奏：Xorg coredump、Discord/lm-studio 崩、amdgpu 顯示卡 DMCUB/INBOX0 卡死、flip timeout）
- **09-18 04:47**：flood #2（前奏：HPET read timeout ×2 各 ~201ms、CPU7 hard lockup）
- **09-18 18:35**：reset pin #1（同日 12:10 曾有 soft lockup——CPU22 卡 1606s）
- **09-19 17:32 / 17:37 / 17:45 / 22:58**：flood #3–6（前奏：DMCUB error、CPU1 hard lockup）
- **09-25 / 09-26 / 09-27**：3× deferred MCE（MC23/MC17/MC19，存活）
- **09-30 01:26–04:12**：flood #7–14（8 連爆；無可見前奏、純瞬死；當時 llama.cpp 服務 + 多 session/多 subagent 重載）
- **10-01 03:36 / 10-02 09:58**：2× deferred MCE（MC12/MC15，存活）
- **10-05 晚**：RAM 由 64G（2×32）換為 128G（4×32）；memtest 64G×2 loops 0 FAIL
- **10-06 04:35**：使用者手動 reboot（04:34:51 停 memtest 之後——時序正常）
- **10-06 04:52 / 05:16**：flood #15–16（無前奏、瞬死；vLLM 雙卡高載中）
- **10-06 09:04**：reset pin #2
- **10-06 09:51–11:19**：乾淨期（0 MCE、0 amdgpu ERROR）
- **10-06 11:19:15**：**flood #17**（CPU:7、Addr `0x7f31…`、無前奏；vLLM 生成中 + 多 session 同時跑；11:19:57 自動重開）
- **10-06 11:19:57 起**：current boot；11:20:02 曾一筆 `clocksource: Watchdog remote CPU 11 read timed out`（boot 時、觀察中）

## 已排除（都驗證過）

- **新 RAM 非成因**：9/17–19、9/30 風暴都在舊 64G 配置時期；兩代 RAM 都發生。
- **kernel 非成因**：跨 7.2.6 / 7.2.7 / 7.2.8 三個版本都發生。
- **工作負載非特定**：llama.cpp（9 月）與 vLLM（10 月）時代都發生。
- **PCIe/AER 無關**：死亡前 boot 內 0 筆 PCIe bus error / AER。
- **本機超頻無關**：無調校紀錄、RAM JEDEC 2400。（hardware/ repo 調校文＝yp-gaming）
- **時段無關**：9/17 傍晚、9/18 一早一晚、9/19 傍晚、9/30 凌晨、10/6 清晨與中午——與「凌晨」無特別關聯；相關的是重載+機率。

## 現行加重因子（未定案）

- 負載時 CPU Tctl 84–90°C（Tjmax 90，貼頂）；GPU junction 74–77°C（正常）。
- 風暴都發生在「雙卡重載 + 多 process 同時開工」時段；但同樣負載也常跑數天無事 → 邊際不穩、機率性。

## 下次異常重置後取證步驟

```bash
journalctl -b -1 -k | grep -B2 -A10 'reset reason'
journalctl -b -1 --no-pager | tail -300 | grep -iE 'lockup|DMCUB|flip_done|watchdog|drm.*ERROR|Hardware Error'
sudo ras-mc-ctl db --table-summary    # rasdaemon，10-06 起
```

同簽名＝已知形態（頻率統計+1）；新簽名＝新問題。

## 候選處置（若風暴再現或加密）

1. 散熱：Phantom Spirit 120 扣具/膏/機殼風道檢視（負載 84–90°C 貼 Tjmax）——便宜、無破壞性，先做無妨。
2. BIOS：確認全預設（PBO/CO/SoC 電壓無殘留）＋查 `A.N3`（2026-08-18）後有無新版。
3. 供電：FSP 1200W SFX-L ATX3；若再現風暴→借測換電源。
4. 記憶體：128G 全量 memtest（現只測過 64G）＋必要時 2-DIMM A/B。
5. 不建議：主動負載複現（風險高、效益低）。
