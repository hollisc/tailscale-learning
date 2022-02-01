# Tailscale

[Tailscale](https://tailscale.com/) is a zero config VPN, creating a secure mesh network for your servers, computers and/or Cloud instances.
  Every device is assigned a static IP and domain regardless of how you are connected to the internet.  It essentially mimics a local network
  for any device regardless of where you are.

## Blog links
Here are a few blog posts that I found extremely helpful.

- [How Tailscale works](https://tailscale.com/blog/how-tailscale-works/)
- [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works/)
- [You can use Tailscale with Kubernetes, you know](https://tailscale.com/blog/kubecon-21/)


## Demo Tailscale on an OpenShift cluster
This demo is based on the Tailscale example for Kubernetes - [https://github.com/tailscale/tailscale/tree/main/docs/k8s](https://github.com/tailscale/tailscale/tree/main/docs/k8s).

### Pre-req

1. Clone [Tailscale](https://github.com/tailscale/tailscale) repository
      ```
      git clone https://github.com/tailscale/tailscale.git
      ```
1. Build Tailscale container image
      ```
      docker build -t quay.io/$USER/tailscale-k8s tailscale/docs/K8s
      docker push quay.io/$USER/tailscale-k8s
      ```
1. Clone this [Tailscale-learning](https://github.com/hollisc/tailscale-learning.git) repository
1. Log in to the OpenShift cluster
1. Create namespace
      ```
      oc new-project demo
      ```
1. Generate a [One-off or Ephemeral Key](https://tailscale.com/kb/1085/auth-keys/) from the Tailscale Admin Console.
   The `One-off key` as the name suggests can only be used once.
1. Update the `tailscale/secret/tailscale-api-key-secret.yaml` with the generated key
1. Create K8s Secret containing key
      ```
      oc apply -f secret/tailscale-api-key-secret.yaml
      ```
1. Create the required serviceaccount, role and rolebinding
      ```
      oc apply -f rbac
      ```

### Demo 1 - Userspace Sidecar

1. Create a userspace sidecar pod
      ```
      oc apply -f userspace-sidecar/userspace-sidecar.yaml
      ```
2. From Tailscale Admin Console, the Machines tab should now show a machine for `nginx` with a connected status.
3. From Tailscale Admin Console, check Settings > Keys and confirm the one time key is gone
4. Bring up nginx on browser and curl using Tailscale IP address
5. Enable MagicDNS by adding a nameserver (ie. CloudFlare - 1.1.1.1 or Google - 8.8.8.8)
6. Bring up nginx using the Tailscale FQDN

### Demo 2 - Subnet Router

1. Switch to Developere view and deploy an Nginx pod from a template
2. Retrieve the ClusterIP for Nginx
3. Identify Pod and Service CIDR for cluster
      ```
      oc get network cluster -o yaml
      ```
4. Update subnet-router.yaml with CIDRs
5. Create subnet-router pod
6. From Tailscale Admin Console, check Machines
      subnet-router should show as connected
7. Enable the routes for the subnet-router
8. Connect to a PodIP
      ```
      oc get po -o wide
      curl http://$POD_IP:$PORT
      ```
9. Connect to a ClusterIP
      ```
      oc get svc
      curl http://$CLUSTER_IP:$PORT
      ```