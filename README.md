# Ubuntu 22.04 Üzerinde "kubeadm" ile Kubernetes Cluster Kurulumu

## Kubernetes Nedir ?

Kubernetes (k8s) Google tarafından geliştirilen bir projedir ve açık kaynaktır. Kubernetes'in temel işlevi konteyner'da çalışmak üzere geliştirilen uygulamaları orkestra etmektir. Orkestra etmek işlevi biraz açalım.

- Kubernetes cluster "kendinden sağlıklıdır". Eğer konteynerlerinizden birisi ölürse kubernetes ortamı hemen tekrardan yenisi oluşturuyor.

- Kubernetes ***Horizontal Scaling*** sağlar. Mevcut kaynak durumunuza göre konteynerlerin sayısını arttırıp azaltabiliyor.

- Kubernetes ***Compute Scheduling*** sağlar. Mevcut kaynak ihtiyacının belirlenmesini ve konteyner özelinde kaynakların atanmasını otomatikleştirir.

- Kubernetes LoadBalance sağlar. Mevcut uygulamalar üzerindeki yükü üzerinde çalıştıkları konteynerler arasında dağıtır.

- Kubernetes Nameserver sağlar. Mevcut konteynerlere erişebilmek için DNS tanımları yapar.

- Kubernetes otomatik Rollout/Rollback sağlar. Kubernetes her yeni deployment'da veya mevcut deployment'daki her değişiklikte önceden belirlenmiş desired duruma göre otomatik olarak bütün instance'ları ve bileşenleri yeniden oluşturur. Deployment sırasında bir sorun yaşandığında ise otomatik olarak bir önceki sorunsuz çalışan deployment'a geri çekilir.

- Kubernetes gizli bilgilerin ve yapılandırma ayarlarının yönetimini sağlar. Uygulama yapaılandırma ayarları, parolalar, API anahtarları, sertifikalar vb bilgilerin yönetimini otomatikleştirir.




