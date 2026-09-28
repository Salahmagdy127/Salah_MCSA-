# 11 — FSRM: File Screening & Quota

**Machine:** PDC

## Steps

1. Add Roles and Features → **File and Storage Services → File and iSCSI Services** →
   ✅ **File Server Resource Manager** → Install.
2. Tools → **File Server Resource Manager**.
3. **Block audio/video/exe** — File Screening Management → **File Screens** → Create File Screen:
   - Path: `C:\Shares\HR` → Derive from template: **Block Audio and Video Files** → Create.
   - Repeat with template **Block Executable Files** (or use Custom properties and check both file groups).
   - Repeat for `C:\Shares\Dev`.
4. **Quota 2 GB** — Quota Management → **Quotas** → Create Quota:
   - Path: `C:\Shares\Public` → Define custom quota properties → Limit **2 GB** → **Hard quota** → Create.

## PowerShell

```powershell
Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools

New-FsrmFileScreen -Path C:\Shares\HR  -Template "Block Audio and Video Files"
New-FsrmFileScreen -Path C:\Shares\Dev -IncludeGroup "Audio and Video Files","Executable Files" -Active

New-FsrmQuota -Path C:\Shares\Public -Size 2GB    # hard quota by default
```

## Verify

- Copy an `.mp3` / `.mp4` / `.exe` into `\\PDC\HR` → **Access denied**.
- Public folder properties (from client) shows 2 GB size.

---

<div dir="rtl">

## ليه عملنا كده؟

- **FSRM**: أداة بتتحكم في **نوع** و**حجم** الملفات اللي تتحط على السيرفر.
- **منع الأغاني والفيديوهات**: عشان الموظفين مايملوش مساحة السيرفر بحاجات شخصية، وده بيوفّر مساحة ووقت الـ backup.
- **منع الـ exe**: حماية من الفيروسات والبرامج اللي ممكن تتشغّل من على فولدر مشترك وتنتشر.
- **Quota 2 GB**: فولدر الـ Public الكل بيكتب فيه، فلازم حد أقصى عشان حد واحد مايملاش الديسك كله.
- **Hard و Soft Quota**:
  - **Hard** = بيمنع فعلاً أول ما توصل للحد.
  - **Soft** = بيبعت تحذير بس (للمراقبة).
- **على فكرة**: الـ File Screen بيعتمد على **الامتداد** (extension)، يعني لو حد غيّر `.mp4` لـ `.txt` هيعدّي.

</div>
