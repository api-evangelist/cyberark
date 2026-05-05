---
title: "Istio OIDC Authentication"
url: "https://developer.cyberark.com/blog/istio-oidc-authentication/"
date: "Sat, 17 Aug 2024 07:15:49 +0000"
author: "Luke Addison"
feed_url: "https://developer.cyberark.com/feed/"
---
<p><em><strong>This post has been updated for Istio version 1.11.4</strong></em></p>
<p>A service mesh is an architectural pattern that provides common network services as a feature of the infrastructure. This typically includes features such as service discovery and policy enforcement to control how services within the mesh can communicate with each other.</p>
<p>Istio is a service mesh implementation that works by running an instance of <a href="https://www.envoyproxy.io/" rel="noopener" target="_blank">Envoy</a> alongside each instance of each of your services to intercept and proxy service traffic. Additionally, fleets of standalone Envoys are deployed to handle traffic entering and leaving the mesh; Istio’s main purpose then is to configure and expose the functionality of Envoy.</p>
<p>In addition to the <a href="https://istio.io/latest/about/service-mesh/#concepts" rel="noopener" target="_blank">core features</a>, Istio also supports powerful extension points, as well as the ability to apply custom configuration to the Envoy sidecars. Here we will describe how Istio can be configured to manage the <a href="https://openid.net/connect/" rel="noopener" target="_blank">OpenID Connect</a> (OIDC) authentication flow for applications running within the mesh to allow both authentication and authorisation decisions to be offloaded to Istio. There are a number of ways to achieve this with Istio however here we look at two solutions and how their integration points have been affected by changes to Istio’s architecture.</p>
<h2>OIDC</h2>
<p>OIDC is an identity layer built upon the OAuth 2.0 protocol which allows the identity of a user to be verified based on authentication to an identity provider. There are <a href="https://openid.net/specs/openid-connect-core-1_0.html#Authentication" rel="noopener" target="_blank">different types of authentication flow</a> which dictate how authentication is handled by the identity provider, but the most common is the <a href="https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth" rel="noopener" target="_blank">Authorization Code Flow</a>, which we will use here.</p>
<p>Typically, when a user first visits an HTTP service that implements the OIDC Authorization Code Flow, they are redirected to an identity provider (for example <a href="https://developers.google.com/identity/protocols/oauth2/openid-connect" rel="noopener" target="_blank">Google</a>, <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-protocols-oidc" rel="noopener" target="_blank">Azure</a> or <a href="https://github.com/dexidp/dex" rel="noopener" target="_blank">dex</a>) where they would login to obtain an Authorization Code. The user is then redirected again back to the original service, passing the Authorization Code as an <a href="https://openid.net/specs/openid-connect-core-1_0.html#AuthResponse" rel="noopener" target="_blank">HTTP query parameter</a>. The service (or OIDC client) can then exchange this Authorization Code (using its Client ID and Client Secret) for a <a href="https://jwt.io/introduction" rel="noopener" target="_blank">JWT</a> called an <a href="https://openid.net/specs/openid-connect-core-1_0.html#IDToken" rel="noopener" target="_blank">ID Token</a>. This JWT is signed by the identity provider (which the service should verify) with fields (or claims) that contain information about the user who logged in (for example their email address) and can be used to make authentication and authorization decisions. Note that since the user does not have access to the service’s Client Secret, they are unable to obtain such a JWT locally. Instead, in order to prevent the user from having to login on every request, the service can implement login sessions; one way in which this can be achieved is by setting an HTTP cookie and then injecting the obtained JWT into subsequent requests based on the presence of this cookie. Session management is discussed further for our setup below.</p>
<p>OIDC is a common way of delegating the responsibility of managing user credentials to a third-party identity provider and a powerful feature of Istio is that it can be leveraged to manage this flow without your backend service needing to be aware OIDC is even being used.</p>
<p>For more information on OIDC and associated terminology, Okta has a great <a href="https://developer.okta.com/blog/2017/07/25/oidc-primer-part-1" rel="noopener" target="_blank">primer</a>.</p>
<h2>Istio 1.4</h2>
<p>Istio 1.4 and earlier included a component called <a href="https://istio.io/v1.4/docs/reference/config/policy-and-telemetry/mixer-overview/" rel="noopener" target="_blank">Mixer</a> which formed part of the <a href="https://istio.io/latest/docs/ops/deployment/architecture/" rel="noopener" target="_blank">Istio control plane</a>. When policy checks were enabled, before Envoy made an upstream connection it would make a logical request to Mixer in order to determine whether the connection was allowed and what action to take. Mixer therefore provided an extension point for Istio, allowing integrations with external components that could make these policy decisions on its behalf.</p>
<p>The <a href="https://istio.io/latest/blog/2019/app-identity-and-access-adapter/" rel="noopener" target="_blank">App Identity and Access Adapter</a> is an example of an external component that interfaces with Mixer in exactly this way; by analyzing attributes sent by Envoy to Mixer when making policy decisions it can work out whether a user has already authenticated to a configured identity provider and if not trigger the OIDC flow. The resulting JWT can then be compared against <a href="https://istio.io/latest/blog/2019/app-identity-and-access-adapter/#applying-web-application-protection" rel="noopener" target="_blank">policy configuration</a> to either allow or deny access to the upstream service.</p>
<h2>Istio 1.5 and Above</h2>
<p>Since Istio 1.5, Mixer has been <a href="https://istio.io/v1.5/docs/tasks/policy-enforcement/enabling-policy/" rel="noopener" target="_blank">deprecated</a> in favour of implementing these extensions within Envoy itself. In particular this reduces the latency of policy decisions that would otherwise require a network call to Mixer.</p>
<p>Here we describe in detail an alternative way to configure Istio to manage the OIDC authentication flow and authorization decisions but without Mixer. For the purpose of this description we will assume we are running on a 1.19 GKE cluster with Istio 1.11.4 installed, but this setup should be compatible with recent versions of both Kubernetes and Istio:</p>
<pre class="prettyprint">gcloud container clusters create istio \
--cluster-version 1.19 \
--machine-type n1-standard-2 \
--enable-autoscaling \
--max-nodes=5
# Install istioctl for Linux or OS X -- if you are on Windows you can download directly from GitHub
# https://github.com/istio/istio/releases
export ISTIO_VERSION=1.11.4
curl -sL https://istio.io/downloadIstioctl | sh -
# Install Istio configured with our custom extension provider to handle the OIDC authentication flow
# https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/#MeshConfig-ExtensionProvider
$HOME/.istioctl/bin/istioctl manifest install -y -f - &lt;&lt;EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
meshConfig:
extensionProviders:
- name: oauth2-proxy
envoyExtAuthzHttp:
service: oauth2-proxy.oauth2-proxy.svc.cluster.local
port: 4180
includeRequestHeadersInCheck:
- cookie
headersToUpstreamOnAllow:
- authorization
headersToDownstreamOnDeny:
- set-cookie
EOF
# Lock down mutual TLS for the entire mesh
# https://istio.io/latest/docs/tasks/security/authentication/mtls-migration/#lock-down-mutual-tls-for-the-entire-mesh
kubectl apply -f - &lt;&lt;EOF
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
name: default
namespace: istio-system
spec:
mtls:
mode: STRICT
EOF
</pre>
<p>We will deploy Nginx to the cluster to act as the test application we want to configure authenticaiton and authorisation for. You will need to configure a domain (I will use <code>nginx.lukeaddison.co.uk</code>) to expose Nginx on. It should be pointing at the following IP address (this may take a few moments to provision):</p>
<pre class="prettyprint">kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
</pre>
<p>We also deploy <a href="https://venafi.com/open-source/cert-manager/" rel="noopener" target="_blank">cert-manager</a> to provision a <a href="https://letsencrypt.org/" rel="noopener" target="_blank">Let’s Encrypt</a> TLS certificate for us.</p>
<p><em>WARNING: Once OIDC authentication is enforced on the Istio ingress gateway, cert-manager will no longer be able to renew the certificate using the </em><a href="https://cert-manager.io/docs/configuration/acme/http01/" rel="noopener" target="_blank"><em>HTTP-01 challenge solver</em></a><em> configured below. One solution would be to use a </em><a href="https://cert-manager.io/docs/configuration/acme/dns01/" rel="noopener" target="_blank"><em>DNS-01 challenge solver</em></a><em> instead</em></p>
<pre class="prettyprint"># Set your Nginx domain
NGINX_DOMAIN=""
# Install cert-manager
CERT_MANAGER_VERSION="v1.6.1"
kubectl apply -f "https://github.com/jetstack/cert-manager/releases/download/$CERT_MANAGER_VERSION/cert-manager.yaml"
# Wait for cert-manager to be ready and then configure ingress with TLS termination. Note that we do
# not configure HTTPS redirection to support Let's Encrypt ACME HTTP-01 challenges
kubectl apply -f - &lt;&lt;EOF
# https://istio.io/latest/docs/tasks/traffic-management/ingress/kubernetes-ingress/#specifying-ingressclass
apiVersion: networking.k8s.io/v1beta1
kind: IngressClass
metadata:
  name: istio
spec:
  controller: istio.io/ingress-controller
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt
    solvers:
    - http01:
        ingress:
          class: istio
---
apiVersion: v1
kind: Namespace
metadata:
  name: nginx
  labels:
    istio-injection: enabled
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: nginx
  namespace: nginx
spec:
  hosts:
  - $NGINX_DOMAIN
  gateways:
  - istio-system/nginx
  http:
  - route:
    - destination:
        port:
          number: 80
        host: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: nginx
  namespace: istio-system
spec:
  secretName: nginx-tls
  issuerRef:
    name: letsencrypt
    kind: ClusterIssuer
  dnsNames:
  - $NGINX_DOMAIN
---
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: nginx
  namespace: istio-system
spec:
  selector:
    app: istio-ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - nginx/$NGINX_DOMAIN
    tls:
      credentialName: nginx-tls
      mode: SIMPLE
EOF</pre>
<p>After a few moments Nginx should become available to the internet which you can verify by curling your domain:</p>
<pre class="prettyprint">curl "https://${NGINX_DOMAIN}"
</pre>
<h2>Authentication</h2>
<p>We can now configure OIDC authentication. The first thing we need to do is to configure our OIDC identity provider; we will be using Google here but any compatible provider should work.</p>
<p>For Google specifically, we need to create a Google OAuth application. The <a href="https://www.kubeflow.org/docs/gke/deploy/oauth-setup" rel="noopener" target="_blank">Kubeflow documentation</a> walks through the steps to achieve this for <a href="https://cloud.google.com/iap" rel="noopener" target="_blank">IAP</a>. The functional differences for us is that we want to specify <code>Authorized domains</code> to be whatever domain Nginx is exposed on (so <code>nginx.lukeaddison.co.uk</code> for me) and in <code>Authorized redirect URIs</code> specify the <code>/oauth2/callback</code> path of the Nginx domain (so <code>https://nginx.lukeaddison.co.uk/oauth2/callback</code> for me).</p>
<p>For other providers the configuration (especially the requirement to whitelist redirect URIs) should be similar. Whichever provider you use the setup process should return a Client ID and Client Secret which you should remember for a later step.</p>
<p>We can now enforce that access to the Nginx service be authenticated using our OIDC provider. Using the discovery URL (supported by most providers) we can retrieve the information required to configure Istio.</p>
<pre class="prettyprint"># https://developers.google.com/identity/protocols/oauth2/openid-connect#discovery
OIDC_DISCOVERY_URL="https://accounts.google.com/.well-known/openid-configuration"
OIDC_DISCOVERY_URL_RESPONSE=$(curl $OIDC_DISCOVERY_URL)
OIDC_ISSUER_URL=$(echo $OIDC_DISCOVERY_URL_RESPONSE | jq -r .issuer)
OIDC_JWKS_URI=$(echo $OIDC_DISCOVERY_URL_RESPONSE | jq -r .jwks_uri)
kubectl apply -f - &lt;&lt;EOF
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: istio-ingressgateway
  namespace: istio-system
spec:
  selector:
    matchLabels:
      app: istio-ingressgateway
  jwtRules:
  - issuer: $OIDC_ISSUER_URL
    jwksUri: $OIDC_JWKS_URI
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: istio-ingressgateway
  namespace: istio-system
spec:
  selector:
    matchLabels:
      app: istio-ingressgateway
  action: CUSTOM
  provider:
    # Extension provider configured when we installed Istio
    name: oauth2-proxy
  rules:
  - {}
EOF
</pre>
<p>The RequestAuthentication resource says that if a request to the ingress gateway contains a <a href="https://oauth.net/2/bearer-tokens" rel="noopener" target="_blank">bearer token</a> in the Authorization header then it must be a valid JWT signed by the specified OIDC provider. Istio will <a href="https://istio.io/latest/docs/tasks/security/authorization/authz-jwt/#allow-requests-with-valid-jwt-and-list-typed-claims" rel="noopener" target="_blank">concatenate the <code>iss</code> and <code>sub</code> fields</a> of the JWT with a <code>/</code> separator which will form the principal of the request. The AuthorizationPolicy says to contact oauth2-proxy for authorization decisions.</p>
<p>Since we have not deployed oauth2-proxy yet, visiting your domain again should now show: <code>RBAC: access denied</code>, so the final thing we need to do is to deploy oauth2-proxy to manage the OIDC flow, retrieve a JWT and inject it into each request.</p>
<h2>oauth2-proxy</h2>
<p>Envoy supports pluggable functionality known as filters which can be chained together and affect how requests are handled. One of Istio’s main roles is to configure these filters across a fleet of Envoys so that they form a mesh, supporting high-level APIs such as <a href="https://istio.io/latest/docs/reference/config/networking/virtual-service/" rel="noopener" target="_blank">VirtualService</a> and <a href="https://istio.io/latest/docs/reference/config/networking/destination-rule/" rel="noopener" target="_blank">DestinationRule</a> for operators to declare their desired mesh behaviour. The extension provider configuration we specified when we first installed Istio, together with the AuthorizationPolicy configuration above, configures the <a href="https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/ext_authz_filter" rel="noopener" target="_blank">external authorization HTTP filter</a> which is responsible for calling out to oauth2-proxy:</p>
<p><em>Before Istio 1.9, the same external authorization configuration could be supplied by applying an </em><a href="https://istio.io/latest/docs/reference/config/networking/envoy-filter/" rel="noopener" target="_blank"><em>EnvoyFilter</em></a></p>
<pre class="prettyprint"># Set your Client ID and Client Secret from your OIDC provider setup
CLIENT_ID=""
CLIENT_SECRET=""
# https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/overview/#generating-a-cookie-secret
COOKIE_SECRET=$(openssl rand -base64 32 | tr -- '+/' '-_')
kubectl apply -f - &lt;&lt;EOF
apiVersion: v1
kind: Namespace
metadata:
  name: oauth2-proxy
  labels:
    istio-injection: enabled
---
apiVersion: v1
kind: Secret
metadata:
  name: oauth2-proxy
  namespace: oauth2-proxy
stringData:
  OAUTH2_PROXY_CLIENT_ID: $CLIENT_ID
  OAUTH2_PROXY_CLIENT_SECRET: $CLIENT_SECRET
  OAUTH2_PROXY_COOKIE_SECRET: $COOKIE_SECRET
---
apiVersion: v1
kind: Service
metadata:
  name: oauth2-proxy
  namespace: oauth2-proxy
spec:
  selector:
    app: oauth2-proxy
  ports:
  - name: http
    port: 4180
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oauth2-proxy
  namespace: oauth2-proxy
spec:
  selector:
    matchLabels:
      app: oauth2-proxy
  template:
    metadata:
      labels:
        app: oauth2-proxy
    spec:
      containers:
      - name: oauth2-proxy
        image: quay.io/oauth2-proxy/oauth2-proxy:v7.2.0
        args:
        - --provider=oidc
        - --cookie-secure=true
        - --cookie-samesite=lax
        - --cookie-refresh=1h
        - --cookie-expire=4h
        - --cookie-name=_oauth2_proxy_istio_ingressgateway
        - --set-authorization-header=true
        - --email-domain=*
        - --http-address=0.0.0.0:4180
        - --upstream=static://200
        - --skip-provider-button=true
        - --whitelist-domain=$NGINX_DOMAIN
        - --oidc-issuer-url=$OIDC_ISSUER_URL
        env:
        - name: OAUTH2_PROXY_CLIENT_ID
          valueFrom:
            secretKeyRef:
              name: oauth2-proxy
              key: OAUTH2_PROXY_CLIENT_ID
        - name: OAUTH2_PROXY_CLIENT_SECRET
          valueFrom:
            secretKeyRef:
              name: oauth2-proxy
              key: OAUTH2_PROXY_CLIENT_SECRET
        - name: OAUTH2_PROXY_COOKIE_SECRET
          valueFrom:
            secretKeyRef:
              name: oauth2-proxy
              key: OAUTH2_PROXY_COOKIE_SECRET
        resources:
          requests:
            cpu: 10m
            memory: 100Mi
        ports:
        - containerPort: 4180
          protocol: TCP
        readinessProbe:
          periodSeconds: 3
          httpGet:
            path: /ping
            port: 4180
EOF</pre>
<p>Note that the above configuration tells oauth2-proxy to store session state as a browser cookie. This has the advantage of being stateless but can lead to large HTTP headers if the JWT returned from the OIDC flow is large; an alternative is to use <a href="https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/session_storage/#redis-storage" rel="noopener" target="_blank">Redis</a> which adds a further operational burden but the size of the cookie is small and constant.</p>
<p>Visiting Nginx again you should be redirected to your OIDC provider. After signing in successfully oauth2-proxy should set an encrypted cookie which on subsequent requests will be decrypted to a JWT and attached as the Authorization header which Istio can validate.</p>
<p>If you want to authenticate multiple services you may want to configure the <code>--cookie-domain</code> and <code>--whitelist-domain</code> flags on the oauth2-proxy Deployment to include multiple subdomains. For example, I could set both of those flags to <code>.lukeaddison.co.uk</code> and then the cookie would be sent for any subdomain of <code>lukeaddison.co.uk</code> meaning I would only need to sign in once. To see the full set of configuration options go <a href="https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/overview" rel="noopener" target="_blank">here</a>.</p>
<p>Another powerful use case is to combine the OIDC authentication configuration with <a href="https://istio.io/latest/blog/2019/proxy/">Istio’s ability to proxy to external services</a>. For example, if I have a service running outside of Kubernetes but that does not have its own identity-aware authentication mechanism, Istio could be used as a reverse proxy to configure access to that service in a similar way to if it was running within the mesh.</p>
<h2>Authorization</h2>
<p>With the above configuration in place, we can now make further authorization decisions based on the attached JWT and corresponding claims. For example, to ensure that the user signed into our configured OAuth application (instead of another Google one) we can restrict access based on the <a href="https://openid.net/specs/openid-connect-core-1_0.html#IDToken" rel="noopener" target="_blank">audience claim</a>. This is straightforward since the audience claim must contain our Client ID:</p>
<pre class="prettyprint"># Use of request.auth.audiences is currently not supported with CUSTOM action so we define the check
# as a separate AuthorizationPolicy
kubectl apply -f - &lt;&lt;EOF
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: istio-ingressgateway-audience
  namespace: istio-system
spec:
  selector:
    matchLabels:
      app: istio-ingressgateway
  rules:
  - when:
    - key: request.auth.audiences
      values:
      - $CLIENT_ID
EOF</pre>
<p>Since oauth2-proxy is making a decision about whether to allow a request depending on the existence of a JWT stored in an encrypted cookie, it shouldn’t be possible for a user to gain access using a JWT from a different source. Nevertheless, representing the expected value natively in Istio provides defence in depth.</p>
<p>We can also configure the ingress gateway Envoys to forward the Authorization header to our upstream service to allow for service specific policy (the following assumes that the email claim is present on the JWT):</p>
<pre class="prettyprint">EMAIL_ADDRESS="luke.addison@jetstack.io"
kubectl apply -f - &lt;&lt;EOF
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: istio-ingressgateway
  namespace: istio-system
spec:
  selector:
    matchLabels:
      app: istio-ingressgateway
  jwtRules:
  - issuer: $OIDC_ISSUER_URL
    jwksUri: $OIDC_JWKS_URI
    # Forward JWT to Nginx sidecar
    forwardOriginalToken: true
---
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: nginx
  namespace: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  jwtRules:
  - issuer: $OIDC_ISSUER_URL
    jwksUri: $OIDC_JWKS_URI
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: nginx
  namespace: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  rules:
  - when:
    - key: request.auth.audiences
      values:
      - $CLIENT_ID
    - key: request.auth.claims[email]
      values:
      - $EMAIL_ADDRESS
EOF</pre>
<p>Restricting access to an email address which differs from yours should give <code>RBAC: access denied</code> as before. Unfortunately, currently <a href="https://github.com/istio/proxy/blob/8ff37d65b3c41e9a28af29a6ba43d146aa8fa3de/src/envoy/http/authn/authn_utils.cc#L54-L55" rel="noopener" target="_blank">only string and string list claims are extracted from the JWT</a>, so in particular the boolean <code>email_verified</code> claim (signifying whether the provider took steps to ensure the email address was controlled by the end user) cannot be matched on. However, as described in the <a href="https://openid.net/specs/openid-connect-core-1_0.html#ClaimStability" rel="noopener" target="_blank">Claim Stability and Uniqueness</a> section of the OIDC specification, the <code>iss</code> and <code>sub</code> claims used together are the only claims that can provide a stable identifier for a user, so if that is the goal then they should be used instead of the <code>email</code> claim. For more details on what is supported by AuthorizationPolicy see the documentation <a href="https://istio.io/latest/docs/reference/config/security/authorization-policy/" rel="noopener" target="_blank">here</a> and <a href="https://istio.io/latest/docs/reference/config/security/conditions/" rel="noopener" target="_blank">here</a>.</p>
<p>The Dex <a href="https://github.com/dexidp/dex/tree/master/examples/example-app" rel="noopener" target="_blank">example-app</a> is a useful tool for retrieving a JWT locally to see the claims your identity provider returns for a particular set of <a href="https://auth0.com/docs/scopes/openid-connect-scopes">scopes</a>. Note the <a href="https://github.com/oauth2-proxy/oauth2-proxy/blob/v7.2.0/pkg/validation/options.go#L138-L144" rel="noopener" target="_blank">scopes requested by oauth2-proxy by default</a>. By configuring oauth2-proxy to request different scopes, you can adjust the claims that are present on the returned JWT and thus the attributes that can be matched on for authorization decisions. Returning group membership for example allows access to particular services to be granted and revoked by simply moving users within your provider, without any changes to the Istio configuration.</p>
<p>As we have demonstrated, a really powerful aspect of this is that our backend service can be completely unaware that OIDC is being used and does not need to support it itself. However, if the service has support for parsing the JWT, then it can also be used to authorize granular access to different features of the service.</p>
<h2>Cleanup</h2>
<pre class="prettyprint">gcloud container clusters delete istio --async
</pre>
<h2>Future</h2>
<p>Another nascent project in this area is <a href="https://github.com/istio-ecosystem/authservice" rel="noopener" target="_blank">authservice</a> which provides an alternative implementation of an <a href="https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/ext_authz_filter" rel="noopener" target="_blank">external authorization</a> endpoint, specifically for OIDC authentication.</p>
<p>Going with the theme of <a href="https://istio.io/latest/blog/2020/wasm-announce/" rel="noopener" target="_blank">WASM extensibility</a> in Istio, managing the OIDC flow may be a good candidate as a WASM extension so that Envoy no longer needs to call out to an external authorization implementation.</p>
<p>The post <a href="https://developer.cyberark.com/blog/istio-oidc-authentication/">Istio OIDC Authentication</a> appeared first on <a href="https://developer.cyberark.com">CyberArk Developer</a>.</p>
