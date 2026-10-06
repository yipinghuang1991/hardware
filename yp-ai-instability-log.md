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

## 社群案例彙整（2026-10-06 讀）

三個來源、同一簽名（MC5/bank 5、`0xbea…0108`、SYND `4d000000`、Exec Unit Ext 0——與本機一字不差；跨 5900X/5800X3D/5950X/5800H/7600/5700G）：

- **Reddit r/pop_os「Data Fabric Sync Flood Event」**（5900X + MSI X570 Tomahawk，約一年前）：
  - 他的修復組合（自述之後沒再炸）：① BIOS 關 **Global C-state Control** ② **CPU NB/SoC Voltage** 模式改 **AMD Overclocking**（截圖讀值 1.108V，A-XMP 3600）③ **Curve Optimizer：Positive、Magnitude 10**（加壓裕度）。
  - 留言：另一條線是 **WD Black SN770 NVMe 韌體 bug**（更新固件修好；跨 Framework 案例）。
- **gist eliottness（Zen 3/4 綜合整理，最完整）**：
  - 7600 修復組合＝`processor.max_cstate=2 + nvme_core.default_ps_max_latency_us=0 + pcie_aspm=off + pcie_ports=native + amdgpu.ppfeaturemask=0xfff73fff` → 2 個月+未再發。
  - 關鍵提醒：**要驗證 C6 真的關掉了**（`processor.max_cstate` 經常是 no-op，需 ZenStates-Linux / BIOS「Global C-state Control」「Power Supply Idle Control → Typical Current Idle」/ amd-disable-c6）。
  - 但不是人人有效：lucperneel 全參數仍每 1–2 週一發（已開 kernel Bugzilla **221909**）；klightspeed（**5950X**）追出 6.18+/7.1+ 的 `amd_pstate` 會忽略 `max_cstate`，最後**定頻**（`amd_pstate=disable` + performance governor + 關 boost）→ 11 天穩；Tavisco（5700G）C6 已關仍炸 → 關 **Cool'n'Quiet** 後穩。
- **Arch BBS #314983（5800X3D + RTX 4090）**：簽名位元級相同；遊戲負載觸發、偶爾開機 10 分鐘內。多變數環境（CO -30 降壓、nvidia 電源限制、mt7921e WiFi、GPU「fell off the bus」）；seth 診斷方向＝WiFi 晶片污染總線 / GPU 掉總線；**未解決**，串後段轉查 nvidia P-state/延遲問題。

### 對 yp-ai 的映射（2026-10-06 現況）

- C-states：只暴露 **POLL/C1/C2**（無 C3）；硬體 C6 需另驗（ZenStates）；未下任何 cstate 參數。
- 頻率管理：**amd-pstate-epp + performance governor + boost ON**（與 klightspeed 的穩定組合不同）。
- ASPM：全部 link 已 **Disabled/不支援** → `pcie_aspm=off` 對我們是 no-op。
- NVMe = Samsung PM983、WiFi = Intel AX200 → 這兩個「裝置 bug」線與我們無關。
- ppfeaturemask：現值 `0xFFF7FFFF`（`/etc/modprobe.d/99-amdgpu-overdrive.conf`）；gist 值 `0xFFF73FFF`（多清 bits14–15）。
- 我們的樣本多在**重載中**觸發（非純 idle）——與社群「idle 深睡」故事不完全吻合，仍可能是電源態轉換邊際。

### 候選試驗（未執行；等決策）

1. BIOS：**Global C-state Control 關** / Cool'n'Quiet 關（先驗 C6 是否還在作用）。
2. BIOS：**SoC 電壓裕度**（AMD Overclocking 模式 / 固定 ~1.10V）。
3. BIOS：**CO Positive +10**（僅在 1+2 不足時）。
4. cmdline 測試：`idle=nomwait`、`usbcore.autosuspend=-1`；`ppfeaturemask` 試 `0xFFF73FFF`。
5. 最後手段：定頻（`amd_pstate=disable` + 關 boost）——對算力機代價大。
6. memtest86+ 裸機全量（含 <4G/低區 pattern）。

### 來源連結

- https://www.reddit.com/r/pop_os/comments/1o4bleo/data_fabric_sync_flood_event/
- https://gist.github.com/eliottness/ded6bce8163689dc426732d0670c7a28
- https://bbs.archlinux.org/viewtopic.php?id=314983
- kernel Bugzilla: https://bugzilla.kernel.org/show_bug.cgi?id=221909
