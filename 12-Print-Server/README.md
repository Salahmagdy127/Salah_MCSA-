# 12 — Print Server

**Machine:** PDC

## Steps

1. Add Roles and Features → ✅ **Print and Document Services** → Role service: **Print Server** → Install.
2. Tools → **Print Management** → Print Servers → PDC → right-click **Printers** → **Add Printer**:
   - Add a TCP/IP or Web Services Printer by IP → e.g. `192.168.1.50` (fake printer is fine for lab)
     → Device type: Generic Network Card → choose a driver (e.g. a generic/Microsoft driver).
   - Share name: e.g. `Printer-HR`.
3. Printer → Properties:
   - **Advanced** → **Available from** `9:00 AM` **To** `4:00 PM`.
   - **Printing Defaults** → Color → **Black & White** (grayscale).
4. **Deploy to users** — right-click printer → **Deploy with Group Policy** →
   Browse → pick a GPO → ✅ The users that this GPO applies to (per user) → Add → OK.
   (Also ✅ **List in the directory** so users can search for it in AD.)

## PowerShell

```powershell
Install-WindowsFeature Print-Server -IncludeManagementTools
Add-PrinterPort -Name "IP_192.168.1.50" -PrinterHostAddress 192.168.1.50
Add-Printer -Name "Printer-HR" -DriverName "Microsoft PS Class Driver" -PortName "IP_192.168.1.50" `
    -Shared -ShareName "Printer-HR" -Published
Set-PrintConfiguration -PrinterName "Printer-HR" -Color $false
```

## Verify

- Client: `gpupdate /force` → log off/on → Settings → Printers → printer appears.
- Printing outside 9–4 → job waits in queue until 9 AM.

---

<div dir="rtl">

## ليه عملنا كده؟

- **Print Server**: بدل ما كل جهاز يتعرّف على الطابعة لوحده، بقى فيه مكان واحد تدير منه الطابعات كلها (drivers، صلاحيات، queue).
- **أبيض وأسود بس**: توفير في الحبر الألوان الغالي، واللي عايز ألوان يتعمله طابعة تانية بصلاحيات.
- **مواعيد 9 لـ 4**: الطابعة تشتغل في مواعيد الشغل بس، فمحدش يطبع حاجات شخصية بعد الشغل، وده توفير كمان.
- **Deploy with GPO**: الطابعة بتظهر عند الموظفين أوتوماتيك أول ما يعملوا login، من غير ما حد من الـ IT يروح لكل جهاز.

</div>
