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
<pre><code>
haeshin@master-ubuntu-2204-k8s:~$ kubectl get nodes -o wide
NAME                       STATUS   ROLES           AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master-ubuntu-2204-k8s     Ready    control-plane   92m     v1.25.3   192.168.1.25   <none>        Ubuntu 22.04.1 LTS   5.15.0-52-generic   containerd://1.6.9
worker-1-ubuntu-2204-k8s   Ready    <none>          8m25s   v1.25.3   192.168.1.26   <none>        Ubuntu 22.04.1 LTS   5.15.0-52-generic   containerd://1.6.9
worker-2-ubuntu-2204-k8s   Ready    <none>          2m5s    v1.25.3   192.168.1.27   <none>        Ubuntu 22.04.1 LTS   5.15.0-52-generic   containerd://1.6.9
</pre></code>
##
