# H2+ 核心升級進度

更新：2026-10-22（Asia/Taipei）。

## 最新追測：十分鐘混合負載（2026-10-22）

保持已安裝版本不變，跑 600 秒雙向 HTTP 加 SPI 讀取：兩向 ping 為
2,894/2,995、2,897/2,995，丟包 3.37%／3.27%，Mac 到路由器對照無丟包。
本地 HTTPS 80/80，反向代理 68/80（10 次 TLS、2 次 CONNECT 期限逾時）；
直接連外對照 79/80，另一次是連線被重設。所有 HTTPS 請求在負載期間開始，
最後約在第 474 秒；其後仍持續大檔負載與 ping 到測試結束。

韌體傳送失敗 1,074/150,701（0.71%），全為重試耗盡。最高溫約 50°C，
主機佇列最長 850 ms、韌體傳送前等待最長約 841 ms。期間有一次 SDIO 資料錯誤與一次
未觀察到 IRQ 的 watchdog 恢復；未觀察到韌體例外或整段斷線，沒有佇列到期。
停止負載後，資料雜湊一致，反向代理重測 20/20 成功。

此結果比前一輪 200 秒測試差，不能宣稱高負載已穩定；本輪未改設定或寫 Flash。
測試服務已清除，正式代理保持運行。詳見 kernel/mainline/XR819-RETRY.md 末段。

## 最新：XR819 傳送重試上限調整（2026-10-22）

已確認上次失聯是使用者拔線關機；今天重新上電後，原有核心、Wi-Fi 及代理正常。
新診斷模組已上板，第一輪 1,250 次傳送失敗全是韌體回報重試耗盡，沒有 TTL 到期
或連線遺失。低 MCS 限制與 RTS/CTS 未證明改善，均已清除。

將韌體傳送重試上限由 7 提高至 15，保留原速率選擇與 policy 終止機制。
交替 A/B/A/B 短測的失敗確認比例為 11.67%／0.43%／2.40%／0.25%；無線環境
未受控制，不能保證全部差異由補丁造成。200 秒網路加 SPI 讀取長測的雙向 ping
收到 988/995、993/995；本地 HTTPS 40/40，反向代理 37/40，仍有三次期限逾時。
目前是改善丟包，尚未根治 TLS 逾時，也不代表長期穩定。

已驗證新完整 SPI 備份，再僅更新 rootfs 分割區；mtd 讀回驗證成功，其他五個
分割區雜湊不變。重啟確認載入新模組，Wi-Fi 與反向代理恢復，HTTPS 重測 20/20，
1 MiB 雙向資料 SHA-256 一致。核心仍是 6.12.113、韌體仍是 .52.58。此版尚未再做
實體斷電冷開機。來源的重試／診斷選項仍預設關閉；板子採用已測試的 opt-in 版本。

詳細數據、選項、雜湊與限制見 kernel/mainline/XR819-RETRY.md。

## 前一輪重負載追查（2026-10-21）

已分離網路／SPI 負載：閒置與純 SPI 的反向代理均 15/15 HTTPS 成功；純網路及
網路加 SPI 均為 11/15，純網路期間 missed interrupt 恢復計數為零。Mac 到路由器
無丟包，板子有大量 Wi-Fi TX 重傳／失敗；CPU 未飽和。不能把 TLS 逾時全歸因於 IRQ。
降低 TCP 佇列上限及 SDIO 時脈均未證明改善，已還原。較低 TX 速率試驗的 HTTPS
為 15/15，但收尾前 SSH 與 USB 串口同時失聯，缺最後狀態，不能宣稱已修復。
速率限制僅 RAM 設定，恢復同次開機後需清除；重啟會回復。

新的主機／韌體 TX 佇列延遲診斷補丁已交叉編譯及反向套用驗證，預設關閉，
尚未上板。此次沒有寫入 Flash；等待恢復供電／串口連線後才能繼續實測。
詳細樣本、限制及後續步驟見 kernel/mainline/XR819-LOAD.md。

## 最新 XR819 調整

2026-10-21 已保存新版 Wi-Fi 韌體 `.52.58`、上游 HT RX 格式修正 #34，以及
IRQ/輪詢競爭診斷與警告分類（只以實際 IRQ 計數變化判定，不把 TX 喚醒當 IRQ）。rootfs 寫入讀回驗證通過，其他五個分割區雜湊
不變；自動重啟已確認載入新韌體/模組並恢復原 Wi-Fi/IP。核心仍為 6.12.113。

雙向各 300 次 ping 的短樣本中，接收端從原韌體/GPIO 的 284/300 改為
新韌體/#34/GPIO 的 298/300；順序測試不能保證是因果，亦非完全無丟包。
重負載加 Flash 備份時仍有丟包及一次 TLS deadline 逾時；事後 20 次 HTTPS
代理重測全數成功。原生 SDIO 與停用 HT 均未證明能根治，所以沒有部署。
#31 也未啟用，舊版本完整備份保留。詳細數據見 kernel/mainline/XR819-IRQ.md。

