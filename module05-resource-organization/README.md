## نظرة عامة على الفصل


سنتعرف على:

- كيف نُعطي تعريفات ذكية للموارد باستخدام **Labels**.
- كيف نُحدد ونختار الموارد بدقة باستخدام **Selectors**.
- كيف نضيف ملاحظات وصفية مفيدة عبر **Annotations**.
- كيف نقسّم الكلاستر إلى بيئات منطقية معزولة باستخدام **Namespaces**.

كل أداة من هذه الأدوات تفتح لنا أبوابًا من **التحكم، التنظيم، والمرونة** في إدارة الكلاستر.


# Labels



 ## ما هي الـ Labels؟**

- عبارة عن أزواج مفتاح/قيمة (key=value).
- تُستخدم لتصنيف الموارد: Pods، Services، Deployments… إلخ.
- لا تؤثر على سلوك الموارد، بل تُستخدم لأغراض التصفية والتنظيم.

📌 مثال:
```yaml
labels:
  app: frontend
  env: production
  ```

##  أين تُستخدم الـ Labels؟

- اختيار الموارد باستخدام `kubectl`.
- ربط الـ Deployments بالـ Pods.
- التحكم في التحديثات، الاستهداف في Services.
- تتبع الموارد حسب البيئة أو الفريق أو الوظيفة.
```
# إنشاء Pod مع Label
kubectl run myapp --image=nginx --labels="env=dev,team=web"

# مشاهدة الـ Labels
kubectl get pods --show-labels

# تعديل أو إضافة Label
kubectl label pod myapp version=1.0

# حذف Label
kubectl label pod myapp version-
```


## الخلاصة

- الـ Labels = نظام تصنيف مرن.
- مفتاح لتنظيم وإدارة الموارد بذكاء.
- ستُستخدم لاحقًا مع **Selectors**.


# Selectors – اختيار الموارد الذكي



### ما هو الـ Selector؟

وسيلة لاختيار الموارد بناءً على القيم الموجودة في الـ Labels.

هناك نوعان أساسيان:

1. **Equality-based Selectors**  
   - `key=value` أو `key!=value`

2. **Set-based Selectors**  
   - `in`, `notin`, `exists`, `!`



### أمثلة عملية

```bash
# اختيار كل الـ Pods التي لديها label env=dev
kubectl get pods -l env=dev

# اختيار كل الـ Pods التي ليست لها label team=backend
kubectl get pods -l 'team!=backend'

# استخدام Set Selector
kubectl get pods -l 'env in (dev, test)'
```



### 🔄 الاستخدام في الموارد

```yaml
selector:
  matchLabels:
    app: frontend
```

أو:

```yaml
selector:
  matchExpressions:
    - key: env
      operator: In
      values:
        - dev
        - staging
```



### ✅ الخلاصة
- Selectors + Labels = إدارة ديناميكية ومرنة.
- تُستخدم في Services، Deployments، ReplicaSets.



# الفرق بين Labels و Annotations

في Kubernetes، لدينا وسيلتان لإرفاق بيانات وصفية بالموارد:
- **Labels**: للفرز والتصفية.
- **Annotations**: للملاحظات والأغراض الوصفية فقط.



### ما هي الـ Annotations؟

- تستخدم لإضافة معلومات وصفية لا تُستخدم في التصفية أو التحديد.
- قد تكون طويلة أو تحتوي بيانات مشفّرة.
- تُستخدم من أدوات خارجية: CI/CD، Monitoring، وغيرها.



### مثال عملي

```yaml
metadata:
  name: mypod
  labels:
    app: nginx
  annotations:
    created-by: "ahmed"
    description: "Test Pod"
    prometheus.io/scrape: "true"
```



### أوامر عملية

```bash
# قراءة الـ Annotations
kubectl get pod mypod -o jsonpath='{.metadata.annotations}'

# تعديل Annotation
kubectl annotate pod mypod description="Test from CI pipeline"

# حذف Annotation
kubectl annotate pod mypod description-
```



