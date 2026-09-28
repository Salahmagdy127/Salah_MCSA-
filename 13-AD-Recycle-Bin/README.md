# 13 — Active Directory Recycle Bin

**Machine:** PDC

## Steps

1. Tools → **Active Directory Administrative Center** (`dsac.exe`).
2. Click `final (local)` in the left pane → right side **Tasks** → **Enable Recycle Bin** → OK → OK.
3. Press **F5** (refresh) → a new container **Deleted Objects** appears.
4. **Test**: delete a test user → go to **Deleted Objects** → right-click user → **Restore**.

## PowerShell

```powershell
Enable-ADOptionalFeature -Identity "Recycle Bin Feature" -Scope ForestOrConfigurationSet -Target final.local

# Restore a deleted user
Get-ADObject -Filter 'samAccountName -eq "ahmed.hr"' -IncludeDeletedObjects | Restore-ADObject
```

## Verify

```powershell
Get-ADOptionalFeature -Filter * | Select Name, EnabledScopes
```

---

<div dir="rtl">

## ليه عملنا كده؟

- **من غير Recycle Bin**: لو مسحت يوزر بالغلط، هتعمله من الأول، وهو هيرجع **بـ SID جديد**، فهيضيع منه كل الجروبات والصلاحيات اللي كانت عنده. أو هتضطر تعمل restore من الـ backup بالـ DSRM، ودي عملية طويلة.
- **مع الـ Recycle Bin**: بتعمل Restore بكليك، واليوزر بيرجع **بكل حاجة**: نفس الـ SID والجروبات والـ attributes.
- **مهم تعرفه**:
  - مرة ما تفعّله **مش هتقدر تقفله تاني** (irreversible).
  - محتاج الـ Forest Functional Level يكون **2008 R2 أو أعلى**.
  - اتفعّله **قبل** ما تحتاجه، لأن الحاجة اللي اتمسحت قبل التفعيل مش هتقدر ترجعها بيه.

</div>
