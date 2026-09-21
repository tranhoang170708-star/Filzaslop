# Hướng dẫn cài FilzaSlop patched trên iOS 26.4 (non-JB, free Apple ID)

## Yêu cầu
- Windows PC
- iPhone iOS 26.4
- Sideloadly (đã cài, có standalone iTunes + iCloud từ apple.com, không phải bản Microsoft Store)
- Free Apple ID
- Python 3 trên Windows (để chạy patch.py)

## Bước 1 — Clone repo
git clone https://github.com/0xjohnnydev/FilzaSlop
cd FilzaSlop

## Bước 2 — Patch source
python3 patch.py com.fox.filzaslop.G78256YMV8
(Thay G78256YMV8 bằng team ID của bạn — nó xuất hiện trong lỗi Sideloadly trước đó.
 Nếu không chắc, dùng com.fox.filzaslop)

## Bước 3 — Push lên GitHub của bạn
- Tạo repo mới trên github.com (private cũng được)
- git remote set-url origin https://github.com/USERNAME/FilzaSlop.git
- git add .
- git commit -m "patch for sideload"
- git push -u origin main

## Bước 4 — Chờ GitHub Actions build
- Vào repo → tab Actions → xem workflow "build" chạy
- Chờ ~3-5 phút
- Build xong → vào workflow run → mục Artifacts → tải "FilzaSlop-dylibs"
- Giải nén ra: FilzaSlop.dylib (và có thể FilzaApplySandboxExt.dylib)

## Bước 5 — Sửa IPA gốc
- Giải nén FilzaSlop-v1.2.0-unsigned.ipa bằng 7-Zip
- Vào Payload/FilzaSlop.app/
- Sửa Info.plist: CFBundleIdentifier = com.fox.filzaslop
  Dùng Python:
    import plistlib
    with open('Info.plist','rb') as f: d = plistlib.load(f)
    d['CFBundleIdentifier'] = 'com.fox.filzaslop'
    with open('Info.plist','wb') as f: plistlib.dump(d, f)
- Xoá thư mục PlugIns/ (nếu có)
- Thay FilzaSlop.dylib cũ bằng file mới build
- Thay FilzaApplySandboxExt.dylib cũ bằng file mới (nếu có)
- Thay FilzaApplySandboxExt.plist bằng bản đã patch (nếu nó nằm trong .app)
- Zip lại: chuột phải thư mục Payload → Send to → Compressed (zipped) folder
- Đổi tên .zip thành FilzaSlop-fox.ipa
- Kiểm tra: mở IPA bằng 7-Zip, phải thấy Payload/ là thư mục gốc

## Bước 6 — Sideload
- Mở Sideloadly
- Kéo FilzaSlop-fox.ipa vào
- Apple ID: tài khoản của bạn
- Bundle ID field: com.fox.filzaslop
- Bấm Start
- Chờ "Done"

## Bước 7 — Trust trên iPhone
- Settings → General → VPN & Device Management → Apple ID của bạn → Trust
- Nếu chưa bật: Settings → Privacy & Security → Developer Mode → ON → reboot

## Bước 8 — Mở app
- Tap FilzaSlop trên home screen
- Nếu thấy file browser hiện được /var/mobile/Containers/Data/Application/... của app khác → escape thành công
- Nếu hiện lỗi "MCM caller identity" → escape fail, patch chưa ăn

## Nếu fail ở bước 8
- Check lại patch.py đã sửa MCMFilzaIntegration.m chưa (grep "MCMSignedCodeIdentifier" → phải thấy return kRequiredIdentifier)
- Check bundle ID trong plist khớp với cái Sideloadly ký
- Vào tab Issues của repo gốc, tìm issue về MCM để tham khảo

## Giới hạn đã biết
- Cert free Apple ID hết hạn sau 7 ngày → phải sideload lại
- Một số build iOS 26.4 có thể không tương thích ngay cả khi patch đúng
- Escape path dựa trên bug có thể bị Apple fix trong 26.5+
