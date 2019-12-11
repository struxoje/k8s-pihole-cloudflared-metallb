# k8s-pihole-cloudflared-metallb
Pi-hole running on Kubernetes, load-balanced with MetalLB, forwarding traffic via DNS-over-HTTPS with Cloudflared

This project is based on work of https://github.com/David-VTUK/k8spihole (setting up Pi-hole with MetalLB) and https://github.com/jcrowthe/k8s-pihole-cloudflared (setting up Pi-hole with Cloudflared).

As I was unable to find finished solution that would combine Pi-hole on k8s with MetalLB load balancer and cloudflared DNS-over-HTTPS, what I did was merged two solutions mentioned above into one project. I also did notice an issue in original k8s-pihole-cloudflared yaml file, as DNS traffic was forwarded via port 53 therefore not utilizing cloudflared cababilities at all as per docker definition file of visibilityspots/cloudflared, cloudflared runs on port 54 and not port 53. 

I did confirm that my changes works well by running tests on https://www.cloudflare.com/ssl/encrypted-sni/

![Result](/images/result.png)

First setup MetalLB by running: 
`kubectl apply -f metallb-config.yaml`

This will setup load balancer and give it an app pool from 192.168.1.200 - 192.168.1.254. Please note that you should limit your DHCP not to use this range.

Pi-hole and cloudflared will be installed by running. But before that change admin password by modifying 
        - name: WEBPASSWORD
          value: admin
in deploy-pihole-cloudflared-fixed.yaml. You can now continue installation and run:
`kubectl apply -f deploy-pihole-cloudflared-fixed.yaml`

After execution you would need to check at what address was assigned to cloudflared service by running:
`kubectl get svc -n pihole-dns`

Look for external IP address of pihole-udp service and if needed change deploy-pihole-cloudflared-fixed.yaml
and following line:           
value: 192.168.1.200 # MODIFY THIS TO YOUR PIHOLE-UDP SVC IP

After that apply updates by running again
`kubectl apply -f deploy-pihole-cloudflared-fixed.yaml`

Fixing IP address like this is far from ideal, but it did exist in original solution, and I didn't have the time to spend to deal with it so contributors are welcome :)

In order to be able to see proper Client IPs and not obscured IP by nodes, we need to set
  externalTrafficPolicy: Local
  
As I was using WeaveNet I also had to set NO_MASQ_LOCAL=1 in order for previous policy to take effect by running
`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.NO_MASQ_LOCAL=1"`

