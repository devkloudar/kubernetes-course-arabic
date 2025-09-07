## نظرة عامة على الوحدة

مرحباً بكم في الوحدة العاشرة من دورة كوبرنيتيس. في هذه الوحدة، سنغطي أحد الجوانب الأساسية لإدارة التطبيقات في الحاويات: **أنظمة التخزين المستدامة**.

بينما تكون الحاويات بطبيعتها عابرة (ephemeral) وتفقد بياناتها عند إعادة التشغيل، توفر كوبرنيتيس عدة آليات لتخزين البيانات بشكل دائم ومستدام. في هذه الوحدة، سنستكشف هذه الآليات بالتفصيل.

سنتعلم في هذه الوحدة:

- أنواع مختلفة من وحدات التخزين (Volumes) وكيفية استخدامها
- مفهوم Persistent Volumes و Persistent Volume Claims
- كيفية عمل Dynamic Provisioning باستخدام StorageClass
- مقدمة عن Container Storage Interface (CSI)
- تطبيق عملي لأنواع التخزين المختلفة

---

## أنواع الـ Volumes (emptyDir, hostPath, configMap, secret)

### 1. emptyDir

- حجم تخزين مؤقت ينشأ عند إنشاء الـ Pod ويُحذف عند حذف الـ Pod
- مفيد للمشاركة المؤقتة للبيانات بين Containers في نفس الـ Pod
- يمكن تخزين البيانات على القرص أو في الذاكرة (tmpfs)


```
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: nginx
    name: nginx-container
    volumeMounts:
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
