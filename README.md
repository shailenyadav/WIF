To connect Google Cloud Platform (GCP) with GitHub using Workload Identity Federation and list down the instances name, you'll need to follow several steps. Please note that the following instructions assume you have already set up a GCP project and have the necessary permissions to create and manage resources.


Step 1: Enable necessary APIs
  1. Go to the GCP Console: https://console.cloud.google.com/.
  2. Open your project.
  3. Click on the "Navigation Menu" button (three horizontal lines) in the upper-left corner and select "APIs & Services" > "Library" from the menu.
  4. Search for the following APIs and enable them if they are not already enabled:
  		 - Identity and Access Management (IAM) API
   		 - Cloud Resource Manager API
   		 - Compute Engine API


Step 2: Set up a service account for Workload Identity Federation
  1. In the GCP Console, go to "IAM & Admin" > "Service Accounts".
  2. Click on "Create Service Account".
  3. Provide a name and description for the service account and click on "Create".
  4. Assign the necessary roles to the service account, such as "Compute Instance Admin" and "Workload Identity User".
  5. Click on "Done".


Step 3: Set up Workload Identity Federation
  1. In the GCP Console, go to "IAM & Admin" > "Workload Identity Federation".
  2. Click on "Add Provider".
  3. Provide a name and description for the provider.
  4. Select "OpenID Connect (OIDC)" as the provider type.
  5. Click on "Create".
  6. In the "Configure Provider" tab, click on "Create Mapping".
  7. Provide a name and description for the mapping.
  8. Click on "Create".
  9. Once pool is created, you need to grant access to your service account.
    -  Click on Pool
    -  Click on Grant Access
    -  Grant Access to your Service Account buy selecting your service account name from the drop-down menu and SAVE.


Step 4: Set up GitHub
  1. Go to the GitHub repository you want to connect with GCP.
  2. Navigate to "Settings" > "Secrets".
  3. Click on "New repository secret".
  4. Provide a name for the secret `PROJECT_ID` and enter the GCP project ID as the value.
  5. Repeat the above steps to create another secret with the name `SERVICE_ACCOUNT` and paste the service account email you created in `Step 2` as the value.
  6. Repeat the above steps to create another secret with the name `POOL_URL` and paste the Workload Identity Federation Pool Url you created in `Step 3` as the value.


Step 5: Configure GitHub Actions for deployment
  1. In your GitHub repository, go to the "Actions" tab.
  2. Click on "Set up a workflow yourself" or "New workflow".
  3. Replace the content of the generated YAML file with the following code:

```#yaml

name: compute instances list using Workload Identity Federation
on:
  push:
    branches:
      - main
jobs:
  gcloud-setup:
    runs-on: ubuntu-latest
    # Add "id-token" with the intended permissions.
    permissions:
      contents: 'read'
      id-token: 'write'

    steps:
    - uses: 'actions/checkout@v3'

    # Configure Workload Identity Federation via a credentials file.
    - id: 'auth'
      name: 'Authenticate to Google Cloud'
      uses: 'google-github-actions/auth@v1'
      with:
        token_format: 'access_token'
        workload_identity_provider: ${{ secrets.POOL_URL }}
        service_account: ${{ secrets.SERVICE_ACCOUNT }}
        
    - name: 'List Compute Instances'
      run: |-
        gcloud compute instances list --project '${{ secrets.PROJECT_ID }}' --zones us-central1-a
```

  4. Save the file.

That's it! Whenever you push changes to the master branch of your GitHub repository, the GitHub Actions workflow will trigger, authenticate with GCP using the service account key.


