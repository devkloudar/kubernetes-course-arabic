 ## نظرة عامة على الوحدة

في هذه الوحدة، سنستكشف:

- مفهوم Ingress وكيفية عمله
- ما هو Ingress Controller وكيفية إعداده
- كيفية تعريف قواعد التوجيه باستخدام Ingress Resources
- إعداد TLS وتأمين الاتصالات باستخدام الشهادات
- تطبيق عملي لأنواع مختلفة من قواعد Ingress



## مفهوم Ingress Controller
- الـ Ingress في Kubernetes هو مورد (Resource) يسمح بتحديد قواعد التوجيه (Routing Rules) للترافيك الخارجي نحو الخدمات (Services) داخل الكلاستر.
- لكن... الـ Ingress لا يعمل وحده، بل يحتاج إلى **Ingress Controller** وهو عبارة عن تطبيق يستمع لهذه القواعد ويقوم بتطبيقها.
- أنواع Ingress Controllers:
  - NGINX Ingress Controller 
  - Traefik
  - HAProxy
  - Istio Gateway

### كيف يعمل؟

```
المستخدم → Ingress Controller → Ingress Resource → Service → Pods
```

1. المستخدم يرسل طلبًا HTTP/HTTPS إلى عنوان الكلاستر (أو الـ LoadBalancer).
2. الـ Ingress Controller يتلقى الطلب.
3. يفحص القواعد المحددة في الـ Ingress Resource.
4. يعيد توجيه الطلب إلى الـ Service المناسب (وفقًا للـ Host أو الـ Path).

---

## إعداد NGINX Ingress Controller

### على Minikube:
```bash
minikube addons enable ingress
```

### على Cluster خارجي:
نستخدم Helm:
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx
```

### التحقق من التثبيت:
```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

---
## تعريف قواعد Ingress (Host, Path)

### إنشاء Ingress Resource أساسي:


```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```
### التوجيه based على المضيف (Host-based):

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  rules:
  - host: app1.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
  - host: app2.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```
### التوجيه based على المسار (Path-based):
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /static
        pathType: Prefix
        backend:
          service:
            name: static-service
            port:
              number: 9000
```
### أنواع pathType:

- **Prefix**: يطابق المسارات التي تبدأ بالقيمة المحددة
- **Exact**: يطابق المسار بالتحديد
- **ImplementationSpecific**: يعتمد على تنفيذ الـ Ingress Controller

---

## TLS وإدارة الشهادات

### إعداد TLS مع Ingress:


```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
      - myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```
### إنشاء TLS Secret:

```
# إنشاء شهادة ذاتية التوقيع (للتجربة فقط)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=myapp.example.com/O=myapp"

# إنشاء Kubernetes secret
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key

# التحقق من الـ secret
kubectl describe secret tls-secret

### استخدام Let's Encrypt مع cert-manager:
```
#### تثبيت cert-manager:

```
# إضافة Helm repository
helm repo add jetstack https://charts.jetstack.io
helm repo update

# تثبيت cert-manager
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.3 \
  --set installCRDs=true
```
#### إنشاء ClusterIssuer:
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```
#### Ingress مع annotations لـ cert-manager:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secured-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
      - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```
---

## العرض العملي


### 1. تثبيت وتكوين NGINX Ingress Controller

```
# تثبيت NGINX Ingress Controller باستخدام Helm
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

# الانتظار حتى يكون جاهزاً
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s

# الحصول على عنوان IP الخارجي
kubectl get svc -n ingress-nginx
```
### 2. نشر تطبيقات نموذجية

