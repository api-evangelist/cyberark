---
title: "Ingress-Nginx is Retiring: Our Practical Journey to Gateway API"
url: "https://developer.cyberark.com/blog/ingress-nginx-is-retiring-our-practical-journey-to-gateway-api/"
date: "Fri, 30 Jan 2026 21:06:07 +0000"
author: "Maria Reynoso"
feed_url: "https://developer.cyberark.com/feed/"
---
<p>Ingress has been the standard way to expose Kubernetes apps since 2015. It went GA in 2020 and gained huge adoption. Ingress-nginx alone has 19k+ stars and more than a thousand contributors. It became the default controller for most teams, including ours.</p>
<p>But Kubernetes traffic management has grown, and so have our requirements for routing, authentication, and security. With the <a href="https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/" rel="noopener" target="_blank">retirement of ingress-nginx</a>, our team used this moment as an opportunity to reevaluate our approach and move to a more modern and flexible model.</p>
<p>This post walks through our real migration from ingress-nginx to Gateway API using Envoy Gateway, including how we evaluated our options, prepared for the migration, validated our new setup, and ensured zero regressions.</p>
<p>Our goal is to give other teams a realistic look at what this kind of transition actually involves in a real, complex production environment and the practical lessons we learned from this journey.</p>
<h2>Introducing our platform &amp; requirements</h2>
<p>Our platform hosts a collection of internal and user-facing services that are exposed through Kubernetes. Our setup relied on ingress-nginx as our primary ingress controller, with oauth2-proxy providing authentication/authorisation along with a variety of nginx annotations providing custom routing behaviours.</p>
<p>Before planning the migration, we spent time evaluating Gateway API and its wide range of implementations and defined a clear set of goals we wanted to achieve:</p>
<ul>
<li>Reduce reliance on annotations and custom controller behaviour</li>
<li>Replace oauth2-proxy with native, platform-level authentication</li>
<li>Prepare for future security and routing requirements</li>
<li>Adopt a Kubernetes standard that will be supported long-term</li>
</ul>
<h2>Gateway API evaluation</h2>
<p>Before selecting a specific Gateway implementation, we evaluated whether Gateway API itself was the right direction.</p>
<p><a href="https://gateway-api.sigs.k8s.io/" rel="noopener" target="_blank">Gateway API</a> is the next-generation networking API for Kubernetes. It is designed as the long-term successor to the Ingress API and provides a more expressive, extensible, and role-oriented model for controlling traffic into and within Kubernetes clusters.</p>
<p>Gateway API introduces well-defined, typed resources for traffic control:</p>
<ul>
<li><strong>GatewayClass</strong>: Defines how the infrastructure behaves (similar to StorageClass)</li>
<li><strong>Gateway</strong>: The actual load balancer / listener</li>
<li><strong>Routes</strong>: Describe how traffic from a Gateway should be mapped to Kubernetes services</li>
</ul>
<pre class="prettyprint">HTTPRoute, TCPRoute, GRPCRoute</pre>
<p>These resources make Gateway API far more maintainable than annotation-driven ingress. Annotation-based configuration is powerful, but also easy to get wrong, e.g., a small or singular/plural mismatch can cause confusion, and debugging those issues isn’t fun. Gateway API’s structured, typed resources can reduce this.</p>
<p>To help evaluate it, we created a small test cluster and deployed the Gateway API CRDs:</p>
<pre class="prettyprint">kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/latest/download/standard-install.yaml
</pre>
<p>This allowed us to test basic listener behaviour, routing, hostname precedence, and the API’s usability before selecting an implementation. We also validated how cert-manager and external-dns work with Gateway API, since both are essential to our setup.</p>
<h2>Evaluating implementations</h2>
<p>There are several <em>production-grade</em> Gateway API controllers available. We decided to evaluate only two options to understand operational complexity, feature gaps, and how much they aligned with our authentication and routing requirements.</p>
<h3>Kgateway (formally Gloo Gateway)</h3>
<p>Kgateway implements the Kubernetes Gateway API by turning Gateway API resources into Envoy or AgentGateway config, depending on whether you&#8217;re handling microservices or AI workloads. It essentially provides a unified control plane for both.</p>
<div>
<table align="center" border="1" cellpadding="10" cellspacing="0" width="100%">
<thead>
<tr>
<th align="center">Benefits</th>
<th align="center">Limitations</th>
</tr>
</thead>
<tbody>
<tr>
<td>Very powerful, evolved from Gloo Gateway with extensive enterprise adoption</td>
<td>Additional features (like OIDC) require other components</td>
</tr>
<tr>
<td>Extensible, with custom CRDs and integrations that let you define advanced policies</td>
<td>More complex to operate</td>
</tr>
<tr>
<td>Uses Envoy for microservices and AgentGateway for AI</td>
<td>More features than we needed</td>
</tr>
</tbody>
</table>
</div>
<p>&nbsp;</p>
<p><strong>Outcome:</strong> Very powerful and flexible, especially for teams that want fine-grained control over Envoy’s data plane. However, it would still require extra components for authentication flows, which didn&#8217;t align with our requirements.</p>
<h3>Envoy Gateway (chosen solution)</h3>
<p>Envoy Gateway is an open-source control plane that manages Envoy Proxy and configures it directly from your Gateway API resources.</p>
<p>What made it our chosen solution:</p>
<div>
<table align="center" border="1" cellpadding="10" cellspacing="0" width="100%">
<thead>
<tr>
<th align="center">Benefits</th>
<th align="center">Limitations</th>
</tr>
</thead>
<tbody>
<tr>
<td>Provides native OIDC authentication and supports JWT validation and claim extraction</td>
<td>Some features are still maturing (filters, transformations, policies)</td>
</tr>
<tr>
<td>Clean integration with Gateway API</td>
<td>Use cases like AI traffic management require additional tooling or custom development</td>
</tr>
<tr>
<td>Simple, focused API surface</td>
<td>The community is still growing</td>
</tr>
<tr>
<td>Provides a 1:1 or 1: many deployment strategy, giving each Gateway its own Envoy deployment for better isolation, scaling, and security.</td>
<td></td>
</tr>
</tbody>
</table>
</div>
<p>&nbsp;</p>
<p>We installed it in our test cluster with:</p>
<pre class="prettyprint">kubectl apply -f https://github.com/envoyproxy/gateway/releases/latest/download/install.yaml</pre>
<p>This provided us with a fully functional environment to experiment with routing, authentication, observability, and operational aspects (PDB, HPA, scaling, etc.).</p>
<p><strong>Outcome:</strong> While some features are still evolving, we believe Envoy Gateway is the best fit for our authentication and routing requirements.</p>
<h2>Preparation &amp; testing strategy</h2>
<p>In our test environment, we validated our setup and ensured we understood how Gateway API and Envoy Gateway behave in practice.</p>
<h3>Gateway resources</h3>
<p>Previously, we had installed Gateway API and Envoy Gateway. Once installed, we created a Gateway to test listener behaviour:</p>
<pre class="prettyprint">apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: test-gateway
  namespace: networking
