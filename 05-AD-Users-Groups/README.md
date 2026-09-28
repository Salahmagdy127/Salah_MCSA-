# 05 — OUs, Users & Groups

**Machine:** PDC

## Steps

1. Tools → **Active Directory Users and Computers** (`dsa.msc`).
2. Right-click `final.local` → New → **Organizational Unit** → create:
   `HR`, `Sales`, `Dev`, `IT`, `Groups`.
3. In **Groups** OU → New → **Group** (Global / Security):
   `HR-Group`, `Sales-Group`, `Dev-Group`, `IT-Group`.
4. In each department OU → New → **User** → fill name + logon name → set password.
5. Open each group → **Members** → Add the users of that department.

## PowerShell

```powershell
$domain = "DC=final,DC=local"
"HR","Sales","Dev","IT","Groups" | ForEach-Object { New-ADOrganizationalUnit -Name $_ -Path $domain }

"HR","Sales","Dev","IT" | ForEach-Object {
    New-ADGroup -Name "$_-Group" -GroupScope Global -GroupCategory Security -Path "OU=Groups,$domain"
}

# Example user
$pw = Read-Host "Password" -AsSecureString
New-ADUser -Name "Ahmed HR" -SamAccountName ahmed.hr -UserPrincipalName ahmed.hr@final.local `
    -Path "OU=HR,$domain" -AccountPassword $pw -Enabled $true
Add-ADGroupMember -Identity HR-Group -Members ahmed.hr
```

## Verify

```powershell
Get-ADOrganizationalUnit -Filter * | Select Name
Get-ADGroupMember HR-Group
```

---

<div dir="rtl">

## ليه عملنا كده؟

- **OU (Organizational Unit)**: فولدر جوه الأكتيف دايركتوري بتنظّم بيه اليوزرز والأجهزة (كل قسم في OU). الأهم إنك **بتربط الـ GPO على الـ OU**، فتقدر تطبّق سياسة على قسم معين بس.
- **Group**: بتستخدمه عشان **الصلاحيات** (على فولدرات، طابعات...). القاعدة الذهبية: **الصلاحية تتدي للجروب، مش لليوزر**. لو موظف جديد دخل HR، تضيفه للجروب بس وخلاص، بدل ما تعدّل صلاحيات 10 فولدرات.
- **الفرق بين OU و Group**: الـ OU للتنظيم وتطبيق الـ GPO، والـ Group للصلاحيات.
- **OU مخصوص للـ Groups**: تنظيم أحسن، عشان الجروبات متبقاش مبعترة وسط اليوزرز.

</div>
