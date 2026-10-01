# jhk-pic

金禾康官網（QDM）首頁用的圖片，優化過的 WebP 版本。

## 資料夾

| 資料夾 | 內容 | 首頁現在用的 |
|---|---|---|
| `jhk-website/`（2026-10-01） | 依客戶的圖片對照表（`jhk-qdm-image-mapping`）命名的 135 張，英文 kebab-case 檔名 | ✅ |
| `JHK_images/`（2026-09-21） | 舊版，中文檔名，來源是 [`jinhekanghealth-beep/new-web`](https://github.com/jinhekanghealth-beep/new-web) 的 `JHKWEB_FINAL_0907/JHK_images/`（commit `a591c7e`，163 張） | 只剩 global.css 的 3 張背景、global.script.js 的浮動列與備用影片縮圖、QDM 頁尾（這些不在首頁區塊裡） |

- 檔名、子資料夾跟來源一致，副檔名換成 `.webp`
- 桌機 banner 另外有 `-<寬度>w` 的尺寸版本，給 `srcset` 用（約 1440、1920 寬，實際寬度挑成縮放後比例不失真的值）
- 舊資料夾不要刪：已經發出去的網址鎖的是舊 commit，刪了也還能用，但留著比較好查

### `jhk-website/` 怎麼來的（2026-10-01）

客戶給了 135 張圖（`jhk-website/<區塊>/<檔名>`，共 32.0 MB）。轉完 9.1 MB（含 banner 尺寸版本）。

- 98 張跟 `JHK_images/` 的來源像素完全相同（81 張檔案一模一樣，17 張是客戶把 WebP 無損轉成 PNG）。其中 94 張直接沿用 `JHK_images/` 裡轉好的 WebP，避免二次壓縮
- 另外 4 張（banner AD08、AD11 的桌機與手機）以前網站沒用到（桌機版當時套了 1600 上限），現在上了 banner，用 banner 規則重新轉
- 桌機 banner AD01–AD07：客戶重新匯出成 2340 寬（AD04 是新設計），重新轉檔
- 證照 35：客戶為了壓到 1 MB 以下重新存過，重新轉檔
- 新聞影片縮圖 29 張（`08-news/news-video-thumbnail-*.jpg`）：就是 YouTube 的 `hqdefault.jpg`。轉 WebP 反而變大（543 KB → 722 KB），所以**保留原本的 JPG**
- 對照表沒有 #45（熱銷商品 PJ_07 活力錠），首頁也已經拿掉

## 網址怎麼用

透過 jsDelivr、**鎖定完整的 commit SHA** 引用（快取一年且內容不可變）：

```
https://cdn.jsdelivr.net/gh/Ludougan123234/jhk-pic@<完整40字元commit-sha>/jhk-website/<資料夾>/<檔名>.webp
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
| 轉出來比原檔大 | 保留原檔（例如 YouTube 縮圖 JPG） |
| 換圖時跟上一版來源像素相同 | 直接沿用上一版轉好的檔案 |
| 色彩 | 來源全部是 sRGB，所以直接移除中繼資料（`-strip`） |

畫質：183 個檔案的 SSIM 全部 ≥ 0.968（1.0 = 跟縮圖後的原圖完全相同，0.95 以上肉眼看不出差異）。

大小：網站實際用到的 105 張，從 41.8 MB 降到 5.9 MB（這是全部尺寸版本的加總，單一裝置實際下載的量更少）。

## 轉檔指令

這個 repo 裡的檔案就是用下面這組指令轉出來的。用同版本的 ImageMagick 重跑，會得到位元組完全相同的檔案（bash 和 zsh 都驗證過）。

### 需要的工具

- ImageMagick 7（本 repo 用 7.1.1-47）：`brew install imagemagick`
- python3（只用來算寬度）

### 轉檔函式

整段貼進終端機（或放進 `~/.zshrc`）就能用：

```bash
# 用法：jhk_webp <來源圖> <輸出.webp> <目標寬度px> [JPG品質，預設 82；banner 用 85]
#   目標寬度 = 圖片在網頁上最大顯示寬度（CSS px）× 2
#   實際寬度會在「目標寬度 ~ 目標寬度×1.03」之間挑一個縮放後長寬比最不失真的值；目標寬度 ≥ 原圖寬度就不縮
jhk_webp() {
  local src="$1" out="$2" target="$3" q="${4:-82}" W H size=""
  read W H < <(magick identify -format '%w %h\n' "${src}[0]")
  if [ "$target" -lt "$W" ]; then
    size=$(python3 -c "
import math
W, H, t = $W, $H, $target
w = min(range(t, min(math.ceil(t * 1.03), W) + 1), key=lambda w: (abs(w * H / W - round(w * H / W)) / w, w))
print(f'{w}x{round(w * H / W)}!' if w < W else '')")
  fi
  local resize=(); [ -n "$size" ] && resize=(-resize "$size")
  case "${src##*.}" in
    png|PNG)   # 有損 q90 跟無損各轉一次，留檔案小的（透明通道都會保留）
      magick "${src}[0]" "${resize[@]}" -strip -quality 90 -define webp:alpha-quality=100 -define webp:method=6 "$out.lossy.webp"
      magick "${src}[0]" "${resize[@]}" -strip -define webp:lossless=true -define webp:method=6 "$out.lossless.webp"
      if [ "$(wc -c < "$out.lossy.webp")" -le "$(wc -c < "$out.lossless.webp")" ]; then
        mv "$out.lossy.webp" "$out"; rm "$out.lossless.webp"
      else
        mv "$out.lossless.webp" "$out"; rm "$out.lossy.webp"
      fi ;;
    webp|WEBP) # 已經是 WebP：不用縮就原檔照搬，避免二次壓縮
      if [ -z "$size" ]; then cp "$src" "$out"
      else magick "${src}[0]" "${resize[@]}" -strip -quality 85 -define webp:method=6 "$out"; fi ;;
    *)         # JPG
      magick "${src}[0]" "${resize[@]}" -strip -quality "$q" -define webp:method=6 "$out" ;;
  esac
  echo "$out  $(magick identify -format '%wx%h' "$out")  $(( $(wc -c < "$out") / 1024 ))KB"
}
```

### 範例

```bash
# 一般照片（證照在網頁上最大顯示約 222px 寬 → ×2 ≈ 443，取整到 450）
jhk_webp 07-certificates/certificate-35.jpg jhk-website/07-certificates/certificate-35.webp 450

