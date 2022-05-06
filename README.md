**Install Docker**: `curl https://releases.rancher.com/install-docker/19.03.sh | bash`

**Rancher 2.x Log Collection**: `wget -O- https://raw.githubusercontent.com/rancherlabs/support-tools/master/collection/rancher/v2.x/logs-collector/rancher2_logs_collector.sh | sudo bash -s`
https://support.rancher.com/hc/en-us/articles/360039113911-The-Rancher-v2-x-log-collector-script

**Rancher 1.6 Log Collection**: `wget -O- https://raw.githubusercontent.com/rancherlabs/support-tools/master/collection/rancher/v1.6/logs-collector/rancher16_logs_collector.sh | sudo bash -s`

**RancherOS Log Collection**: `sudo curl https://raw.githubusercontent.com/rancher/os/master/scripts/tools/collect_rancheros_info.sh | sh`

**Rancher 2.x Usage Collection**: `wget -O- https://raw.githubusercontent.com/rancherlabs/support-tools/master/collection/rancher/v2.x/usage-collector/rancher2_usage_collector.sh | sudo bash -s`

**Extended Cleanup script (DESTRUCTIVE)**: `wget -O- https://github.com/rancherlabs/support-tools/raw/master/extended-rancher-2-cleanup/extended-cleanup-rancher2.sh | sudo bash -s`

**Extended Cleanup script URL**: https://github.com/rancherlabs/support-tools/tree/master/extended-rancher-2-cleanup

**RBAC Lookup Tool**: https://github.com/FairwindsOps/rbac-lookup

**Get rancher-wins service logs**: `Get-EventLog -LogName Application -Source rancher-wins > wins.log`

**Get a secret contents (escape periods in the key)**: `kubectl get secret <SECRET_NAME> -o jsonpath="{.data.<DATA>}" | base64 --decode`    

**Install specific Docker version on Ubuntu**:
```
apt-cache show docker-ce | grep Version
apt install docker-ce=<version string> docker-ce-cli=<version string>
```

**View /etc/resolv.conf of coredns container***: ```docker cp `docker ps | grep k8s_coredns | head -n1 | awk '{print $1}'`:/etc/resolv.conf -```

***View /etc/coredns of coredns container***: ```docker cp `docker ps | grep k8s_coredns | awk '{print $1}' | head -n1`:/etc/coredns -```

**Run http-echo container**: `docker run -p 8080:80 -p 8443:443 --name echo -d mendhak/http-https-echo`

**Run basic HTTP forward proxy**: `docker run -d --name='tinyproxy' -p 80:8888 dannydirect/tinyproxy:latest ANY`

**Get etcd metrics**: `curl --cacert /etc/kubernetes/ssl/kube-ca.pem --key /etc/kubernetes/ssl/kube-etcd-104-248-25-11-key.pem --cert /etc/kubernetes/ssl/kube-etcd-104-248-25-11.pem https://127.0.0.1:2379/metrics`

**Check remote certificate**: `openssl s_client -showcerts -connect <server-name>:443 -servername <server-name>`

**Connect over TLS to a server**: `openssl s_client -connect <ip-address>:6443`

**Connect to kubeapi with the right certificates**: `curl --cert <client-cert-path> --key <client-key-path> --cacert <ca-cert-path> https://<ip-address>:6443`

**Inspect certificate**: `openssl x509 -in <cert-path> -noout -text`

**Check certificate expiration**: `openssl x509 -startdate -enddate -noout -in <cert-path>`

**Check certificate expriation for cluster certs**: ```for cert in `ls -d /etc/kubernetes/ssl/* | grep -v key | grep pem`; do echo $cert; openssl x509 -startdate -enddate -noout -in $cert; done```

**Verify that the certificate is signed by a specific CA**: `openssl verify -verbose -CAfile <ca-cert-path> <cert-path>`

**Verify that the certificate is signed by CA and intermediate**: `openssl verify -CAfile <ca-cert-path> -untrusted <intermediate-cert-path> <cert-path>`

**Verify that the cert matches its key**:
```
openssl x509 -noout -modulus -in <cert-path> | openssl md5
openssl rsa -noout -modulus -in <key-path> | openssl md5
```

**Validate Kubernetes component certificates signed by kube-ca**:
```
for certificate in $(ls -1 kube-apiserver.pem kube-controller-manager.pem kube-etcd*.pem kube-node.pem kube-proxy.pem kube-scheduler.pem  | grep -v key); do openssl verify -verbose -CAfile kube-ca.pem $certificate > /dev/null 2>&1; if [ $? -ne 0 ]; then echo $certificate is wrong; fi; done
```

**Add user to Docker group**: `usermod -aG docker $USER`

**Lauch Gitlab with SSL:**: https://github.com/GetchaDEAGLE/gitlab-https-docker/blob/master/docker-compose.yml

**Get Pod IPs**:
```
for ctid in $(docker ps -q --filter name=k8s_POD); do echo "$ctid"; docker inspect --format '{{ .Name }}' $ctid; nsenter --net=$(docker inspect --format '{{ .NetworkSettings.SandboxKey }}' $ctid) ifconfig eth0 | grep 'inet addr'; done
```

**Get kubeconfig for Rancher custom cluster**: https://gist.github.com/superseb/f6cd637a7ad556124132ca39961789a4

**Get cluster object from etcdbackup resource**: `kubectl get etcdbackups -n <cluster-id> <backup-id> -o jsonpath={.status.clusterObject} | base64 -d | gunzip | jq`

**Retrieve .rkestate file from RKE >= v0.2.0 launched cluster (https://gist.github.com/superseb/5e173da28116cfcfe5177ea6d866fbd5)**:
```
docker run --rm --net=host -v $(docker inspect kubelet --format '{{ range .Mounts }}{{ if eq .Destination "/etc/kubernetes" }}{{ .Source }}{{ end }}{{ end }}')/ssl:/etc/kubernetes/ssl:ro --entrypoint bash $(docker inspect $(docker images -q --filter=label=org.label-schema.vcs-url=https://github.com/rancher/hyperkube.git) --format='{{index .RepoTags 0}}' | tail -1) -c 'kubectl --kubeconfig /etc/kubernetes/ssl/kubecfg-kube-node.yaml -n kube-system get configmap full-cluster-state -o json | jq -r .data.\"full-cluster-state\" | jq -r .' > cluster.rkestate
```

**Remove Windows carriage return characters in vim**: `:%s/\r//g`

**Curl kube-apiserver on RKE1 controlplane node**:
```
docker cp kubelet:/usr/local/bin/kubectl .
wget https://gist.githubusercontent.com/Oats87/9b27310f5d2937adc87a03ee97675802/raw/eca91c84e48aa0fb52248c661fdf1646d657196e/generate_new_kubeconfig.sh
sudo bash generate_new_kubeconfig.sh
export KUBECONFIG=new_kube_config.yml
TOKEN=$(./kubectl -n cattle-system get secret `kubectl -n cattle-system get sa cattle -o jsonpath={.secrets[0].name}` -o jsonpath={.data.token} | base64 -d)
curl -v -k -H "Authorization: Bearer $TOKEN" https://127.0.0.1:6443/api/v1/namespaces
```

**Runnning a proxy with SSL interception**: https://scubarda.com/2020/03/23/configure-squid-proxy-for-ssl-tls-inspection-https-interception/

**Add my SSH keys to a host**: `wget -O- https://github.com/axeal.keys >> ~/.ssh/authorized_keys`

**Create Namespaces from YAML spec list**: `grep namespace: <spec>.yaml | awk '{print $2}' | sort | uniq | xargs -I '{}' kubectl create ns '{}'`
