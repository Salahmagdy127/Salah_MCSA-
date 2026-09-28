# 10 — Shared Folders & Advanced NTFS Permissions

**Machine:** PDC

## Steps

1. Create folders, e.g. `C:\Shares\HR`, `C:\Shares\Dev`, `C:\Shares\Public`.
2. **Share** — folder → Properties → Sharing → **Advanced Sharing** → ✅ Share this folder →
   **Permissions** → remove `Everyone` → add the department group (e.g. `HR-Group`) → **Full Control** (or Change).
3. **NTFS (Security tab)** — Properties → Security → Edit → add `HR-Group` → **Modify**.
4. **Prevent delete** — Security → **Advanced** → Add → Select principal `HR-Group` →
   Type: **Deny** → **Show advanced permissions** → check only:
   - ✅ Delete subfolders and files
   - ✅ Delete
   → OK.
5. Map the share to users via GPO → see [07 — Drive Maps](../07-Group-Policy/README.md#74-map-network-drive).

## PowerShell

```powershell
New-Item C:\Shares\HR -ItemType Directory
New-SmbShare -Name HR -Path C:\Shares\HR -FullAccess "FINAL\HR-Group"

# NTFS: Modify for HR-Group
icacls C:\Shares\HR /grant "FINAL\HR-Group:(OI)(CI)M"
# NTFS: Deny delete (D = Delete, DC = Delete child)
icacls C:\Shares\HR /deny "FINAL\HR-Group:(OI)(CI)(D,DC)"
```

## Verify

- From client (as HR user): `\\PDC\HR` → create a file ✔ → try to delete ✖ Access denied.
- From a Sales user: `\\PDC\HR` → Access denied.

---

<div dir="rtl">

## ليه عملنا كده؟

- **Share Permissions و NTFS Permissions**: فيه نوعين صلاحيات:
  - **Share** = بتتطبق لما تدخل على الفولدر **من الشبكة** بس.
  - **NTFS (Security)** = بتتطبق **دايماً**، من الشبكة أو من على الجهاز نفسه.
  - الصلاحية الفعلية = **الأقل** بين الاتنين (Most Restrictive).
  - الـ Best Practice: خلي الـ Share واسعة (Full Control للجروب)، والتحكم الحقيقي بالـ NTFS.
- **شيل Everyone**: عشان مش أي حد في الشبكة يدخل على فولدر الـ HR.
- **Deny Delete**: الموظف يقدر يضيف ويعدّل بس **مايمسحش**، وده بيحمي الملفات من المسح بالغلط أو بقصد.
- **ليه Deny صريح (Explicit Deny)؟**: الـ **Deny دايماً بيكسب الـ Allow**. فحتى لو اليوزر واخد Allow من جروب تاني، الـ Deny هيمنعه. بس **استخدمه بحذر**، لأن لو الأدمن نفسه في الجروب ده، هيتمنع هو كمان.

</div>
