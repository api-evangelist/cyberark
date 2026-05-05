---
title: "GKE Workload Identity Federation for Kubernetes Principals"
url: "https://developer.cyberark.com/blog/gke-workload-identity-federation-for-kubernetes-principals/"
date: "Sat, 10 Aug 2024 21:27:54 +0000"
author: "Paul Jones"
feed_url: "https://developer.cyberark.com/feed/"
---
<p>In this post, we’ll take a look at a new change to Workload Identity Federation on GKE that can reduce the amount of configuration and overhead required for IAM resources, and see it in action with cert-manager using Cloud DNS.</p>
<p>GKE Workload Identity enables a Kubernetes Service Account (KSA) to authenticate with Google Cloud APIs without needing to manage keys or credentials.</p>
<p>By using this feature, the KSA’s token is used to verify its identity and receive a Google Cloud Service Account (GSA) access token which can be used to authenticate and access Google Cloud APIs.</p>
<p>As recommended in the <a href="https://www.cisecurity.org/benchmark/kubernetes" rel="noopener" target="_blank">CIS GKE Benchmarks v1.5.0</a> (5.2.2), users should: ‘Prefer using dedicated Google Cloud Service Accounts and Workload Identity’.</p>
<p>This mitigates against generating, storing and rotating Google Service Account Keys, and adheres to the 5.2.1 recommendation of ‘Ensure GKE clusters are not running using the Compute Engine default service account’ which by default has overly permissive access.</p>
<p>Previously, Workload Identity on GKE used Service Account Impersonation to grant Kubernetes Service Accounts access to Google Cloud APIs by impersonating Google Cloud Service Accounts and inheriting their associated IAM permissions.</p>
<p>Now, Workload Identity Federation for GKE allows Kubernetes Service Accounts to be referenced directly using a <a href="https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity#kubernetes-resources-iam-policies" rel="noopener" target="_blank">principal identifier</a> in IAM Policies without using impersonation.</p>
<h3>Before:</h3>
<pre class="prettyprint">$ gcloud iam service-accounts create my-gsa

$ gcloud projects add-iam-policy-binding jetstack-paul --member “serviceAccount:my-gsa@jetstack-paul.iam.gserviceaccount.com” --role “roles/viewer”

$ gcloud iam service-accounts add-iam-policy-binding my-gsa@jetstack-paul.iam.gserviceaccount.com --role roles/iam.workloadIdentityUser --member “serviceAccount:jetstack-paul.svc.id.goog[default/my-ksa]”

$ kubectl create serviceaccount my-ksa

$ kubectl annotate serviceaccount my-ksa iam.gke.io/gcp-service-account=my-gsa@jetstack-paul.iam.gserviceaccount.com
</pre>
<h3>After:</h3>
<pre class="prettyprint">$ gcloud projects add-iam-policy-binding jetstack-paul --role=roles/viewer  --member=principal://iam.googleapis.com/projects/993897508389/locations/global/workloadIdentityPools/jetstack-paul.svc.id.goog/subject/ns/default/sa/my-ksa \
   --condition=None
</pre>
<p>The benefits of removing the need to impersonate Google Service Accounts are:</p>
<ul>
<li>Fewer IAM Policy bindings to manage
<ul>
<li>Previously, each KSA required an IAM Policy Binding to the GSA that granted the <code>workloadIdentityUser</code> IAM Role to impersonate</li>
</ul>
</li>
<li>No superfluous GSA
<ul>
<li>There is no longer the need for GSAs to impersonate as IAM policy bindings can name KSAs as principals</li>
</ul>
</li>
<li>No more annotating Kubernetes Service Accounts with the Google Service Account to impersonate
<ul>
<li>This can be especially painful when setting annotations for resources in templates, or consuming public templates that don’t support annotations in their inputs</li>
</ul>
</li>
</ul>
<p>Let’s look at a concrete example of where GKE Workload Identity Federation can come in useful for everyday applications.</p>
<p>To use GKE Workload Identity Federation, create a GKE cluster with a Workload Pool. This is created by default on Autopilot clusters.</p>
<pre class="prettyprint">$ gcloud container clusters create example \
   --location=europe-west2-a \
   --workload-pool=jetstack-paul.svc.id.goog
</pre>
<p>Next, let’s configure a Kubernetes Service Account to use with cert-manager and Google Cloud DNS.</p>
<p>Creating an IAM Policy Binding can grant a Kubernetes Service Account principal the desired Google Cloud IAM permissions. Previously, these permissions would have been granted to a Google Service Account which the Kubernetes Service Account would impersonate.</p>
<pre class="prettyprint">$ gcloud projects add-iam-policy-binding project/jetstack-paul \
   --role=roles/dns.admin \
