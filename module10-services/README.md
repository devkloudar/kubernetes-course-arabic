## نظرة عامة على الوحدة

مرحباً بكم في الوحدة الحادية عشرة من دورة كوبرنيتيس. في هذه الوحدة، سنغطي أحد المفاهيم الأساسية والأكثر أهمية في كوبرنيتيس: **الـ Services**.

الـ Services هي وسيلة أساسية لتعريض التطبيقات التي تعمل داخل الـ Pods للشبكة، سواء داخل الكلستر أو خارجه. نظراً لأن الـ Pods في كوبرنيتيس عابرة (يمكن إنشاؤها وحذفها ديناميكياً)، فإن عناوين IP الخاصة بها تتغير باستمرار. هنا يأتي دور الـ Services لتوفير عنوان ثابت وموثوق للوصول إلى مجموعة من الـ Pods.

في هذه الوحدة، سنستكشف:

- أنواع الـ Services المختلفة وكيفية عملها
- متى نستخدم كل نوع من أنواع الـ Services
- كيفية تكوين الـ Services وتوصيلها بالـ Pods
- مفاهيم متقدمة مثل Headless Services
- تطبيق عملي لأنواع الـ Services المختلفة



## ClusterIP

دعنا الأن نتكلم على أول نوع من ال Services وهو ClusterIP 

وهو النوع الافتراضي على كوبيرنيتيس، يعني إن قمت بإنشاء Service معين دون تحديد النوع الذي تريده، فنوع السيرفيس الذي سيُنَشأ هو من نوع ClusterIP
### ما هو ClusterIP Service؟

- **ClusterIP** هو النوع الافتراضي من الـ Services في كوبرنيتيس
    
- يعرض الـ Service على عنوان IP داخلي داخل الكلستر
- يمكن فقط للـ Pods والتطبيقات داخل الكلستر الوصول إلى هذا الـ Service
- مثالي للاتصال بين ميكروسيرفيس داخل الكلستر


### كيف يعمل؟

- يقوم الـ Service بإنشاء عنوان IP ثابت وداخلي
- يوجه الحركة إلى الـ Pods التي تطابق الـ selector المحدد
- يستخدم الـ kube-proxy لتنفيذ التوجيه إما عبر iptables أو ipvs

### مثال تكوين ClusterIP:
```
apiVersion: v1
kind: Service
metadata:
  name: my-internal-service
spec:
  type: ClusterIP  # هذا هو النوع الافتراضي، يمكن حذفه
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80      # port الذي يعرضه الـ Service
      targetPort: 8080  # port على الـ Pods
```
### حالات الاستخدام:

- الاتصال بين ميكروسيرفيس داخل الكلستر
- خدمات قاعدة بيانات داخلية
- واجهات API للاستخدام الداخلي فقط
- أي خدمة لا تحتاج إلى التعريض للعالم الخارجي



## NodePort

**NodePort** هو نوع من أنواع الـ Services 
في كوبرنيتيس الذي يعرض تطبيقك على منفذ (port) ثابت في نطاق محدد (30000-32767) على **كل عقدة (Node)** في الكلستر. هذا يسمح بالوصول إلى الخدمة من خارج الكلستر باستخدام عنوان IP أي عقدة متبوعة برقم الـ NodePort.
### ما هو NodePort Service؟

- **NodePort** يعرض الـ Service على port ثابت على كل node في الكلستر
    
- يمكن الوصول إلى الـ Service من خارج الكلستر باستخدام `<NodeIP>:<NodePort>`
- يقوم تلقائياً بإنشاء ClusterIP داخلي أيضاً

### كيف يعمل؟

- يفتح port ثابت في النطاق 30000-32767 على كل node
- أي حركة تصل إلى هذا port على أي node يتم توجيهها إلى الـ Service
- الـ Service بدوره يوجه الحركة إلى الـ Pods المناسبة

### مثال تكوين NodePort:


```
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80           # port داخلي للـ Service
      targetPort: 8080   # port على الـ Pods
      nodePort: 30007    # port على الـ Nodes (اختياري، إذا لم يتم تحديده يختار عشوائياً)
```

### 1. إنشاء تطبيق بسيط (Deployment)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-container
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
```
### 2. إنشاء خدمة NodePort

```
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport-service
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
    - protocol: TCP
      port: 80          # المنفذ الداخلي للخدمة
      targetPort: 80    # المنفذ الذي يعمل عليه الـ Pod
      nodePort: 30007   # المنفذ على الـ Node (اختياري، بين 30000-32767)
```

### 1. حفظ الملف باسم `nodeport-example.yaml`

### 2. تطبيق التكوين

```
kubectl apply -f nodeport-example.yaml
```
### 3. التحقق من الخدمة

```
kubectl get services
kubectl describe service web-nodeport-service
```
### 4. الوصول للتطبيق

باستخدام أي من الطرق التالية:
```
- `http://<ANY_NODE_IP>:30007`
    
- `curl http://<ANY_NODE_IP>:30007`
``` 

##  عرض المعلومات المفصلة

```
# عرض الـ Services
kubectl get svc

# عرض تفاصيل الـ Service
kubectl describe svc web-nodeport-service

