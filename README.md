# Ubuntu 22.04 Üzerinde "kubeadm" ile Kubernetes Cluster Kurulumu
![This is an image](https://github.com/hae-shin/ubuntu-2204-kubernetes-cluster/blob/main/kubernetes.png)

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

## Kubernetes'in Bileşenleri

![image](https://user-images.githubusercontent.com/116150600/201122730-677a4ec3-5998-4d8a-a418-dee3d50e2682.png)

# Control Plane Bileşenleri

- **kube-apiserver:**
- **etcd:**
- **kube-scheduler:**
- **kube-controller-manager:** 
- **cloud-controller-manager:** 


# Node Bileşenleri

- **kubelet:**
- **kube-proxy:**
- **Container runtime:** 

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
192.168.1.25 master-ubuntu-2204-k8s
192.168.1.26 worker-1-ubuntu-2204-k8s
192.168.1.27 worker-2-ubuntu-2204-k8s
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

Link: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

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
Yapılan değişikliği kalıcılaştırmak için /etc/fstab dosyasını düzenleyip swap satırını yoruma almalıyız.
<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ sudo cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-rTj4l11pVn7sbKJOHXKBwFacO9dABZFAuSpWPP2y0YtYDGdfAUrgNlhsBdInWKmA / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/a8ad5fbe-816a-4d75-ad98-7710f1bacedf /boot ext4 defaults 0 1
/swap.img       none    swap    sw      0       0
 </pre></code>
## Kernel Modülünün Aktif Edilmesi
<pre><code>
sudo modprobe overlay
sudo modprobe br_netfilter
</pre></code>
# sysctl'in Yapılandırılması
/etc/sysctl.d/kubernetes.conf dosyasının oluşturup aşağıdaki satırları ekliyoruz.
<pre><code>
sudo vi /etc/sysctl.d/kubernetes.conf
</pre></code>
<pre><code>
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
</pre></code>
Ardından değişiklikleri görebilmesi için sysctl'i reload ediyoruz.
<pre><code>
sudo sysctl --system
</pre></code>

## Konteyner Çalışma Ortamının Kurulumu

Pod'ların içerisinde konteynerlerin çalışabilmesi için tüm Node'larda konteyner çalışma ortamı kurulmalıdır. Üç farklı seçeneğimiz bulunuyor;

- Docker
- CRI-O
- Containerd

Biz Containerd seçeneği ile devam edeceğiz kurulumumuza.

# Modül Yüklemelerinin Kalıcılaştırıyoruz.

/etc/modules-load.d/k8s.conf dosyasını oluşturup. İçerisine aşağıdaki satırları ekliyoruz.
<pre><code>
sudo vi /etc/modules-load.d/k8s.conf
</pre></code>

<pre><code>
overlay
br_netfilter
</pre></code>

# Gerekli Paketleri Yüklüyoruz.
<pre><code>
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
</pre></code>

# Docker Repository'sini ekliyoruz.
<pre><code>
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
</pre></code>
# Containerd'yi kuruyoruz.
<pre><code>
sudo apt update
sudo apt install -y containerd.io
</pre></code>

# Containerd yapılandırma ayarlarını oluşturuyoruz.
<pre><code>
sudo su -
mkdir -p /etc/containerd
containerd config default>/etc/containerd/config.toml
</pre></code>

# Containerd'yi yeniden başlatalım.

<pre><code>
sudo systemctl restart containerd
sudo systemctl enable containerd
systemctl status containerd
</pre></code>

![image](https://user-images.githubusercontent.com/116150600/200186159-9c11626f-6cd8-4725-bb36-36eca5dab4d1.png)

# cgroup driver

systemd cgroup driver'ını kullanmak istiyorsak /etc/containerd/config.toml dosyamıza aşağıdaki satırı eklememiz gerekiyor.
<pre><code>
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = true
</pre></code>


## Control Plane'nin başlatılması (Master Node'da)

İlk olarak br_netfilter module'unun duğru biçimde yüklenip yüklenmediğini kontrol edelim.
<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ lsmod | grep br_netfilter
br_netfilter           32768  0
bridge                307200  1 br_netfilter
</pre></code>
Ardından kubelet servisini aktifleştirelim.
<pre><code>
sudo systemctl enable kubelet
</pre></code>
Sonraki aşamada ise Control Plane'nin bileşenlerini(etcd, kube-apiserver vb.) kubeadm ile çekiyoruz.
<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ sudo kubeadm config images pull
[config/images] Pulled registry.k8s.io/kube-apiserver:v1.25.3
[config/images] Pulled registry.k8s.io/kube-controller-manager:v1.25.3
[config/images] Pulled registry.k8s.io/kube-scheduler:v1.25.3
[config/images] Pulled registry.k8s.io/kube-proxy:v1.25.3
[config/images] Pulled registry.k8s.io/pause:3.8
[config/images] Pulled registry.k8s.io/etcd:3.5.4-0
[config/images] Pulled registry.k8s.io/coredns/coredns:v1.9.3
</pre></code>
Eğer birden fazla cri socket bulunuyorsa "--cri-socket" parametresi ile kullanmak istediğinizi belirtmelisiniz.
<pre><code>
sudo kubeadm config images pull --cri-socket /var/run/crio/crio.sock
sudo kubeadm config images pull --cri-socket /run/containerd/containerd.sock
sudo kubeadm config images pull --cri-socket /run/cri-dockerd.sock 
</pre></code>
Şimdi kubeadm ile cluster kurulumunu başlatalım.
<pre><code>
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16
</pre></code>
Aşağıdaki çıktı cluster kurulumunun başarıyla sonuçlandığını gösteriyor.
<pre><code>
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.25:6443 --token rxlytp.m963obz7840h3mx4 \
        --discovery-token-ca-cert-hash sha256:2f0ee2ed94a0fc8b8679a7b6225f0b622182a145a12a3c95d21423f1971a533d 
</pre></code>
Yukarıdaki çıktıda bize söylediği gibi .kube gizli dosyasını oluşturup içerisine config'imizi atıyoruz
<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config^C
haeshin@master-ubuntu-2204-k8s:~$ mkdir -p $HOME/.kube
haeshin@master-ubuntu-2204-k8s:~$ sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
haeshin@master-ubuntu-2204-k8s:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
haeshin@master-ubuntu-2204-k8s:~$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.1.25:6443
CoreDNS is running at https://192.168.1.25:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
</pre></code>
## Network plugin kurulumu
flannel network plugin'i kullanacağız.
<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
--2022-11-10 12:44:16--  https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.108.133, 185.199.109.133, 185.199.110.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4583 (4.5K) [text/plain]
Saving to: ‘kube-flannel.yml’

kube-flannel.yml                         100%[================================================================================>]   4.48K  --.-KB/s    in 0.001s  

2022-11-10 12:44:16 (6.88 MB/s) - ‘kube-flannel.yml’ saved [4583/4583]


</pre></code>
haeshin@master-ubuntu-2204-k8s:~$ ls
kube-flannel.yml
<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ kubectl apply -f kube-flannel.yml
namespace/kube-flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
</pre></code>

<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ kubectl get pods -n kube-flannel
NAME                    READY   STATUS    RESTARTS   AGE
kube-flannel-ds-nqcqs   1/1     Running   0          60s
</pre></code>

<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ kubectl get nodes -o wide
NAME                     STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master-ubuntu-2204-k8s   Ready    control-plane   30m   v1.25.3   192.168.1.25   <none>        Ubuntu 22.04.1 LTS   5.15.0-52-generic   containerd://1.6.9
</pre></code>
## Worker Node'ların Cluster'a dahil edilmesi
<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ kubeadm token create --print-join-command
kubeadm join 192.168.1.25:6443 --token sfimd9.3hovd3re054g6mqs --discovery-token-ca-cert-hash sha256:2f0ee2ed94a0fc8b8679a7b6225f0b622182a145a12a3c95d21423f1971a533d 
</pre></code>
<pre><code>
haeshin@worker-1-ubuntu-2204-k8s:~$ sudo kubeadm join 192.168.1.25:6443 --token sfimd9.3hovd3re054g6mqs --discovery-token-ca-cert-hash sha256:2f0ee2ed94a0fc8b8679a7b6225f0b622182a145a12a3c95d21423f1971a533d 
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: missing optional cgroups: blkio
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
</pre></code>

<pre><code>
haeshin@worker-2-ubuntu-2204-k8s:~$ sudo kubeadm join 192.168.1.25:6443 --token sfimd9.3hovd3re054g6mqs --discovery-token-ca-cert-hash sha256:2f0ee2ed94a0fc8b8679a7b6225f0b622182a145a12a3c95d21423f1971a533d 
[sudo] password for haeshin: 
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: missing optional cgroups: blkio
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
</pre></code>

![image](https://user-images.githubusercontent.com/116150600/201110731-d9b98d4e-3e0a-468e-80c1-218fbf82c9d9.png)

## Kubernetes Dashboard Kurulum

<pre><code>

haeshin@master-ubuntu-2204-k8s:~$ wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
--2022-11-10 14:50:30--  https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.111.133, 185.199.109.133, 185.199.110.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7621 (7.4K) [text/plain]
Saving to: ‘recommended.yaml’

recommended.yaml                         100%[================================================================================>]   7.44K  --.-KB/s    in 0.002s  

2022-11-10 14:50:30 (3.06 MB/s) - ‘recommended.yaml’ saved [7621/7621]

haeshin@master-ubuntu-2204-k8s:~$ ls
kube-flannel.yml  recommended.yaml


---

haeshin@master-ubuntu-2204-k8s:~$ kubectl apply -f recommended.yaml 
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created


haeshin@master-ubuntu-2204-k8s:~$ kubectl get pods -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-64bcc67c9c-jvhfv   1/1     Running   0          49s
kubernetes-dashboard-66c887f759-4qdkv        1/1     Running   0          49s

haeshin@master-ubuntu-2204-k8s:~$ kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
dashboard-metrics-scraper   ClusterIP   10.100.12.31     <none>        8000/TCP   6m1s
kubernetes-dashboard        ClusterIP   10.107.108.167   <none>        443/TCP    6m1s

haeshin@master-ubuntu-2204-k8s:~$ kubectl --namespace kubernetes-dashboard patch svc kubernetes-dashboard -p '{"spec": {"type": "NodePort"}}'
service/kubernetes-dashboard patched

haeshin@master-ubuntu-2204-k8s:~$ kubectl get svc -n kubernetes-dashboard kubernetes-dashboard -o yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard","namespace":"kubernetes-dashboard"},"spec":{"ports":[{"port":443,"targetPort":8443}],"selector":{"k8s-app":"kubernetes-dashboard"}}}
  creationTimestamp: "2022-11-10T14:50:49Z"
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
  resourceVersion: "12601"
  uid: 0e4cdcb1-9cbb-4eb9-a7ae-e4437b573777
spec:
  clusterIP: 10.107.108.167
  clusterIPs:
  - 10.107.108.167
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 31121
    port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}

haeshin@master-ubuntu-2204-k8s:~$ kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.100.12.31     <none>        8000/TCP        7m50s
kubernetes-dashboard        NodePort    10.107.108.167   <none>        443:31121/TCP   7m50s

haeshin@master-ubuntu-2204-k8s:~$ vim nodeport_dashboard_patch.yaml
haeshin@master-ubuntu-2204-k8s:~$ cat nodeport_dashboard_patch.yaml 
spec:
  ports:
  - nodePort: 32000
    port: 443
    protocol: TCP
    targetPort: 8443

haeshin@master-ubuntu-2204-k8s:~$ kubectl -n kubernetes-dashboard patch svc kubernetes-dashboard --patch "$(cat nodeport_dashboard_patch.yaml)"
service/kubernetes-dashboard patched

haeshin@master-ubuntu-2204-k8s:~$ kubectl get deployments -n kubernetes-dashboard 
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
dashboard-metrics-scraper   1/1     1            1           10m
kubernetes-dashboard        1/1     1            1           10m

haeshin@master-ubuntu-2204-k8s:~$ kubectl get pods -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-64bcc67c9c-jvhfv   1/1     Running   0          11m
kubernetes-dashboard-66c887f759-4qdkv        1/1     Running   0          11m
haeshin@master-ubuntu-2204-k8s:~$ kubectl get service -n kubernetes-dashboard 
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.100.12.31     <none>        8000/TCP        11m
kubernetes-dashboard        NodePort    10.107.108.167   <none>        443:32000/TCP   11m

haeshin@master-ubuntu-2204-k8s:~$ vim admin-haeshin.yml
haeshin@master-ubuntu-2204-k8s:~$ cat admin-haeshin.yml 
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-haeshin
  namespace: kube-system
haeshin@master-ubuntu-2204-k8s:~$ kubectl apply -f admin-haeshin.yml 
serviceaccount/admin-haeshin created
haeshin@master-ubuntu-2204-k8s:~$ vim admin-haeshin-rbac.yml
haeshin@master-ubuntu-2204-k8s:~$ cat admin-haeshin-rbac.yml 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-haeshin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-haeshin
    namespace: kube-system
haeshin@master-ubuntu-2204-k8s:~$ kubectl apply -f admin-haeshin-rbac.yml 
clusterrolebinding.rbac.authorization.k8s.io/admin-haeshin created
haeshin@master-ubuntu-2204-k8s:~$ ls
admin-haeshin-rbac.yml  admin-haeshin.yml  kube-flannel.yml  nodeport_dashboard_patch.yaml  recommended.yaml

haeshin@master-ubuntu-2204-k8s:~$ kubectl create token admin-haeshin -n kube-system
eyJhbGciOiJSUzI1NiIsImtpZCI6ImtBa2lJYWtXQW50Sks0MjUteXFLZTkxRW93b2lwQkIxamxleGt2VDgzeGsifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjY4MDk3Mzc4LCJpYXQiOjE2NjgwOTM3NzgsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi1oYWVzaGluIiwidWlkIjoiNDk4MTAzNDUtODM2NS00ZGY4LTg2MTMtMTZhNGQ5OTExMjk1In19LCJuYmYiOjE2NjgwOTM3NzgsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbi1oYWVzaGluIn0.aotmASqgx9yxpOrNF8JQgTaJv50LTMNFL4BfXzJBGvwb6tNKmk8h3F55w4gVZt1ZPbW3igqxGMogWKCgvAZkwr6sKzUyJOtPzo3KX7fDKlCsYhsZkRdx5A7zgTdzbyZYBVlYCpmibqrlLaae56QudUF-6tuPOa0kBRAHh815SVC5cT1pDYNVgBMGySqOJXqV9G47T7K47TMP72cW1d6Mjw8RZ_qx9LZZGNzmdTiROKGiAWsk0almzq-1ij5s2ZlYIS8-kano4VEtJU1eP9Yayf4ONviElR6yRUawqQPgg_8VIeOweoZvX7XLExbjRSEz9740cqWNuPRFvWqFLcWzTw



</pre></code>

![image](https://user-images.githubusercontent.com/116150600/201135821-a88dc50a-284c-491a-948c-d9e08c199e7c.png)

![image](https://user-images.githubusercontent.com/116150600/201136093-59f1bb6f-0149-43f8-bb79-3de642f31117.png)


## Kubernetes Monitoring

git clone https://github.com/prometheus-operator/kube-prometheus.git

haeshin@master-ubuntu-2204-k8s:~/kube-prometheus$ ls
build.sh            CONTRIBUTING.md      example.jsonnet  go.mod   jsonnetfile.json           kustomization.yaml  manifests   scripts
CHANGELOG.md        developer-workspace  examples         go.sum   jsonnetfile.lock.json      LICENSE             README.md   tests
code-of-conduct.md  docs                 experimental     jsonnet  kubescape-exceptions.json  Makefile            RELEASE.md

haeshin@master-ubuntu-2204-k8s:~/kube-prometheus$ kubectl create -f manifests/setup
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com created
namespace/monitoring created

haeshin@master-ubuntu-2204-k8s:~/kube-prometheus$ kubectl get ns monitoring
NAME         STATUS   AGE
monitoring   Active   76s


haeshin@master-ubuntu-2204-k8s:~/kube-prometheus$ kubectl create -f manifests/
alertmanager.monitoring.coreos.com/main created
networkpolicy.networking.k8s.io/alertmanager-main created
poddisruptionbudget.policy/alertmanager-main created
prometheusrule.monitoring.coreos.com/alertmanager-main-rules created
secret/alertmanager-main created
service/alertmanager-main created
serviceaccount/alertmanager-main created
servicemonitor.monitoring.coreos.com/alertmanager-main created
clusterrole.rbac.authorization.k8s.io/blackbox-exporter created
clusterrolebinding.rbac.authorization.k8s.io/blackbox-exporter created
configmap/blackbox-exporter-configuration created
deployment.apps/blackbox-exporter created
networkpolicy.networking.k8s.io/blackbox-exporter created
service/blackbox-exporter created
serviceaccount/blackbox-exporter created
servicemonitor.monitoring.coreos.com/blackbox-exporter created
secret/grafana-config created
secret/grafana-datasources created
configmap/grafana-dashboard-alertmanager-overview created
configmap/grafana-dashboard-apiserver created
configmap/grafana-dashboard-cluster-total created
configmap/grafana-dashboard-controller-manager created
configmap/grafana-dashboard-grafana-overview created
configmap/grafana-dashboard-k8s-resources-cluster created
configmap/grafana-dashboard-k8s-resources-namespace created
configmap/grafana-dashboard-k8s-resources-node created
configmap/grafana-dashboard-k8s-resources-pod created
configmap/grafana-dashboard-k8s-resources-workload created
configmap/grafana-dashboard-k8s-resources-workloads-namespace created
configmap/grafana-dashboard-kubelet created
configmap/grafana-dashboard-namespace-by-pod created
configmap/grafana-dashboard-namespace-by-workload created
configmap/grafana-dashboard-node-cluster-rsrc-use created
configmap/grafana-dashboard-node-rsrc-use created
configmap/grafana-dashboard-nodes-darwin created
configmap/grafana-dashboard-nodes created
configmap/grafana-dashboard-persistentvolumesusage created
configmap/grafana-dashboard-pod-total created
configmap/grafana-dashboard-prometheus-remote-write created
configmap/grafana-dashboard-prometheus created
configmap/grafana-dashboard-proxy created
configmap/grafana-dashboard-scheduler created
configmap/grafana-dashboard-workload-total created
configmap/grafana-dashboards created
deployment.apps/grafana created
networkpolicy.networking.k8s.io/grafana created
prometheusrule.monitoring.coreos.com/grafana-rules created
service/grafana created
serviceaccount/grafana created
servicemonitor.monitoring.coreos.com/grafana created
prometheusrule.monitoring.coreos.com/kube-prometheus-rules created
clusterrole.rbac.authorization.k8s.io/kube-state-metrics created
clusterrolebinding.rbac.authorization.k8s.io/kube-state-metrics created
deployment.apps/kube-state-metrics created
networkpolicy.networking.k8s.io/kube-state-metrics created
prometheusrule.monitoring.coreos.com/kube-state-metrics-rules created
service/kube-state-metrics created
serviceaccount/kube-state-metrics created
servicemonitor.monitoring.coreos.com/kube-state-metrics created
prometheusrule.monitoring.coreos.com/kubernetes-monitoring-rules created
servicemonitor.monitoring.coreos.com/kube-apiserver created
servicemonitor.monitoring.coreos.com/coredns created
servicemonitor.monitoring.coreos.com/kube-controller-manager created
servicemonitor.monitoring.coreos.com/kube-scheduler created
servicemonitor.monitoring.coreos.com/kubelet created
clusterrole.rbac.authorization.k8s.io/node-exporter created
clusterrolebinding.rbac.authorization.k8s.io/node-exporter created
daemonset.apps/node-exporter created
networkpolicy.networking.k8s.io/node-exporter created
prometheusrule.monitoring.coreos.com/node-exporter-rules created
service/node-exporter created
serviceaccount/node-exporter created
servicemonitor.monitoring.coreos.com/node-exporter created
clusterrole.rbac.authorization.k8s.io/prometheus-k8s created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-k8s created
networkpolicy.networking.k8s.io/prometheus-k8s created
poddisruptionbudget.policy/prometheus-k8s created
prometheus.monitoring.coreos.com/k8s created
prometheusrule.monitoring.coreos.com/prometheus-k8s-prometheus-rules created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s-config created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
rolebinding.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s-config created
role.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s created
role.rbac.authorization.k8s.io/prometheus-k8s created
service/prometheus-k8s created
serviceaccount/prometheus-k8s created
servicemonitor.monitoring.coreos.com/prometheus-k8s created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
clusterrole.rbac.authorization.k8s.io/prometheus-adapter created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-adapter created
clusterrolebinding.rbac.authorization.k8s.io/resource-metrics:system:auth-delegator created
clusterrole.rbac.authorization.k8s.io/resource-metrics-server-resources created
configmap/adapter-config created
deployment.apps/prometheus-adapter created
networkpolicy.networking.k8s.io/prometheus-adapter created
poddisruptionbudget.policy/prometheus-adapter created
rolebinding.rbac.authorization.k8s.io/resource-metrics-auth-reader created
service/prometheus-adapter created
serviceaccount/prometheus-adapter created
servicemonitor.monitoring.coreos.com/prometheus-adapter created
clusterrole.rbac.authorization.k8s.io/prometheus-operator created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-operator created
deployment.apps/prometheus-operator created
networkpolicy.networking.k8s.io/prometheus-operator created
prometheusrule.monitoring.coreos.com/prometheus-operator-rules created
service/prometheus-operator created
serviceaccount/prometheus-operator created
servicemonitor.monitoring.coreos.com/prometheus-operator created


haeshin@master-ubuntu-2204-k8s:~/kube-prometheus$ kubectl get pods -n monitoring
NAME                                   READY   STATUS    RESTARTS      AGE
alertmanager-main-0                    2/2     Running   1 (57s ago)   79s
alertmanager-main-1                    2/2     Running   1 (57s ago)   79s
alertmanager-main-2                    2/2     Running   1 (57s ago)   79s
blackbox-exporter-59cccb5797-2fphc     3/3     Running   0             3m33s
grafana-76d67c8b47-j852d               1/1     Running   0             3m31s
kube-state-metrics-6d68f89c45-9k4xg    3/3     Running   0             3m31s
node-exporter-hv9rg                    2/2     Running   0             3m31s
node-exporter-r9k9s                    2/2     Running   0             3m30s
node-exporter-zkgmw                    2/2     Running   0             3m30s
prometheus-adapter-757f9b4cf9-ffn9n    1/1     Running   0             3m29s
prometheus-adapter-757f9b4cf9-l6x7f    1/1     Running   0             3m29s
prometheus-k8s-0                       2/2     Running   0             75s
prometheus-k8s-1                       1/2     Running   0             74s
prometheus-operator-67f59d65b8-6zzxr   2/2     Running   0             3m29s

haeshin@master-ubuntu-2204-k8s:~/kube-prometheus$ kubectl get svc -n monitoring
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
alertmanager-main       ClusterIP   10.108.222.85    <none>        9093/TCP,8080/TCP            9m1s
alertmanager-operated   ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   6m47s
blackbox-exporter       ClusterIP   10.96.187.142    <none>        9115/TCP,19115/TCP           9m1s
grafana                 ClusterIP   10.109.185.203   <none>        3000/TCP                     9m
kube-state-metrics      ClusterIP   None             <none>        8443/TCP,9443/TCP            8m59s
node-exporter           ClusterIP   None             <none>        9100/TCP                     8m59s
prometheus-adapter      ClusterIP   10.108.151.237   <none>        443/TCP                      8m57s
prometheus-k8s          ClusterIP   10.99.223.82     <none>        9090/TCP,8080/TCP            8m58s
prometheus-operated     ClusterIP   None             <none>        9090/TCP                     6m43s
prometheus-operator     ClusterIP   None             <none>        8443/TCP                     8m57s




haeshin@master-ubuntu-2204-k8s:~/kube-prometheus$ kubectl --namespace monitoring patch svc prometheus-k8s -p '{"spec": {"type": "NodePort"}}'
service/prometheus-k8s patched
haeshin@master-ubuntu-2204-k8s:~/kube-prometheus$ kubectl --namespace monitoring patch svc alertmanager-main -p '{"spec": {"type": "NodePort"}}'
service/alertmanager-main patched
haeshin@master-ubuntu-2204-k8s:~/kube-prometheus$ kubectl --namespace monitoring patch svc grafana -p '{"spec": {"type": "NodePort"}}'
service/grafana patched

haeshin@master-ubuntu-2204-k8s:~/kube-prometheus$ kubectl -n monitoring get svc  | grep NodePort
alertmanager-main       NodePort    10.108.222.85    <none>        9093:31426/TCP,8080:31696/TCP   16m
grafana                 NodePort    10.109.185.203   <none>        3000:32291/TCP                  16m
prometheus-k8s          NodePort    10.99.223.82     <none>        9090:32335/TCP,8080:31338/TCP   16m


![image](https://user-images.githubusercontent.com/116150600/201350278-1fba01db-5abc-487b-8060-5f28d528c2eb.png)
