# Route53 Mapping Service

This is a Kubernetes controller that polls services (in all namespaces) that are
configured with the label `dns=route53` and adds the appropriate alias to the
domain specified by the annotation `domainName=sub.mydomain.io`. Multiple
domains and top level domains are also supported:
`domainName=.mydomain.io,sub1.mydomain.io,sub2.mydomain.io`.

The project is created by wearemolecule, and maintained at
[wearemolecule/route53-kubernetes](https://github.com/wearemolecule/route53-kubernetes).

## Usage

### Deploy To Cluster

```
# Version 1.2.0
# https://github.com/wearemolecule/route53-kubernetes/tree/v1.2.0
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/kops/master/addons/monitoring-standalone/v1.2.0.yaml
```

### Service Configuration

Add the `dns: route53` label and your target DNS entry in a `domainName`
annotation. Example below:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-awesome-app
  labels:
    app: my-awesome-app
    dns: route53
  annotations:
    domainName: "test.mydomain.tld"
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: |-
      arn:aws:acm:us-east-1:659153740712:certificate/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"
spec:
  selector:
    app: my-awesome-app
  ports:
    - name: http
      port: 80
      protocol: TCP
    - name: https
      port: 443
      protocol: TCP
  type: LoadBalancer
```

An `A` record for `test.mydomain.tld` will be created as an alias to the ELB
that is configured by Kuberntes (see `service.beta.kubernetes.io/aws-load-
balancer` annotations). This assumes that a hosted zone exists in Route53 for
`mydomain.tld`. Any record that previously existed for that dns record will be
updated.



