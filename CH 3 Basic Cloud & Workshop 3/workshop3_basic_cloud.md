# R2DE Workshop3 
### Upload ไฟล์ข้อมูล เข้า Google Cloud Storage
&nbsp;<br>
&nbsp;<br>
#### วิธี Upload Data เข้า Google Cloud Storage
&nbsp;<br>
#### 1 สร้าง Bucket 
* 1.1 สร้างผ่าน Cloud Console บนเว็บ UI
* 1.2 ใช้คำสั่ง gsutil command ผ่าน Cloud Shell
  ![create_bucket](https://github.com/srrytn92/Road-to-Data-Engineer/assets/83905993/f693b608-105e-4c27-8de8-0447b735f2bb)

#### 2. อัปโหลดข้อมูล
* 2.1 อัปโหลดข้อมูลผ่าน Cloud Console บนเว็บ UI
* 2.2 ใช้คำสั่ง gsutil ผ่าน Cloud Shell
 ![cp_file2](https://github.com/srrytn92/Road-to-Data-Engineer/assets/83905993/62e81cad-9be2-487f-983f-f2f4de23bde3)
* 2.3 ใช้โค้ด Python ผ่าน Python SDK library <br />
[Python Code](https://github.com/srrytn92/Road-to-Data-Engineer/blob/main/CH%203%20Basic%20Cloud%20%26%20Workshop%203/workshop3%20-%20upload-download.py)
