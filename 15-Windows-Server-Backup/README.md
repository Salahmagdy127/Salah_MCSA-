# 15 — Windows Server Backup (Scheduled, to Network Share)

**Machines:** PDC (backed up) → CORE (stores the backup)

## Steps

1. **On CORE** — create a share for backups:
   ```powershell
   New-Item C:\Backup -ItemType Directory
   New-SmbShare -Name Backup -Path C:\Backup -FullAccess "FINAL\Domain Admins","FINAL\PDC$"
   ```
2. **On PDC** — Add Roles and Features → Features → ✅ **Windows Server Backup** → Install.
3. Tools → **Windows Server Backup** → Local Backup → **Backup Schedule...**:
   - Backup configuration: **Full server (recommended)**
   - Time: **Once a day** at **11:00 PM**
   - Destination: **Back up to a shared network folder** → `\\CORE\Backup`
   - Enter credentials of an account with write access → Finish.

## PowerShell

```powershell
Install-WindowsFeature Windows-Server-Backup

# One-off full backup to the share (test)
wbadmin start backup -backupTarget:\\CORE\Backup -allCritical -vssFull -quiet
```

## Verify

- Windows Server Backup console → **Scheduled** shows next backup 11:00 PM.
- Run **Backup Once** to test → files appear in `\\CORE\Backup\WindowsImageBackup\PDC`.
- `wbadmin get versions -backupTarget:\\CORE\Backup`

---

<div dir="rtl">

## ليه عملنا كده؟

- **Backup**: أي سيرفر ممكن الهارد بتاعه يبوظ، أو ييجي فيروس (Ransomware)، أو حد يمسح حاجة. الـ backup هو آخر خط دفاع.
- **Full Server**: بياخد كل حاجة، الـ OS والـ System State (يعني قاعدة بيانات الـ AD) والداتا، فتقدر ترجّع السيرفر كله (Bare Metal Recovery).
- **الساعة 11 بالليل**: بعد مواعيد الشغل، عشان الـ backup مايبطّأش السيرفر والناس شغالة عليه.
- **على سيرفر تاني**: لو حطيت الـ backup على نفس السيرفر وهو وقع، **الـ backup هيضيع معاه**. عشان كده دايماً يبقى على مكان منفصل.
- **خد بالك**: الـ backup على network share **بيعمل overwrite** على النسخة القديمة كل مرة، يعني مفيش غير آخر نسخة بس. في الشغل الحقيقي استخدم ديسك مخصوص للـ backup، أو حلول زي Veeam، وطبّق قاعدة **3-2-1**: ٣ نسخ، على نوعين تخزين مختلفين، ونسخة منهم بره الشركة.

</div>
