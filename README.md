# ninusd — คู่มือ Nintendo Switch 2 (Static Site)

เว็บไซต์นี้เป็น **ไฟล์ HTML เดียว** (`index.html`) ไม่มี build step, ไม่มี dependency
ดังนั้นบน Vercel ต้องรันแบบ **Static / Other** ไม่ใช่ Node.js

## สาเหตุของ error

```
[BUILD_FAILED] No entrypoint found in "/vercel/path0"
Set package.json "main" to a server file, or add one of: app.js, index.js, server.js ...
```

Vercel เข้าใจโปรเจกต์นี้ว่าเป็น **Node.js server** (เพราะเจอ `package.json` / ตั้ง Framework Preset
เป็น Node.js) แล้วหาไฟล์ server entrypoint ไม่เจอ แต่โปรเจกต์นี้เป็น static HTML
จึงไม่มี entrypoint — วิธีแก้คือทำให้ Vercel มองเห็นว่าเป็น static site

## โครงสร้างที่ถูกต้อง

```
ninusd/
├── index.html     ← ตัวเว็บ (ต้องชื่อ index.html และอยู่ที่ root)
├── vercel.json    ← ปิด build/install, เปิด cleanUrls
└── .gitignore
```

## สิ่งที่ต้องทำ

1. **ลบไฟล์ที่ไม่ใช่ static ออกจาก repo** (ถ้ามี): `package.json`, `package-lock.json`,
   `server.js`, `app.js`, โฟลเดอร์ `api/`, `node_modules/`
   (ถ้ายังมี `package.json` Vercel จะพยายามรัน Node builder อีก)

2. **ตรวจ Root Directory** ใน Vercel → Settings → General → Root Directory
   ต้องว่าง หรือ `/` (ถ้าไฟล์ HTML อยู่ในโฟลเดอร์ย่อย ให้ชี้ Root Directory ไปที่โฟลเดอร์นั้น)

3. **ตรวจ Build settings** ใน Vercel → Settings → Build & Deployment
   - Framework Preset: **Other**
   - Build Command: **ปิด (Override = off)**
   - Output Directory: **ปิด (Override = off)**
   - Install Command: **ปิด (Override = off)**

4. Push ขึ้น GitHub แล้ว Redeploy — **วิธีปลอดภัย (แนะนำ)**: clone repo เดิมมาแก้

```powershell
cd "D:\ผู้ใช้"
git clone https://github.com/mpalg1453-wq/ninusd.git ninusd-fix
cd ninusd-fix
git rm -r --cached . -q
Remove-Item -Recurse -Force * -Exclude .git -ErrorAction SilentlyContinue
Copy-Item "D:\ผู้ใช้\ninusah\*" . -Recurse -Force
git add -A
git commit -m "fix: deploy as static site (index.html) instead of node entrypoint"
git push origin main
```

**วิธีลัด** (ถ้า repo มีแค่ไฟล์เว็บนี้ ไม่มีอะไรต้องเก็บ):

```powershell
cd "D:\ผู้ใช้\ninusah"
git init
git add -A
git commit -m "fix: deploy static site (index.html) instead of node entrypoint"
git branch -M main
git remote add origin https://github.com/mpalg1453-wq/ninusd.git
git push -u origin main --force
```

> `--force` เขียนทับประวัติเดิมบน GitHub ทั้งหมด — ใช้เฉพาะเมื่อมั่นใจว่าไม่มีไฟล์อื่นตกหล่น

## ทดสอบในเครื่อง

```powershell
npx --yes serve .        # แล้วเปิด http://localhost:3000
```

หรือเปิด `index.html` ด้วยเบราว์เซอร์ได้เลย (ไฟล์เดียว ไม่ต้องมี server)
