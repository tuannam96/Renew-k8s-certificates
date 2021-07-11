#Hướng dẫn renew certificate kubernetes manual
##kiểm tra cert hiện tại
1. Phi vào nút master với quyền root và chạy lệnh sau
```kubeadm alpha certs check-expiration```
output sẽ có dạng như sau, trường hợp của tôi là còn 273 ngày
```
CERTIFICATE                EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
admin.conf                 Sep 17, 2020 21:24 UTC   273d            no
apiserver                  Sep 17, 2020 21:24 UTC   273d            no
apiserver-etcd-client      Sep 17, 2020 21:24 UTC   273d            no
apiserver-kubelet-client   Sep 17, 2020 21:24 UTC   273d            no
controller-manager.conf    Sep 17, 2020 21:24 UTC   273d            no
etcd-healthcheck-client    Sep 17, 2020 21:24 UTC   273d            no
etcd-peer                  Sep 17, 2020 21:24 UTC   273d            no
etcd-server                Sep 17, 2020 21:24 UTC   273d            no
front-proxy-client         Sep 17, 2020 21:24 UTC   273d            no
scheduler.conf             Sep 17, 2020 21:24 UTC   273d            no
```
2. Chạy các lệnh sau để sao lưu các chứng chỉ Kubernetes hiện có:

```mkdir -p $HOME/fcik8s-old-certs/pki
/bin/cp -p /etc/kubernetes/pki/*.* $HOME/fcik8s-old-certs/pki
ls -l $HOME/fcik8s-old-certs/pki/```

output sẽ có dạng như sau
```
total 56
-rw-r--r-- 1 root root 1261 Sep  4  2019 apiserver.crt
-rw-r--r-- 1 root root 1090 Sep  4  2019 apiserver-etcd-client.crt
-rw------- 1 root root 1679 Sep  4  2019 apiserver-etcd-client.key
-rw------- 1 root root 1679 Sep  4  2019 apiserver.key
-rw-r--r-- 1 root root 1099 Sep  4  2019 apiserver-kubelet-client.crt
-rw------- 1 root root 1679 Sep  4  2019 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1025 Sep  4  2019 ca.crt
-rw------- 1 root root 1675 Sep  4  2019 ca.key
-rw-r--r-- 1 root root 1038 Sep  4  2019 front-proxy-ca.crt
-rw------- 1 root root 1675 Sep  4  2019 front-proxy-ca.key
-rw-r--r-- 1 root root 1058 Sep  4  2019 front-proxy-client.crt
-rw------- 1 root root 1679 Sep  4  2019 front-proxy-client.key
-rw------- 1 root root 1675 Sep  4  2019 sa.key
-rw------- 1 root root  451 Sep  4  2019 sa.pub
```

3. Chạy các lệnh sau để sao lưu các tệp cấu hình hiện có:
```
/bin/cp -p /etc/kubernetes/*.conf $HOME/fcik8s-old-certs
ls -ltr $HOME/fcik8s-old-certs
```

đầu ra sẽ có dạng như sau
```
total 36
-rw------- 1 root root 5451 Sep  4  2019 admin.conf
-rw------- 1 root root 5595 Sep  4  2019 kubelet.conf
-rw------- 1 root root 5483 Sep  4  2019 controller-manager.conf
-rw------- 1 root root 5435 Sep  4  2019 scheduler.conf
drwxr-xr-x 2 root root 4096 Dec 19 21:21 pki
```

4. Chạy lệnh sau để sao lưu cấu hình thư mục $HOME
```
mkdir -p $HOME/fcik8s-old-certs/.kube
/bin/cp -p ~/.kube/config $HOME/fcik8s-old-certs/.kube/.
ls -l $HOME/fcik8s-old-certs/.kube/.
```
output sẽ có dạng
```
-rw------- 1 root root 5451 Sep  4  2019 config
```

5. Chạy lệnh sau để renew toàn bộ cert
```kubeadm alpha certs renew all
```

output sẽ có dạng
```certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healtcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed
```

6. Chạy lại lệnh ban đầu để kiểm tra cert

```
kubeadm alpha certs check-expiration
```
output sẽ có dạng
```
CERTIFICATE                EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
admin.conf                 Dec 20, 2021 02:35 UTC   364d            no      
apiserver                  Dec 20, 2021 02:35 UTC   364d            no      
apiserver-etcd-client      Dec 20, 2021 02:35 UTC   364d            no      
apiserver-kubelet-client   Dec 20, 2021 02:35 UTC   364d            no      
controller-manager.conf    Dec 20, 2021 02:35 UTC   364d            no      
etcd-healthcheck-client    Dec 20, 2021 02:35 UTC   364d            no      
etcd-peer                  Dec 20, 2021 02:35 UTC   364d            no      
etcd-server                Dec 20, 2021 02:35 UTC   364d            no      
front-proxy-client         Dec 20, 2021 02:35 UTC   364d            no      
scheduler.conf             Dec 20, 2021 02:35 UTC   364d            no
```

7. xác nhận kubelet đã chạy và có thể kết nối từ các node worker, k8s master hoạt động
8. Đợi ít phút và chạy lại lệnh sau để kiểm tra:
```
kubectl get nodes
```
nếu output có dạng
```
The connection to the server <ip>:6443 was refused - did you specify the right host or port?
```

9. thì chạy lệnh sau. (nếu không có vấn đề gì mà show ra như bước 14 thì vui lòng bỏ qua bước này)
```
diff $HOME/fcik8s-old-certs/kubelet.conf /etc/kubernetes/kubelet.conf
```
nếu không có output có nghĩa là file kubelet.conf đã không updated thông tin về các cert mới.

10. Update file /etc/kubernetes/kubelet.conf và hiển thị sự khác biệt với phiên bản cũ hơn
```
cd /etc/kubernetes
sudo kubeadm alpha kubeconfig user --org system:nodes --client-name system:node:$(hostname) > kubelet.conf
diff $HOME/fcik8s-old-certs/kubelet.conf /etc/kubernetes/kubelet.conf
```
Nếu output hiển thị sự khác biệt ở 2 file kubelet.conf thì thông tin về certificate đã được updated.

11. chạy command
```
diff ~/.kube/config $HOME/fcik8s-old-certs/.kube/config
```
nếu không có output thì config file đang chứa cấu hình trỏ đến key và cert kết hạn. (đen)

12. update client-certificate-data và client-key-data trong file ~/.kube/config với giá trị đã được update trong file /etc/kubernetes/kubelet.conf:

```cat /etc/kubernetes/kubelet.conf
```
copy hết output sau dòng client-key-data:
trong file ~/.kube/config, replace các thông tin sau client-key-data: với text đã copy ở bước trước
```
cat /etc/kubernetes/kubelet.conf
```
làm tương tự với client-certificate-data:
13. Restart kubelet service:
```
systemctl daemon-reload&&systemctl restart kubelet
```
14. Verify master and worker nodes are available:
```
kubectl get nodes
```

