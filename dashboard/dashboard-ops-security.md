---
title: Secure TiDB Dashboard
summary: Learn how to improve the security of TiDB Dashboard.
aliases: ['/docs/dev/dashboard/dashboard-ops-security/']
---

# Secure TiDB Dashboard

Although you need to sign into TiDB Dashboard before accessing it, TiDB Dashboard is designed to be accessed by trusted user entities by default. When you want to provide TiDB Dashboard to external network users or untrusted users for access, take the following measures to avoid security vulnerabilities.

## Enhance security of TiDB users

### Set a strong password for the TiDB `root` user

The account system of TiDB Dashboard is consistent with that of TiDB SQL users. By default, TiDB's `root` user has no password, so accessing TiDB Dashboard does not require password authentication, which will give the malicious visitor high privileges, including executing privileged SQL statements.

It is recommended that you set a strong password for the TiDB `root` user. See [TiDB User Account Management](/user-account-management.md) for details. Alternatively, you can disable the TiDB `root` user.

### Create a least-privileged user for TiDB Dashboard

The account system of TiDB Dashboard is consistent with that of TiDB SQL. Users accessing TiDB Dashboard are authenticated and authorized based on TiDB SQL user's privileges. Therefore, TiDB Dashboard requires limited privileges, or merely the read-only privilege. You can configure users to access TiDB Dashboard based on the principle of least privilege, thus avoiding access of high-privileged users.

It is recommended that you create a least-privileged SQL user to access and sign in to TiDB Dashboard. This avoids access of high-privileged users and improves security. See [TiDB Dashboard User Management](/dashboard/dashboard-user.md) for details.

## Use a firewall to block untrusted access

> **Note:**
>
> TiDB v6.5.0 (and later) and TiDB Operator v1.4.0 (and later) support deploying TiDB Dashboard as an independent Pod on Kubernetes. Using TiDB Operator, you can access the IP address of this Pod to start TiDB Dashboard. This port does not communicate with other privileged interfaces of PD and no extra firewall is required if provided externally. For details, see [Deploy TiDB Dashboard independently in TiDB Operator](https://docs.pingcap.com/tidb-in-kubernetes/dev/get-started#deploy-tidb-dashboard-independently).

TiDB Dashboard provides services through the PD client port, which defaults to <http://IP:2379/dashboard/>. Although TiDB Dashboard requires identity authentication, other privileged interfaces (such as <http://IP:2379/pd/api/v1/members>) in PD carried on the PD client port do not require identity authentication and can perform privileged operations. Therefore, exposing the PD client port directly to the external network is extremely risky.

It is recommended that you take the following measures:

+ Use a firewall to prohibit a component from accessing **any** client port of the PD component via the external network or untrusted network.

    > **Note:**
    >
    > TiDB, TiKV, and other components need to communicate with the PD component through the PD client port, so do not block access to the internal network between components. Otherwise, the cluster will become unavailable.

+ See [Use TiDB Dashboard behind a Reverse Proxy](/dashboard/dashboard-ops-reverse-proxy.md) to learn how to configure the reverse proxy to safely provide the TiDB Dashboard service on another port to the external network.

### How to open access to TiDB Dashboard port when deploying multiple PD instances

> **Warning:**
>
> This section describes an unsafe access solution, which is for the test environment only. **DO NOT** use this solution in the production environment.

In the test environment, you might need to configure the firewall to open the TiDB Dashboard port for external access.

When multiple PD instances are deployed, only one of the PD instances actually runs TiDB Dashboard, and browser redirection occurs when you access other PD instances. Therefore, you need to ensure that the firewall is configured with the correct IP address. For details of this mechanism, see [Deployment with multiple PD instances](/dashboard/dashboard-ops-deploy.md#deployment-with-multiple-pd-instances).

When using the TiUP deployment tool, you can view the address of the PD instance that actually runs TiDB Dashboard by running the following command (replace `CLUSTER_NAME` with the cluster name):

{{< copyable "shell-regular" >}}

```bash
tiup cluster display CLUSTER_NAME --dashboard
```

The output is the actual TiDB Dashboard address.

> **Note:**
>
> This feature is available only in the later version of the `tiup cluster` deployment tool (v1.0.3 or later).
>
> <details>
> <summary>Upgrade TiUP Cluster</summary>
>
> ```bash
> tiup update --self
> tiup update cluster --force
> ```
>
> </details>

The following is a sample output:

```bash
http://192.168.0.123:2379/dashboard/
```

In this example, the firewall needs to be configured with inbound access for the `2379` port of the `192.168.0.123` open IP, and the TiDB Dashboard is accessed via <http://192.168.0.123:2379/dashboard/>.

## Reverse proxy only for TiDB Dashboard

As mentioned in [Use a firewall to block untrusted access](#use-a-firewall-to-block-untrusted access), the services provided under the PD client port include not only TiDB Dashboard (located at <http://IP:2379/dashboard/>), but also other privileged interfaces in PD (such as <http://IP:2379/pd/api/v1/members>). Therefore, when using a reverse proxy to provide TiDB Dashboard to the external network, ensure that the services **ONLY** with the `/dashboard` prefix are provided (**NOT** all services under the port) to avoid that the external network can access the privileged interface in PD through the reverse proxy.

It is recommended that you see [Use TiDB Dashboard behind a Reverse Proxy](/dashboard/dashboard-ops-reverse-proxy.md) to learn a safe and recommended reverse proxy configuration.

## Enable TLS for reverse proxy

To further enhance the security of the transport layer, you can enable TLS for reverse proxy, and even introduce mTLS to authenticate user certificates.

See [Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html) and [HAProxy SSL Termination](https://www.haproxy.com/blog/haproxy-ssl-termination/) for more details.

## Other recommended safety measures

- [Enable TLS Authentication and Encrypt the Stored Data](/enable-tls-between-components.md)
- [Enable TLS Between TiDB Clients and Servers](/enable-tls-between-clients-and-servers.md)
