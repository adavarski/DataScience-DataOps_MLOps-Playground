SAAS: IAM:Keycloak + JupyterHUB/JupyterLAB + Spark clusters(per tenant/user) with k8s as Cluster Manager 


The remainder of this

Creates a user-provisioned Spark clusters connected to other SaaS servcies (Kafka, ELK, etc) directly within the cluster as shownas bellow:

<img>

JupyterLab, brings a robust and extendable suite of data science capabilities along with a command-line terminal. Operating JupyterLab within the cluster creates an incredibly
efficient environment for both traditional data science, analytics, and experimentation, along with opportunities for development and operations through closer interaction with the Kubernetes API.
The following sections demonstrate the setup of a Kubernetes Namespace, sample RBAC, and ServiceAccount permissions allowing JupyterLab access to Kubernetes resources. JupyterHub is configured to
provision JupyterLab environments(Spark driver with jupyter inside + spark workers/executors we can run via SparkSession), authenticating against Keycloak. So SaaS is based on JupyterHub/Lab.

JupyterHub (per user/tenant):
```
singleuser:
  image:
    name: davarski/spark301-k8s-minio-jupyter
    tag: 2.0.0
  defaultUrl: "/lab"
```


Example Notebook(Ref: Demo6: using spark driver pod and running 2 spark workers and Spark using k8s as Cluster Manager):

```
import pyspark

conf = pyspark.SparkConf()

# Kubernetes is a Spark master in our setup. 
# It creates pods with Spark workers, orchestrates those 
# workers and returns final results to the Spark driver 
# (“k8s://https://” is NOT a typo, this is how Spark knows the “provider” type). 
conf.setMaster("k8s://https://kubernetes.default:443") 

# Worker pods are created from the base Spark docker image.
# If you use another image, specify its name instead.
conf.set(
    "spark.kubernetes.container.image", 
    "davarski/spark301-k8s-minio-base:1.0.0") 

# Authentication certificate and token (required to create worker pods):
conf.set(
    "spark.kubernetes.authenticate.caCertFile", 
    "/var/run/secrets/kubernetes.io/serviceaccount/ca.crt")
conf.set(
    "spark.kubernetes.authenticate.oauthTokenFile", 
    "/var/run/secrets/kubernetes.io/serviceaccount/token")

# Service account which should be used for the driver
conf.set(
    "spark.kubernetes.authenticate.driver.serviceAccountName", 
    "spark-driver") 

# 2 pods/workers will be created. Can be expanded for larger workloads.
conf.set("spark.executor.instances", "2") 

# The DNS alias for the Spark driver. Required by executors to report status.
conf.set(
    "spark.driver.host", "spark-jupyter") 

# Port which the Spark shell should bind to and to which executors will report progress
conf.set("spark.driver.port", "29413") 

# Initialize spark context, create executors
sc = pyspark.SparkContext(conf=conf)

sc._conf.getAll()

!kubectl get pod

# Create a distributed data set to test to the session
t = sc.parallelize(range(10))

# Calculate the approximate sum of values in the dataset
r = t.sumApprox(3)
print('Approximate sum: %s' % r)
```


Keycloak
Keycloak is a free, open source identity and access management application sponsored by Red Hat. Keycloak provides the ability to create
and manage user accounts, or connect to an existing LDAP or Active
Directory. Third-party applications may authenticate users through
OpenID Connect, OAuth 2.0, and SAML 2.0.

Keycloak provides a turnkey solution for identity management and
third-party authentication well suited to the requirements of the SaaS  data
platform. The following section implements a
single-node Keycloak instance, used later for JupyterHub to
authenticate users before provisioning JupyterLab instances for them.

kubectl apply -f 



Realm, Client, and User
Keycloak provides identity management and authentication to multiple
tenants through the configuration of realms. JupyterHub is configured
later to authenticate users using Oauth2, belonging
to the realm "saas". A Keycloak client associated with a realm grants
access to applications such as JupyterHub looking to authenticate users.
This section sets up a realm, client, and user used to provision JupyterLab
servers later.
Using a web browser, visit the new Ingress https://auth.data.davar.com/auth/ as set up via yaml manifests. Log in to Keycloak
using the "sysop" credential defined in /9000-keycloak/15-secret.yml . After logging in, master
is the default realm shown in the upper left of the user interface and
depicted bellow:

<img>

Open the "Add realm" menu by clicking the drop-
down to the right of the realm title and create the new realm "saas".


