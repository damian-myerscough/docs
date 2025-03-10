---
title: "Armory Agent for Kubernetes"
linkTitle: "Armory Agent for Kubernetes"
weight: 20
description: >
  The Armory Agent is a lightweight, scalable service that monitors your Kubernetes infrastructure and streams changes back to Spinnaker's Clouddriver service.
---
![Proprietary](/images/proprietary.svg)
## Advantages of using the Armory Agent for Kubernetes

* Scalability
  * Caching and deployment scales to thousands of Kubernetes clusters for your largest applications.
  * By leveraging the Kubernetes `watch` mechanism, the Agent detects changes to Kubernetes and streams them in real time over a single TCP connection per cluster to Spinnaker<sup>TM</sup>.
  * The Agent optimizes how infrastructure information is cached, making _Force Cache Refresh_ almost instantaneous. This means optimal performance for your end users and your pipeline executions.

* Security
  * Keep your Kubernetes API servers private from Spinnaker.
  * Only the information Spinnaker needs leaves the cluster.
  * Decentralize your account management. Using Kubernetes Service Accounts, teams control what Spinnaker can do. Add or remove accounts in real time. and then use them without restarting Spinnaker.
  * Use Kubernetes Service Accounts or store your `kubeconfig` files in one of the supported [secret engines]({{< ref "secrets#supported-secret-engines" >}}), or provision them via the method of your choice as Kubernetes secrets.


* Usability
  * Use the Agent alongside Spinnaker and benefit from performance improvements.
  * Use the Agent in a target cluster and get Kubernetes accounts automatically registered.
  * Use YAML, a HELM chart, or a Kustomize template to inject the Agent into newly provisioned Kubernetes clusters, and immediately make those clusters software deployment targets.
  * Use the Agent with little operational overhead or changes in the way you currently manage Spinnaker.

Check out the Quick Start [guide]({{< ref "armory-agent-quick" >}}) to deploy the Agent on your Kubernetes infrastructure.

## Deployment topologies

### Spinnaker service mode

In this mode, the Agent is installed as a new Spinnaker service (`spin-kubesvc`) and can be configured like other services.

![Spinnaker service mode](/images/armory-agent/in-cluster-mode.png)

If you provision clusters automatically, the Agent can dynamically reload accounts when `kubesvc.yaml` changes. You could, for example, configure accounts in a `configMap` mounting to `/opt/spinnaker/config/kubesvc-local.yaml`.  The Agent reflects `configMap` within seconds after [etcd](https://etcd.io/) sync.

### Infrastructure mode

In infrastructure mode, multiple Agent deployments handle different groups of Kubernetes clusters. Each deployment is configured separately.

![Infra mode](/images/armory-agent/agent-infra-mode.png)

> Account name must still be unique across all your infrastructure. Clouddriver will reject new accounts with a name that matches a different cluster.

### Agent mode

In this mode, the Agent acts as a piece of infrastructure. It authenticates  using a [service account token](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens). You use
[RBAC service account permissions](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#service-account-permissions) to configure what the Agent is authorized to do.

If Spinnaker is unable to communicate with the Agent, Spinnaker attempts to reconnect during a defined grace period. If Spinnaker still can't communicate with the Agent after the grace period has expired, the Agent's cluster is removed from Spinnaker.

![Agent mode](/images/armory-agent/agent-mode.png)

## Communication with Clouddriver

The Armory Agent does outbound calls only, except for a local health check, over a single [gPRC](https://grpc.io/) connection to Clouddriver. The connection can be over TLS or mTLS. You can terminate TLS:

1. On Clouddriver: in the case of running the Agent in Spinnaker Service mode or if declaring `spin-clouddriver-grpc` as a network load balancer.
2. On a gRPC proxy that directs request to the `spin-clouddriver-grpc` service.

Spinnaker will use the bidirectional communication channel to receive changes from Kubernetes accounts as well as send operations to the Agent.

### Information sent by the Agent

The Agent sends the following information about the cluster it is watching back to Spinnaker:

- Account properties as configured in `kubernetes.accounts[]`.
- Kubernetes API server host, certificate fingerprint, version.
- All the Kubernetes objects it is configured to watch and has permissions to access. You can ignore certain Kubernetes [kinds](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) (`kubernetes.accounts[].omitKinds`) or configure specific kinds to watch (`kubernetes.accounts[].kinds`).

> The Agent always scrubs data from `Secret` in memory before it is sent and even before that data makes it onto the Agent's memory heap.

## Security

Since the Armory Agent does outbound calls only, you can have agents running on-premises or in public clouds such as AWS, GCP, Azure, Oracle, or Alibaba.

What Spinnaker can do in the target cluster is limited by what it is running as:

- a `serviceAccount` in Agent mode
- a `kubeconfig` setup for infrastructure or Spinnaker service mode

Communications are secured with TLS and optionally mTLS.

Furthermore, in [Agent mode](#agent-mode), Spinnaker never gets credentials and account registration is dynamic.


## Scalability

Each Agent can scale to 100s of Kubernetes clusters. The more types of Kubernetes objects the Agent has to watch, the more memory it uses. Memory usage is bursty. You can control burst with `budget`. See [Agent options]({{< ref "agent-options/#options" >}})) for configuration information.

Scaling the Agent can mean:

- Scaling the Kubernetes `Deployment` it is part of.
- Sharding Kubernetes clusters into groups that can in turn be scaled as described in [Infrastructure mode](#infrastructure-mode) above.

You can also mix deployment strategies if you have complex Kubernetes infrastructure and permissions:

- Run as a Spinnaker service
- Run in target clusters
- Run next to traditional Spinnaker Kubernetes accounts

## Supported versions

{{< include "agent/agent-compat-matrix.md" >}}

For a full list of previous releases, see this [page](https://armory.jfrog.io/artifactory/manifests/).