# gcp-kubernetes-workload-identity-and-mapping-gcp-service-account-with-gke

### Here is the formatted and organized set of commands and explanations:

# Step 1: Create or Update GKE Cluster
## Create a New Cluster
### Enable Workload Identity and the GKE Metadata Server in the node pool configuration when creating the cluster:

    gcloud container clusters create my-cluster \
    --location=us-central1-c \
    --workload-pool=my-project-id.svc.id.goog

## Update an Existing Cluster
### Enable Workload Identity on an existing cluster:

    gcloud container clusters update disearch-cluster \
    --location=us-central1-c \
    --workload-pool=my-project-id.svc.id.goog

# Step 2: Create or Update Node Pool with Metadata Server
## Create a New Node Pool
### Enable the GKE Metadata Server when creating the node pool:

    gcloud container node-pools create my-node-pool \
    --cluster=my-cluster \
    --region=us-central1 \  or --zone us-central1-c
    --workload-metadata=GKE_METADATA

## Update an Existing Node Pool
### Enable the GKE Metadata Server on an existing node pool:

    gcloud container node-pools update my-node-pool \
    --cluster=my-cluster \
    --region=us-central1 \ or --zone us-central1-c(works)
    --workload-metadata=GKE_METADATA

# Step 3: Create Service Account in Kubernetes Namespace
## Create a service account named gke-sa in the test namespace:

      kubectl create serviceaccount gke-sa --namespace=test

# Step 4: Add Service Account to Deployment File
## Create a deployment and service for Nginx in a single file, with the service account named gke-sa:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: doc-chat-deployment
      namespace: test
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: doc-chat
      template:
        metadata:
          labels:
            app: doc-chat
        spec:
          serviceAccountName: gke-sa      
          containers:
          - name: doc-chat
            image: gcr.io/disearch-vertexai/doc_chat:latest
            imagePullPolicy: Always
            ports:
            - containerPort: 5000
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: doc-chat-service
      namespace: test
      annotations:
        networking.gke.io/load-balancer-type: "Internal"
    spec:
      selector:
        app: doc-chat
      ports:
        - protocol: TCP
          port: 80
          targetPort: 5000
      type: LoadBalancer

# Step 5: Create Service Account in GCP Project
## Create a service account named gcp-sa in the GCP project:

    gcloud iam service-accounts create gcp-sa --project=my-project-id

# Step 6: Grant Roles to GCP Service Account
## Assign the necessary roles to the gcp-sa service account like i have assign for my use case:

    gcloud projects add-iam-policy-binding my-project-id \
    --member "serviceAccount:gcp-sa@my-project-id.iam.gserviceaccount.com" \
    --role "roles/apigateway.admin"

    gcloud projects add-iam-policy-binding my-project-id \
        --member "serviceAccount:gcp-sa@my-project-id.iam.gserviceaccount.com" \
        --role "roles/cloudfunctions.admin"
    
    gcloud projects add-iam-policy-binding my-project-id \
        --member "serviceAccount:gcp-sa@my-project-id.iam.gserviceaccount.com" \
        --role "roles/discoveryengine.admin"
    
    gcloud projects add-iam-policy-binding my-project-id \
        --member "serviceAccount:gcp-sa@my-project-id.iam.gserviceaccount.com" \
        --role "roles/secretmanager.admin"
    
    gcloud projects add-iam-policy-binding my-project-id \
        --member "serviceAccount:gcp-sa@my-project-id.iam.gserviceaccount.com" \
        --role "roles/iam.serviceAccountTokenCreator"
    
    gcloud projects add-iam-policy-binding my-project-id \
        --member "serviceAccount:gcp-sa@my-project-id.iam.gserviceaccount.com" \
        --role "roles/storage.admin"
    
    gcloud projects add-iam-policy-binding my-project-id \
        --member "serviceAccount:gcp-sa@my-project-id.iam.gserviceaccount.com" \
        --role "roles/storage.objectAdmin"

# Step 7: Bind GCP Service Account to GKE Service Account
## Bind the GCP service account to the GKE service account:

    gcloud iam service-accounts add-iam-policy-binding gcp-sa@my-project-id.iam.gserviceaccount.com \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:my-project-id.svc.id.goog[test/gke-sa]"

# Step 8: Annotate Kubernetes Service Account
## Annotate the Kubernetes service account so that GKE sees the link between the service accounts:

    kubectl annotate serviceaccount gke-sa \
    --namespace test \
    iam.gke.io/gcp-service-account=gcp-sa@my-project-id.iam.gserviceaccount.com
