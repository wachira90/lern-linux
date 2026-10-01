# `/etc/skel` (Skeleton Directory) 

คือโฟลเดอร์แม่แบบสำหรับสร้าง Home Directory ของผู้ใช้ใหม่ เมื่อสร้างผู้ใช้ ระบบจะคัดลอกไฟล์จาก `/etc/skel` ไปยัง `/home/<username>`

> จุดสำคัญ: การแก้ `/etc/skel` มีผลเฉพาะผู้ใช้ที่สร้างใหม่หลังจากแก้ไขเท่านั้น ผู้ใช้เดิมจะไม่เปลี่ยนแปลงอัตโนมัติ

## 1. ตรวจสอบไฟล์เริ่มต้น

```bash
sudo ls -la /etc/skel
```

โดยทั่วไปอาจพบ:

```text
.bash_logout
.bashrc
.profile
```

หน้าที่หลัก:

- `.bashrc` — ทำงานเมื่อเปิด Interactive Bash shell
- `.profile` — ทำงานเมื่อ Login shell เริ่มต้น
- `.bash_logout` — ทำงานเมื่อออกจาก Login shell
- `.bash_profile` — บาง Distribution ใช้แทน `.profile`

## 2. สำรองข้อมูลก่อนปรับแต่ง

```bash
sudo cp -a /etc/skel /etc/skel.backup
```

ตรวจสอบ:

```bash
sudo ls -la /etc/skel.backup
```

หากต้องการคืนค่า:

```bash
sudo cp -a /etc/skel.backup/. /etc/skel/
```

## 3. เพิ่ม Profile เริ่มต้น

แนะนำให้สร้างไฟล์แยก แล้วเรียกใช้จาก `.bashrc` เพื่อดูแลรักษาง่าย:

```bash
sudo nano /etc/skel/.company_profile
```

ตัวอย่างเนื้อหา:

```bash
# Company default shell profile

export EDITOR=vim
export HISTSIZE=5000
export HISTFILESIZE=10000
export HISTCONTROL=ignoreboth

alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias grep='grep --color=auto'

umask 027
```

จากนั้นเพิ่มคำสั่งโหลดไฟล์นี้ใน `/etc/skel/.bashrc`:

```bash
sudo nano /etc/skel/.bashrc
```

เพิ่มท้ายไฟล์:

```bash
# Load organization default profile
if [ -f "$HOME/.company_profile" ]; then
    . "$HOME/.company_profile"
fi
```

ใช้ `$HOME` แทนการเขียน `/home/<username>` แบบตายตัว เพื่อรองรับ Home Directory ที่อยู่ตำแหน่งอื่น

## 4. เพิ่มโครงสร้าง Directory เริ่มต้น

ตัวอย่างสร้างโฟลเดอร์มาตรฐาน:

```bash
sudo mkdir -p /etc/skel/bin
sudo mkdir -p /etc/skel/logs
sudo mkdir -p /etc/skel/projects
sudo mkdir -p /etc/skel/.config
```

เพิ่มไฟล์ต้อนรับ:

```bash
sudo nano /etc/skel/README.txt
```

ตัวอย่าง:

```text
Welcome!

Directories:
- bin      Personal commands and scripts
- logs     Application logs
- projects Development projects
```

ตรวจสอบผล:

```bash
sudo find /etc/skel -maxdepth 2 -printf '%M %u:%g %p\n'
```

## 5. กำหนด Permission

โดยทั่วไปไฟล์ต้นแบบควรเป็นของ `root`:

```bash
sudo chown -R root:root /etc/skel
```

กำหนด Permission พื้นฐาน:

```bash
sudo find /etc/skel -type d -exec chmod 755 {} \;
sudo find /etc/skel -type f -exec chmod 644 {} \;
```

หากมี Script ที่ต้อง Execute:

```bash
sudo chmod 755 /etc/skel/bin/example-script
```

ระวังคำสั่ง `find ... chmod` เพราะจะทำให้ไฟล์ทุกไฟล์มี Permission เหมือนกัน ควรกำหนดไฟล์ที่มีข้อมูลสำคัญแยกต่างหาก เช่น:

```bash
sudo chmod 600 /etc/skel/.company_private_config
```

อย่างไรก็ตาม ไม่ควรเก็บ Password, Private Key, API Key หรือ Secret ไว้ใน `/etc/skel`

## 6. สร้างผู้ใช้ใหม่และทดสอบ

### Ubuntu/Debian

```bash
sudo adduser testuser
```

หรือใช้ `useradd`:

