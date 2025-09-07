
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: 
      sizeLimit: 500Mi  # حد اختياري للحجم
```
### 2. hostPath

- يربط ملف أو دليل من نظام ملفات الـ Node إلى الـ Pod
- مفيد للوصول إلى بيانات النظام على الـ Node
- تحذير: غير محمول بين Nodes وقد يشكل خطر أماني
    
```
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example
  labels:
    app: log-collector
spec:
  containers:
  - name: log-collector-container
    image: alpine
    command: ["sh", "-c"]
    args:
    - while true; do
        echo "$(date) - Collecting logs from host" >> /var/log/host-app.log;
        sleep 30;
      done
    volumeMounts:
    - name: host-log-volume
      mountPath: /var/log
      readOnly: false
  volumes:
  - name: host-log-volume
    hostPath:
      path: /var/log
      type: DirectoryOrCreate
  restartPolicy: OnFailure
```
### 3. configMap

- يحقن بيانات من ConfigMap إلى الـ Pod كملفات أو متغيرات بيئة
- مفيد لتخزين بيانات التكوين غير الحساسة

```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: test-container
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: special-config  # اسم الـ ConfigMap
      items:
      - key: game.properties
        path: game.properties
```
### 4. secret

- يحقن بيانات حساسة من Secret إلى الـ Pod
- البيانات مشفرة بـ base64 وتُعرض كملفات عادية داخل الـ Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: test-container
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret  # اسم الـ Secret
      items:
      - key: username
        path: my-username
```
---

## Persistent Volume (PV) و Persistent Volume Claim (PVC)

### Persistent Volume (PV)

## 📦 ما هو الـ Persistent Volume (PV)؟

الـ **Persistent Volume** هو مورد تخزين في الـ Cluster تم توفيره بواسطة المسؤول (Admin). إنه مورد مستقل عن Pods له دورة حياة مستقلة.

### خصائص الـ PV:

- **مستقل عن الـ Pods**: يبقى موجوداً حتى بعد حذف الـ Pods
- **يدعم أنواع تخزين متعددة**: NFS, iSCSI, Cloud Storage (EBS, Azure Disk, etc.)
    
- **له سعة تخزين محددة**
    
- **له سياسات وصول** (Access Modes) مختلفة
    

## 📋 ما هو الـ Persistent Volume Claim (PVC)؟

الـ **Persistent Volume Claim** هو طلب من قبل المستخدم (User/Developer) لتخزين. يعمل كواجهة بين الـ Pod والـ PV.

### خصائص الـ PVC:

- **طلب للتخزين**: يحدد الحجم ونوع الوصول المطلوب
    
- **يربط الـ Pod بالـ PV**: من خلال الـ Volume
    
- **يدعم ديناميكية التوفير**: مع StorageClass
    
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
### Persistent Volume Claim (PVC)

- طلب من المستخدم لتخزين مستدام
- يشبه "طلب" لموارد التخزين من الـ PVs المتاحة
- يربط الـ PVC مع الـ Pod لاستخدام التخزين المستدام
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
### ربط PVC مع Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
  - name: pv-container
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: pv-storage
  volumes:
  - name: pv-storage
    persistentVolumeClaim:
      claimName: pv-claim  # اسم الـ PVC
```
---

## StorageClass و Dynamic Provisioning

### StorageClass

- يصف "فئات" التخزين المتاحة في الكلستر
- يسمح بإدارة ديناميكية للـ PVs
- كل موفر تخزين (AWS EBS, GCE PD, etc.) له معاملات مختلفة
    

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs  # موفر التخزين
parameters:
  type: gp3
  fsType: ext4
  iops: "10000"
  throughput: "500"
reclaimPolicy: Retain  # أو Delete
allowVolumeExpansion: true  # يسمح بتوسعة الحجم
volumeBindingMode: WaitForFirstConsumer
```
### Dynamic Provisioning

- إنشاء الـ PVs تلقائياً عند إنشاء الـ PVCs
    
- يلغي الحاجة إلى إنشاء الـ PVs يدوياً
    
- يعتمد على الـ StorageClass المحدد في الـ PVC
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  storageClassName: fast-ssd  # يشير إلى StorageClass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```
---

## CSI (Container Storage Interface)

### ما هو CSI؟

- معيار مفتوح يسمح لأنظمة التخزين بالتكامل مع أنظمة orchstration مثل كوبرنيتيس
- يفصل منطق التخزين عن نواة كوبرنيتيس
- يسمح لمزودي التخزين بتطوير مشغلات (drivers) دون الحاجة إلى تعديل كود كوبرنيتيس الأساسي

### مكونات CSI:

