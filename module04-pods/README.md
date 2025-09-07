
## 📚 مصادر خطة الدراسة – الوحدة 4: الـ Pods

فيما يلي مجموعة من الموارد الموثوقة والمفيدة لتعميق فهمك بالـ Pods في Kubernetes:

### 📘 الوثائق الرسمية (Official Docs)
- [Kubernetes Pods (Official)](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Multi-Container Pods](https://kubernetes.io/docs/concepts/workloads/pods/#multi-container-pods)
- [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Ephemeral Containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)

### 💡 مصادر إضافية ومقالات متقدمة
- [How Kubernetes Pods Work – Learnk8s Blog](https://learnk8s.io/kubernetes-pods)
- [Pod Troubleshooting Guide](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pod-replication-controller/)

### 🛠️ أوامر `kubectl` المهمة في هذه الوحدة
```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl exec -it <pod-name> -- /bin/sh
kubectl logs <pod-name>
kubectl apply -f pod.yaml
```