# 透明 PNG
jhk_webp 01-navigation/navigation-phone-button.png jhk-website/01-navigation/navigation-phone-button.webp 600

# banner（品質 85）：主檔 + srcset 用的尺寸
#   banner 最寬顯示 1920px → 目標 3840；原圖只有 2340 寬的話就維持原寬
#   srcset 尺寸：1440、1920，原圖比 2880 寬才加 2880
jhk_webp 02-banner/banner-01-desktop.jpg jhk-website/02-banner/banner-01-desktop.webp 3840 85
for w in 1440 1920; do jhk_webp 02-banner/banner-01-desktop.jpg /tmp/v.webp $w 85; done
# ↑ 每個尺寸依輸出顯示的實際寬度改名，例如 1484x657 → banner-01-desktop-1484w.webp，
#   srcset 的 w 描述符也要填實際寬度（1484w），不是 1440w
```

### 函式實際執行的指令

以 `-resize 1480x655!` 為例（`!` 代表照指定寬高縮放，寬高已經先算成跟原圖同比例）：

```bash
# JPG（banner 把 82 改成 85）
magick "input.jpg[0]" -resize "1480x655!" -strip -quality 82 -define webp:method=6 output.webp

# PNG：兩種都轉，留檔案小的
magick "input.png[0]" -resize "1480x655!" -strip -quality 90 -define webp:alpha-quality=100 -define webp:method=6 output.lossy.webp
magick "input.png[0]" -resize "1480x655!" -strip -define webp:lossless=true -define webp:method=6 output.lossless.webp

# 需要縮的 WebP
magick "input.webp[0]" -resize "1480x655!" -strip -quality 85 -define webp:method=6 output.webp
```

注意事項：
- `"檔名[0]"` 代表只取第一個影格。一定要加引號：zsh 會把沒加引號的 `$src[0]` 當成陣列下標，展開成空字串，magick 就會卡住等 stdin
- 不需要縮圖時，拿掉 `-resize ...` 那一段就好

### 目標寬度怎麼量

用 Chrome DevTools 的 Elements 面板，滑鼠停在圖片上會顯示 *Rendered size*。
在手機（375–412）、平板（768、1024）、桌機（1366、1440、1920）幾種寬度各看一次，取最大的 CSS 寬度 × 2。
背景圖（`background-size: cover`）要看實際畫出來的圖片寬度，不是元素寬度。

## 新增或替換圖片

1. 用上面的 `jhk_webp` 轉檔，放到 `jhk-website/` 對應的資料夾（檔名照客戶對照表的英文檔名，副檔名改 `.webp`）
2. commit、push
3. 把 QDM 裡 jsDelivr 網址的 SHA 全部換成新的 commit SHA（要用完整 40 字元）
