## Data-Driven Analytics/Data Science(ML/DeepML) k8s-based SaaS Platform for Data Analyst/Data Engineer/Data Scientist/DataOps/MLOps playground (MVP/POC/environmints)

### Used stacks and products:

VPN (WireGuard, k8s:Kilo); Monitoring stacks (Prometheus-based, TIG:Telegraf+InfluxDB+Grafana, Sensu, Zabbix); Indexing and Analytics/Debugging and Log management stacks (ELK/EFK); Pipeline: Messaging/Kafka stack (Kafka cluster, Zookeper cluster, Kafka Replicator, Kafka Connect, Schema Registry); Routing and Transformation (Serverless:OpenFaaS; ETL:Apache NiFi); Data Lake/Big Data (MinIO s3-compatable Object Storage); DWHs (HIVE SQL-Engine with MinIO:s3, Presto SQL query engine with Hive/Cassandra/MySql/Postgresql/etc. as data sources); Apache Spark for large-scale distributed Big Data processing and data analytics with Delta Lake (Lakehouses) and MinIO(S3); Machine Learning/Deep Learning/AutoML (TensorFlow, Pandas/Koalas, Keras, Scikit-learn, Spark MLlib, etc.; k8s:Model Development with AutoML: Kubeflow(MLflow, etc.) and k8s:AI Model Deployment (Seldon Core); Spark ML with S3(MinIO) as Data Source); GitLab/Jenkins/Jenkins X/Argo CD In-Platform CI/CD (GitOps); Identity and Access Management (IAM:Keycloak); JupyterHub/JupyterLab for data science; HashiCorp Vault cluster; k8s Persistent Volumes (Rook Ceph, Gluster); etc.

Summary: k8s-based Analytics/ML/DeepML SaaS using Big Data/Data Lake: MinIO (s3-compatable object storage) with Hive(s3) SQL-Engine/DWHs (instead of Snowflake as big data platform for example), Apache Spark(Hive for metadata)/Delta Lake(lakehouses)/Jupyter/etc. (instead of Databricks for example) + Kafka stack + ELK/EFK + Serverless(OpenFaaS) + ETL(Apache NiFi) + ML/DeepML/AutoML + GitOps. 

<img src="https://github.com/adavarski/DataScience-DataOps_MLOps-Playground/blob/main/k8s/003-pictures/saas-v2.0.0.png" width="900">

Note: For building Analytics/ML SaaS platform we can also use cloud-native SaaSs as reference (or build SaaS based on cloud-native SaaSs): Snowflake(SaaS/DWaaS) as big data solution on a single data platform (DWH, S3, etc.) + Databricks(cloud-based big data processing SaaS: unified analytics platform for ML & Data Science) + AWS S3/MKS/SQS/ELK/Lambda/etc.


### PaaS/SaaS MVP/POC/Development environments used:

- k8s (k8s-local: k3s, minikube, kubespray) `Note: Default development environment: k3s` 
- k8s AWS (KOPS for k8s clusters deploy on AWS, and k8s Operators/Helm Charts/YAML manifests for creating k8s deployments (PasS&SaaS services).  

### PaaS/SaaS objectives:

Platform as a Service (PaaS) will be data-driven and data-science platform allowing end user to develop, run, and manage applications without the complexity of building and maintaining the infrastructure.

Software as a Service (SaaS) will be "on-demand software", accessed/used by end users using a web browser.

### Productionalize SaaS: k8s (KOPS, Rancher, managed k8s)

k8s-based data-driven Analytics/ML/DeepML SaaS platform provisioning/deploy : KOPS for k8s provisioning on AWS(default); Rancher (custom provisioning): k3s(for branches development)/RKE(for on-prem k8s) and Rancher(for the public clouds) with AWS EKS/Azure AKS/Google GKE; managed k8s on AWS/Azure/GCP/OCP/etc. k8s Operators/Helm Charts/YAML manifests for creating k8s deployments (SaaS services).





