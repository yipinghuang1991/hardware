# Commands for Stressing RAM

所有工具均已安裝於本機（Arch Linux, Ryzen 5800X3D, 32GB dual-rank Micron E-die @ 3733 MT/s）。

## Monitoring

即時監控 MCE / WHEA / hardware errors：

```bash
sudo dmesg -w | grep -iE 'mce|aer|hardware'
```

測試後檢查累積 MCE：

```bash
journalctl -k --grep=mce
```

確認記憶體速度與 training fallback：

```bash
sudo dmidecode -t memory | grep -E 'Speed|Type|Size'
```

## stress-ng

通用 CPU/RAM 壓力測試。`--cpu` 全核負載，`--vm` 記憶體壓力，`--tz` 顯示溫度區域。

```bash
# 全核 CPU 壓力，5 分鐘
stress-ng --cpu $(nproc) --timeout 5m --metrics-brief --tz

# 記憶體頻寬壓力（vm workers 寫滿 RAM）
stress-ng --vm 2 --vm-bytes 16G --vm-method flip --timeout 30m --metrics-brief

# 混合 CPU + 記憶體
stress-ng --cpu $(nproc) --vm 2 --vm-bytes 8G --timeout 30m --metrics-brief --tz
```

**用途：** 熱循環測試（配合 `cpu_thermal_cycle` 腳本）、快速 smoke test。
**不適用：** 精細記憶體 timing 驗證——用 mprime / stressapptest 替代。

## mprime (Prime95)

記憶體穩定性的 gold standard。torture test 模式 `-t` 有三種 FFT 大小：

```bash
# 互動模式（選 FFT 大小）
mprime -t

# Small FFTs（L1/L2/L3 cache，測	CPU/IMC）
# → 選 option 1

# Large FFTs（memory controller & RAM）
# → 選 option 2

# Blend（混合，預設）
# → 選 option 3
```

非互關閉（用於腳本自動化，配合 `cpu_thermal_cycle_mprime`）：

```bash
# 直接跑 torture test（預設 Small FFTs），背景執行
mprime -t &

# 搭配定時 kill（mprime 無 --timeout）
(sleep 300; kill $!) &
mprime -t
```

**驗證用途：**
- Stage 1 stability：Small FFTs 30 min（FCLK + primaries + SoC + ProcODT）
- Stage 2 stability：Large FFTs 1 hour（secondaries + tertiaries）
- Pass criteria：`results.txt` 中零 rounding error、零 FATAL ERROR

**版本：** v30.19, build 20, Linux64

## stressapptest

Google 開發的 user-space 記憶體壓力測試。直接測 memory cells、data line signal integrity、rank switching。

```bash
# 基礎：2 小時，32GB 覆蓋，高 CPU 負載模式
stressapptest -s 7200 -M 30000 -W

# 快速 smoke test：5 分鐘
stressapptest -s 300 -M 2600 -W

# 低 CPU 負載純記憶體測試（不加 -W）
stressapptest -s 3600 -M 30000

# 多執行緒 + verbose
stressapptest -s 7200 -M 30000 -W -m 16 -v 10
```

**用途：**
- Step 3 of validation protocol：2 小時全覆蓋
- Catches：cell-level corruption, data line SI, rank-to-rank switching（SD/DD timings）
- Pass criteria：exit code 0, no miscompare errors

**參數說明：**
- `-s` 秒數
- `-M` MB 測試容量（建議 ≤總 RAM − 系統保留）
- `-W` 高 CPU 負載模式（更激進的 memory copy）
- `-m` threads, `-i` invert threads, `-C` CPU stress threads
- `-v` verbosity 0-20

## OCCT

圖形化 + CLI 混合工具，支援 CPU/RAM/IMC 混合壓力。

```bash
# CLI 模式需要 GUI 環境（Qt），純 headless 不支援
# 推薦在桌面環境下執行

# 快速 RAM 測試（需 GUI）
occt
```

**用途：** 桌面環境下的額外交叉驗證。Headless 環境用 stressapptest / mprime 替代。

## y-cruncher

Pi 計算 benchmark，高負載壓力測試 IMC 和 FCLK。TM5 的 Linux 替代方案。

```bash
# 互動模式
y-cruncher

# 直接跑預設 benchmark（1b digits, all components）
y-cruncher benchmark -b

# 指定 Pi 位數
y-cruncher crunch 1e10 -o /tmp/pi_output
```

**用途：**
- TM5 Windows-only 時的 Linux 替代（ram_tuning.md Step 4）
- 特別擅長壓力測試 FCLK + IMC 交互
- 持續跑 30+ 分鐘作為 memory stability indicator

**版本：** v0.8.7 Build 9547-gcc

## memtester

Linux 原生 user-space 記憶體測試（若已安裝）。

```bash
# 測試 28GB，1 輪
sudo memtester 28G 1

# 測試 28GB，3 輪
sudo memtester 28G 3
```

**用途：** 輕量級替代方案。不如 mprime / stressapptest 激進，但適合快速 sanity check 後快速排錯。

---

## Quick Reference — Validation Protocol 對應

| Protocol Step | Tool | Command | Duration |
|---|---|---|---|
| Step 1 | Boot check | `dmidecode -t memory \| grep Speed` | POST |
| Step 2 | mprime Large FFTs | `mprime -t` → option 2 | 4 hr |
| Step 3 | stressapptest | `stressapptest -s 7200 -M 30000 -W` | 2 hr |
| Step 4 | TM5 (Windows) / y-cruncher (Linux) | `y-cruncher crunch 1e10` | 30+ min |
| Step 5 | dmesg monitoring | `journalctl -k --grep=mce` | Each test |
