****Grafana Installation and Azure Monitor Integration – Report**


**Steps Taken:**


1. Azure VM Setup

   
   Created a new Azure resource group:

       az group create --name MyResourceGroup --location canadacentral

   Created an Ubuntu 22.04 VM using:


       az vm create --resource-group MyResourceGroup --name MyGrafanaVM --image Ubuntu2204 --admin-username azureuser --generate-ssh-keys


   Retrieved the public IP of the VM for browser access.


2.  Grafana Installation


        SSH’d into the VM using:

         ssh azureuser@<public-ip>

      Installed Grafana by configuring the apt repository and installing via apt-get.

       Enabled and started the Grafana service:


         sudo systemctl enable grafana-server
         sudo systemctl start grafana-server

3.Firewall Configuration
    
  Allowed inbound traffic on port 3000 (Grafana’s default port):

     sudo ufw allow 3000/tcp
     sudo ufw enable
     
   Also added an Inbound Port Rule in Azure Portal to allow TCP 3000 traffic.

   
   
4.Accessing Grafana


  Accessed the Grafana web UI from the browser at:
              
    http://<public-ip>:3000

  Logged in using default credentials:


   Username: admin, Password: admin (then changed password on first login).


5. Azure CLI Setup in VM

   Installed Azure CLI on the VM:

       Curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   
   Logged into Azure using managed identity:

       az login --identity
   
6. Permissions Assignment

   Retrieved the VM's Managed Identity Principal ID.

   

   Assigned required roles using:

        az role assignment create --assignee <principal-id> --role "Monitoring Reader" --scope /subscriptions/<subscription-id>
        az role assignment create --assignee <principal-id> --role "Reader" --scope /subscriptions/<subscription-id>
   
 
7. Log Analytics Workspace

   Discovered Grafana couldn’t detect any Log Analytics workspaces.

   Created a new workspace:

       az monitor log-analytics workspace create --resource-group MyResourceGroup --workspace-name MyLogAnalyticsWorkspace --location canadacentral

   
     Assigned Reader role on the workspace to the VM’s identity.


9. Connecting Grafana to Azure Monitor

   
   In Grafana, added Azure Monitor as a data source.
   

    Selected Managed Identity as the authentication method.

   

   Clicked "Save & Test", which successfully connected to Azure Monitor and detected the Log Analytics workspace.


 ## ❗ Issues Encountered & Resolutions

| Issue | Resolution |
|-------|------------|
| `az: command not found` | Installed Azure CLI using Microsoft’s official script: `curl -sL https://aka.ms/InstallAzureCLIDeb \| sudo bash`. |
| SSH Permission Denied (`Permission denied (publickey)`) | Regenerated SSH keys using `ssh-keygen` and reset them in Azure Portal via "Reset SSH Public Key". |
| Grafana port 3000 not accessible in browser | Opened port 3000 in both VM using `sudo ufw allow 3000/tcp` and in Azure Network Security Group via inbound rule. |
| Grafana not listening on external IP | Edited `/etc/grafana/grafana.ini`, set `http_addr = 0.0.0.0`, and restarted Grafana. |
| No Log Analytics workspaces found | Created a workspace using `az monitor log-analytics workspace create` and verified its existence with `az monitor log-analytics workspace list`. |
| Azure Monitor connection failed | Assigned `Monitoring Reader` and `Reader` roles to the VM’s managed identity using `az role assignment create`. |


**Screenshots**

![Screenshot 2025-03-18 141152](https://github.com/user-attachments/assets/962bb1c1-2682-42e3-ae9d-77e4bc7c671d)
![Screenshot 2025-03-18 141431](https://github.com/user-attachments/assets/3c0f285c-99c7-48eb-a3f9-694f0c7cb6af)
![Screenshot 2025-03-18 142452](https://github.com/user-attachments/assets/3217bae1-4b11-4ee3-ada8-93e415fd8bb4)
![Screenshot 2025-03-20 204201](https://github.com/user-attachments/assets/2dd75bde-e442-4d81-8dea-46f65906dbb5)
![Screenshot 2025-03-20 204324](https://github.com/user-attachments/assets/13a31e68-ca0b-42f0-a28f-995e8a0015e4)
![Screenshot 2025-03-20 204553](https://github.com/user-attachments/assets/3bd9ae0d-541e-4e67-860b-1d957ed6f542)
![Screenshot 2025-03-20 204611](https://github.com/user-attachments/assets/dae79004-3f4d-40e6-81f8-2140ca9a1ec0)
![Screenshot 2025-03-20 205514](https://github.com/user-attachments/assets/2bea820f-3799-4acd-b38a-0890058ff85e)
![Screenshot 2025-03-20 211710](https://github.com/user-attachments/assets/ade77168-bb9e-4833-9b6b-616bab5072dd)
![Screenshot 2025-03-20 211613](https://github.com/user-attachments/assets/25299e03-1caa-42c0-b900-30d4cbd6718c)


**Deleted resource group**
![Screenshot 2025-03-20 211758](https://github.com/user-attachments/assets/e60b5c52-cb49-45c8-805d-d3c349148ef3)

 
