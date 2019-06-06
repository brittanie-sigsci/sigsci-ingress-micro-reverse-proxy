# Signal Sciences Kubernetes Sidecar Example

This example deployment method for Kubernetes is using NGINX with the Signal Sciences Native Module and a sidecar container with the Signal Sciences Agent. The configuration shown can be used for any other type of agent/module pair and gives a simple example of how to set things up.

The two containers communicate over a Unix Socket file shared via Shared Persistent Volume. In my example I will use minikube with a NFS Server as the Persistent Volume type.

## Setting up the Environment

1. Download and install minikube from https://kubernetes.io/docs/tasks/tools/install-minikube/
2. Download and install kubectl from https://kubernetes.io/docs/tasks/tools/install-kubectl/
3. Run: `minikube start`
    - Note on Windows you will need to do something like `minikube start --vm-driver hyperv --hyperv-virtual-switch "Default Switch"` in order to work with Hyper-V
4. Run: `minikube addons enable dashboard`
5. Run: `kubectl apply -f build_env.yaml` 
    - Please note you will need to update the following in the yaml:
        + The images (`image:trickyhu/sigsci-debian-nginx:latest`, `image:trickyhu/sigsci-agent-alpine:latest`) with your own unless you want to test with mine.
        + The value for `SIGSCI_NGINX_PORT` if you don't want to use the default `8282` if you change this you will also need to update `- containerPort: 8282`
        + The value for `SIGSCI_HOSTNAME` if you do not want to use the default
        + The value for `SIGSCI_SECRETACCESSKEY`
        + The value for `SIGSCI_ACCESSKEYID`
        + The `SIGSCI_ACCESSKEYID` will need to be updated for the Ingress, the Nginx Service, the Apache Service, and the Reverse Proxy Service
        + The POD Volume Mount to a Persistent Volume (PV) that exists in your kubernetes Cluster
        ````
        volumes:
          - name: host-mount
            nfs:
              path: /mnt/sharedfolder
              server: NFS_SERVER_NAME
        ````
6. Run: `kubectl --namespace ingress-nginx expose deployment sigsci-nginx-example --type=NodePort`
7. Run: `kubectl --namespace ingress-nginx expose deployment reverse-proxy --type=NodePort`
8. Run: `kubectl --namespace ingress-nginx expose deployment microservice-a`
9. Run: `kubectl --namespace ingress-nginx expose deployment microservice-b`
10. Run: `minikube service sigsci-nginx-example`
11. You can access the ingress controller easily by doing `minikube service ingress-nginx`
    - Apache service will be http://minikubeip:ingress_port/apache
    - NGIINX service will be http://minikubeip:ingress_port/nginx
    - RP Service will be http://minikubeip:rp_port

At this point you should now have a browser window pop up with a simple html page.

_Side Note: If you would like to set up a NFS Server you can follow the directions from https://vitux.com/install-nfs-server-and-client-on-ubuntu/ like I did in an Ubuntu VM_

### NFS Server Steps

1. Run: `sudo apt update`
2. Run: `sudo apt install nfs-kernel-server`
3. Run: `sudo mkdir -p /mnt/sharedfolder`
4. Run: `sudo chown nobody:nogroup /mnt/sharedfolder`
5. Run: `sudo chmod 777 /mnt/sharedfolder`
6. Edit: `/etc/exports`
    - Add: /mnt/sharedfolder XXX.XXX.XXX.XXX/24(rw,sync,no_subtree_check)
7. Run: `sudo exportfs -a`
8. Run: `sudo systemctl restart nfs-kernel-server`
9. Run: `sudo ufw allow from XXX.XXX.XXX.XXX/24 to any port nfs`

You can now use this NFS Server in the POD Volume Mount `server: NFS_SERVER_NAME`