spec:
  gatewayClassName: envoy
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      hostname: "*.example.test"
      tls:
        mode: Terminate
        certificateRefs:
          - name: cert

</pre>
<p>And an HTTPRoute:</p>
<pre class="prettyprint">kind: HTTPRoute
spec:
  parentRefs:
    - name: test-gateway
  hostnames:
    - "test".example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: "/abc123def/portal"
      backendRefs:
        - name: portal-service
          port: 8080
</pre>
<p>Then we could test:</p>
<ul>
<li>Listener and hostname matching</li>
<li>HTTP to HTTPS redirection</li>
<li>TLS termination</li>
<li>Routing behaviour using HTTPRoutes</li>
</ul>
<h3>Authentication testing</h3>
<p>One of our biggest requirements was replacing oauth2-proxy, and as we mentioned previously, Envoy Gateway offers native OIDC support, which can be implemented by a <a href="https://gateway.envoyproxy.io/docs/tasks/security/oidc/#create-a-securitypolicy" rel="noopener" target="_blank">SecurityPolicy</a>. We expected this to be fairly straightforward, and it was at the beginning until we faced a limitation.</p>
<p>We started on creating the SecurityPolicy:</p>
<pre class="prettyprint">apiVersion: gateway.envoyproxy.io/v1alpha1
kind: SecurityPolicy
metadata:
  name: demo-auth