--member=principal://iam.googleapis.com/projects/0123456789012/locations/global/workloadIdentityPools/jetstack-paul.svc.id.goog/subject/ns/cert-manager/sa/cert-manager \
   --condition=None
</pre>
<p>When using principal identifiers in IAM Policy Bindings, the Workload Identity Pool is shared across all clusters within a project. This means that if two clusters in the same Workload Identity Pool both contain a Kubernetes Service Account with the same name (in the same namespace), the principal identifier will match both Service Accounts and therefore they will each be granted the same permissions.</p>
<p>This <a href="https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity#identity_sameness" rel="noopener" target="_blank">identity sameness</a> stems from there only being a single Workload Identity Pool per Google Cloud Project (at the time of writing) which includes all GKE clusters and therefore all workload identities in these clusters. <a href="https://cloud.google.com/iam/docs/managing-conditional-role-bindings" rel="noopener" target="_blank">IAM Policy Conditions</a> can be used to restrict a policy to a specific principal, however, to isolate workload identities then separate Google Cloud Projects and Workload Identity Pools should be used to separate principals across clusters.</p>
<p>With the Kubernetes Service Account now having the necessary IAM Permissions, cert-manager can be deployed using the Kubernetes Service account which will be issued with a token with authorization to access the required Cloud DNS APIs.</p>
<pre class="prettyprint">helm upgrade --install cert-manager jetstack/cert-manager \
 --namespace cert-manager \
 --create-namespace \
 --set installCRDs=true \
 --set global.leaderElection.namespace=cert-manager \
 --set extraArgs={--issuer-ambient-credentials}
</pre>
<p>As normal, an <code>Issuer</code> can be deployed that uses Google CloudDNS to solve DNS01 ACME challenges for requested <code>Certificates</code>.</p>
<pre class="prettyprint">apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
 name: cloud-dns
spec:
 acme:
   email: paul.jones@jetstack.io
   server: https://acme-staging-v02.api.letsencrypt.org/directory
   privateKeySecretRef:
     name: issuer-account-key
   solvers:
   - dns01:
       cloudDNS:
         project: jetstack-paul
–--
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
 name: example-com
spec:
 secretName: example-com-tls
 issuerRef:
   name: cloud-dns
 dnsNames:
 - example.paul-gcp.jetstacker.net
</pre>
<p>All examples can be found in this repo: <a href="https://github.com/paulwilljones/gke-cert-manager-wi-fed" rel="noopener" target="_blank">https://github.com/paulwilljones/gke-cert-manager-wi-fed</a></p>
<p>As we have seen, workloads in GKE can now access Google Cloud APIs by having a single IAM policy that directly references the Kubernetes Service Account as a principal.</p>
<p>To create this IAM allow policy, we used <code>gcloud</code> but it can easily be done with any IaC tool that manages Google Cloud resources. See these examples for how to administer Workload Identity Federation on GKE:</p>
<h2>Terraform</h2>
<pre class="prettyprint">resource "google_project_iam_custom_role" "cert_manager" {
 project     = data.google_project.project.project_id
 role_id     = "certmanagertf"
 title       = "Cert Manager" permissions = ["dns.resourceRecordSets.create", "dns.resourceRecordSets.list", "dns.resourceRecordSets.get", "dns.resourceRecordSets.update", "dns.resourceRecordSets.delete", "dns.changes.get", "dns.changes.create", "dns.changes.list", "dns.managedZones.list"]
}

resource "google_project_iam_member" "cert_manager" {
 project = data.google_project.project.project_id
 role    = google_project_iam_custom_role.cert_manager.name
 member  = "principal://iam.googleapis.com/projects/${data.google_project.project.number}/locations/global/workloadIdentityPools/${data.google_project.project.project_id}.svc.id.goog/subject/ns/cert-manager/sa/cert-manager"
}
</pre>
<p><a href="https://github.com/paulwilljones/gke-cert-manager-wi-fed/tree/develop/terraform" rel="noopener" target="_blank">https://github.com/paulwilljones/gke-cert-manager-wi-fed/tree/develop/terraform</a></p>
<h2>Config Connector</h2>
<pre class="prettyprint">apiVersion: iam.cnrm.cloud.google.com/v1beta1
kind: IAMCustomRole
metadata:
 annotations:
   cnrm.cloud.google.com/project-id: jetstack-paul
 name: certmanagerkcc		#intentional naming to comply with IAM naming