# عرض الـ Pods والعنوانات الداخلية
kubectl get pods -o wide

# عرض عناوين الـ Nodes
kubectl get nodes -o wide
```
##  مثال للوصول للتطبيق

إذا كان عنوان الـ Node هو `192.168.1.100`، يمكن الوصول للتطبيق عبر:

```

curl http://192.168.1.100:30007
```
أو فتح المتصفح والذهاب إلى:

```

http://192.168.1.100:30007
```
##  هيكل عمل NodePort


المستخدم → NodeIP:NodePort → Service → Pod
    ↓
أي node في الكلستر → نفس المنفذ → نفس الخدمة → نفس التطبيق

##  إعدادات متقدمة

### تحديد نطاق مخصص لـ NodePort (يتطلب صلاحيات)

```
apiVersion: v1
kind: Service
metadata:
  name: custom-nodeport-service
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 31000 # يجب أن يكون بين 30000-32767
```
### خدمة متعددة المنافذ

```
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
      nodePort: 30443
```
##  التنظيف

```
kubectl delete -f nodeport-example.yaml
# أو
kubectl delete deployment web-app
kubectl delete service web-nodeport-service
```
### حالات الاستخدام:



- التطوير والاختبار المحلي
- عندما تحتاج إلى تعريض خدمة للعالم الخارجي ولكن لا يتوفر LoadBalancer
- للوصول المباشر إلى الخدمات من خارج الكلستر



## LoadBalancer

### ما هو LoadBalancer Service؟

- **LoadBalancer** يعرض الـ Service خارجياً باستخدام موفر cloud load balancer
    
- يعمل على إنشاء Load Balancer خارجي في موفر السحابة (AWS, GCP, Azure, etc.)
- يوفر عنوان IP خارجي ثابت للوصول إلى الخدمة

### كيف يعمل؟

- يتكامل مع موفر السحابة لإنشاء Load Balancer
- يوجه الحركة من Load Balancer إلى الـ Nodes على الـ NodePort
- ثم يتم توجيه الحركة إلى الـ Pods المناسبة


### مثال تكوين LoadBalancer:

```
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"  # لـ AWS NLB
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  # يمكن تحديد عنوان IP ثابت إذا كانت السحابة تدعمه
  # loadBalancerIP: "78.11.24.19"
```
### حالات الاستخدام:

- تعريض الخدمات للعالم الخارجي في بيئات الإنتاج
    
- عندما تحتاج إلى موازنة حمل خارجية
    
- للخدمات التي تحتاج إلى عنوان IP خارجي ثابت
    



##  العرض العملي


في هذا القسم العملي، سنقوم بما يلي:

### 1. إنشاء وتجربة ClusterIP Service


```
# إنشاء تطبيق نموذجي
kubectl create deployment nginx --image=nginx --replicas=3

# إنشاء ClusterIP Service
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF

# اختبار الوصول من داخل الكلستر
kubectl run test-$RANDOM --rm -it --image=busybox -- /bin/sh
wget -q -O- nginx-clusterip.default.svc.cluster.local

# مشاهدة تفاصيل الـ Service
kubectl describe svc nginx-clusterip
```
### 2. إنشاء وتجربة NodePort Service

```
# إنشاء NodePort Service
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
EOF

# معرفة عناوين الـ Nodes
kubectl get nodes -o wide

# اختبار الوصول من خارج الكلستر (استبدل <node-ip> بعنوان node الفعلي)
curl http://<node-ip>:30080
```

## مصادر خطة الدراسة

### الوثائق الرسمية:

- [Kubernetes Services Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
    
- [Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)
    
- [Headless Services](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)
    
- [ExternalName Services](https://kubernetes.io/docs/concepts/services-networking/service/#externalname)
    

### مقالات ودروس:

- [Kubernetes Networking Guide](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
    
- [Understanding Kubernetes Services](https://medium.com/google-cloud/understanding-kubernetes-services-i7a0079c7fb5)
    
- [Kubernetes Service Types Explained](https://www.magalix.com/blog/kubernetes-services-101)
    

### أدوات مساعدة:

- [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
    
- [Network Policy Editor](https://cilium.io/blog/2021/02/09/network-policy-editor)
    
- [Service Mesh Comparison](https://servicemesh.es/)
    

### مشاريع عملية:

- بناء تطبيق ميكروسيرفيس مع اتصال بين الخدمات باستخدام ClusterIP
    
- تعريض تطبيق ويب للعالم الخارجي باستخدام LoadBalancer
    
- تكوين اتصال آمن مع قاعدة بيانات خارجية باستخدام ExternalName
    
- إعداد Stateful application مع Headless Services
    
- مراقبة وتحليل أداء الـ Services باستخدام أدوات المراقبة
    

---

في الختام، تعتبر الـ Services من المكونات الأساسية في كوبرنيتيس التي توفر اتصالاً موثوقاً ومستقراً بين التطبيقات داخل الكلستر ومع العالم الخارجي. فهم أنواع الـ Services المختلفة ومتى نستخدم كل منها هو أمر بالغ الأهمية لتصميم وتشغيل التطبيقات بشكل فعال في بيئة كوبرنيتيس.