### الخلاصة

|      | Labels           | Annotations        |
|------|------------------|--------------------|
| الغرض | تصنيف/اختيار     | وصف/ملاحظات فقط    |
| تُستخدم من | Kubernetes + أدوات | أدوات فقط         |
| مثال | `env=prod`       | `created-by=admin` |

- استخدم **Labels** للإدارة والتصفية.
- استخدم **Annotations** لإضافة ملاحظات داخلية مهمة.



# Namespaces – فصل البيئات في Kubernetes

### 📌 المقدمة
في Kubernetes، عندما تبدأ بزيادة عدد الموارد، يصبح من المهم عزلها من بعضها البعض.

وهنا تأتي أهمية **Namespaces**.

---

### ما هو الـ Namespace؟

- طريقة لتقسيم الكلاستر إلى مساحات عمل منطقية.
- كل Namespace يمكن أن يحتوي على مجموعة مستقلة من الموارد.
- يساعد في فصل البيئات (dev, staging, prod) أو الفرق أو التطبيقات.



### المزايا

- عزل الموارد (Pods، Services…).
- سهولة التحكم بالصلاحيات (RBAC).
- تنظيم الكلاستر بطريقة منظمة ومنطقية.



### أوامر مفيدة

```bash
# إنشاء Namespace
kubectl create namespace dev

# تنفيذ أمر داخل Namespace
kubectl get pods -n dev

# تعيين Namespace كافتراضي
kubectl config set-context --current --namespace=dev

# حذف Namespace
kubectl delete namespace dev
```

---

### الممارسات الجيدة

- استخدم Namespace لكل بيئة (`dev`, `prod`, `test`).
- افصل الفرق عبر Namespaces.
- لا تضع كل شيء في `default`.

---

### الخلاصة
- الـ Namespaces = العزل والتنظيم.
- أساس إدارة بيئات متعددة داخل نفس الكلاستر.




## 📚 مصادر خطة الدراسة - الفصل 5: تنظيم الموارد في Kubernetes

### 🔹 Labels & Selectors
- <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/" target="_blank">📖 المفاهيم الأساسية: Labels و Selectors</a>  
- <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/" target="_blank">📖 أفضل الممارسات في اختيار Labels</a>  
- <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/" target="_blank">📖 Field Selectors</a>  
- <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/label-selectors/" target="_blank">📖 Label Selectors</a>  

### 🔹 Annotations
- <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/" target="_blank">📖 المفاهيم الأساسية: Annotations</a>  
- <a href="https://cloud.google.com/kubernetes-engine/docs/how-to/creating-managing-labels#using_annotations" target="_blank">📖 استخدام Annotations في Google Kubernetes Engine</a>  

### 🔹 Namespaces
- <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/" target="_blank">📖 المفاهيم الأساسية: Namespaces</a>  
- <a href="https://kubernetes.io/docs/tasks/administer-cluster/namespaces/" target="_blank">📖 إدارة Namespaces في Kubernetes</a>  
- <a href="https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/" target="_blank">📖 شرح عملي: Namespaces Walkthrough</a>  

### 🔹 موارد إضافية
- <a href="https://kubernetes.io/docs/reference/kubectl/cheatsheet/" target="_blank">📖 kubectl Cheatsheet</a>  
- <a href="https://www.youtube.com/watch?v=Be5XA1G3OfU" target="_blank">🎥 فيديو: Kubernetes Labels & Selectors Explained</a>  
- <a href="https://www.youtube.com/watch?v=K0jR2VE0-8k" target="_blank">🎥 فيديو: Kubernetes Namespaces Simplified</a>  

---

✅ هذه المصادر ستساعدك على:  
1. فهم دور **Labels** و **Selectors** في تنظيم وإدارة الموارد.  
2. معرفة كيفية استخدام **Annotations** لإضافة بيانات وصفية.  
3. إدارة وفصل البيئات باستخدام **Namespaces**.  
4. تطبيق هذه المفاهيم عمليًا عبر أوامر **kubectl**.  