## 結果

Orange Pi Zero H2+ 已從 Linux 3.4.113+ 升級至 **6.12.113-mizy-h2plus**，
寫入 16 MiB SPI，保留 miZy 使用者環境、XR819 Wi-Fi、HTTPS CONNECT 代理
及反向 SSH。沒有安裝完整 Armbian，也尚未安裝 Tailscale。

最終修正後已完成不需手動介入的軟體重啟測試：

- `uname -r`：`6.12.113-mizy-h2plus`，四核心、512 MiB RAM。
- XR819 自動連回 保存的 SSID，WPA2-PSK/CCMP，狀態 `COMPLETED`。
- 原 Wi-Fi MAC 和 DHCP 位址保留；DNS 可解析外網。
- `/dev/net/tun` 存在。
- 反向 SSH 自動重連，經 VPS 代理實測 `CONNECT=200 HTTPS=200`。
- kernel、rootfs、DTB、boot script 寫入後均通過 `mtd verify`；最後補寫
  rootfs 與 script 後再度驗證成功。U-Boot 與 user 分割區 SHA-256 維持一致。
- 最終修正後尚未另外要求實體斷電冷開機測試；以上為自動軟體重啟的結果。

## SPI 配置

| 分割區 | 起點 | 大小 |
|---|---|---|
| uboot | `0x000000` | 512 KiB |
| script | `0x080000` | 64 KiB |
| dtb | `0x090000` | 64 KiB |
| kernel | `0x0a0000` | 4 MiB |
| initrd | `0x4a0000` | `0x9e0000` bytes |
| user | `0xe80000` | 1.5 MiB |

uImage 為 3,824,003 bytes；miZy uInitrd 為 7,352,384 bytes。
UART 保持 115200/8N1，U-Boot 保持 2017.01-hyphop，沒有偵測到 TF 卡。

## 必要相容修正

- Linux 6.12.113 與 fifteenhex/xradio 固定來源、雜湊驗證及交叉編譯。
- JFFS2 啟用 LZO，SPI 使用原本 64 KiB 擦除區塊。
- 原設定在掛載 OverlayFS 前匯入 upper，使用獨立 workdir。
- Legacy squashfs initrd 使用 chroot 交接；RAM cpio bootstrap 使用 switch_root。
- 暫存 user 掛載點放在可寫的 `/mnt/user`。
- 停用舊 Wi-Fi hotplug 重啟流程，明確載入 WPA2 加密模組。
- DTB 預先配置裝置原 MAC，防止舊 U-Boot 用 ethernet1 alias 蓋掉。
- `initrd_high=0xffffffff` 保留 rootfs 在安全的 `0x44000000`，避免舊 U-Boot
  把 7 MiB rootfs 搬到 `0x41000000` 以下而空間不足。
- 停用舊 CPU/DRAM 暫存器直接寫入，改由 mainline 驅動管理。

## 備份、紀錄及重建

最新升級前完整 SPI 備份 SHA-256：
`67df5fe78919934b2a50008307a90666a6f18700c7c806bf70067685bb47f213`。

本機忽略的 `build/artifacts/` 保存升級前後備份、候選映像及實機記錄：
`spi-autoboot-final.log`、`flash-final-fix.log`。備份包含憑證與私鑰，不可上傳 git。
舊版備份另在同層 `artifacts/mizy-backup-20261021/`。

核心來源及建置流程位於 kernel 子模組的 `mainline/`，分支為
`upgrade/h2plus-linux-6.12`。詳細流程見該目錄 README。

先前 legacy kexec 未能啟動新核心；實機驗證採 U-Boot legacy uImage/uInitrd，
原 U-Boot 不支援 FIT。Armbian 只曾作為驅動方案參考。

## XR819 穩定性查核

2026-10-21 查核：現用 xradio commit 是上游最新合併版本，與 Armbian 相同。
運作約 37 分鐘仍保持連線，但日誌累計 67 次 missed interrupt。20 次 gateway
ping 無丟包，不能據此宣稱長期穩定。未合併的 ARP、HT 接收和韌體修正
尚未安裝；詳見 kernel/mainline/XR819-STATUS.md。


## XR819 修正實測

PR #31（ARP cipher）及 #34（HT RX metadata）已編譯並透過串口在 RAM 載入測試。
未寫入 Flash。短測各版本的代理均正常；加載 16 MiB 雙向 SSH 傳輸及 20 次
HTTPS 請求的兩分鐘測試中，修正版 ping 遺失 7/120，原版對照遺失 2/120。
資料 SHA-256 全部一致，兩版本 HTTPS 都為 20/20 成功。無法證明穩定性改善，
因此目前回到原版驅動，沒有永久套用。詳細數據與限制見 kernel/mainline/XR819-STATUS.md。