spec:
 title: Cert Manager
 resourceID: certmanagerkcc
 permissions:
   - dns.resourceRecordSets.create
   - dns.resourceRecordSets.list
   - dns.resourceRecordSets.get
   - dns.resourceRecordSets.update
   - dns.resourceRecordSets.delete
   - dns.changes.get
   - dns.changes.create
   - dns.changes.list
   - dns.managedZones.list
 stage: GA
---
apiVersion: iam.cnrm.cloud.google.com/v1beta1
kind: IAMPolicyMember
metadata:
 name: cert-manager-kcc
 namespace: cert-manager
 annotations:
   cnrm.cloud.google.com/project-id: jetstack-paul
spec:
 member: principal://iam.googleapis.com/projects/993897508389/locations/global/workloadIdentityPools/jetstack-paul.svc.id.goog/subject/ns/cert-manger/sa/cert-manager
 role: projects/jetstack-paul/roles/certmanagerkcc
 resourceRef:
   kind: Project
   external: projects/jetstack-paul
</pre>
<p><a href="https://github.com/paulwilljones/gke-cert-manager-wi-fed/tree/develop/kcc" rel="noopener" target="_blank">https://github.com/paulwilljones/gke-cert-manager-wi-fed/tree/develop/kcc </a></p>
<h2>Crossplane</h2>
<pre class="prettyprint">apiVersion: cloudplatform.gcp.upbound.io/v1beta1
kind: ProjectIAMCustomRole
metadata:
 name: certmanagerxp
spec:
 forProvider:
   permissions:
     - dns.resourceRecordSets.create
     - dns.resourceRecordSets.list
     - dns.resourceRecordSets.get
     - dns.resourceRecordSets.update
     - dns.resourceRecordSets.delete
     - dns.changes.get
     - dns.changes.create
     - dns.changes.list
     - dns.managedZones.list
   title: Cert Manager (Crossplane)
–--
apiVersion: cloudplatform.gcp.upbound.io/v1beta1
kind: ProjectIAMMember
metadata:
 name: cert-manager-xp
 namespace: cert-manager
spec:
 forProvider:
   project: jetstack-paul
   member: principal://iam.googleapis.com/projects/993897508389/locations/global/workloadIdentityPools/jetstack-paul.svc.id.goog/subject/ns/cert-manager/sa/cert-manager
   role: projects/jetstack-paul/roles/certmanagerxp
</pre>
<p><a href="https://github.com/paulwilljones/gke-cert-manager-wi-fed/tree/develop/crossplane" rel="noopener" target="_blank">https://github.com/paulwilljones/gke-cert-manager-wi-fed/tree/develop/crossplane </a></p>
<h2>Pulumi</h2>
<pre class="prettyprint">cert_manager_role = gcp.projects.IAMCustomRole(
   "cert-manager",
   project=my_project.projects[0].project_id,
   role_id="certmanagerpulumi",
   title="Cert Manager (Pulumi)",
   permissions=[
       "dns.resourceRecordSets.create",
       "dns.resourceRecordSets.list",
       "dns.resourceRecordSets.get",
       "dns.resourceRecordSets.update",
       "dns.resourceRecordSets.delete",
       "dns.changes.get",
       "dns.changes.create",
       "dns.changes.list",
       "dns.managedZones.list",
   ],
)

project = gcp.projects.IAMMember(
   "project",
   project=my_project.projects[0].project_id,
   role=cert_manager_role.name,
   member=f"principal://iam.googleapis.com/projects/{my_project.projects[0].number}/locations/global/workloadIdentityPools/{my_project.projects[0].project_id}.svc.id.goog/subject/ns/cert-manager/sa/cert-manager",
)

manager = CertManager(
   "cert-manager",
   install_crds=True,
   helm_options=ReleaseArgs(namespace="cert-manager", create_namespace=True),
   extra_args=["--issuer-ambient-credentials"],
   global_=CertManagerGlobalArgs(
       leader_election=CertManagerGlobalLeaderElectionArgs(namespace="cert-manager")
   )
)
</pre>
<p><a href="https://github.com/paulwilljones/gke-cert-manager-wi-fed/tree/develop/pulumi" rel="noopener" target="_blank">https://github.com/paulwilljones/gke-cert-manager-wi-fed/tree/develop/pulumi</a></p>
<p>For more information on GKE Workload Identity Federation, see the documentation <a href="https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity" rel="noopener" target="_blank">here</a>.</p>
<p>The post <a href="https://developer.cyberark.com/blog/gke-workload-identity-federation-for-kubernetes-principals/">GKE Workload Identity Federation for Kubernetes Principals</a> appeared first on <a href="https://developer.cyberark.com">CyberArk Developer</a>.</p>