```
# نشر تطبيق ويب 1
kubectl create deployment web-app1 --image=nginx
kubectl expose deployment web-app1 --port=80 --name=web-service1

# نشر تطبيق ويب 2
kubectl create deployment web-app2 --image=httpd
kubectl expose deployment web-app2 --port=80 --name=web-service2

# نشر تطبيق API
kubectl create deployment api-app --image=containous/whoami
kubectl expose deployment api-app --port=80 --name=api-service
```
### 3. إنشاء قواعد Ingress مختلفة
```
# إنشاء ملف host-based-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  rules:
  - host: app1.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service1
            port:
              number: 80
  - host: app2.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service2
            port:
              number: 80
---
```
# إنشاء ملف path-based-ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /web1
        pathType: Prefix
        backend:
          service:
            name: web-service1
            port:
              number: 80
      - path: /web2
        pathType: Prefix
        backend:
          service:
            name: web-service2
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```
```

# تطبيق قواعد Ingress
kubectl apply -f host-based-ingress.yaml
kubectl apply -f path-based-ingress.yaml

# مشاهدة تفاصيل Ingress
kubectl get ingress
kubectl describe ingress host-based-ingress
```
### 4. اختبار قواعد التوجيه

```
# الحصول على عنوان IP الـ Ingress Controller
INGRESS_IP=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

# اختبار التوجيه based على المضيف
curl -H "Host: app1.local" http://$INGRESS_IP
curl -H "Host: app2.local" http://$INGRESS_IP

# اختبار التوجيه based على المسار
curl http://$INGRESS_IP/web1
curl http://$INGRESS_IP/web2
curl http://$INGRESS_IP/api

# إضافة سجلات DNS محلية (على الجهاز المحلي)
echo "$INGRESS_IP app1.local app2.local myapp.local" | sudo tee -a /etc/hosts
```
### 5. إعداد TLS وتأمين الاتصالات

```

# إنشاء شهادة TLS للتجربة
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=myapp.local"

# إنشاء TLS secret
kubectl create secret tls myapp-tls --cert=tls.crt --key=tls.key

# إنشاء Ingress مع TLS
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
      - myapp.local
    secretName: myapp-tls
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service1
            port:
              number: 80
EOF

# اختبار HTTPS
curl -k https://myapp.local
```
### 6. استخدام annotations المتقدمة

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: advanced-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
spec:
  tls:
  - hosts:
      - myapp.local
    secretName: myapp-tls
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /api(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```
---

## مصادر خطة الدراسة

### الوثائق الرسمية:

- [Kubernetes Ingress Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [Ingress API Reference](https://kubernetes.io/docs/reference/kubernetes-api/service-resources/ingress-v1/)

### مقالات ودروس:

- [Kubernetes Ingress Tutorial](https://www.digitalocean.com/community/tutorials/an-introduction-to-kubernetes-ingress)
- [NGINX Ingress Annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)
- [Cert-manager Documentation](https://cert-manager.io/docs/)

### أدوات مساعدة:

- [Helm Charts](https://helm.sh/)
- [Cert-manager](https://cert-manager.io/)
- [Kustomize](https://kustomize.io/)

- [Kubernetes Official Ingress Docs](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [NGINX Ingress Controller GitHub](https://github.com/kubernetes/ingress-nginx)
- [TLS Secrets in Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/#tls-secrets)
- [kubectl Cheat Sheet - Ingress](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### مشاريع عملية:

- إعداد Ingress Controller في بيئة حقيقية
- تكوين TLS مع Let's Encrypt
- إنشاء قواعد توجيه معقدة لتطبيق متعدد الخدمات
- مراقبة وأداء الـ Ingress Controller
- تطبيق سياسات الأمان والتحكم في الوصول

---

في الختام، يعد Ingress مكوناً حيوياً في أي بنية تحتية لKubernetes، حيث يوفر طريقة قوية ومرنة لإدارة الدخول إلى التطبيقات من العالم الخارجي. من خلال إتقان المفاهيم والتقنيات التي تم تناولها في هذه الوحدة، ستتمكن من تصميم وتنفيذ أنظمة دخول متطورة وآمنة لتطبيقاتك في بيئة Kubernetes.


