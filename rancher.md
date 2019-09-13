curl https://releases.rancher.com/install-docker/17.03.sh | bash

apt install -y libltdl7  

wget https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/docker-ce_17.03.2~ce-0~ubuntu-xenial_amd64.deb  

dpkg -i docker-ce_17.03.2~ce-0~ubuntu-xenial_amd64.deb

usermod -aG docker $USER

openssl x509 -in certificate.crt -text -noout

openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt

docker run -d --restart=always --name registry -v "$(pwd)"/certs:/certs -e REGISTRY_HTTP_ADDR=0.0.0.0:443 -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key -p 8443:443 registry:2



docker run -d -p 8443:443 --restart=always   --name registry -v "$(pwd)"/auth:/auth -e "REGISTRY_AUTH=htpasswd"   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm"   -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd -v "$(pwd)"/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key -e REGISTRY_HTTP_ADDR=0.0.0.0:443 registry:2

git checkout tags/<tag_name> -b <branch_name>

Self-signed CA and cert:

https://gist.github.com/fntlnz/cf14feb5a46b2eda428e000157447309

https://gist.github.com/xenogenesi/1b2137f769aa80b6c99d573071f5d086

Gitlab with SSL:

https://github.com/GetchaDEAGLE/gitlab-https-docker/blob/master/docker-compose.yml



git clone -c http.sslCAPath="C:\\Users\\jsmith\\certificates\\gitlab" -c http.sslCAInfo="C:\\Users\\jsmith\\certificates\\gitlab\\server-cert.pem" -c http.sslVerify=1 https://git.example.com/jsmith/gitlab-ce.git



for certificate in $(ls -1 kube-apiserver.pem kube-controller-manager.pem kube-etcd*.pem kube-node.pem kube-proxy.pem kube-scheduler.pem  | grep -v key); do openssl verify -verbose -CAfile kube-ca.pem $certificate > /dev/null 2>&1; if [ $? -ne 0 ]; then echo $certificate is wrong; fi; done



for ctid in $(docker ps -q --filter name=k8s_POD); do echo "$ctid"; docker inspect --format '{{ .Name }}' $ctid; nsenter --net=$(docker inspect --format '{{ .NetworkSettings.SandboxKey }}' $ctid) ifconfig eth0 | grep 'inet addr'; done



https://github.com/rancher/os/blob/master/scripts/tools/collect_rancheros_info.sh

Cleanup script https://gist.github.com/superseb/06539c6dcd377e118d72bfefdd444f81

Custom cluster kubeconfig

https://gist.github.com/superseb/f6cd637a7ad556124132ca39961789a4



ssh -f ubuntu@52.59.227.249 -L 8080:localhost:80 -N



aws elb describe-load-balancers --load-balancer-name RancherAdmin-CLB

aws elb describe-load-balancer-policies --load-balancer-name RancherAdmin-CLB

For both the load-balancer and ec2 instance security groups:

aws ec2 describe-security-groups --group-ids <sg id>



ssh -N -D 9090 [USER]@[SERVER_IP]

https://gist.github.com/superseb/f44d51cc42c7e5a0049d393d6d2563e7



https://github.com/kubernetes/ingress-nginx/issues/3588

-generator=run-pod/v1

kubectl run --it --rm --image=nicolaka/netshoot sh

nslookup configuration-server

curl configuration-server

docker ps --filter name=k8s_POD_configuration-server

nsenter --net=$(docker inspect <ctid> --format '{{ .NetworkSettings.SandboxKey }}')



kind: Service

apiVersion: v1

metadata:

  name: configuration-server

spec:

  selector:

    app: configuration-server

  ports:

  - name: configuration-server

    protocol: TCP

    port: 8080





https://github.com/rancher/rancher/issues/18921#issuecomment-473055835

https://github.com/rancher/rancher/issues/18921#issuecomment-473071223



hdparm -t /dev/sda

centos-7-x64

kubectl get clusters c-fwk9r -o jsonpath="{.status.appliedSpec.rancherKubernetesEngineConfig.nodes[0].sshKey}"



cat OBoHoPwL.json | jq -r '.cards[] | select(.idList=="5c9213c62de43465a325c215") | [.name, .desc] | @csv'



sed -E 's/ZD-([1-9]+)/https\:\/\/rancher\.zendesk.com\/agent\/tickets\/\1/'



https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#without-kubectl-proxy

