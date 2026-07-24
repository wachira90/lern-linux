# **LVM (Logical Volume Manager)**

เป็นระบบบริหารจัดการพื้นที่เก็บข้อมูลบน Linux ที่มีความยืดหยุ่นสูงกว่าการแบ่งพาร์ทิชัน (Partition) แบบดั้งเดิม จุดเด่นของ LVM คือสามารถขยาย (Extend) หรือลด (Reduce) ขนาดพื้นที่ของดิสก์ได้อย่างอิสระ รวมถึงสามารถนำดิสก์หลายๆ ก้อนมารวมเป็นก้อนเดียวกันได้ โดยไม่ต้องหยุดการทำงานของระบบ (Zero Downtime)

เพื่อทำความเข้าใจการใช้งาน ต้องรู้จักองค์ประกอบหลัก 3 ส่วนของ LVM ดังนี้:

1. **PV (Physical Volume):** ดิสก์จริง (เช่น `/dev/sdb`) หรือพาร์ทิชัน (เช่น `/dev/sdc1`) ที่ถูกกำหนดให้พร้อมใช้งานสำหรับ LVM
2. **VG (Volume Group):** การนำ PV หลายๆ ก้อนมารวมกันเป็น "สระพื้นที่ส่วนกลาง" (Storage Pool)
3. **LV (Logical Volume):** การจำลองพาร์ทิชันที่ดึงพื้นที่มาจาก VG เพื่อนำไปฟอร์แมตและเมาท์ (Mount) ใช้งานจริง

---

## ขั้นตอนการสร้างและใช้งาน LVM

*สถานการณ์สมมติ: คุณมีดิสก์เปล่าก้อนใหม่เพิ่มเข้ามาในระบบคือ `/dev/sdb` และ `/dev/sdc*`

### 1. การสร้าง Physical Volume (PV)

เตรียมดิสก์จริงให้ระบบ LVM รู้จัก

```bash
# สร้าง PV บนดิสก์ทั้งสองก้อน
sudo pvcreate /dev/sdb /dev/sdc

# ตรวจสอบสถานะของ PV
sudo pvs
# หรือดูรายละเอียดแบบเต็ม
sudo pvdisplay

```

### 2. การสร้าง Volume Group (VG)

นำ PV ที่สร้างไว้มารวมกันเป็นกลุ่มพื้นที่ สมมติว่าตั้งชื่อกลุ่มนี้ว่า `vg_data`

```bash
# สร้าง VG ชื่อ vg_data โดยใช้ดิสก์ sdb และ sdc
sudo vgcreate vg_data /dev/sdb /dev/sdc

# ตรวจสอบสถานะของ VG เพื่อดูขนาดพื้นที่รวมทั้งหมด
sudo vgs
# หรือดูรายละเอียดแบบเต็ม
sudo vgdisplay

```

### 3. การสร้าง Logical Volume (LV)

ดึงพื้นที่จาก VG มาสร้างเป็นพาร์ทิชันสำหรับใช้งาน สมมติว่าสร้าง LV ชื่อ `lv_app`

```bash
# สร้าง LV ขนาด 50GB
sudo lvcreate -n lv_app -L 50G vg_data

# หรือหากต้องการใช้พื้นที่ทั้งหมดที่เหลืออยู่ใน VG
sudo lvcreate -n lv_app -l 100%FREE vg_data

# ตรวจสอบสถานะของ LV
sudo lvs
# หรือดูรายละเอียดแบบเต็ม
sudo lvdisplay

```

### 4. การ Format และ Mount ใช้งาน

ตอนนี้คุณจะได้พาธของดิสก์ที่พร้อมใช้งานคือ `/dev/vg_data/lv_app` (หรือสามารถอ้างอิงผ่าน `/dev/mapper/vg_data-lv_app` ก็ได้)

