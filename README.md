# Using docker with gcp (Google Cloud Platform)

1. Activate Artifact Registry and Cloud Build APIs:

```bash
gcloud services enable artifactregistry.googleapis.com
gcloud services enable cloudbuild.googleapis.com
```

2. Have the cloudbuild.yaml file with the following content:

```bash
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/<project-id>/<image-name>', '<path-to-dockerfile>']
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/<project-id>/<image-name>']
```

Where:
- `<project-id>` is the project id from gcp
- `<image-name>` is the desired name of the image
- `<path-to-dockerfile>` can be `.` or `./Dockerfile` or `./dockerfiles/Dockerfile`

Example:
```bash
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/nifty-byway-410709/testing', 'dockerfiles/test.dockerfile']
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/nifty-byway-410709/testing']
```
3. Connect github repository to gcp

4. Create a trigger for the cloudbuild.yaml file in gcp
This can e.g. be set up to build every time you push to the repository's main branch.

5. The build can be found in the Artifact Registry (containers)

6. Pull the image (like you would normally do - this requires authentication between Github and gcp)

Authenticate docker with gcp (only needs to be done once):
```bash
gcloud auth configure-docker
```
Pull the image:

```bash
docker pull gcr.io/<project-id>/<image_name>:<image_tag>
```



# Create virtual machines (Compute Engines)

## From the console

Using CPU
```bash
gcloud compute instances create <desired-instance-name> \
    --zone europe-west1-b \
    --image-family=pytorch-latest-cpu \
    --image-project=deeplearning-platform-release
```

Using GPU
```bash
gcloud compute instances create <desired-instance-name> \
    --zone europe-west4-a \
    --image-family=pytorch-latest-gpu \
    --image-project=deeplearning-platform-release \
    --accelerator="type=nvidia-tesla-v100,count=1" \
    --metadata="install-nvidia-driver=True" \
    --maintenance-policy TERMINATE
```

To connect:
```bash
gcloud beta compute ssh <instance-name>
```

To build a specific image you can see which ones are available by (from terminal, example with pytorch):
```bash
gcloud compute images list --project deeplearning-platform-release --filter="family:pytorch*"
```


## From dockerfile (NB: this does not work as intended)
1. Create local dockerfile.
For this example a simple dockerfile `gcp_vm_tester.dockerfile` is created.

2. Build the image:
```bash
docker build -f gcp_vm_tester.dockerfile.dockerfile . -t gcp_vm_tester:latest
```

3. Make tag for image and push to gcp repository:

```bash
docker tag gcp_vm_tester gcr.io/<project-id>/gcp_vm_tester
docker push gcr.io/<project-id>/gcp_vm_tester
```

4. Create instance with docker image:

```bash
gcloud compute instances create-with-container <desired-instance-name> \
    --container-image=gcr.io/<project-id>/gcp_vm_tester \
    --zone europe-west1-b

gcloud compute instances create-with-container test-with-dockerimage \
    --container-image=gcr.io/nifty-byway-410709/gcp_vm_tester \
    --zone europe-west1-b
```
NB: `<desired-instance-name>` must be unique and without "_".


5. Connect to instance:

```bash
gcloud compute ssh --zone "europe-west1-b" "<desired-instance-name>" --project "<project-id>"
```

