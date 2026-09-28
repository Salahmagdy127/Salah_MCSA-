# 07 — Group Policy (GPO)

**Machine:** PDC → Tools → **Group Policy Management** (`gpmc.msc`)

---

## 7.1 Password & Lockout Policy (Default Domain Policy)

Right-click **Default Domain Policy** → Edit →
`Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies`

| Policy | Value |
|---|---|
| Password Policy → Maximum password age | **60 days** |
| Password Policy → Minimum password length | **6** characters |
| Password Policy → Password must meet complexity requirements | **Enabled** |
| Password Policy → Enforce password history | **3** passwords |
| Account Lockout → Account lockout threshold | **4** invalid attempts |
| Account Lockout → Account lockout duration | **60 minutes** |
| Account Lockout → Reset account lockout counter after | 60 minutes |

## 7.2 Enable Remote Desktop for all computers

New GPO (e.g. `Enable-RDP`) linked to the domain →
`Computer Configuration > Policies > Administrative Templates`:

- `Windows Components > Remote Desktop Services > Remote Desktop Session Host > Connections`
  → **Allow users to connect remotely by using Remote Desktop Services** = Enabled
- `Network > Network Connections > Windows Defender Firewall > Domain Profile`
  → **Windows Defender Firewall: Allow inbound Remote Desktop exceptions** = Enabled

## 7.3 Restrictions (Task Manager, Control Panel, USB)

New GPO (e.g. `User-Restrictions`) linked to the department OUs:

| Setting | Path |
|---|---|
| Remove Task Manager = Enabled | `User Config > Policies > Admin Templates > System > Ctrl+Alt+Del Options` |
| Prohibit access to Control Panel and PC settings = Enabled | `User Config > Policies > Admin Templates > Control Panel` |
| All Removable Storage classes: Deny all access = Enabled | `Computer (or User) Config > Policies > Admin Templates > System > Removable Storage Access` |

## 7.4 Map Network Drive

GPO → `User Configuration > Preferences > Windows Settings > Drive Maps` → New → Mapped Drive:
- Action: **Create** (or Update) · Location: `\\PDC\HR` · Drive letter: e.g. `H:`
- (Optional) **Common → Item-level targeting** → Security Group = `HR-Group`

## Apply & Verify

```powershell
gpupdate /force          # on client
gpresult /r              # which GPOs applied
rsop.msc                 # GUI result
```

---

<div dir="rtl">

## ليه عملنا كده؟

- **GPO**: بدل ما تظبط إعدادات كل جهاز بإيدك، بتعمل سياسة واحدة من السيرفر وتتطبق على مئات الأجهزة أوتوماتيك.
- **الباسورد في Default Domain Policy**: سياسة الباسورد للأكونتات الدومين **مابتشتغلش غير على مستوى الدومين**. لو حطيتها في GPO مربوط على OU، مش هتأثر على أكونتات الدومين. (لو عايز سياسة مختلفة لجروب معين، في حاجة اسمها Fine-Grained Password Policy.)
- **Complexity + Length + History**: عشان الموظفين مايحطوش باسوردات سهلة زي `123456`، ولا يرجعوا لنفس الباسورد القديم.
- **Lockout بعد 4 محاولات**: حماية من الـ **Brute Force**، أي حد بيجرّب باسوردات كتير ورا بعض.
- **RDP بالـ GPO**: فريق الـ IT يقدر يدخل على أي جهاز عن بُعد من غير ما يقوم من مكانه. ولازم تفتح الفايروال كمان، وإلا الإعداد لوحده مش هيكفي.
- **منع Task Manager / Control Panel**: عشان اليوزر العادي مايقفلش برامج الحماية ولا يغيّر إعدادات الجهاز.
- **منع الـ USB**: حماية من **تسريب الداتا** (Data Leakage) ومن الفيروسات اللي بتيجي على الفلاشات.
- **Drive Maps**: الموظف يلاقي فولدر القسم بتاعه ظاهر كـ drive (زي `H:`) أول ما يعمل login، من غير ما يعرف يعني إيه مسار شبكة.
- **User vs Computer Configuration**: الـ User settings بتتطبق على اليوزر في أي جهاز يدخل عليه، والـ Computer settings بتتطبق على الجهاز أياً كان مين اللي داخل عليه.

</div>
