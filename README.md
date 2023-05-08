# Argo CD Demo

## Apa itu Argo CD?

Argo CD adalah sebuah alat untuk melakukan continuous delivery (CD) secara deklaratif dan GitOps untuk Kubernetes. Argo CD dapat menarik kode yang diperbarui dari repositori Git dan menerapkannya langsung ke sumber daya Kubernetes. Ini memungkinkan pengembang untuk mengelola konfigurasi infrastruktur dan pembaruan aplikasi dalam satu sistem¹.

Argo CD mengikuti prinsip GitOps yang menggunakan repositori Git sebagai sumber kebenaran untuk mendefinisikan keadaan aplikasi yang diinginkan. Manifes Kubernetes dapat ditentukan dengan beberapa cara, seperti kustomize, helm, jsonnet, atau YAML biasa. Argo CD akan secara otomatis menyinkronkan konfigurasi aplikasi dengan keadaan yang dinyatakan saat ini di repositori Git².

Argo CD memiliki beberapa fitur utama, seperti:

- Penerapan otomatis aplikasi ke lingkungan target yang ditentukan
- Penerapan gaya GitOps yang mendeklarasikan keadaan aplikasi yang diinginkan di Git
- Strategi penerapan yang berbeda seperti rollback, blue-green, canary, dll.
- Integrasi SSO dengan penyedia SSO seperti OIDC, OAuth2, LDAP, SAML 2.0, GitHub, GitLab, Microsoft, dll.
- Multi-tenancy dan kebijakan RBAC untuk otorisasi
- Analisis status kesehatan sumber daya aplikasi
- Web UI yang memberikan tampilan real-time aktivitas aplikasi
- CLI untuk otomatisasi dan integrasi CI/CD
- Metrik Prometheus³

## Struktur Folder

Struktur folder untuk demo ini terlihat seperti ini:

```
├── staging
│   ├── balance # new subfolder for balance service 
│   │    ├── Chart.yaml  
│   │    └── values.yaml 
│   ├── search # new subfolder for search service 
│   │    ├── Chart.yaml  
│   │    └── values.yaml 
│   ├── items # new subfolder for items service 
│   │    ├── Chart.yaml  
│   │    └── values.yaml 
│   ├── order # new subfolder for order service 
│   │    ├── Chart.yaml  
│   │    └── values.yaml 
│   └── chart # new subfolder for chart service 
│        ├── Chart.yaml  
│        └── values.yaml 
├── production
│   ├── balance # new subfolder for balance service 
│   │    ├── Chart.yaml  
│   │    └── values.yaml 
│   ├── search # new subfolder for search service 
│   │    ├── Chart.yaml  
│   │    └── values.yaml 
│   ├── items # new subfolder for items service 
│   │    ├── Chart.yaml  
│   │    └── values.yaml 
│   ├── order # new subfolder for order service 
│   │    ├── Chart.yaml  
│   │    └── values.yaml 
│   └── chart # new subfolder for chart service 
        ├── Chart.yaml  
        └── values.yaml 
└── applicationsets
    └── apps.yaml 

```

Folder staging dan production berisi subfolder untuk setiap layanan yang ingin diterapkan ke klaster Kubernetes. Subfolder tersebut berisi file Chart.yaml dan values.yaml untuk

## Cara Menggunakan ApplicationSet

ApplicationSet adalah sebuah sumber daya Kubernetes yang memungkinkan Anda untuk membuat, memodifikasi, dan mengelola beberapa aplikasi Argo CD sekaligus. ApplicationSet menggunakan template dan generator untuk membuat aplikasi Argo CD berdasarkan nilai-nilai yang dihasilkan oleh generator. Generator dapat menghasilkan nilai-nilai dari berbagai sumber, seperti klaster Kubernetes, repositori Git, file JSON atau YAML, atau daftar statis.

ApplicationSet memiliki dua bagian utama dalam spesifikasinya: spec.generator dan spec.template.

### spec.generator

spec.generator adalah bagian yang menentukan generator yang digunakan untuk menghasilkan nilai-nilai yang akan disuplai ke template. Generator dapat berupa salah satu dari:

