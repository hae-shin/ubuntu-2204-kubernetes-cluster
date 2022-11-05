# Ubuntu 22.04 Üzerinde "kubeadm" ile Kubernetes Cluster Kurulumu
![This is an image](https://github.com/hae-shin/ubuntu-2204-kubernetes-cluster/blob/main/kubernetes.png)

## Kubernetes Nedir ?

Kubernetes (k8s) Google tarafından geliştirilen bir projedir ve açık kaynaktır. Kubernetes'in temel işlevi konteyner'da çalışmak üzere geliştirilen uygulamaları orkestra etmektir. Orkestra etmek işlevi biraz açalım.

- Kubernetes cluster ***Self-Healing***. Eğer konteynerlerinizden birisi ölürse kubernetes ortamı hemen tekrardan yenisi oluşturuyor.

- Kubernetes ***Horizontal Scaling*** sağlar. Mevcut kaynak durumunuza göre konteynerlerin sayısını arttırıp azaltabiliyor.

- Kubernetes ***Compute Scheduling*** sağlar. Mevcut kaynak ihtiyacının belirlenmesini ve konteyner özelinde kaynakların atanmasını otomatikleştirir.

- Kubernetes ***LoadBalance*** sağlar. Mevcut uygulamalar üzerindeki yükü üzerinde çalıştıkları konteynerler arasında dağıtır.

- Kubernetes ***Nameserver*** sağlar. Mevcut konteynerlere erişebilmek için DNS tanımları yapar.

- Kubernetes otomatik ***Rollout/Rollback*** sağlar. Kubernetes her yeni deployment'da veya mevcut deployment'daki her değişiklikte önceden belirlenmiş desired duruma göre otomatik olarak bütün instance'ları ve bileşenleri yeniden oluşturur. Deployment sırasında bir sorun yaşandığında ise otomatik olarak bir önceki sorunsuz çalışan deployment'a geri çekilir.

- Kubernetes gizli bilgilerin ve yapılandırma ayarlarının yönetimini sağlar. Uygulama yapaılandırma ayarları, parolalar, API anahtarları, sertifikalar vb bilgilerin yönetimini otomatikleştirir.

- Kubernetes ***Volume Management*** sağlar. Uygulamaların kullanımına göre depolama ihtiyaçlarını karşılar ve otomatikleştirir.

## Kubernetes Cluster Node'lardan oluşur.

Node kavramını Türkçe karşılığı olan boğum, yumru gibi kelimeler ile açıklayabilir, Kubernetes Cluster'ın birden fazla boğumdan oluşan bütüncül bir yapı olduğunu söyleyebiliriz. Node'lar rolleri özelinde birbirlerinden ayrılırlar;

- ***Master Node:*** Worker Node'lardan, service'lerden, pod'lardan ve diğer bileşenlerden gelen API isteklerini karşılar, yönetir ve gereğini yapar. Cluster'ın beynidir. Control Plane olarak etcd, controller-manager ve scheduler 'ı çalıştırır.

- ***Woker Node:*** Konteynerlerin çalıştıkları ortamdır. Aynı zamanda Control Plane'e üzerinde çalışan konteynerler ve güncel durum hakkında sürekli bilgi verir. kubectl ve kubeproxy aracılığı ile API server'la haberleşir.

## Gerekli Sistem Kaynakları

Kubernetes Cluster'ın sağlıklı biçimde çalışabilmesi için;

- Node başına 2GB RAM ve 2VCPU kaynak
- ***Private Registry*** kullanımlayacaksa internet erişimi
- Node'lar arası tam erişim

sağlanmalıdır.

## Cluster Yapısı

Burada bir Master iki Worker Node olmak üzere High Availibility sağlayabilmek için 3 adet Node kullanacağız.

| Sunucu Rolleri | Sunucu Hostname'leri      | Kaynaklar      |	IP Adresi    |
| -------------- | --------------------      | ---------      | ---------    |
| Master Node	   | master-ubuntu-22.04-k8s	 | 3GB RAM, 2VCPU	| 192.168.1.25 |
| Worker Node	   | worker-1-ubuntu-22.04-k8s | 3GB RAM, 2VCPU	|	192.168.1.26 |
| Worker Node	   | worker-2-ubuntu-22.04-k8s | 3GB RAM, 2VCPU	|	192.168.1.27 |

## Host Dosyalarının Düzenlenmesi

Her Node için ayrı ayrı **/etc/hosts** dosyası düzenlenmeli Node'ların hostname üzerinden birbirine erişebilmeleri için aşağıdaki üç satır dosyanın sonuna eklenmelidir.

<pre><code>
sudo vi /etc/hosts
</pre></code>

<pre><code>
**192.168.1.25** master-ubuntu-22.04-k8s
**192.168.1.26** worker-1-ubuntu-22.04-k8s
**192.168.1.27** worker-2-ubuntu-22.04-k8s
</pre></code>

Değişiklikten sonra ping komutu ile kontrol edilebilir.

<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ ping worker-1-ubuntu-22.04-k8s
PING worker-1-ubuntu-22.04-k8s (192.168.1.26) 56(84) bytes of data.
64 bytes from worker-1-ubuntu-22.04-k8s (192.168.1.26): icmp_seq=1 ttl=64 time=0.630 ms
64 bytes from worker-1-ubuntu-22.04-k8s (192.168.1.26): icmp_seq=2 ttl=64 time=0.784 ms
64 bytes from worker-1-ubuntu-22.04-k8s (192.168.1.26): icmp_seq=3 ttl=64 time=0.565 ms
^C
--- worker-1-ubuntu-22.04-k8s ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2023ms
rtt min/avg/max/mdev = 0.565/0.659/0.784/0.091 ms
haeshin@master-ubuntu-2204-k8s:~$ ping worker-2-ubuntu-22.04-k8s
PING worker-2-ubuntu-22.04-k8s (192.168.1.27) 56(84) bytes of data.
64 bytes from worker-2-ubuntu-22.04-k8s (192.168.1.27): icmp_seq=1 ttl=64 time=0.922 ms
64 bytes from worker-2-ubuntu-22.04-k8s (192.168.1.27): icmp_seq=2 ttl=64 time=0.659 ms
64 bytes from worker-2-ubuntu-22.04-k8s (192.168.1.27): icmp_seq=3 ttl=64 time=0.773 ms
^C
--- worker-2-ubuntu-22.04-k8s ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2506ms
rtt min/avg/max/mdev = 0.659/0.784/0.922/0.107 ms
</pre></code>

## Makinelerin Güncellenmesi

Kubernetes Cluster kurulumu için gerekli paketler ve bağımlılıklar için Ubuntu sunucularımızın her birinin güncellenmiş ve yükseltilmiş olması gerekiyor.
<pre><code>
sudo apt-get update 
sudo apt-get upgrade -y
</pre></code>
Güncelleme ve yükseltme sonrası sunucuların yeniden başlatılması gerekiyor.
<pre><code>
sudo reboot -f
</pre></code>

## kubelet, kubectl ve kubeadm 'in kurulumu

Güncellemenin ardından sunucuları yeniden başlattıysak Kubernetes'in repo'sunu ve gerekli bazı araçları tüm Node'lara ekliyoruz.

<pre><code>
sudo apt install curl apt-transport-https -y
curl -fsSL  https://packages.cloud.google.com/apt/doc/apt-key.gpg|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/k8s.gpg
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
</pre></code>

Güncellemeyi tekrar yapıp gerekli paketleri yüklüyoruz.
</pre></code>
sudo apt update
sudo apt install wget curl vim git kubelet kubeadm kubectl -y
</pre></code>

Ardından kubelet, kubeadm ve kubectl için güncellemeleri durduruyoruz.
</pre></code>
haeshin@master-ubuntu-2204-k8s:/proc$ sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
</pre></code>

Aşağıdaki gibi versiyon kotrolü yapabiliriz
<pre><code>
haeshin@worker-2-ubuntu-2204-k8s:~$ kubectl version --client && kubeadm version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"25", GitVersion:"v1.25.3", GitCommit:"434bfd82814af038ad94d62ebe59b133fcb50506", GitTreeState:"clean", BuildDate:"2022-10-12T10:57:26Z", GoVersion:"go1.19.2", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
kubeadm version: &version.Info{Major:"1", Minor:"25", GitVersion:"v1.25.3", GitCommit:"434bfd82814af038ad94d62ebe59b133fcb50506", GitTreeState:"clean", BuildDate:"2022-10-12T10:55:36Z", GoVersion:"go1.19.2", Compiler:"gc", Platform:"linux/amd64"}
</pre></code>




## swap Alanının Devredışı Bırakılması

Aşağıdaki komutla tüm Node'larda swap alanını devredışı bırakmamız gerekiyor.
<pre><code>
sudo swapoff -a 
</pre></code>
Yine aşağıdaki komutla swap alnının sıfırlanıp sıfırlanmadığını kontrol edebilirsiniz.
<pre><code>
haeshin@master-ubuntu-2204-k8s:/proc$ free -h
               total        used        free      shared  buff/cache   available
Mem:           2.9Gi       186Mi       2.5Gi       1.0Mi       268Mi       2.6Gi
Swap:             0B          0B          0B
</pre></code>
Ardından kalıcı olarar swap alanını kaldırmak için /etc/fstab dosyasını düzenlememiz gerekiyor.
<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ sudo vim /etc/fstab
#/swap.img      none    swap    sw      0       0
</pre></code>
Tekrardan kontrol ediyoruz
<pre><code>
sudo mount -a
free -h
</pre></code>
## Kernel Modülünün ve sysctl'in yapılandırılması


## Konteyner Çalışma Ortamının Kurulumu (Hem Master Hem Worker Node'da)

## Control Plane'nin başlatılması (ilk olarak Master Node'da)

## Worker Node'ların Cluster'a dahil edilmesi

##
