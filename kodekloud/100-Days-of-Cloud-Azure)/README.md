<img width="927" height="161" alt="image" src="https://github.com/user-attachments/assets/eab107d8-1c1e-4b3a-a762-06c562cafd43" />

# DAY 1
## Create SSH Key Pair for Azure Vritual Machine
<img width="947" height="1156" alt="image" src="https://github.com/user-attachments/assets/5a0518b4-e54a-42c0-a994-b273fd9c61bf" />

### Web Links References: 
- https://learn.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest
- https://learn.microsoft.com/en-us/cli/azure/sshkey?view=azure-cli-latest
- https://learn.microsoft.com/en-us/azure/virtual-machines/linux/mac-create-ssh-keys

### STEPS
<img width="975" height="483" alt="image" src="https://github.com/user-attachments/assets/5f573f11-c819-49f1-8f00-533f74a264be" />
<img width="975" height="740" alt="image" src="https://github.com/user-attachments/assets/162bacac-592a-49ae-9a17-b8de8da91588" />
<img width="975" height="788" alt="image" src="https://github.com/user-attachments/assets/88de5710-f6df-4ca4-9751-bc98feaea5f3" />
<img width="975" height="654" alt="image" src="https://github.com/user-attachments/assets/467016a9-e96b-4625-bc60-e0bbe17d2666" />
<img width="975" height="768" alt="image" src="https://github.com/user-attachments/assets/2eb1ba43-c2ea-4e49-a8a4-ef3b80f1ed6b" />
<img width="975" height="681" alt="image" src="https://github.com/user-attachments/assets/c6348bc0-7cc3-4cb4-a290-07a206ff2312" />
<img width="975" height="715" alt="image" src="https://github.com/user-attachments/assets/3c131361-0e24-4cfd-9e67-33667fbe989f" />
<img width="975" height="737" alt="image" src="https://github.com/user-attachments/assets/65a4603e-f490-416a-8874-bc47b6399bbc" />

```bash
# 1. Retrieve credentials
showcreds

# 2. Authenticate using Device Code (due to AADSTS50126 error)
az login --use-device-code

# 3. Identify the Resource Group
az group list -o table

# 4. First Attempt to create SSH Key (Failed due to typo in flag)
az sshkey create --name "datacenter-kp" --resource--group "kml_rg_main-c1e1e59a8ab344ea"

# 5. Second Attempt (Success - Typo corrected)
az sshkey create --name "datacenter-kp" --resource-group "kml_rg_main-c1e1e59a8ab344ea"

# 6. Verify Resource in Azure
az sshkey list --resource-group "kml_rg_main-c1e1e59a8ab344ea" -o table

# 7. Verify Local Key Files
ls -l /root/.ssh/

```