- clusters: generator ini menghasilkan nilai-nilai dari klaster Kubernetes yang terdaftar di Argo CD
- git: generator ini menghasilkan nilai-nilai dari file atau direktori di repositori Git
- list: generator ini menghasilkan nilai-nilai dari daftar statis yang ditentukan dalam spesifikasi
- jsonnet: generator ini menghasilkan nilai-nilai dari file jsonnet
- scmpath: generator ini menghasilkan nilai-nilai dari jalur SCM (misalnya GitHub)

Generator dapat memiliki parameter tambahan tergantung pada jenisnya. Misalnya, generator git dapat memiliki parameter seperti repoURL, revision, directories, files, rekursif, dll.

### spec.template

spec.template adalah bagian yang menentukan template aplikasi Argo CD yang akan dibuat oleh ApplicationSet. Template ini mirip dengan sumber daya aplikasi Argo CD biasa, tetapi dengan dukungan untuk substitusi parameter. Parameter dapat berupa salah satu dari:

- { {name}}: nama klaster atau nama elemen dalam generator list
- { {server}}: alamat server klaster
- { {path}}: jalur ke file atau direktori dalam repositori Git
- { {path.basename}}: nama dasar jalur (tanpa direktori induk)
- { {path.basenameNormalized}}: nama dasar jalur yang dinormalisasi (dengan karakter yang tidak didukung diganti dengan -)
- { {path [n]}}: komponen jalur ke-n (misalnya, untuk jalur /clusters/clusterA, path [0]: clusters, path [1]: clusterA)
- { {cluster.labels.foo}}: label klaster dengan kunci foo
- { {values.foo}}: nilai khusus yang ditentukan dalam generator list dengan kunci foo

Template dapat memiliki parameter tambahan tergantung pada jenis sumbernya. Misalnya, template dengan sumber helm dapat memiliki parameter seperti chart, valuesFiles, dll.

Berikut adalah contoh ApplicationSet yang menggunakan generator git dan template helm:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: apps
spec:
  generators:
  - git:
      repoURL: https://github.com/example/repo.git 
      revision: HEAD 
      directories:
      - path: 'staging/*' # match any subfolder under staging 
        cluster: staging         
        url: https://1.2.3.4
      - path: 'production/*' # match any subfolder under production 
        cluster: production         
        url: https://9.8.7.6
  template:
    metadata:
      name: '{{cluster}}-{{path.basename}}'
    spec:
      project: default 
      source:
        repoURL: https://github.com/example/chart.git # use helm chart from this repo
        chart: '{{path.basename}}' # use the subfolder name as the chart name
        targetRevision: HEAD 
      destination:
        server: '{{url}}'
        namespace: apps # use apps as the namespace for all applications
```

ApplicationSet ini akan membuat aplikasi Argo CD untuk setiap subfolder di bawah staging dan production. Misalnya, aplikasi Argo CD dengan nama staging-apps-balance akan menerapkan helm chart balance yang ada di repositori https://github.com/example/chart.git ke ruang nama apps di klaster staging.

## Penerapan Argocd

1. Install argocd CLI di komputer Anda. Anda dapat mengunduh versi terbaru dari https://github.com/argoproj/argo-cd/releases/latest atau menggunakan perintah curl.

2. Install argocd server di kluster Kubernetes Anda. Anda dapat menggunakan perintah kubectl berikut:

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

3. Akses argocd API server dengan salah satu cara berikut¹:
    - Ubah tipe layanan argocd-server menjadi LoadBalancer
    - Konfigurasikan ingress untuk argocd-server
    - Gunakan kubectl port-forward untuk menghubungkan ke argocd-server

4. Login menggunakan CLI dengan perintah berikut¹:

```
argocd login <ARGOCD_SERVER>
```

5. Terapkan ApplicationSet CRD ke kluster Kubernetes Anda dengan perintah kubectl berikut:

```
kubectl apply -f applicationset.yaml
```

6. Periksa status aplikasi yang dihasilkan oleh ApplicationSet dengan perintah berikut:

```
argocd app list
```

7. Selesai! Anda telah berhasil install argocd dan apply applicationsetnya.

---- Semua text di atas di generate ole AI ---