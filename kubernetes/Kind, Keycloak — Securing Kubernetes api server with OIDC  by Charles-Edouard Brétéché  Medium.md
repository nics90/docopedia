
Securing Kubernetes control plane can be a **challenging task**, especially when a company grows and more people come and go to work in a shared Kubernetes cluster.

One important tool to setup as early as possible is an **identity provider**, let it be a giant SaaS provider like Google or Okta or something more self-managed, it will allow **centralized user and permissions management**.

In this article I’m going to create a local Kubernetes cluster with [**Kind**](https://kind.sigs.k8s.io/) and deploy [**Keycloak**](https://www.keycloak.org/) identity provider. The Kubernetes **api server** will be configured to **authenticate users against Keycloak** (using OIDC) and autorisation will be implemented with **RBAC** depending on user groups.

## First, check the docs

Let’s jump to the [Kubernetes docs](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#openid-connect-tokens) to see how an identity provider can be configured.

Basically, the flow is the one described below (a user has a couple of tokens, configures `kubectl` with them and the api server validates those tokens for authentication/autorisation):

![](https://miro.medium.com/max/875/1*UHiAKX-xtkkUYrpRACHwyQ.png)

There will be some configuration to do on the **client side** for `kubectl` to work and **server side** to make the api server able to validate the tokens.

We will talk about client side later. On the server side we need to pass `--oidc-issuer-url` and `--oidc-client-id` arguments when starting the api server (other optional arguments can be needed too).

This sounds simple enough but there’s something in the docs that needs our attention:

> For an identity provider to work with Kubernetes it must:  
> \- Support [OpenID connect discovery](https://openid.net/specs/openid-connect-discovery-1_0.html); not all do.  
> \- Run in TLS with non-obsolete ciphers  
> \- Have a CA signed certificate (even if the CA is not a commercial CA or is self signed)

It means that we will need a **certificate that can be trusted by the api server**.

As Keycloak will run in the cluster, we need to generate the certificate ourselves and find a way to get this certificate considered in our Kind cluster nodes that run the api server.

Without that, the **api server won’t trust our Keycloak instance** and will not be able to validate the tokens.

## Create a root certificate

The very first thing to do is to create our **root certificate**, we will use it when creating the Kind cluster in the next step.

Creating a certificate is easily done with `openssl`:

```
# create a folder to store certificatesmkdir -p .ssl# generate an rsa keyopenssl genrsa -out .ssl/root-ca-key.pem 2048# generate root certificateopenssl req -x509 -new -nodes -key .ssl/root-ca-key.pem \  -days 3650 -sha256 -out .ssl/root-ca.pem -subj "/CN=kube-ca"
```

At this point, our root certificate lives in `.ssl/root-ca.pem` this is the certificate we want our cluster nodes to trust.

## Create Kind cluster

With our root certificate in `.ssl/root-ca.pem` we can now create our Kind cluster, asking Kind to **mount the certificate from the host to the container**.

Additionally we need to **configure the api server to use our Keycloak instance** (even if it doesn’t exist yet, we need to get this configuration done at cluster creation time).

Let’s create a cluster with the following spec:

```
# create cluster with our generated certificate# and pass necessary arguments to api serverkind create cluster --image kindest/node:v1.23.1 --config - <<EOFkind: ClusterapiVersion: kind.x-k8s.io/v1alpha4kubeadmConfigPatches:- |-  kind: ClusterConfiguration  apiServer:    extraArgs:      oidc-client-id: kube      oidc-issuer-url: https://keycloak.kind.cluster/auth/realms/master      oidc-username-claim: email      oidc-groups-claim: groups      oidc-ca-file: /etc/ca-certificates/keycloak/root-ca.pemnodes:- role: control-plane  extraMounts:  - hostPath: $PWD/.ssl/root-ca.pem    containerPath: /etc/ca-certificates/keycloak/root-ca.pem    readOnly: true- role: workerEOF
```

Next we are going to setup a couple core components (**MetalLB**, **ingress-nginx** and **dnsmasq**) to be able to talk with services in our local cluster.

## MetalLB, ingress-nginx and dnsmasq

I already wrote about **MetalLB**, **ingress-nginx** and **dnsmasq** in previous articles. Please refer to those stories if you need more details about why and how to set them up.

Or just run the script below if you want to get them installed quickly.

```
# install metallbKIND_NET_CIDR=$(docker network inspect kind \  -f '{{(index .IPAM.Config 0).Subnet}}')METALLB_IP_START=$(echo ${KIND_NET_CIDR} | sed "s@0.0/16@255.200@")METALLB_IP_END=$(echo ${KIND_NET_CIDR} | sed "s@0.0/16@255.250@")METALLB_IP_RANGE="${METALLB_IP_START}-${METALLB_IP_END}"helm upgrade --install --wait --timeout 15m \  --namespace metallb-system --create-namespace \  --repo https://metallb.github.io/metallb metallb metallb \  --values - <<EOFconfigInline:  address-pools:  - name: default    protocol: layer2    addresses:    - ${METALLB_IP_RANGE}EOF# install ingress-nginxhelm upgrade --install --wait --timeout 15m \  --namespace ingress-nginx --create-namespace \  --repo https://kubernetes.github.io/ingress-nginx \  ingress-nginx ingress-nginx \  --values - <<EOFdefaultBackend:  enabled: trueEOF# retrieve local load balancer IP addressLB_IP=$(kubectl get svc -n ingress-nginx ingress-nginx-controller \  -o jsonpath='{.status.loadBalancer.ingress[0].ip}')# point kind.cluster domain (and subdomains) to our load balancerecho "address=/kind.cluster/$LB_IP" | \  sudo tee /etc/dnsmasq.d/kind.k8s.conf# restart dnsmasqsudo systemctl restart dnsmasq
```

## Deploy and configure Keycloak

It’s not time to deploy Keycloak and configure it.

First, we will deploy it using [Helm](https://helm.sh/), then we will create the certificate from the root certificate we created earlier, and finally we will configure it using [terraform](https://www.terraform.io/).

**Deploy Keycloak Helm chart**

```
helm upgrade --install --wait --timeout 15m \  --namespace keycloak --create-namespace \  --repo https://charts.bitnami.com/bitnami keycloak keycloak \  --reuse-values --values - <<EOFauth:  createAdminUser: true  adminUser: admin  adminPassword: admin  managementUser: manager  managementPassword: managerproxyAddressForwarding: trueingress:  enabled: true  hostname: keycloak.kind.cluster  annotations:    kubernetes.io/ingress.class: nginx  tls: true  extraTls:  - hosts:    - keycloak.kind.cluster    secretName: keycloak.kind.cluster-tlspostgresql:  enabled: true  postgresqlPassword: passwordEOF
```

**Create the certificate**

```
# create certificate configuration filecat <<EOF > .ssl/req.cnf[req]req_extensions = v3_reqdistinguished_name = req_distinguished_name[req_distinguished_name][ v3_req ]basicConstraints = CA:FALSEkeyUsage = nonRepudiation, digitalSignature, keyEnciphermentsubjectAltName = @alt_names[alt_names]DNS.1 = keycloak.kind.clusterEOF# generate private keyopenssl genrsa -out .ssl/key.pem 2048# create certificate signing requestopenssl req -new -key .ssl/key.pem -out .ssl/csr.pem \  -subj "/CN=kube-ca" \  -addext "subjectAltName = DNS:keycloak.kind.cluster" \  -sha256 -config .ssl/req.cnf# create certificateopenssl x509 -req -in .ssl/csr.pem \  -CA .ssl/root-ca.pem -CAkey .ssl/root-ca-key.pem \  -CAcreateserial -sha256 -out .ssl/cert.pem -days 3650 \  -extensions v3_req -extfile .ssl/req.cnf# create secret used by keycloak ingresskubectl create secret tls -n keycloak keycloak.kind.cluster-tls \  --cert=.ssl/cert.pem \  --key=.ssl/key.pem
```

At this point **the Kubernetes api server should be able to talk with the Keycloak** instance running in our cluster, we now need to configure it.

**Configure Keycloak with terraform**

We are going to use terraform to create a couple users, groups, and an application in Keycloak, those users and groups will later be used to describe RBAC permissions.

```
cat <<'EOF' > keycloak.tfterraform {  required_providers {    keycloak = {      source  = "mrparkers/keycloak"      version = "3.6.0"    }  }}# configure keycloak providerprovider "keycloak" {  client_id                = "admin-cli"  username                 = "admin"  password                 = "admin"  url                      = "https://keycloak.kind.cluster"  tls_insecure_skip_verify = true}locals {  realm_id = "master"  groups   = ["kube-dev", "kube-admin"]  user_groups = {    user-dev   = ["kube-dev"]    user-admin = ["kube-admin"]  }}# create groupsresource "keycloak_group" "groups" {  for_each = toset(local.groups)  realm_id = local.realm_id  name     = each.key}# create usersresource "keycloak_user" "users" {  for_each       = local.user_groups  realm_id       = local.realm_id  username       = each.key  enabled        = true  email          = "${each.key}@domain.com"  email_verified = true  first_name     = each.key  last_name      = each.key  initial_password {    value = each.key  }}# configure use groups membershipresource "keycloak_user_groups" "user_groups" {  for_each  = local.user_groups  realm_id  = local.realm_id  user_id   = keycloak_user.users[each.key].id  group_ids = [for g in each.value : keycloak_group.groups[g].id]}# create groups openid client scoperesource "keycloak_openid_client_scope" "groups" {  realm_id               = local.realm_id  name                   = "groups"  include_in_token_scope = true  gui_order              = 1}resource "keycloak_openid_group_membership_protocol_mapper" "groups" {  realm_id        = local.realm_id  client_scope_id = keycloak_openid_client_scope.groups.id  name            = "groups"  claim_name      = "groups"  full_path       = false}# create kube openid clientresource "keycloak_openid_client" "kube" {  realm_id                     = local.realm_id  client_id                    = "kube"  name                         = "kube"  enabled                      = true  access_type                  = "CONFIDENTIAL"  client_secret                = "kube-client-secret"  standard_flow_enabled        = false  implicit_flow_enabled        = false  direct_access_grants_enabled = true}# configure kube openid client default scopesresource "keycloak_openid_client_default_scopes" "kube" {  realm_id  = local.realm_id  client_id = keycloak_openid_client.kube.id  default_scopes = [    "email",    keycloak_openid_client_scope.groups.name,  ]}EOFterraform init && terraform apply -auto-approve
```

Running the script above will create two groups (`**kube-admin**` and `**kube-dev**`) and two users :

-   `**user-admin**` is a member of the `**kube-admin**` group
-   `**user-dev**` is a member of the `**kube-dev**` group

Next we will configure RBAC permissions based on groups.

We usually don’t want to manage permissions at the user level but at the group level, this way granting permissions to a user boils down to adding him in the right group.

## Create RBAC permissions

This time we’re going to **configure RBAC permissions** in the cluster for the two groups we created in the previous step (we will use Kubernetes [built-in roles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles) for that):

-   `**kube-admin**` members will be granted `**cluster-admin**` role
-   `**kube-dev**` members will be granted `**edit**` role

```
kubectl apply -f - <<EOFkind: ClusterRoleBindingapiVersion: rbac.authorization.k8s.io/v1metadata:  name: kube-adminsubjects:- kind: Group  name: kube-admin  apiGroup: rbac.authorization.k8s.ioroleRef:  apiGroup: rbac.authorization.k8s.io  kind: ClusterRole  name: cluster-admin---kind: ClusterRoleBindingapiVersion: rbac.authorization.k8s.io/v1metadata:  name: kube-devsubjects:- kind: Group  name: kube-dev  apiGroup: rbac.authorization.k8s.ioroleRef:  apiGroup: rbac.authorization.k8s.io  kind: ClusterRole  name: editEOF
```

## Configure kubectl

Last step is to configure `kubectl` to use our Keycloak instance and manage the token used when talking with the Kubernetes api server.

We will fetch `**id_token**` and `**refresh_token**` from Keycloak and use this to configure `kubectl` credentials/contexts.

```
# helper functionkubectl_config(){  local ISSUER=https://keycloak.kind.cluster/auth/realms/master  local ENDPOINT=$ISSUER/protocol/openid-connect/token  local ID_TOKEN=$(curl -k -X POST $ENDPOINT \    -d grant_type=password \    -d client_id=kube \    -d client_secret=kube-client-secret \    -d username=$1 \    -d password=$1 \    -d scope=openid \    -d response_type=id_token | jq -r '.id_token')  local REFRESH_TOKEN=$(curl -k -X POST $ENDPOINT \    -d grant_type=password \    -d client_id=kube \    -d client_secret=kube-client-secret \    -d username=$1 \    -d password=$1 \    -d scope=openid \    -d response_type=id_token | jq -r '.refresh_token')  local CA_DATA=$(cat .ssl/cert.pem | base64 | tr -d '\n')  kubectl config set-credentials $1 \    --auth-provider=oidc \    --auth-provider-arg=client-id=kube \    --auth-provider-arg=client-secret=kube-client-secret \    --auth-provider-arg=idp-issuer-url=$ISSUER \    --auth-provider-arg=id-token=$ID_TOKEN \    --auth-provider-arg=refresh-token=$REFRESH_TOKEN \    --auth-provider-arg=idp-certificate-authority-data=$CA_DATA  kubectl config set-context $1 --cluster=kind-kind --user=$1}# setup config for our userskubectl_config user-adminkubectl_config user-dev
```

Now, you should have **three contexts** in your `kubectl` config:

```
$ kubectl config get-contexts CURRENT   NAME         CLUSTER     AUTHINFO     NAMESPACE*         kind-kind    kind-kind   kind-kind              user-admin   kind-kind   user-admin             user-dev     kind-kind   user-dev
```

-   `**kind-kind**` is the admin context that was created by Kind
-   `**user-admin**` and `**user-dev**` are contexts linked to our Keycloak users

## Testing time

Finally we can test the new contexts by switching to `**user-dev**` and trying to create a namespace:

```
$ kubectl config use-context user-dev                  Switched to context "user-dev".$ kubectl create ns testError from server (Forbidden): namespaces is forbidden: User "user-dev@domain.com" cannot create resource "namespaces" in API group "" at the cluster scope
```

Doing the same thing with `**user-admin**` should work fine:

```
$ kubectl config use-context user-admin                                                                                                                                                                     Switched to context "user-admin".$ kubectl create ns testnamespace/test created
```

Everything works as expected 🎉

-   **RBAC** permissions are managed on a per group basis in the cluster
-   **users** and **groups** are managed completely separately in **Keycloak**
-   `kubectl` knows how to maintain **credential tokens** and send them to the Kubernetes api server when making requests
-   Kubernetes api server knows how to validate tokens and authenticate the user behind a request

## Wrapping it up

In the end, getting everything to work was not that easy and illustrates well how **securing Kubernetes is a difficult task**.

I’ts worth the effort though as **security is not option**.

Securing the Kubernetes api server of a local cluster is not that crucial, but being able to do it is a nice thing as it allows us to **test things locally** before making changes in a production cluster.