Next, navigate to Clients in the left-hand navigation of the new Datalab
realm. Click Create and fill in the “Add Client” form to add a new client
named "saas" shown bellow:


<img>

After adding the new "saas" client, click the Credentials tab to retrieve
the generated secret, as shown:

JupyterHub is later configured
to use the client ID "saas" and the generated secret for permission to
authenticate users against the Keycloak "saas" realm.


Configure the new datalab client (under the Setting tab) by switching
Authorization Enabled to on. Provide Valid Redirect URIs, in this case,
https://saas.data.davar.com/hub/oauth_callback later defined in
the “JupyterHub” section. Review:


<img>

Finally, create one or more users in the "saas" realm by choosing Users
under the Manage section of the left-hand menu. After adding a user, assign
a password under the Credentials tab. Use a strong password; any users
assigned to this realm are later given access to a JupyterLab environment with
permissions to read and write data and execute code from within the cluster.

SaaS Namespace

Tip Use Kubernetes ResourceQuota objects for fine-grain
restriction to resources for a given Namespace, including the total
number of Pods and PersistentVolumeClaims allowed, CPU, memory,
and storage class restrictions.

This section sets up the Namespace "saas" along with a
ServiceAccount and RBAC permissions used by JupyterLab and JupyterHub.

The default service account assigned to Pods in this cluster does not
have access to the Kubernetes API. The following creates a service account
assigned to JupyterLab Pods provisioned by JupyterHub.


JupyterHub

JupyterHub “spawns, manages, and proxies multiple instances of the
single-user Jupyter notebook server.” 21 This section installs JupyterHub
into the development cluster and configures it to authenticate users with
Keycloak and spawn JupyterLab (notebook with spark driver preinstalled) servers into the "saas"
Namespace. Additionally, the "saas" Role defined in the previous
section grants JupyterHub limited access to the Kubernetes API.

JupyterHub (along with Keycloak) for use as a multi-tenant(SaaS users/tenants) provisioner of JupyterLab(Spark clusters) environments managing one
or more Jupyter Notebooks.

values.yml:

-Within the proxy section, set the secretToken to
a 32-character string of random hex values. The
official documentation recommends using the
following command:


``
$ openssl rand -hex 32
```
-Within the singleuser section, note the container
image davarski/spark301-k8s-minio-jupyter(JupyterLab image). A variety of Jupyter
Notebook images may be used here; however, in
this case, the specified image represents a highly
customized version developed (with Spark and needed libs)



-Within the hub section, extraConfig is used to
inject additional configuration not directly exposed
by the Helm chart. In this case, the configuration
instructs KubeSpawner to spawn JupyterLab Pods
in the "saas" Namespace and configured to use
the "saas" ServiceAccount defined earlier.

Additionally, within the hub section, extraEnv is
used to populate environment variables required
by the GenericOAuthenticator defined later in
values.yml. Note the Keycloak realm "saas",
created earlier and defined in the
environment variables OAUTH2_AUTHORIZE_
URL and OAUTH2_TOKEN_URL.

-Within the auth section, the
GenericOAuthenticator is configured with a
client_id and client_secret set up earlier in the
Keycloak "saas" realm. Note the "saas" realm is
part of the token_url and userdata_url paths.

JupyterHub is configured to run in the cluster and in the Namespace
data and configured to spawn single-user JupyterLab servers (Pods) in the
"saas" namespace. After applying configuration JupyterHub may take
several minutes to boot as it must preload large JupyterLab images. Once
JupyterHub has fully booted, launch a new JupyterLab instance (with a
user created in Keycloak under the datalab realm) by visiting https://
saas.data.davar.com.




JupyterLab
JupyterLab is “the next-generation web-based user interface for Project Jupyter,” a feature-rich data science
environment. Project Jupyter began in 2014 and has seen massive
adoption; 

Jupyter Notebooks are a browser-based (or web-based) IDE.

Kubernetes is a natural fit for provisioning and serving
JupyterLab environments through JupyterHub, as demonstrated in the
previous section.
Streamlining the development of machine learning and statistical models
has driven the success of Project Jupyter. Many data science activities, such as
machine learning, require static, immutable data sets to achieve reproducible
results from experimentation. However, operating Jupyter environments with
static data alongside real-time event streams, indexes, and the full power of
Kubernetes distributed computing is an opportunity to offer a variety of data
science functionality directly in the center of a data platform.
The following sections demonstrate brief examples of working directly
with the data and control plane from within the cluster, connecting
JupyterLab notebooks with MinIO, etc.