1. **CSI Driver**: برنامج يتكامل مع نظام التخزين الخارجي
2. **Node Plugin**: يعمل على كل node ويدعم عمليات mount/unmount
3. **Controller Plugin**: يعمل في الـ control plane ويدعم إنشاء/حذف الـ volumes

### مثال استخدام CSI مع AWS EBS:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com  # موفر CSI
parameters:
  type: gp3
  encrypted: "true"
  kmsKeyId: arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
```
### فوائد CSI:

- دعم لأنظمة تخزين أحدث دون انتظار تحديثات كوبرنيتيس
- تطوير مستقل لمشغلات التخزين
- دعم ميزات متقدمة مثل التصوير (snapshots)، الاستنساخ (cloning)، والتوسعة (expansion)

---

## العرض العملي

في هذا القسم العملي، سنقوم بما يلي:

### 1. تجربة أنواع Volumes المختلفة


```
# إنشاء Pod باستخدام emptyDir
kubectl apply -f emptyDir-pod.yaml

# إنشاء Pod باستخدام hostPath
kubectl apply -f hostPath-pod.yaml

# التحقق من وجود البيانات في المسارات المحددة
kubectl exec -it pod-name -- ls /path/to/mount
```
### 2. إدارة PV و PVC
```
# إنشاء PersistentVolume
kubectl apply -f pv.yaml

# إنشاء PersistentVolumeClaim
kubectl apply -f pvc.yaml

# التحقق من ربط PVC مع PV
kubectl get pv
kubectl get pvc

# إنشاء Pod يستخدم PVC
kubectl apply -f pv-pod.yaml
```

### 3. استخدام Dynamic Provisioning مع StorageClass

```
# إنشاء StorageClass
kubectl apply -f storageclass.yaml

# إنشاء PVC تشير إلى StorageClass
kubectl apply -f dynamic-pvc.yaml

# سينشئ النظام PV تلقائياً
kubectl get pv
kubectl get pvc
```
### 4. استخدام CSI Driver (مثال مع AWS EBS)

```
# تثبيت AWS EBS CSI Driver (إذا لم يكن مثبتاً)
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"

# إنشاء StorageClass باستخدام مشغل CSI
kubectl apply -f ebs-storageclass.yaml

# إنشاء PVC
kubectl apply -f ebs-pvc.yaml

# استخدام PVC في Pod
kubectl apply -f ebs-pod.yaml
```
### 5. اختبار استمرارية البيانات
```
# كتابة بيانات إلى volume
kubectl exec -it pod-name -- sh -c "echo 'test data' > /path/to/file"

# حذف Pod وإعادة إنشائه
kubectl delete pod pod-name
kubectl apply -f pod.yaml

# التحقق من استمرارية البيانات
kubectl exec -it pod-name -- cat /path/to/file
# يجب أن يظهر "test data"
```
---

## مصادر خطة الدراسة

### الوثائق الرسمية:

- [Kubernetes Volumes Documentation](https://kubernetes.io/docs/concepts/storage/volumes/)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Container Storage Interface (CSI)](https://kubernetes.io/docs/concepts/storage/volumes/#csi)
    

### مقالات ودروس:

- [Kubernetes Storage Best Practices](https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-for-storage)
- [Understanding Kubernetes Storage Options](https://www.digitalocean.com/community/tutorials/understanding-kubernetes-storage-options)
- [CSI Driver Development Guide](https://kubernetes-csi.github.io/docs/)
    

### أدوات ومشغلات تخزين شائعة:

- [AWS EBS CSI Driver](https://github.com/kubernetes-sigs/aws-ebs-csi-driver)
- [Azure Disk CSI Driver](https://github.com/kubernetes-sigs/azuredisk-csi-driver)
- [Google PD CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver)
- [Rook (Ceph Storage for Kubernetes)](https://rook.io/)
- [Longhorn (Distributed Block Storage)](https://longhorn.io/)
    

### مشاريع عملية:

- نشر تطبيق قاعدة بيانات (MySQL/PostgreSQL) مع تخزين مستدام
- بناء نظام تخزين موزع باستخدام Rook/Ceph
- تكوين snapshots تلقائية للبيانات الهامة
- تطبيق سياسات احتفاظ البيانات (Data Retention Policies)
    

---

في الختام، يعد فهم أنظمة التخزين في كوبرنيتيس أمراً بالغ الأهمية لإدارة التطبيقات stateful بشكل فعال. من خلال إتقان المفاهيم والتقنيات التي تم تناولها في هذه الوحدة، ستتمكن من تصميم أنظمة تخزين قوية وموثوقة لتطبيقاتك في بيئة كوبرنيتيس.
