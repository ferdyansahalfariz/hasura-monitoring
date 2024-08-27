## pre Deployment

### Git push branch staging repo hasura-chm
1. Push repo CHM-Schema ke space Bitbucket yg sudah tersedia di URL https://bitbucket.bri.co.id/projects/HSR/repos/datahub-hasura-chm
   
![image](https://github.com/user-attachments/assets/3e12d019-4926-4e08-92ef-dff7eb7d08b6)

### Git push branch staging repo hasura-chm-schema
1. Push repo CHM-Schema ke space Bitbucket yg sudah tersedia di URL https://bitbucket.bri.co.id/projects/HSR/repos/datahub-hasura-chm-schema

![image](https://github.com/user-attachments/assets/b89d8825-0df9-409e-9d39-5b98747415d8)

### Pembuatan multibranch pipeline jenkins baru untuk hasura-chm-schema
1. Buka jenkins dashboard dan klik new item masukan nama pipeline dan pilih multibranch pipeline

![image](https://github.com/user-attachments/assets/786e2d3a-54d8-49f5-8f90-cfba3988c183)
   
2. Masukan branch source dari repo git dan masukan url dan kredential bitbucket, dan pilih branch name dengan opsi Filter by name (with regular expression), isikan 'staging' lalu simpan

![image](https://github.com/user-attachments/assets/16eb0007-9744-44c2-a74b-eb00c190923e)

### Pembuatan jenkins credential untuk menyimpan admin secret
1. buka jenkins dashboard -> Manage jenkins -> manage credentials

![image](https://github.com/user-attachments/assets/3ba371ba-e76c-4156-a4ca-9fed815976ec)

2. Add credential pada global credentials, dan pilih secret text untuk menyimpan hasura admin secret, masukan isinya dan save

![image](https://github.com/user-attachments/assets/8a8eaf97-daf7-4ec5-b000-e09b5136dfa8)

### Pembuatan namespace di kubernetes
1. Buka kubernetes dashboard dan buat namespace baru pada menu add resource
![image](https://github.com/user-attachments/assets/65505683-2c0a-4bf2-a355-c969525d88cf)

1. Buka DB editor

![image](https://github.com/user-attachments/assets/6d083d6b-3013-47b3-9d56-89cb512a38bb)

2. Create new database untuk hasura-chm-schema

![image](https://github.com/user-attachments/assets/24734fb5-9c7d-4f80-8cb8-396c5a9d637f)

3. Pastikan database hasura-chm-schema sudah terbuat

![image](https://github.com/user-attachments/assets/851367d0-3bbf-4521-9eec-a8bdc9767872)

### Pengecekan config di Dockerfile & Jenkinsfile
1. Buka repo bitbucket dan periksa file Dockerfile, periksa apakah image yang digunakan sudah menggunakan image HGE dengan tag v2.40.2

![image](https://github.com/user-attachments/assets/b8f2ab54-76cf-4177-8911-0e3744e0719b)

2. Buka juga file Jenkisfile dan periksa stage "apply metadata" setelah stage `deploy staging` yang sudah merujuk ke hasura endpoint yang tepat sesuai dengan `values.staging.yaml` di folder helm

![image](https://github.com/user-attachments/assets/23686648-0a26-4de2-b6cd-30e60fc9cef3)


### Pembuatan kubesecret 	

1. Akses SSH server yang bisa kubectl ke Kubernetes Cluster

2. Buat secret pada kubernetes sesuai nama yang digunakan di values.staging.yaml dengan command kubectl dengan contoh berikut: https://drive.google.com/file/d/1N1c9tfDR_EJK1xrkhZV5IGr_l_Dnai2O/view?usp=drive_link

3. Jika tidak memiliki akses SSH ke server jenkins, maka buka kubernetes dashboard, add resource dan masukan file yaml seperti berikut: https://drive.google.com/file/d/1siZvv3GnX9CMifmdljAdP-meoeki_5bs/view?usp=sharing

### Pembuatan image pull secret	

1. Masuk ke kubernetes dashboard, masuk ke namespace datahub-hasura dan buat resource baru menggunakan yaml dengan tipe `.dockerconfigjson` yang berisi auth url registry beserta username, password dan email yang memiliki akses, upload

![image](https://github.com/user-attachments/assets/e2e7dca7-48db-4800-8da5-76eb58417f2b)

### Penambahan etc hosts untuk staging.datahub.co.id	

1. SSH ke server jenkins staging dan masuk ke `etc/hosts`, tambahkan alias pada ip tempat hasura di instance dengan alias `staging.datahub.bri.co.id`, simpan

2. Lakukan ping ke `staging.datahub.bri.co.id` dan pastikan sudah terkoneksi
   
### Installasi hasura cli di server jenkins	

1. SSH ke server jenkins, dan lakukan instalasi hasura cli seperti biasa sampai selesai
   
2. konfirmasi instalasi dengan menjalankan command `hasura` dan pastikan hasura cli sudah terinstall

![image](https://github.com/user-attachments/assets/c0eb1481-ab18-444c-9d34-2361d3f1ea2c)

### Pengecekan values.staging.yaml
1. buka values.staging.yaml pada folder `helm/datahub-hasura-chm/` dan lakukan pengecekan pada bagian env, pastikan tidak terdapat value yang menyimpan data penting seperti kredensial, admin secret dsb

![image](https://github.com/user-attachments/assets/895dc2ed-65a4-4436-ab61-77d8795b3e3b)

## Deployment

### Deployment hasura-chm di environment staging
1. Lakukan deployment service dengan klik `build now` pada multibranch pipeline datahub-hasura-chm

![image](https://github.com/user-attachments/assets/71487cb7-3cbb-420c-8e78-21eb8317918e)

2. Cek logs pada jenkins

![image](https://github.com/user-attachments/assets/b4027883-05a2-4267-87b8-1195894ec643)

### Deployment hasura-chm-schema di environment staging
1. Lakukan deployment service dengan klik `build now` pada multibranch pipeline datahub-hasura-chm-schema

![image](https://github.com/user-attachments/assets/71487cb7-3cbb-420c-8e78-21eb8317918e)

2. Cek logs pada jenkins

![image](https://github.com/user-attachments/assets/b4027883-05a2-4267-87b8-1195894ec643)

## Post Deployment

1. Akses ke url hasura instance dengan menambahkan `/healthz` di belakangnya dan pastikan sudah berstatus OK

![image](https://github.com/user-attachments/assets/d486416e-8b07-4125-b220-69bdcfcacc70)

2. Akses hasura console, buka settings dan cek health metadata

![image](https://github.com/user-attachments/assets/beead863-a111-4c2e-a645-d827f9cd0ed6)

3. Jika terjadi inconsistency metadata pada hasura, lakukan fixing metadata melalui file yaml nya dengan mengikuti daftar inconsistency nya pada bagian metadata status di console, setelahnya maka lakukan `hasura metadata apply`

![image](https://github.com/user-attachments/assets/2cdf0ccc-c342-403a-bd8c-00f0e2881634)