spec:
  targetRefs:
    - group: gateway.networking.k8s.io
      kind: Gateway
      name: primary
  oidc:
    provider:
      issuer: "https://auth.example.com/"
    clientIDRef:
      Name: auth-config
    clientSecret:
      name: auth-config
    redirectURL: "https://echo.example.com/callback"
    scopes: ["openid", "email", "profile"]
    cookieDomain: "example.com"
  jwt:
    providers:
      - name: oidc-id-token
        issuer: "https://auth.example.com/"
        remoteJWKS:
          uri: "https://auth.example.com/.well-known/jwks.json"
        extractFrom:
          cookies:
            - id_token
        claimToHeaders:
          - claim: email
            header: x-authenticated-email

</pre>
<p>Then we discovered during this testing:</p>
<ul>
<li>We applied a <strong>SecurityPolicy</strong> to a <strong>Gateway</strong> that defined OIDC authentication and then tried to apply another SecurityPolicy on an <strong>HTTPRoute</strong> that defined additional authorization rules. This didn’t work as we expected since according to the <a href="https://gateway.envoyproxy.io/docs/concepts/gateway_api_extensions/security-policy/#precedence" rel="noopener" target="_blank">docs</a>, if an HTTPRoute has its own SecurityPolicy, it takes precedence over the Gateway’s policy for that route, but it doesn’t combine with the Gateway-level policy and you must define the complete policy for that route, if you want different behaviour. <strong>This means SecurityPolicies don’t merge</strong>; the route-level policy completely overrides the gateway-level policy for that route.</li>
<li>All referenced secrets (clientIDRef, clientSecret) must be in the same namespace as the SecurityPolicy.</li>
</ul>
<p>These surprising challenges became our major test focus and easily would have caused production issues if we discovered them late.</p>
<h2>JWT validation and claim injection</h2>
<p>We validated JWT behaviour using:</p>
<pre class="prettyprint">extractFrom:
  cookies:
    - id_token
claimToHeaders:
  - claim: email
    header: x-authenticated-email

</pre>
<p>We intentionally tested with misspelled claim names or invalid signature scenarios.</p>
<h3>Cert-manager</h3>
<ul>
<li>Upgraded to 1.19.x for Gateway API support.</li>
<li>Migrated certificate issuance to reference Gateway API HTTPRoutes in addition to ingress objects.</li>
<li>Ensured our Gateway control plane could consume cert-manager issued certificates correctly.</li>
<li>The cert-manager team’s <a href="https://cert-manager.io/announcements/2025/11/26/ingress-nginx-eol-and-gateway-api/" rel="noopener" target="_blank">announcement</a> about ingress-nginx EOL reinforced the importance of making this change early.</li>
</ul>
<h3>External-dns</h3>
<ul>
<li>Updated to enable Gateway API support so it can watch HTTPRoutes</li>
</ul>
<pre class="prettyprint">sources:
  - gateway-httproute
