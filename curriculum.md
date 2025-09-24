
# 📚 محتوى الدورة

## الفصل 0: مقدمة الدورة
- مقدمة إلى الدورة  
- نظرة عامة على الوحدة  
- ماذا ستتعلم في هذه الدورة  
- لمن هي هذه الدورة + طريقة التعلم  
- مصادر خطة الدراسة  



## الفصل 1: مقدمة إلى الحاويات و Kubernetes
- نظرة عامة على الوحدة  
- ما هي الحاويات (Containers)؟  
- فوائد استخدام الحاويات  
- مفاهيم Microservices  
- نظرة سريعة على Docker  
- ما هو Kubernetes ولماذا نحتاجه؟  
- مقارنة مع أنظمة التشغيل التقليدية والتطبيقات  
- العرض العملي (Demo)  
- مصادر خطة الدراسة  



## الفصل 2: أساسيات Kubernetes
- نظرة عامة على الوحدة  
- مفهوم الـ Cluster (العُقد Master/Worker)  
- المكونات الأساسية (API Server, Scheduler, Controller Manager, etcd, Kubelet, Kube-proxy)  
- العرض العملي  
- مصادر خطة الدراسة  



## الفصل 3: تثبيت Kubernetes وبداية العمل عليه
- نظرة عامة على الوحدة  
- تجهيز بيئة العمل باستخدام Minikube  
- تثبيت Minikube على الحاسوب الخاص  
  - MacOS  
  - Linux  
  - Windows  
- ما هو kubectl ولماذا نحتاجه؟  
- تشغيل أول Cluster واختبار أول Pod  
- مصادر خطة الدراسة  



## الفصل 4: الـ Pods
- نظرة عامة على الوحدة  
- ما هو الـ Pod ولماذا يعتبر أصغر وحدة؟  
- مكونات الـ Pod (Containers, Volumes, Networking)  
- أنواع الـ Pods (Single vs Multi-Container)  
- Init Containers  
- Ephemeral Containers  
- إدارة الـ Pods باستخدام kubectl  
- الممارسات الجيدة في تصميم الـ Pods  
- العرض العملي  
  - Demo 1: إنشاء Pod على Kubernetes  
  - Demo 2: إنشاء Pod بحاويتين  
  - Demo 3: Init Containers  
  - Demo 4: إنشاء Ephemeral Containers  
- مصادر خطة الدراسة  



## الفصل 5: تنظيم الموارد في Kubernetes
- نظرة عامة على الوحدة  
- Labels  
- Selectors  
- Annotations  
- Namespaces  
- العرض العملي  
- مصادر خطة الدراسة  



## الفصل 6: موارد تشغيل التطبيقات (Workload Resources)
- نظرة عامة على الوحدة  
- ReplicaSet  
- Deployments  
- DaemonSet  
- StatefulSet  
- Jobs و CronJobs  
- العرض العملي  
- مصادر خطة الدراسة  



## الفصل 7: إدارة الحالة باستخدام الـ Deployments
- نظرة عامة على الوحدة  
- إنشاء وتحديث Deployments  
- Rolling Updates و Rollbacks  
- إدارة النسخ (Replicas)  
- استراتيجيات النشر  
- العرض العملي (Demo)  
- مصادر خطة الدراسة  



## الفصل 8: ConfigMaps
- نظرة عامة على الوحدة  
- ما هو ConfigMap؟  
- إنشاء ConfigMap من ملف أو متغيرات  
- حقن القيم في الـ Pods (Env, Files)  
- تحديث ConfigMaps وإعادة التحميل  
- أفضل الممارسات  
- العرض العملي  
- مصادر خطة الدراسة  



## الفصل 9: Secrets
- نظرة عامة على الوحدة  
- ما هو Secret؟  
- أنواع الـ Secrets (Opaque, TLS, Docker Registry)  
- التشفير (base64 و KMS)  
- ربط الـ Secrets مع الـ Pods  
- أفضل الممارسات (RBAC، CSI، External Secrets)  
- العرض العملي  
- مصادر خطة الدراسة

- 
## الفصل 10: الـ Services
- نظرة عامة على الوحدة  
- ClusterIP  
- NodePort  
- LoadBalancer  
- ExternalName  
- Headless Services  
- العرض العملي  
- مصادر خطة الدراسة  



## الفصل 11: Ingress
- نظرة عامة على الوحدة  
- مفهوم Ingress Controller  
- إعداد NGINX Ingress Controller  
- تعريف قواعد Ingress (Host, Path)  
- TLS وإدارة الشهادات  
- العرض العملي  
- مصادر خطة الدراسة


   ## الفصل 12: التخزين (Volumes & Storage)
- نظرة عامة على الوحدة  
- أنواع الـ Volumes (emptyDir, hostPath, configMap, secret)  
- Persistent Volume (PV) و Persistent Volume Claim (PVC)  
- StorageClass و Dynamic Provisioning  
- CSI (Container Storage Interface)  
- العرض العملي  
- مصادر خطة الدراسة  

