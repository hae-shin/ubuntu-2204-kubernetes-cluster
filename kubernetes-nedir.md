
## Kubernetes Nedir ?

Kubernetes (k8s) Google tarafından geliştirilen bir projedir ve açık kaynaktır. Kubernetes'in temel işlevi konteyner'da çalışmak üzere geliştirilen uygulamaları orkestra etmektir. Konteyner teknolojisini ve orkestra etmek işlevini biraz açalım.

![image](https://user-images.githubusercontent.com/116150600/201118995-c5b4283f-97ef-4284-99f1-47bc9709b7ab.png)


- Kubernetes cluster ***Self-Healing***. Eğer konteynerlerinizden birisi ölürse kubernetes ortamı hemen tekrardan yenisi oluşturuyor.

- Kubernetes ***Horizontal Scaling*** sağlar. Mevcut kaynak durumunuza göre konteynerlerin sayısını arttırıp azaltabiliyor.

- Kubernetes ***Compute Scheduling*** sağlar. Mevcut kaynak ihtiyacının belirlenmesini ve konteyner özelinde kaynakların atanmasını otomatikleştirir.

- Kubernetes ***LoadBalance*** sağlar. Mevcut uygulamalar üzerindeki yükü üzerinde çalıştıkları konteynerler arasında dağıtır.

- Kubernetes ***Nameserver*** sağlar. Mevcut konteynerlere erişebilmek için DNS tanımları yapar.

- Kubernetes otomatik ***Rollout/Rollback*** sağlar. Kubernetes her yeni deployment'da veya mevcut deployment'daki her değişiklikte önceden belirlenmiş desired duruma göre otomatik olarak bütün instance'ları ve bileşenleri yeniden oluşturur. Deployment sırasında bir sorun yaşandığında ise otomatik olarak bir önceki sorunsuz çalışan deployment'a geri çekilir.

- Kubernetes gizli bilgilerin ve yapılandırma ayarlarının yönetimini sağlar. Uygulama yapaılandırma ayarları, parolalar, API anahtarları, sertifikalar vb bilgilerin yönetimini otomatikleştirir.

- Kubernetes ***Volume Management*** sağlar. Uygulamaların kullanımına göre depolama ihtiyaçlarını karşılar ve otomatikleştirir.

**Kaynak:** https://kubernetes.io/docs/concepts/overview/
