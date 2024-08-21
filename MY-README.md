# Deplying harbor on k8s

## notes
```yaml
expose:
  type: clusterIP

  clusterIP:
    # The name of ClusterIP service
    name: harbor
    # The ip address of the ClusterIP service (leave empty for acquiring dynamic ip)
    staticClusterIP: ""
    ports:
      # The service port Harbor listens on when serving HTTP
      httpPort: 80
      # The service port Harbor listens on when serving HTTPS
      # My note: httpsPort changed from 443 to 8443 as port-forward on 443 locally is not possible
      # domain of internal access to harbor and external access needs to be the same
      httpsPort: 8443
    # Annotations on the ClusterIP service
    annotations: {}
    # ClusterIP-specific labels
    labels: {}

# If Harbor is deployed behind the proxy, set it as the URL of proxy
# My Note: external url is set the same like accessing the service from internal
externalURL: https://harbor.harbor.svc.cluster.local:8443

# My note: setting up the storage class for all services
storageClass: "nfs-client"

```

## steps on each node of the k8s
- link: https://medium.com/@hosseinyousefi/harbor-kubernetes-containerd-c1f98782375e

- download the certificate from project / repo: save it as: harbor.crt
- copy the content to: /usr/share/ca-certificates/mozilla/harbor.crt
- append "mozilla/harbor.crt" in /etc/ca-certificates.conf

run command:
```shell
update-ca-certificates
systemctl restart containerd
```

update /etc/containerd/config.toml
```ỳaml
[plugins."io.containerd.grpc.v1.cri".registry]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
     [plugins."io.containerd.grpc.v1.cri".registry.mirrors."harbor.harbor.svc.cluster.local:8443/wandersnap"]
          endpoint = ["https://harbor.harbor.svc.cluster.local:8443/wandersnap"]
     [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.harbor.svc.cluster.local:8443/wandersnap".tls]
          insecure_skip_verify = true
     [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.harbor.svc.cluster.local:8443/wandersnap".auth]
          username = "admin"
          password = "Harbor12345"
     [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://registry-1.docker.io"]
```

run command:
```shell
update-ca-certificates
systemctl restart containerd
```