

# Infrastructure Setup

First create requried resources (AKS clusters, AppInsights, KeyVaults etc) on Azure for your team. These resources will be edited by the hackteams.
It also creates a storage account for TF state, found in RG state<teamname><location>.
1. Open Cloudshell on shell.azure.com, if required create a storage account (only for cloud shell sessions) in your subscription and open Bash
2. Clone this repo 
```
git clone https://github.com/azure-adventure-day/aad-team.git
```

3. You'll find resource definitions as code (infrastructure as code) in a bunch of *.tf files in the terrafrom folder. Deploy them using the following command but replace <team_name> with your actual team's name, <region> with your preferred region and <subscription_id> with your subscription id.
Make the file executable first.
```
chmod +x ./deploy-team.sh
./deploy-team.sh <team_name> northeurope <subscription_id>
```

After the script has been executed you will see two resource groups, one holding TF storage, the other one holding e.g. the AKS cluster etc.

**Note: You don't have to wait until the terraform deployment has finished completely to continue with the next steps. Check if the Azure services like Azure SQL are already deployed in the Azure Portal and continue working with the tasks below!**


## Gameengine Setup

1. An instance of Azure SQL and a SQL DB  should be deployed to your RG. Reset the Azure SQL Server admin password and adjust the firewall in the Azure Portal for Azure SQL Server so you can work with the DB.
2. Also allow other Azure Services to access SQL Server so your cluster can talk to the DB.
3. In the Azure Portal open your SQL Database,  go to the Query editor and execute the scripts in the DatabaseScripts folder.
4. Take a note of the SQL database connection string.
5. Switch to the GameEngine folder and modify the blackbox_gameengine_deployment.yaml file to reference your connection strings. Make sure you set the password correctly in the DB connection string.
6. You can deploy directly from Cloud Shell. Run the following command to be able to use kubectl with your aks cluster.
```
az aks-get credentials -n <aks_cluster_name> -g <resource_group_name>
```
7. Deploy the game engine using kubectl. 
```
kubectl apply -f blackbox_gameengine_deployment.yaml
```
8. Figure out the public endpoint which can be used to call your game engine. Use the command below and get the public IP of the gameengine service. 
```
kubectl get services --field-selector metadata.name=blackboxgameengine --output=jsonpath={.items..status.loadBalancer.ingress..ip}
```
9. If you found your endpoint, you can call it on http://<YOUR_ENDPOINT_IP>/Match in the browser.

10. Important: **You have to provide this URL in the team portal**, so gambling can start. **But make sure you deploy your gamebot first (see instructions below) otherwise the engine will fail and you will get malus points.**


## Gamebot Setup
You can deploy directly from ghcr.io/ricardoniepel/azure-adventure-day-coach/gamebot:latest without an additional build. 



### Deployment

1. Deploy your bot.
```
kubectl apply -f gamebot_deployment.yaml
```

2. Test your Bot
You can get the IP of your bot by running 
```
kubectl get services --field-selector metadata.name=arcadebackend --output=jsonpath={.items..status.loadBalancer.ingress..ip}
```

You can test your bot by posting something like this to your bot's public IP http://A.B.C.D/pick.

```
curl --location --request POST 'http://<IP>/pick' --header 'Content-Type: application/json' --data-raw '{"Player1Name":"daniel","MatchId":"42"}'
```

You can test your engine by posting something like this to your engine's public IP.
```
curl --location --request POST 'http://<IP>/Match' --header 'Content-Type: application/json' --data-raw '{"ChallengerId":"daniel","Move": "Rock"}'
```
For subsequent requests, make sure you put the gameid from the response into the request.

## Register your game endpoint

**Register your game endpoint in your team portal. As soon as you have registered it customers will start gambling and your performance is being measured. So make sure everything works as expected up front!**


### Optional: Build and push bot image manually
If you change the source code you can build and push the container manually or use a prepared github action (check the .github/workflows folder). 
To modify, build and push the bot before deploying it, follow these steps:

1. Build the image you want to use, eg like this.
```
docker build -t yourContainerRegistry/Gamebot .
```
2. Change the file gamebot_deployment.yaml to reference your bot image.
Then publish your bot image:
```
docker push yourContainerRegistry/Gamebot
```