&lt;/pre</pre>
<h3>Side-by-side operation with ingress-nginx</h3>
<p>Finally, we ran nginx and Envoy side-by-side for several days making sure routing and authentication worked.</p>
<h2>Introducing new architecture</h2>
<p><img alt="new architecture" class="aligncenter size-full wp-image-8331" height="499" src="https://developer.cyberark.com/wp-content/uploads/2026/01/new-architecture.png" width="1204" /></p>
<h3>Gateway</h3>
<p>A single Gateway resource defines:</p>
<ul>
<li>One or more network listeners (e.g., HTTP, HTTPS)</li>
<li>Wildcard or specific hostnames</li>
<li>TLS configuration</li>
<li>Allowed namespaces for routes</li>
</ul>
<h3>Routes</h3>
<p>Each legacy ingress became an HTTPRoute, except TLS, which is now handled at the Gateway level.</p>
<p>Routes define:</p>
<ul>
<li>The hostnames they apply to. These must match one of the Gateway’s listener hostnames; otherwise, the Route will not be accepted.</li>
<li>How requests are matched</li>
<li>Optional filters (redirects, rewrites, authentication hooks)</li>
</ul>
<h3>Authentication</h3>
<p>One of the most significant improvements was replacing oauth2-proxy with native OIDC support built into Envoy Gateway.</p>
<p>Envoy Gateway now handles:</p>
<ul>
<li>Redirecting users to the identity provider</li>
<li>Handling callback exchanges</li>
<li>Validating ID tokens</li>
<li>Managing user sessions via secure cookies</li>
<li>Extracting JWT claims and attaching them to upstream requests</li>
</ul>
<p>As we saw in the preparation and testing section, this is all done through a SecurityPolicy.</p>
<p>This reduced the number of moving parts in the authentication flow, significantly making this easier to troubleshoot and reducing the cognitive overload of multiple components within the platform being maintained, scaled and secured.</p>
<h2>Limitations &amp; security considerations</h2>
<p>We identified several considerations when adopting Gateway API and Envoy Gateway:</p>
<h3>Technical</h3>
<ul>
<li>Some features (filters, transformations) are still maturing</li>
<li>Requires understanding listener precedence and route attachment rules</li>
<li>Policy resources (authentication, JWT, rate limiting) may change as the API matures</li>
<li>Knowledge of Envoy and its configuration language (XDS) to understand how/what the config will look like.</li>
</ul>
<h3>Security</h3>
<ul>
<li>Session cookies must be scoped correctly</li>
<li>JWT validation must be configured carefully to avoid leaking sensitive information.</li>
</ul>
<h3>TLS &amp; certificate management</h3>
<ul>
<li>cert-manager needed to be upgraded to v1.19 to fully support Gateway API</li>
<li>Certificates are now attached at the Gateway listener level instead of per ingress, which changes how they are requested and validated</li>
</ul>
<h3>Migration complexity</h3>
<ul>
<li>Dual-stack operation required:
<ul>
<li>Running both nginx and Envoy Gateway simultaneously</li>
<li>Maintaining two sets of routing configuration</li>
<li>Careful DNS management to avoid conflicts</li>
</ul>
</li>
<li>OIDC and JWT are separate concepts and must be configured correctly</li>
</ul>
<p>We should take the time to emphasise here that this wasn’t just a simple YAML swap. We modified around 60 files, from Helm templates to certificate and DNS configurations.</p>
<h2>Ensuring no regressions</h2>
<p>To ensure a smooth migration, we validated:</p>
<ul>
<li>All routes resolve as expected</li>
<li>Authentication flows using Envoy Gateway’s native OIDC worked consistently</li>
<li>Claims are mapped correctly into headers</li>
<li>TLS certificates were issued and rotated correctly through cert-manager</li>
<li>Applications do not require changes</li>
<li>A complete rollback plan (DNS back to nginx)</li>
</ul>
<p>We had a gradual migration process:</p>
<ul>
<li>Dual stack deployment: Both systems running simultaneously</li>
<li>DNS cutover</li>
<li>Monitor and validate</li>
<li>Decommission nginx</li>
</ul>
<h2>Final Outcome</h2>
<p>In the end, we successfully retired the ingress-nginx controller and switched our traffic routing over to the Kubernetes Gateway API, all with zero downtime and no impact on our customers. From start to finish, this took around 2 weeks of engineering effort.<br />
The benefits and improvements we noticed were:</p>
<ul>
<li><strong><em>Cleaner configuration</em></strong>: We could replace loads of annotations with the resources provided by Gateway API. Using Gateway and HTTPRoutes has made our routing configuration easier to understand and maintain.</li>
<li><strong><em>Aligned with Kubernetes standards</em></strong>: By adopting Gateway API, our platform now uses Kubernetes native APIs, which ensures long-term support and flexibility.</li>
<li><strong><em>Simpler operations</em></strong>: Replacing oauth2-proxy with Envoy native OIDC, as well as getting rid of those bunch of annotations, made things easier to operate and debug.</li>
<li><strong><em>Ready for the future:</em></strong> Gateway API features are maturing and becoming the right choice for advanced traffic management in Kubernetes. By moving now, we can take advantage of upcoming features or capabilities with a minor rework or fewer challenges.</li>
</ul>
<h2>Takeaways for other platform teams</h2>
<p>Finally, here are our main takeaways we can share with other platform teams:</p>
<table align="center" border="1" cellpadding="10" cellspacing="0" width="100%">
<thead>
<tr>
<th align="center"></th>
<th align="center">Do</th>
<th align="center">Don’t</th>
</tr>
</thead>
<tbody>
<tr>
<td align="center"><strong>Start with evaluation, not implementation</strong></td>
<td align="center">
<ul style="text-align: left; display: inline-block;">
<li>Read Gateway API documentation thoroughly</li>
<li>Compare multiple implementations (Envoy Gateway, Traefik, Kgateway, etc.)</li>
<li>Evaluate options against your specific requirements</li>
<li>Put together a simple checklist to compare different options (e.g., https://github.com/howardjohn/gateway-api-bench)</li>
</ul>
</td>
<td align="center">
<ul style="text-align: left; display: inline-block;">
<li>Jump to the most popular solution</li>
<li>Expect your existing setup to translate to Gateway API straight away</li>
</ul>
</td>
</tr>
<tr>
<td align="center"><strong>Build a complete prototype first</strong></td>
<td align="center">
<ul style="text-align: left; display: inline-block;">
<li>Deploy a full stack in a non-production environment</li>
<li>Test all critical paths (auth, TLS, routing, error handling)</li>
<li>Document findings and gaps</li>
</ul>
</td>
<td align="center">
<ul style="text-align: left; display: inline-block;">
<li>Skip failure scenarios; they might show up in production</li>
<li>Rely on vendor claims without testing them yourself</li>
</ul>
</td>
</tr>
<tr>
<td align="center"><strong>Plan for dual-stack operation</strong></td>
<td align="center">
<ul style="text-align: left; display: inline-block;">
<li>Support both old and new architectures simultaneously</li>
<li>Use feature flags for gradual rollout</li>
<li>Keep the rollback path ready and tested</li>
<li>Monitor both systems during migration</li>
</ul>
</td>
<td align="center">
<ul style="text-align: left; display: inline-block;">
<li>Don’t try to migrate everything in a single step for critical systems</li>
<li>Remove the old system until the new system is proven</li>
<li>Assume migration will go perfectly</li>
</ul>
</td>
</tr>
<tr>
<td align="center"><strong>Document everything</strong></td>
<td align="center">
<ul style="text-align: left; display: inline-block;">
<li>Write runbooks before production deployment</li>
<li>Create troubleshooting guides</li>
<li>Document rollback procedures</li>
<li>Share knowledge with the entire team</li>
</ul>
</td>
<td align="center">
<ul style="text-align: left; display: inline-block;">
<li>Keep migration knowledge in one person&#8217;s head</li>
<li>Skip documentation “because we’re in a hurry”</li>
<li>Forget to update the docs after lessons learned</li>
</ul>
</td>
</tr>
</tbody>
</table>
<p></p>
<p>The post <a href="https://developer.cyberark.com/blog/ingress-nginx-is-retiring-our-practical-journey-to-gateway-api/">Ingress-Nginx is Retiring: Our Practical Journey to Gateway API</a> appeared first on <a href="https://developer.cyberark.com">CyberArk Developer</a>.</p>