```bash
# 1. ฟอร์แมตระบบไฟล์ (ตัวอย่างใช้ ext4, หากต้องการใช้ xfs ให้เปลี่ยนเป็น mkfs.xfs)
sudo mkfs.ext4 /dev/vg_data/lv_app

# 2. สร้าง Directory สำหรับเมาท์
sudo mkdir -p /data_app

# 3. เมาท์พื้นที่เพื่อใช้งาน
sudo mount /dev/vg_data/lv_app /data_app

# 4. ตรวจสอบพื้นที่
df -h /data_app

```

*(หมายเหตุ: เพื่อให้ระบบเมาท์อัตโนมัติเมื่อรีบูตเครื่อง อย่าลืมนำข้อมูลไปตั้งค่าในไฟล์ `/etc/fstab`)*

---

## การบริหารจัดการ LVM ที่พบบ่อย (Day-2 Operations)

### การขยายขนาดพื้นที่ (Extend LV)

นี่คือจุดเด่นที่สุดของ LVM เมื่อพื้นที่ `/data_app` ของคุณเต็ม คุณสามารถขยายได้ทันที

**กรณีที่ 1: Volume Group (vg_data) ยังมีพื้นที่เหลืออยู่**

```bash
# ขยายขนาด LV เพิ่มอีก 20GB
sudo lvextend -L +20G /dev/vg_data/lv_app

# หรือขยายให้เต็มพื้นที่ VG ที่เหลืออยู่
sudo lvextend -l +100%FREE /dev/vg_data/lv_app

# หลังจากขยาย LV แล้ว ต้องขยาย File System ให้มองเห็นพื้นที่ใหม่ด้วย
# สำหรับระบบไฟล์ ext4:
sudo resize2fs /dev/vg_data/lv_app

# สำหรับระบบไฟล์ xfs:
sudo xfs_growfs /data_app

```

**กรณีที่ 2: Volume Group (vg_data) พื้นที่เต็มแล้ว**
คุณต้องเสียบดิสก์ก้อนใหม่ (สมมติว่าเป็น `/dev/sdd`) เพื่อขยาย VG ก่อน

```bash
# 1. สร้าง PV ใหม่
sudo pvcreate /dev/sdd

# 2. ขยาย VG โดยเพิ่มดิสก์ก้อนใหม่เข้าไป
sudo vgextend vg_data /dev/sdd

# 3. ขยาย LV และ File System ตามขั้นตอนใน "กรณีที่ 1"

```

### การลดขนาดพื้นที่ (Reduce LV)

*คำเตือน: มีความเสี่ยงที่ข้อมูลจะสูญหาย ควร Backup ข้อมูลทุกครั้ง และระบบไฟล์ **XFS ไม่รองรับ** การลดขนาด (รองรับเฉพาะ ext2/3/4)*

```bash
# 1. ต้อง Unmount ก่อนเสมอ
sudo umount /data_app

# 2. ตรวจสอบความสมบูรณ์ของระบบไฟล์
sudo e2fsck -f /dev/vg_data/lv_app

# 3. ลดขนาด File System ก่อน (เช่น ลดเหลือ 30GB)
sudo resize2fs /dev/vg_data/lv_app 30G

# 4. ลดขนาด LV
sudo lvreduce -L 30G /dev/vg_data/lv_app

# 5. เมาท์กลับคืน
sudo mount /dev/vg_data/lv_app /data_app

```

### การลบ LVM (Remove LVM)

หากต้องการยกเลิกระบบ LVM ทิ้ง ต้องทำย้อนกลับจาก LV -> VG -> PV ตามลำดับ

```bash
# 1. Unmount พื้นที่
sudo umount /data_app

# 2. ลบ Logical Volume (ระบบจะถามยืนยัน ให้ตอบ y)
sudo lvremove /dev/vg_data/lv_app

# 3. ลบ Volume Group
sudo vgremove vg_data

# 4. ลบ Physical Volume (คืนค่าดิสก์เป็นดิสก์ปกติ)
sudo pvremove /dev/sdb /dev/sdc

```
