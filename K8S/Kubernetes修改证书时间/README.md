### 1.查看证书有效日期

```shell
# 切换到证书所在目录
cd /etc/kubernetes/pki/
# 查看证书信息
openssl x509 -in apiserver.crt -text -noout
----------------------------------------------
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 5133984457178512100 (0x473f93a9ac807ae4)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=kubernetes
        Validity
            Not Before: Nov  9 06:55:56 2021 GMT
            Not After : Nov  9 06:55:56 2022 GMT
        Subject: CN=kube-apiserver
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:bb:95:31:52:5d:a6:d7:a8:3b:25:4a:f5:03:e0:
                    37:42:da:14:db:4b:e0:11:2e:e9:d7:15:4a:fa:a2:
                    d5:e8:90:cf:0c:ec:08:4c:39:fc:22:fc:6a:a3:1f:
                    f4:1e:e6:3e:bc:aa:e3:86:64:17:78:84:4f:52:dc:
                    cc:39:ad:a3:85:6e:c2:b7:05:58:04:c9:fa:40:a3:
                    8c:a1:b3:fc:c8:6c:b1:41:9b:b9:6c:24:03:19:23:
                    02:e7:b2:d8:d2:5e:1d:d5:d0:38:98:45:66:ac:e4:
                    4b:1b:32:5d:8f:f4:48:21:c0:fc:d8:73:b1:f2:18:
                    30:7b:42:98:20:e1:b2:13:00:0b:d9:23:17:8e:28:
                    83:96:b4:3b:58:a4:54:f1:1c:55:0b:fe:ca:49:fe:
                    90:7a:9b:0a:19:3a:4f:d1:7f:50:75:71:7f:ed:c3:
                    24:70:4b:60:e7:2c:74:7c:e9:0d:67:c0:2f:7f:6f:
                    f4:22:9d:e5:8e:5b:db:2b:1a:24:55:0f:c5:be:82:
                    8c:d7:8d:14:13:c1:9e:aa:92:56:73:5c:0e:d2:5c:
                    32:11:a8:ad:66:a3:b6:ec:6f:9a:7c:e5:ab:7e:e2:
                    e5:a4:47:51:f1:45:01:22:a8:f9:6d:11:73:9b:cd:
                    8f:e6:4d:48:84:8e:f6:dc:39:cb:c9:52:e6:75:2f:
                    e1:25
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Subject Alternative Name: 
                DNS:k8s-master, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP Address:10.96.0.1, IP Address:192.168.2.132
    Signature Algorithm: sha256WithRSAEncryption
         86:5a:f8:d1:dd:06:30:e6:4f:9c:50:80:c4:4b:f2:49:d1:1d:
         a7:be:e3:61:df:ab:ce:eb:d0:a8:0a:9b:9b:f2:f7:78:1b:44:
         70:15:89:31:cb:7d:8e:c8:a9:ef:43:d6:5f:22:64:d6:da:a2:
         aa:7e:21:01:a0:b9:0e:f4:0d:59:4b:62:3f:9b:3a:5d:87:05:
         59:c9:4f:bd:5e:7a:60:fe:e6:4c:a1:f2:28:93:ce:f3:0f:89:
         bf:7f:21:e9:3f:b3:b4:01:4b:6f:85:89:a7:0b:ee:9e:57:ee:
         95:7b:9a:5c:81:79:d4:51:30:1f:fa:ce:58:82:82:f5:03:6f:
         7d:d8:0e:6d:3f:58:26:4d:c6:da:d5:73:55:79:31:df:d8:f3:
         bd:3f:ab:0e:02:a1:cd:f5:cb:37:50:1c:f2:2c:b2:59:ba:ae:
         8b:2d:b2:7d:80:72:2f:3b:55:0a:22:27:6c:12:d0:bd:07:e2:
         84:38:02:c8:c0:47:b0:93:34:d5:de:b9:5d:a9:49:32:e8:d7:
         87:3e:a2:02:23:b5:64:6c:58:71:a8:9c:2e:13:af:fb:9f:21:
         4f:28:16:6b:ed:75:ed:47:be:c8:63:95:fc:cd:6f:56:da:21:
         07:e2:3c:9b:42:6e:a2:2f:dd:60:4b:29:b7:84:06:ad:46:2b:
         2c:7d:be:0f
```

### 2.go环境安装

（1）在**GO中文社区**下载安装包

（2）解压至**/usr/local/**目录下

```shell
tar -zxvf go...... /usr/local/
```

（3）配置环境变量

```shell
# 添加环境变量
vim  /etc/profile

-----------------------------------
export PATH=$PATH:/usr/local/go/bin
----------------------------------

# 刷新变量
source  /etc/profile

# 检查版本信息
go version
```

（4）更换证书

```shell
# 拉取git（首先安装git）
git clone https://github.com/kubernetes/kubernetes.git

# 查看当前版本
kubeadm version
----------------------
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:56:30Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
----------------------
# 根据当前版本，检出相应版本文件
git checkout -b remotes/origin/release-1.18.0 v1.18.0

# 编辑文件
vim cmd/kubeadm/app/util/pkiutil/pki_helpers.go

# 找到最下面生成证书的模板语句
-------------------------------------
......
// NewSignedCert creates a signed certificate using the given CA certificate and key
func NewSignedCert(cfg *CertConfig, key crypto.Signer, caCert *x509.Certificate, caKey crypto.Signer) (*x509.Certificate, error) {
        const duration36500d = time.Hour * 24 * 365 * 10

        serial, err := cryptorand.Int(cryptorand.Reader, new(big.Int).SetInt64(math.MaxInt64))
        if err != nil {
                return nil, err
        }
        if len(cfg.CommonName) == 0 {
                return nil, errors.New("must specify a CommonName")
        }
        if len(cfg.Usages) == 0 {
                return nil, errors.New("must specify at least one ExtKeyUsage")
        }

        certTmpl := x509.Certificate{
                Subject: pkix.Name{
                        CommonName:   cfg.CommonName,
                        Organization: cfg.Organization,
                },
                DNSNames:     cfg.AltNames.DNSNames,
                IPAddresses:  cfg.AltNames.IPs,
                SerialNumber: serial,
                NotBefore:    caCert.NotBefore,
                NotAfter:     time.Now().Add(duration36500d).UTC(),
                KeyUsage:     x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature,
                ExtKeyUsage:  cfg.Usages,
        }
        certDERBytes, err := x509.CreateCertificate(cryptorand.Reader, &certTmpl, caCert, key.Public(), caKey)
        if err != nil {
                return nil, err
        }
......
-------------------------------------
# 声明时间变量：const duration36500d = time.Hour * 24 * 365 * 100
# 添加变量： NotAfter:     time.Now().Add(duration36500d).UTC(),
# 保存退出编辑
# 执行以下语句
make WHAT=cmd/kubeadm GOFLAGS=-v

# 执行完后，备份新生成的文件至/root/下
cp _output/bin/kubeadm /root/

# 备份原有的kubeadm
cp /usr/bin/kubeadm /usr/bin/kubeadm.old

# 新文件替换旧文件
cd ~
cp kubeadm /usr/bin/

# 赋予权限
chmod a+x /usr/bin/kubeadm

# 切换到k8s目录
cd /etc/kubernetes/

# 备份原有证书
cp -r pki/ pki.old

# 切到root目录下，生成证书
kubeadm alpha certs renew all
--------------------------------
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed
--------------------------------
# 切换到证书目录
cd /etc/kubernetes/pki

# 查看证书信息
openssl x509 -in apiserver.crt -text -noout
```

