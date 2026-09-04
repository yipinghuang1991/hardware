# R9700 VRAM OC — ECC/RAS 偵測 & 穩定性 SOP

硬體: AMD Radeon AI PRO R9700 32GB (gfx1201 / Navi 48, 256-bit GDDR6, 20 Gbps stock)
日期: 2026-09-04

## 關鍵事實: R9700 有 Memory ECC (Linux Only)

與消費級 9070 XT (同 die, 無 ECC) 不同 — AMD 官方 spec 明列:
**Memory ECC Support: Yes (Linux Only)**

後果: VRAM 超頻過頭時不是只有 crash 或 silent corruption 兩種結局 —
ECC 會先嘗試修正 (correctable error, ce), 修正有成本 → **t/s 下降**。
這讓「t/s 監測」在 R9700 上是有意義的穩定度指標 (Gemini 說法在此卡成立)。

## RAS sysfs 節點 (Gaming PC, card1)

```
/sys/class/drm/card1/device/ras/
├── umc_err_count        # ue: uncorrectable / ce: correctable / de: deferred
├── event_state          # Fatal Error / Poison 計數
├── gpu_vram_bad_pages   # 壞頁清單 (空 = 無壞頁)
├── features             # 0x101 = UMC RAS 支援
└── version / schema
```

檢查指令 (免 root, 唯讀):

```bash
cat /sys/class/drm/card1/device/ras/umc_err_count
# 預期輸出: ue: 0 / ce: 0 / de: 0
```

2026-09-04 基線: **ue:0 ce:0 de:0**, event_state 全 0, 無壞頁。

## VRAM OC 穩定性 SOP — 三層偵測

| 層級 | 訊號 | 意義 | 動作 |
| ---- | ---- | ---- | ---- |
| 1 | `ce` count 開始增加 | ECC 修正活動 = 已到/超過穩定邊界, 效能隱形損失 | **退一檔** (或判定此為極限) |
| 2 | `ue` > 0 / crash / hang / driver reset | 已過頭 | 退一檔以上, 檢查 dmesg |
| 3 | LiveBench accuracy 掉 | 最終驗證 (含未被 ECC 抓到的 corruption) | 長時間跑分確認 |

**誤判陷阱**: 只看「沒 crash」會把 ECC 正在擦屁股的狀態當成穩 —
那時 t/s 已掉, 不是免費的 OC。

## 測試流程

1. 固定 cap (如 165W) + VDDGFX_OFFSET (記錄當下值)
2. MCLK 每步 +50 (effective; sysfs actual = /2)
3. 跑真實負載 (LiveBench / llama-bench) 10-30 min
4. 比對前後 `umc_err_count` — ce 不動 = 乾淨, 可繼續; ce 增加 = 極限
5. crash 邊界往下退一檔即 bin 極限

## 2026-09-04 測試數據 (165-180W 區間)

MCLK effective vs core (cap sweep, LiveBench 負載):

| Cap | core med (trim) |
| --- | --- |
| 150W | 2015 |
| 160W | 2160 |
| 165W | 2306 |
| 170W | 2390 |
| 175W | 2502 |
| 180W | 2524 |
| 185W | 2610 |
| 190W | 2633 |

VRAM bin 探索 (cap 165W):

| MCLK (eff) | 結果 |
| --- | --- |
| 2668 | 穩 (30min watchdog, 0 crash) |
| 2718 | 穩 |
| 2742 | 測試中 |
| 2768 | crash @ −100mV |
| 2800 | crash @ −100mV |

**電壓 × MCLK 交互發現**: −100mV + MCLK 2668 長時間跑 crash (短 sweep 抓不到);
0mV + 2668 跑 30min 完全穩。MCLK OC 吃掉電壓裕度 — 高 MCLK 時 undervolt 要淺。

## 待辦 / 下一步

- [ ] 2742 跑穩後確認 `umc_err_count` ce 仍為 0
- [ ] 確認 2742 之後的 bin 極限點
- [ ] 定案組合: cap / MCLK / VDDGFX_OFFSET 三維最佳化
