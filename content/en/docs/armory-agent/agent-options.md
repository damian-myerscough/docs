---
title: Armory Agent Configuration Options
linkTitle: Agent Options
weight: 3
description: >
  Learn how to configure the Armory Agent based on installation mode and environment restrictions. This guide contains a detailed list of configuration options.
---
![Proprietary](/images/proprietary.svg)
## Where to configure the Agent

Set these options at the agent level in the `kubesvc.yaml` configuration file. If deploying as a non-Spinnaker<sup>TM</sup> service, you need to specify a `clouddriver.grpc` endpoint (e.g. `grpc.spinnaker.example.com:443`).

## Kubernetes account

At a minimum you will need to add an account, give it a name, and set its Spinnaker permissions.

### Spinnaker Service and Infrastructure modes

In these modes, you set up multiple accounts per agent. Your configuration should look like:

```yaml
kubernetes:
  accounts:
    - name: account-01
      kubeconfigFile: /kubeconfigfiles/kubecfg-account01.yaml
    - ...  
```

> If you are migrating accounts from Clouddriver, you can just copy the same block of configuration here. Unused properties are ignored.


### Agent mode

In agent mode, your configuration should look like:

```yaml
kubernetes:
  accounts:
    - name: account-01
      serviceAccount: true
```


## Options

