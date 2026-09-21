# jhk-pic

金禾康官網（QDM）首頁用的圖片，優化過的 WebP 版本。

- 來源：[`jinhekanghealth-beep/new-web`](https://github.com/jinhekanghealth-beep/new-web) 的 `JHKWEB_FINAL_0907/JHK_images/`（commit `a591c7e`，163 張）
- 資料夾結構、檔名跟來源一致，只是副檔名統一換成 `.webp`
- 桌機 banner（`02_Banner/AD0x_PC`）另外有 `-<寬度>w` 的尺寸版本，給 `srcset` 用（約 1440、1920、2880 寬，實際寬度挑成縮放後比例不失真的值）

## 網址怎麼用

透過 jsDelivr、**鎖定完整的 commit SHA** 引用（快取一年且內容不可變）：

```
https://cdn.jsdelivr.net/gh/Ludougan123234/jhk-pic@<完整40字元commit-sha>/JHK_images/<資料夾>/<檔名>.webp
```

- 不要用 `@main`：jsDelivr 會把分支網址快取 12 小時，換圖後會有新舊混雜
- 不要用短 SHA（例如 `@a1b2c3d`）：jsDelivr 會把它當成分支處理，一樣不會永久快取
- 換圖流程：先 push，再把 HTML 裡的 SHA 換成新的 commit

## 轉檔規則

| 項目 | 規則 |
|---|---|
| 寬度 | 在 10 種螢幕寬度（375–2560px）量出最大顯示寬度 × 2（Retina 桌機也清楚），不放大原圖；寬度會微調成縮放後長寬比跟原圖一致的值 |
| 網站沒用到的圖 | 寬度上限 1600px |
| JPG | WebP 品質 82；banner 用 85 |
| PNG（含透明） | 有損 q90 跟無損各轉一次，取檔案較小的，透明通道保留 |
| 本來就是 WebP 且不用縮 | 原檔照搬，不重新壓縮 |
| 色彩 | 來源全部是 sRGB，所以直接移除中繼資料（`-strip`） |

畫質：183 個檔案的 SSIM 全部 ≥ 0.968（1.0 = 跟縮圖後的原圖完全相同，0.95 以上肉眼看不出差異）。

大小：網站實際用到的 105 張，從 41.8 MB 降到 5.9 MB（這是全部尺寸版本的加總，單一裝置實際下載的量更少）。

## 新增圖片

1. 量這張圖在網頁上的最大顯示寬度（CSS px），乘以 2，就是輸出寬度
2. 轉檔：`magick input.jpg -resize <寬度>x -strip -quality 82 -define webp:method=6 output.webp`
3. push 後，把 HTML 裡 jsDelivr 網址的 SHA 換成新的 commit
