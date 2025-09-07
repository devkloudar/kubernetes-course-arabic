## نظرة عامة على الوحدة

الـ Secrets هي كائنات في Kubernetes تُستخدم لتخزين وإدارة المعلومات الحساسة مثل كلمات المرور وروابط الاتصال وقواعد البيانات ومفاتيح API والشهادات الرقمية. بدلاً من تضمين هذه المعلومات مباشرة في ملفات التكوين أو كود التطبيق، توفر كوبرنيتيس طريقة آمنة ومنفصلة لإدارتها.

في هذه الوحدة، سنغطي:

- مفهوم الـ Secrets وأنواعها المختلفة
- كيفية تشفير البيانات داخل Secrets
- ربط الـ Secrets مع الـ Pods لاستخدامها من قبل التطبيقات
- أفضل الممارسات لإدارة Secrets بأمان
- عرض عملي لتطبيق ما نتعلمه



## ما هو Secret؟

الـ Secret في كوبرنيتيس هو كائن (object) يُستخدم لتخزين وإدارة البيانات الحساسة مثل:

- بيانات الاعتماد (كلمات المرور، tokens، المفاتيح)
- مفاتيح التشفير (SSH، TLS)
- معلومات الاتصال بقواعد البيانات
- أي بيانات حساسة أخرى لا يجب أن تكون في صورة plain text

الفرق الرئيسي بين الـ Secret والـ ConfigMap:

- الـ Secrets مصممة خصيصاً للبيانات الحساسة
- الـ Secrets مشفرة (base64) افتراضياً

- كوبرنيتيس تتخذ احتياطات إضافية لحماية الـ Secrets (مثل تقليل الكتابة على القرص)
    

```

apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=  # admin
  password: MWYyZDFlMmU2N2Rm  # 1f2d1e2e67df
```

---

## أنواع الـ Secrets (Opaque, TLS, Docker Registry)

### 1. Opaque Secrets

- النوع الافتراضي للـ Secrets
- تُستخدم لأي بيانات ثنائية أو بيانات نصية عادية
- يجب أن تكون البيانات مشفرة بـ base64 قبل إضافتها

```
apiVersion: v1
kind: Secret
metadata:
  name: opaque-secret
type: Opaque
data:
  username: dXNlcg==  # user
  password: cGFzcw==  # pass
```
### 2. TLS Secrets

- مُخصصة لتخزين شهادات TLS/SSL
- تحتوي على حقلين إلزاميين: `tls.crt` و `tls.key`
- تُستخدم لتأمين اتصالات HTTPS في Ingress controllers

```
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-private-key>
```
### 3. Docker Registry Secrets

- تُستخدم لتخزين بيانات اعتماد Docker registry
- تُستخدم عند سحب images من registries خاصة
- النوع: `kubernetes.io/dockerconfigjson`

```
kubectl create secret docker-registry myregistrykey \
  --docker-server=DUMMY_SERVER \
  --docker-username=DUMMY_USERNAME \
  --docker-password=DUMMY_PASSWORD \
  --docker-email=DUMMY_EMAIL
```
---

## التشفير (base64 و KMS)

### تشفير Base64

- الـ Secrets في كوبرنيتيس تخزن البيانات مشفرة بـ base64    
- base64 ليس تشفيراً أمنياً، بل هو encoding لتمثيل البيانات الثنائية كنص
- يمكن فك تشفيرها بسهولة، لذا لا تعتبر آلية أمان قوية بحد ذاتها
    
```
# تشفير نص إلى base64
echo -n 'mysecretpassword' | base64
# bXlzZWNyZXRwYXNzd29yZA==

# فك تشفير base64 إلى نص
echo 'bXlzZWNyZXRwYXNzd29yZA==' | base64 --decode
# mysecretpassword
```
### التشفير باستخدام KMS (Key Management Service)

- في بيئات الإنتاج، يجب تشفير الـ Secrets عند تخزينها (encryption at rest)
- كوبرنيتيس تدعم تشفير الـ Secrets باستخدام موفري KMS خارجيين
- يتطلب تهيئة EncryptionConfiguration في cluster