| Settings                                                                                                                                                                    | Type              | Default                 | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `clouddriver.grpc`                                                                                                                                                          | string (hostname) | `spin-clouddriver-grpc:9091` | Hostname of the Clouddriver or gRPC proxy endpoint  |
| `clouddriver.insecure`  | boolean           | false                   | If true, we’re connecting to a non TLS server |
| `clouddriver.tls.serverName` | string | none | Server name on the remote certificate (override from the hostname) |
| `clouddriver.tls.insecureSkipVerify` | boolean | false | Do not verify the endpoint's certificate |
| `clouddriver.tls.clientCertFile`<br>`clouddriver.tls.clientKeyFile`<br>`clouddriver.tls.clientKeyFilePassword`  | string<br>string<br>string| none<br>none<br>none | Client certificate file for mTLS<br>Client key file if not included in the certificate<br>Password the key file if needed|
| `clouddriver.tls.cacertFile` | string | none |  If provided, verify endpoint certificate with the trust store. Otherwise, the system trust store is used. |
| `clouddriver.auth.token`| string | none | <span class="badge badge-primary">0.3.0+</span> Optional bearer token added to each request back to the endpoint. |
| `clouddriver.auth.tokenCommand.command`<br>`clouddriver.auth.tokenCommand.args`<br>`clouddriver.auth.tokenCommand.format`<br>`clouddriver.auth.tokenCommand.refreshIntervalSeconds` | string<br>[]string<br>string<br>integer | none<br>none<br>[]<br>0 | <span class="badge badge-primary">0.3.0+</span> Allows to invoke a command every `refreshIntervalSeconds` seconds that outputs either the token (`format` is `raw`) or a JSON object with an attribute of `token` if `format` is `json` or left empty. `args` is the optional list of parameters to the command.|
| `clouddriver.noProxy` | boolean | false | <span class="badge badge-primary">0.3.1+</span> Ignore the `HTTP_PROXY`, `HTTPS_PROXY`, and `NO_PROXY` environment variables when connecting back to the control plane (Spinnaker) |
| `logging.file`  | string            | stdout if not defined   | File to save logs to  |
| `logging.level`| string | `INFO` | Log level. Can be any of (case insensitive):<br>`panic`, `fatal` , `error`, `warn` (or `warning`),  `info`, `debug`, `trace` |
| `kubernetes.noProxy` | boolean | false | <span class="badge badge-primary">0.3.1+</span> Ignore the `HTTP_PROXY`, `HTTPS_PROXY`, and `NO_PROXY` environment variables when connecting to any Kubernetes cluster |
| `kubernetes.reconnectTimeoutMs` | integer           | 5000 | How long to wait before reconnecting to Spinnaker |
| `kubernetes.accounts[].name`  | string | none, required          | Name of the Kubernetes cluster in Spinnaker. Spinnaker [still needs to accept that name](../#infrastructure-mode). |
| `kubernetes.accounts[].kubeConfigFile`  | string            | none  | Path to the kubeconfig file if not using `serviceAccount`|
| `kubernetes.accounts[].insecure` | boolean           | false | Do not verify the TLS certificate of the Kubernetes API server<br>Don’t use without a good reason. |
| `kubernetes.accounts[].context`  | string            | empty | If provided, will use the given context of the configured kubeconfig |
| `kubernetes.accounts[].oAuthScopes` | []string          | empty                   | List of OAuth scope when authenticating with gcp provider<br>https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl#authentication  |
| `kubernetes.accounts[].serviceAccount`                                                                                                                                      | boolean           | false                   | If true and the Agent runs in Kubernetes - use the current service account to call to the current API server. In that mode, you don’t need to provide a kubeconfig file.|
| `kubernetes.accounts[].namespaces`                                                                                                                                          | []string          | empty                   | <span class="badge badge-primary">0.4.0+</span> Whitelist of namespaces to monitor.<br>This comes at a greater cost of multiplying the resources by the number of namespaces.|
| `kubernetes.accounts[].omitNamespaces` | []string          | empty | Blacklist of namespaces <br>This comes at a greater cost of multiplying the resources by the number of namespaces.<br>NOT CURRENTLY IMPLEMENTED |
| `kubernetes.accounts[].onlyNamespacedResources`                                                                                                                                          | boolean          | false                   | <span class="badge badge-primary">0.4.0+</span> If true, the Agent will ignore non-namespaced resources; namespaces must be whitelisted with `namespaces` setting and CRDs with `customResourceDefinitons`.|
| `kubernetes.accounts[].kinds` | []string          | empty                   | If not empty, only kinds in the list will be cached. Use the format `<kind>.<apiGroup>` (e.g. `Deployment.apps`)|
| `kubernetes.accounts[].omitKinds`  | []string          | empty                   | List of kinds not to cache.|
| `kubernetes.accounts[].customResourceDefinitions`  | []{kind: <string>}          | empty                   | <span class="badge badge-primary">0.4.0+</span> List of CustomResourceDefinition to expose to Spinnaker. This is not needed if `onlyNamespacedResources` is left off. The format of `kind` is `Kind.group`.|
| `kubernetes.accounts[].metrics` | boolean | false  | When true, sends pod metrics back to Spinnaker every 20s |
| `kubernetes.accounts[].permissions` | list               | empty                   | List of permissions (currently `READ` or `WRITE`) with a list of roles authorized. For more information, see [Permissions format](#permissions-format).|
| `kubernetes.accounts[].maxResumableResourceAgeMs` | integer           | 300000 (5m)             | When connecting to Spinnaker, the Agent asks Clouddriver for the latest resource version known per resource that is not older than that setting.<br><br>The resource version is used to resume the watch without first doing a list - saving memory and time. There’s no guarantee that the resource version is still known. If not “remembered” by the Kubernetes API server, a `list`  call will be used.<br><br>https://kubernetes.io/docs/reference/using-api/api-concepts/#efficient-detection-of-changes |
| `kubernetes.accounts[].onlySpinnakerManaged`   | boolean           | false   | Only return Spinnaker managed resources<br>NOT IMPLEMENTED in the Agent but added to the plugin see `kubesvc.runtime.defaults.onlySpinnakerManaged`|
| `kubernetes.accounts[].noProxy` | boolean | false | <span class="badge badge-primary">0.3.1+</span> Ignore the `HTTP_PROXY`, `HTTPS_PROXY`, and `NO_PROXY` environment variables when connecting to that Kubernetes cluster |
| `server.host` | string            | localhost               | hostname of the server health check |
| `server.port` | integer           | 8082                    | port of the server health check |
| `server.ssl.enabled`, `server.ssl.certFile`, `server.ssl.keyFile`, `server.ssl.keyPassword`, `server.ssl.caCertFile`, `server.ssl.keyFilePassword`, `server.ssl.clientAuth` |                   |                         | Various options to control TLS config. Don’t bother, it’s just for the health endpoint.                                                                                                                                                                                                                                                                                                                                                                                                         |
| `prometheus.enabled`                                                                                                                                                        | boolean           | false                   | Enable Prometheus handler                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `prometheus.port`                                                                                                                                                           | integer           | 8008                    | Port to expose Prometheus metrics on. Responds to both `/metrics` (standard) and `/prometheus_metrics` (Spinnaker default)                                                                                                                                                                                                                                                                                                                                                                      |
| `tasks.totalBudget`                                                                                                                                                         | integer           | 0                       | If > 0, limits the number of tasks that can be started. Tasks have different cost. Watches are considered free because they are part of the normal operations of the Agent.                                                                                                                                                                                                                                                                                                                       |
| `tasks.budgetPerAccount`                                                                                                                                                    | integer           | 0                       | Same as above but per account. If both settings are provided, they’re both checked.                                                                                                                                                                                                                                                                                                                                                                                                             |
| `tasks.queueCheckFrequencyMs`                                                                                                                                               | integer           | 2000                    | Frequency at which the Agent will check for new tasks to launch. Once launched a task is not stopped until explicitly requested (account unregistered or connection to Spinnaker lost)                                                                                                                                                                                                                                                                                                            |
| `pprof.enabled`                                                                                                                                                             | boolean           | false                   | Enable pprof endpoint. Useful for troubleshooting, slowness, memory leaks, and more!<br>https://github.com/google/pprof/blob/master/doc/README.md                                                                                                                                                                                                                                                                                                                                               |
| `pprof.port`                                                                                                                                                                | integer           | 6060                    | Port on which to respond to pprof requests                                                                                                                                       |
| `secrets.vault.*`                                                                                                                                                           | object            | none                    | [Vault configuration]({{< ref "secrets-vault#1-kubernetes-service-account-recommended" >}}) |

### Permissions format

Permissions for the Agent use a format that is slightly different than the format that Clouddriver uses for permissions:

```
kubernetes:
  accounts:
    - name: my-k8s-account
      permissions:
        - READ: ['role1', 'role2']
        - WRITE: ['role3', 'role4']
```


## Restricted environments

### Network access

The Agent needs access to its control plane (Spinnaker) as well to the various Kubernetes clusters it is configured to monitor. You can control which traffic should go through an HTTP proxy via the usual `HTTP_PROXY`, `HTTPS_PROXY`, and `NO_PROXY` environment variables.

A common case is to force the connection back to the control plane via a proxy but bypass it for Kubernetes clusters. In that case, define the environment variable `HTTPS_PROXY=https://my.corporate.proxy` and use the `kubernetes.noProxy: true` setting to not have to maintain the list of Kubernetes hosts in `NO_PROXY`.


### Kubernetes authorization

The Agent should be configured to access each Kubernetes cluster it monitors with a service account. You can limit what Spinnaker can do via the role you assign to that service account. For example, if you'd like Spinnaker to see `NetworkPolicies` but not deploy them:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: agent-role
rules:
- apiGroups: ["networking.k8s.io"]
  resources: ["networkpolicies"]
  verbs: [ "get", "list", "watch"]
...
```

### Namespace restrictions

You can limit the Agent to monitoring specific namespaces by listing them under `namespaces`. If you need to prevent the Agent from accessing cluster-wide (non-namespaced) resources, use the `onlyNamespacedResources` setting.

A side effect of disabling cluster-wide resources is that [CustomResourceDefinitions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) won't be known (and therefore deployable by Spinnaker). `CustomResourceDefinitions` are cluster-wide resources, but the custom resources themselves may be namespaced. To workaround the limitation, you can define `customResourceDefinitions`. Both namespaces and CRDs will be sent to Spinnaker as "synthetic" resources. They won't be queried or watched, but their existence will be known by Spinnaker.

```yaml
kubernetes:
    accounts:
        - name: production
          ...
          # Restricts the agent to namespaces `ns1` and `ns2`
          namespaces:
            - ns1
            - ns2
          # Prevents the Agent from querying non-namespaced resources
          onlyNamespacedResources: true
          # Whitelist CRDs so Spinnaker
          customResourceDefinitions:
            - kind: ServiceMonitor.monitoring.coreos.com
            - kind: SpinnakerService.spinnaker.armory.io

```


