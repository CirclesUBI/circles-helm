# Graph protocol for Circles

This chart installs graph protocol for circles with the following configuration:
- index node
- query node + proxy

It also add an extra middlewares to avoid spamming to the range of private IPs from IPFS

⚠️ This installation assumes that you use Traefik as ingress controller. For nginx the configuration of the ingress should change.