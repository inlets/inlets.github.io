# Cloud Native Tunnel

You can use inlets to connect HTTP and TCP services between networks securely. Through an encrypted websocket, inlets can penetrate firewalls, NAT, captive portals, and other restrictive networks lowering the barrier to entry.

VPNs traditionally require up-front configuration like subnet assignment and ports to be opened in firewalls. A tunnel with inlets can provide an easy-to-use, low-maintenance alternative to VPNs and other site-to-site networking solutions. 

You can run inlets as a stand-alone binary, in Docker, integrated into Kubernetes for Ingress, or with cloud APIs. All services can be kept private in the target network, or exposed publicly.

## Use-cases

You can use inlets as a stand-alone networking tool, or integrate it into a platform to extend functionality for your users.

For companies:

* Low-maintenance, secure, and quick alternative to a VPN
* Provide APIs endpoints to consume data from third-parties and partners
* For command and control of: services within private VPCs, IoT devices and Point of Sale (PoS)
* Create a hybrid cloud between existing servers and public cloud for: CI, e2e testing, or for accessing legacy systems
* As an alternative to an expensive data-center uplink such as [AWS Direct Connect](https://aws.amazon.com/directconnect/) or [Azure Express Route](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-introduction).

For teams and individuals:

* As a zero-touch VPN - penetrating NAT, firewalls, captive portals and hotel WiFi
* When integrating with API that use webhooks such as Stripe, PayPal or GitHub, you can test live with your local machine
* To get a public IP address for your IngressController or services on a Kubernetes cluster
* To access your homelab or Raspberry Pi cluster using SSH
* As a freelancer, you can share a blog or website with a client or with your team
* As an alternative to SaaS and or proprietary offerings such as [Ngrok](https://ngrok.io) and [Argo Tunnel](https://www.cloudflare.com/en-gb/products/argo-tunnel/)

## Concept

There are two flavours of inlets tunnels depending on your needs:

* [inlets-pro](https://github.com/inlets/inlets-pro) - L4 tunnel for any TCP traffic including automatic encryption with TLS. Commercial license: support & enterprise solutions available.
* [inlets](https://github.com/inlets/inlets) - L7 tunnel for HTTP/HTTPS with OSS MIT license.

In the diagram we can see a developer has exposed a Node.js website on his or her laptop through the use of inlets and a server that has a public IPv4 address.

![Conceptual diagram for inlets](images/conceptual.png)

The remote server is called an "exit-node" or "exit-server" because that is where traffic from the private network appears. The user's laptop has gained a "VirtualIP" and users on the Internet can now connect to it using that IP.

## Exit-servers

Exit-servers run the inlets server component (the control-plane) on a given port and accept incoming requests from the inlets client. You can create these manually and customise them, or automate them through tooling such as [Terraform](https://www.terraform.io) or bash.

You can create and manage your own exit-servers, or use an existing computer and run `inlets-pro server` or `inlets server`.

There are two community projects which automate the creation and configuration of your exit-servers. Both inlets and -pro are supported on: DigitalOcean, Packet, Scaleway, Civo, AWS, Azure and GCP.

* [inletsctl](https://github.com/inlets/inletsctl)  - create exit-servers for inlets/-pro
* [inlets-operator](https://github.com/inlets/inlets-operator) - Kubernetes automation to create exit-servers for inlets/-pro including a CRD

## Get started

* [Quick-start: expose a HTTP server with a public IP](/get-started/quickstart-http?id=expose-a-http-server-with-a-public-ip)

* [Quick-start: get SSH access from anywhere](/get-started/quickstart-tcp-ssh?id=get-ssh-access-from-anywhere)

* [Video: Get public endpoints in seconds with inlets and create-react-app](https://www.youtube.com/watch?v=jrAqqe8N3q4&feature=youtu.be)

* [inlets PRO CLI reference](https://github.com/inlets/inlets-pro/blob/master/docs/cli-reference.md)

* [Tunnel PostgreSQL or MariaDB with TLS](https://gist.github.com/alexellis/995b518a653d172dda4b0901dcdaa391)

* [Expose your private Grafana dashboards with TLS](https://blog.alexellis.io/expose-grafana-dashboards/)

Kubernetes:

* [Quick-start: expose Your IngressController and get TLS from LetsEncrypt](/get-started/quickstart-ingresscontroller-cert-manager?id=expose-your-ingresscontroller-and-get-tls-from-letsencrypt)

* [Get kubectl access to your private cluster from anywhere](https://blog.alexellis.io/get-private-kubectl-access-anywhere/)

* [Quick-start: expose a Kubernetes Pod with KinD](/get-started/quickstart-k8s?id=expose-nginx-from-your-kubernetes-cluster-with-kind)

* [Get a private Docker registry with auth and TLS](https://blog.alexellis.io/get-a-tls-enabled-docker-registry-in-5-minutes/)

## Pricing

### inlets PRO

Buy an inlets PRO license for personal or commercial use:

[inlets PRO features and pricing](/pricing/?id=pricing)

### Professional services

inlets, inlets-operator and inletsctl are all OSS and distributed under the MIT license.

Contact [sales@openfaas.com](mailto:sales@openfaas.com) for a quote on commercial support, training, and custom solutions for inlets or inlets PRO.

## Connect with the community

Follow [@inletsdev](https://twitter.com/inletsdev) on Twitter.

Join the `#inlets` channel on [OpenFaaS Slack](https://slack.openfaas.io/)

[Contribute to this documentation on GitHub](https://github.com/inlets/docs/)

### Buy SWAG

If you are using the project and want to support the community, then buy some SWAG to say thanks for our work.

![T-shirt and hoodie](images/inlets-swag.jpg)

**Buy your own [inlets t-shirt, hoodie or mug](https://store.openfaas.com/)**

### Featured on

inlets is proud to be featured on the Cloud Native Landscape in the Service Proxy category.

<p><a href="https://landscape.cncf.io"><img width="200px" src="/images/cncf-landscape-left-logo.svg" alt="Cloud Native Landscape"></a></p>