```bash
sudo useradd -m -s /bin/bash testuser
sudo passwd testuser
```

### RHEL/Rocky/AlmaLinux

```bash
sudo useradd -m -s /bin/bash testuser
sudo passwd testuser
```

ตัวเลือกสำคัญ:

- `-m` — สร้าง Home Directory
- `-s /bin/bash` — กำหนด Login shell
- `-k /etc/skel` — ระบุ Skeleton Directory

ตัวอย่างระบุ `/etc/skel` อย่างชัดเจน:

```bash
sudo useradd -m -k /etc/skel -s /bin/bash testuser
sudo passwd testuser
```

ตรวจสอบไฟล์ที่ถูกคัดลอก:

```bash
sudo ls -la /home/testuser
```

ตรวจสอบเจ้าของไฟล์:

```bash
sudo find /home/testuser -maxdepth 2 -printf '%u:%g %M %p\n'
```

ระบบควรเปลี่ยนเจ้าของไฟล์ที่คัดลอกเป็น `testuser:testuser` โดยอัตโนมัติ

ทดลองเข้าสู่ระบบ:

```bash
sudo -iu testuser
```

ตรวจสอบค่า:

```bash
echo "$EDITOR"
alias ll
umask
pwd
```

## 7. นำ Template ไปใช้กับผู้ใช้เดิม

`/etc/skel` ไม่อัปเดตผู้ใช้เดิมโดยอัตโนมัติ หากต้องการคัดลอกไฟล์ที่ยังไม่มี สามารถใช้:

```bash
sudo cp -an /etc/skel/. /home/existinguser/
sudo chown -R existinguser:existinguser /home/existinguser
```

ความหมายของ `cp -an`:

- `-a` — รักษาโครงสร้างและ Attribute
- `-n` — ไม่เขียนทับไฟล์เดิม

อย่าใช้คำสั่งเขียนทับ `.bashrc` หรือ `.profile` โดยไม่สำรอง เพราะผู้ใช้อาจมีการตั้งค่าส่วนตัวอยู่แล้ว

แนวทางปลอดภัยกว่าคือคัดลอกเฉพาะไฟล์องค์กร:

```bash
sudo install -o existinguser -g existinguser -m 644 \
    /etc/skel/.company_profile \
    /home/existinguser/.company_profile
```

แล้วเพิ่มการเรียกใช้ใน `.bashrc` ของผู้ใช้โดยตรวจสอบก่อนว่าไม่มีบรรทัดซ้ำ

## 8. `/etc/skel` กับ `/etc/profile.d/` ต่างกันอย่างไร

ใช้ `/etc/skel` เมื่อ:

- ต้องการให้ผู้ใช้ได้รับไฟล์ของตนเอง
- ผู้ใช้สามารถแก้ไขการตั้งค่าภายหลังได้
- ต้องการสร้างโฟลเดอร์หรือไฟล์ตัวอย่างใน Home
- ต้องการตั้งค่าเฉพาะผู้ใช้ใหม่

ใช้ `/etc/profile.d/*.sh` เมื่อ:

- ต้องการตั้งค่ากลางให้ผู้ใช้ทุกคน
- ต้องการให้ผู้ใช้เดิมและผู้ใช้ใหม่ได้รับค่าเดียวกัน
- ต้องการให้ผู้ดูแลระบบควบคุมการตั้งค่า

ตัวอย่าง:

```bash
sudo nano /etc/profile.d/company.sh
```

```bash
export COMPANY_ENV="production"
```

กำหนด Permission:

```bash
sudo chown root:root /etc/profile.d/company.sh
sudo chmod 644 /etc/profile.d/company.sh
```

โปรดทราบว่า `/etc/profile.d/` มักถูกโหลดโดย Login shell ส่วน Interactive non-login shell อาจมีพฤติกรรมต่างกันตาม Distribution และ shell ที่ใช้งาน

## ตัวอย่าง `/etc/skel` ที่แนะนำ

```text
/etc/skel/
├── .bash_logout
├── .bashrc
├── .company_profile
├── .profile
├── .config/
├── bin/
├── logs/
├── projects/
└── README.txt
```

แนวทางที่ดีที่สุดคือแก้ไข `/etc/skel` อย่างระมัดระวัง ทดสอบด้วยบัญชีชั่วคราวก่อนใช้งานจริง และเก็บเฉพาะ Configuration ทั่วไป—ไม่เก็บข้อมูลลับหรือค่าที่ผูกกับผู้ใช้รายใดรายหนึ่งครับ
