# 06 — Join Windows 11 Client to the Domain

**Machine:** CLIENT (Windows 11 **Pro/Enterprise** — Home can't join a domain)

## Steps

1. Make sure client got an IP from DHCP (`ipconfig /all` → DNS = `192.168.1.2`).
2. `ping final.local` → must reply.
3. (Recommended) Rename PC first → restart.
4. `sysdm.cpl` → **Computer Name** → Change → **Domain**: `final.local`.
5. Enter a domain account (e.g. `FINAL\Administrator`) → "Welcome to the final.local domain" → Restart.
6. On login screen → **Other user** → log in with a domain user (e.g. `ahmed.hr`).
7. On PDC: move the computer object from **Computers** container to the right OU (so OU GPOs apply).

## PowerShell

```powershell
Add-Computer -DomainName final.local -Credential FINAL\Administrator -Restart
```

## Verify

```powershell
whoami            # final\ahmed.hr
echo %logonserver%   # (cmd) \\PDC or \\CORE
```

---

<div dir="rtl">

## ليه عملنا كده؟

- **Join Domain**: عشان الجهاز يبقى تحت إدارة السيرفر، فاليوزر يدخل بأكونت الدومين من أي جهاز، والـ GPOs تتطبق عليه، والصلاحيات تبقى مركزية.
- **Rename قبل الـ Join**: اسم الجهاز بيتسجل في الـ AD والـ DNS. لو غيرته بعد الـ Join ممكن يعمل مشاكل في الـ computer account.
- **Ping final.local الأول**: لو مردّش يبقى المشكلة في الـ DNS، والـ Join أكيد هيفشل. صلّح الـ DNS الأول.
- **Windows Home مابيعملش Join**: لازم Pro أو Enterprise.
- **نقل الجهاز لـ OU**: الـ container اللي اسمه Computers مش OU، فمينفعش تربط عليه GPO. عشان كده لازم تنقل الجهاز لـ OU.

</div>
