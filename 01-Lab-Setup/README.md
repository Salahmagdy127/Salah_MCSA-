# 01 — Lab Setup (VMware Workstation)

## Steps

1. Create 3 VMs in VMware Workstation:
   | VM | OS | Suggested RAM |
   |---|---|---|
   | PDC | Windows Server (Desktop Experience / GUI) | 4 GB |
   | CORE | Windows Server (Server Core — no GUI) | 2–4 GB |
   | CLIENT | Windows 11 | 4 GB |
2. Put all VMs on the **same virtual network** (e.g. the same VMnet / NAT / Bridged) so they see each other.
3. For **PDC** and **CORE** (they will run Hyper-V later):
   - Power off the VM → **VM Settings → Processors** → check
     **Virtualize Intel VT-x/EPT or AMD-V/RVI**.
4. (Optional) Take a **snapshot** of each VM after OS install — easy rollback.

## Verify

- All VMs boot and have network adapters on the same VMnet.

---

<div dir="rtl">

## ليه عملنا كده؟

- **3 أجهزة**: ده أقل setup بيمثّل شركة حقيقية: سيرفر رئيسي، وسيرفر احتياطي، وجهاز موظف.
- **Server Core**: من غير واجهة رسومية، فبيستهلك RAM وديسك أقل، والثغرات فيه أقل (attack surface أصغر) لأن مفيش GUI تتضرب. في الشغل الحقيقي أغلب الـ DCs بيبقوا Core.
- **نفس الشبكة**: عشان الأجهزة تشوف بعض وتقدر تعمل Join للدومين.
- **Virtualize VT-x**: الـ Hyper-V محتاج الـ CPU يدعم الـ virtualization. وإحنا أصلاً جوه VM، فلازم VMware "يعدّي" الميزة دي للـ VM، وده اسمه **Nested Virtualization**. من غيرها تثبيت الـ Hyper-V هيفشل.

</div>
