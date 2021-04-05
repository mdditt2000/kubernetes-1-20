# My Notes

## Restarting K8S after reboot

systemctl enable docker
systemctl start docker
systemctl disable firewalld
systemctl stop firewalld
systemctl enable kubelet
systemctl start kubelet
export KUBECONFIG=/etc/kubernetes/admin.conf
