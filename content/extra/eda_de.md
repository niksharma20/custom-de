# 

##

**Step 1: Define the Decision Environment**  
Create a file called [decision-environment.yml](/custom_image_definations/decision-environment.yml)  

**Step 2: Running the Build**  
```bash
# 1. Log in to ensure your system can pull the AAP 2.6 base layer
podman login registry.redhat.io

# 2. Run the build and tag the image 'mega-ee' version 1.0
ansible-builder build -f decision-environment.yml -t ansible-nikhil-de:v1.0 --extra-build-cli-args "--platform linux/amd64"
```

**Step 2**: Clear the Cache and Rebuild  

```bash
# Clear out cache directories  
rm -rf context/  

# Re-run build  
ansible-builder build -f decision-environment.yml -t ansible-nikhil-de:v1.0 --extra-build-cli-args "--platform linux/amd64"
```

**Step 4: Verifying Your New Image**  

```bash
podman run --rm ansible-nikhil-de:v1.0 python3 -m ansible galaxy collection list
```
**Step 5: Push it to Quay.io**  
```bash
# 1. Login to Quay
podman login quay.io

# 2. Tag the image for your Quay account
podman tag ansible-nikhil-de:v1.0 quay.io/YOUR_QUAY_USERNAME/ansible-nikhil-de:v1.0

# 3. Push it up
podman push quay.io/YOUR_QUAY_USERNAME/ansible-nikhil-de:v1.0
```
> [!NOTE]
> remember to create matching the image name in quay.io under your organisation.  
> **Registry**: quay.io  
> **Image Name**: quay.io/niksharma20/ansible-nikhil-de:v1.0  
> **Image** [ansible-nikhil-de:v1.0](quay.io/niksharma20/ansible-nikhil-de:v1.0)  
