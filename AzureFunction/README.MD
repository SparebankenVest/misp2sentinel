# MISP to Microsoft Sentinel

This code can be used to create an Azure Function that writes threat intelligence from a MISP instance to Microsoft Sentinel. The code is a modified version of [script.py](https://github.com/cudeso/misp2sentinel/blob/main/script.py) in this repository with Azure Function specific code from [zolderio/misp-to-sentinel](https://github.com/zolderio/misp-to-sentinel) modified slightly.


## Installation

### Prerequisities
- An Azure Subscription 
- A Microsoft Sentinel Instance
- API and URL of your MISP instance

### Full Instructions

Full instructions in [INSTALL.md](../docs/INSTALL.MD)

### Summary

1. Create an App in the same Microsoft tenant where the Sentinel instance resides. The app requires the *ThreatIndicators.ReadWrite.OwnedBy* and *ThreatIndicators.Read.All* (Application type) permission.
2. Create a Keyvault in your Azure subscription
3. Add a new secret with the name "tenants" and the following value (its possible to add multiple Sentinel instances, it will loop all occurences):
```
{"<TENANT_ID_WITH_APP>": {"id": "<APP_ID>", "secret": "APP_SECRET"} }
```
4. Add a new secret with the name "mispkey" and the value of your MISP API key
5. Create an Azure Function in your Azure subscription
6. Modify config.py to your needs (event filter). 
7. Upload the code to your Azure Function. 
   * If you are using VSCode, this can be done by clicking the Azure Function folder and selecting "Deploy to Function App", provided you have the Azure Functions extension installed.
7. Add a "New application setting" (env variable) to your Azure Function named `tenants`. Create a reference to the key vault previously created (`@Microsoft.KeyVault(SecretUri=https://<keyvaultname>.vault.azure.net/secrets/tenants/)`).
8. Do the same for the `mispkey` secret (`@Microsoft.KeyVault(SecretUri=https://<keyvaultname>.vault.azure.net/secrets/mispkey/)`)
9. Add a "New application setting" (env variable) called `mispurl` and add the URL to your MISP-server (`https://<mispurl>`)
10. Add a "New application setting" (env variable) `timerTriggerSchedule` and set it to run. If you're running against multiple tenants with a big filter, set it to run once every two hours or so. 
   * The `timerTriggerSchedule` takes a cron expression. For more information, see [Timer trigger for Azure Functions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer?tabs=python-v2%2Cin-process&pivots=programming-language-python).
   * Run once every two hours cron expression: `0 */2 * * *`


#### Multi-tenant support

If you want to push to multiple Azure tenants using a single app registration, you can accomplish this by making the app created in step 1 a multi tenant app. Customers can then consent to the app in their own tenant using the following url schema: 
 
`https://login.microsoftonline.com/common/adminconsent?client_id=<APP_ID>&sso_reload=true` 
 
This will allow you to add multiple tenants to the `tenants` variable in step 3. 
Please note that this will require you to add a redirect URL in the app registrations configuration, like `https://portal.azure.com`.

## Usage

If the installation was successful, the MISP data should be written to your Sentinel instance on 00:00 every day. Use the following query to test if the data was written successfully:

```
ThreatIntelligenceIndicator
```

## Credits

Credits for this update goes to:

* [cudeso](https://github.com/cudeso/misp2sentinel) for his work on upgrading the original MISP2Sentinel script
* [zolderik](https://github.com/zolderio/misp-to-sentinel/tree/main) for his work on the Azure Function. 

I've simply spliced them together and added some minor improvements.