مثال لـ EncryptionConfiguration:

```
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - kms:
          name: myKmsPlugin
          endpoint: unix:///tmp/kms.socket
          cachesize: 100
          timeout: 3s
      - identity: {} # fallback
```
---
## Video 3:  ربط الـ Secrets مع الـ Pods
## ربط الـ Secrets مع الـ Pods

هناك عدة طربق لربط الـ Secrets مع الـ Pods:

### 1. كمتغيرات بيئة (Environment Variables)

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: password
```
### 2. كملفات (Volume Mount)

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: my-secret
      items:
      - key: username
        path: my-group/my-username
```
### 3. كملف منفرد داخل Volume

```
volumes:
- name: secret-volume
  secret:
    secretName: my-secret
    # يتم تخزين كل مفتاح كملف منفصل
```
---

## Video 4: أفضل الممارسات

## أفضل الممارسات (RBAC، CSI، External Secrets)

### 1. استخدام RBAC (Role-Based Access Control)

- تقييد الوصول إلى الـ Secrets باستخدام الأذونات الدقيقة
- مبدأ الامتياز الأقل (Least Privilege Principle)
    

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: secret-access-role
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["my-secret"]
  verbs: ["get", "watch", "list"]
```
### أفضل ممارسات إضافية:

- تدوير الـ Secrets بشكل دوري
- تدقيق الوصول إلى الـ Secrets (auditing)
- تجنب استخدام الـ Secrets في سجلات التطبيقات (logs)
- استخدام أدوات مسح الأمان للكشف عن الـ Secrets المعرضة للخطر
    



## العرض العملي

في هذا القسم العملي، سنقوم بما يلي:

1. **إنشاء Secrets بأنواعها المختلفة**:
    ```
    # إنشاء Opaque secret من ملف
    kubectl create secret generic app-secret \
      --from-literal=username=admin \
      --from-literal=password=secret
    
    # إنشاء TLS secret
    kubectl create secret tls tls-secret \
      --cert=path/to/cert/file \
      --key=path/to/key/file
 ``   
1. **ربط Secrets مع Pods**:
    - إنشاء pod يستخدم Secrets كمتغيرات بيئة
    - إنشاء pod يستخدم Secrets كملفات mounted
2. **إدارة Secrets باستخدام أدوات خارجية**:
    - تهيئة External Secrets Operator
    - مزامنة Secrets من AWS Secrets Manager        
3. **تطبيق أفضل ممارسات الأمان**:
    - إنشاء أدوار RBAC للتحكم في الوصول إلى Secrets
    - استخدام Network Policies لعزل الـ Pods التي تصل إلى Secrets



## مصادر خطة الدراسة

### الوثائق الرسمية:

- [Kubernetes Secrets Documentation](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Managing Secrets in Kubernetes](https://kubernetes.io/docs/tasks/configmap-secret/)
- [Encryption at Rest for Secrets](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

### مقالات ودروس:

- [Kubernetes Secrets Management Best Practices](https://blog.aquasec.com/kubernetes-security-best-practices)
- [Using External Secrets with Kubernetes](https://github.com/external-secrets/external-secrets)

### أدوات مساعدة:

- [External Secrets Operator](https://external-secrets.io/)
- [Secrets Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/)
- [Kubernetes Vault Integration](https://www.vaultproject.io/docs/platform/k8s)

### مشاريع عملية:

- إنشاء تطبيق يستخدم Secrets للاتصال بقاعدة بيانات
- تأمين تطبيق ويب باستخدام TLS Secrets
- بناء خط أنابيب CI/CD مع إدارة آمنة للأسرار



في الختام، تعتبر إدارة الأسرار بشكل آمن أحد الجوانب الحرجة في إدارة تطبيقات Kubernetes. من خلال فهم المفاهيم والتقنيات التي تم تناولها في هذه الوحدة، ستتمكن من حماية البيانات الحساسة في تطبيقاتك بشكل فعال وتطبيق أفضل ممارسات الأمان في بيئة Kubernetes.
