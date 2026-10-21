# Setup CloudFlare Webhook Fan Out

* Login to CloudFlare
* On the left click Compute -> `Workers & Pages`
* Click `Create Application`
* Select `Start with Hello World`
* Change the worker name to reflect that this is where you send your success webhooks
* Leave `Protect with CloudFlare Access` disabled
* Click `Deploy`
* Click `Edit Code` on the top right
* Delete all existing code
* Paste in the code below for the success webhook
* Replace the URLs in the top of the code with your webhook URLs
    * Masked means it will hide e-mail, order number etc.
    * There is no masking for declines/misc at the moment so it's just one list of URLs
* **REPEAT THIS PROCESS FOR DECLINE AND MISC WEBHOOKS (USING THE SPECIFIC CODE BELOW**

[Code for Success Webhook](./Success_Webhook_Code.md)  
[Code for Fail Webhook](./Fail_Webhook_Code.md)  
[Code for Misc Webhook](./Misc_Webhook_Code.md)   
