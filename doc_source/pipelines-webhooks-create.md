--------

The procedures in this guide support the new console design\. If you choose to use the older version of the console, you will find many of the concepts and basic procedures in this guide still apply\. To access help in the new console, choose the information icon\.

--------

# Create a Webhook for Your GitHub Pipeline<a name="pipelines-webhooks-create"></a>

After you use the AWS CLI to manually create a webhook, you must register the webhook in GitHub\. A designated AWS endpoint is used for the webhook and is supplied by the put\-webhook command\.

To use the AWS CLI to create a webhook, call the put\-webhook command and supply the following:
+ A name that uniquely identifies the webhook\. This name must be unique within the region of the account for the pipeline\.
+ A secret in the JSON file to be used for GitHub authorization\.<a name="proc-cli-gh-webhook"></a>

**To create and register your webhook**

1. In a text editor, create and save a JSON file for the webhook you want to create\. Use this sample file for a webhook named `my-webhook`:

   ```
   {"webhook": 
     {"name": "my-webhook",
      "targetPipeline": "pipeline_name",
      "targetAction": "source_action_name",
      "filters": [
       {
         "jsonPath": "$.ref", 
         "matchEquals": "refs/heads/{Branch}"
       }
      ],
      "authentication": "GITHUB_HMAC",
      "authenticationConfiguration": {"SecretToken":"secret"}
     }
   }
   ```

1. Call the put\-webhook command and include the `--cli-input` and `--region` parameters\.

   The following sample command creates a webhook with the `webhook_json` JSON file\.

   ```
   aws codepipeline put-webhook --cli-input-json file://webhook_json.json --region "eu-central-1"
   ```

1. In the output shown in this example, the URL and ARN are returned for a webhook named `my-webhook`\.

   ```
   {
       "webhook": {
           "url": "https://webhooks.domain.com/trigger111111111EXAMPLE11111111111111111",
           "definition": {
               "authenticationConfiguration": {
                   "SecretToken": "secret"
               },
               "name": "my-webhook",
               "authentication": "GITHUB_HMAC",
               "targetPipeline": "pipeline_name",
               "targetAction": "Source",
               "filters": [
                   {
                       "jsonPath": "$.ref",
                       "matchEquals": "refs/heads/{Branch}"
                   }
               ]
           },
           "arn": "arn:aws:codepipeline:eu-central-1:ACCOUNT_ID:webhook:my-webhook"
       }
   }
   ```

1. Call the register\-webhook\-with\-third\-party command and include the `--webhook-name` parameter\.

   The following sample command registers a webhook named `my-webhook`\.

   ```
   aws codepipeline register-webhook-with-third-party --webhook-name my-webhook
   ```

If you are updating a pipeline to use webhooks, you must also use the following procedure to turn off periodic checks\.<a name="proc-cli-flag-github"></a>

**To edit your pipeline's PollForSourceChanges parameter**

1. Run the get\-pipeline command to copy the pipeline structure into a JSON file\. For example, for a pipeline named `MyFirstPipeline`, you would type the following command: 

   ```
   aws codepipeline get-pipeline --name MyFirstPipeline >pipeline.json
   ```

   This command returns nothing, but the file you created should appear in the directory where you ran the command\.

1. Open the JSON file in any plain\-text editor and edit the source stage by changing or adding the `PollForSourceChanges` parameter\. In this example, for a repository named `UserGitHubRepo`, the parameter is set to `false` \.

   **Why am I making this change? ** Changing this parameter turns off periodic checks so you can use event\-based change detection only\.

   ```
   "configuration": {
     "Owner": "darlaker",
     "Repo": "UserGitHubRepo",
     "PollForSourceChanges": "false",
     "Branch": "master",
     "OAuthToken": "****"
     },
   ```

1. If you are working with the pipeline structure retrieved using the get\-pipeline command, you must edit the structure in the JSON file by removing the `metadata` lines from the file\. Otherwise, the update\-pipeline command cannot use it\. Remove the `"metadata"` section from the pipeline structure in the JSON file, including the : `{ }` and the `"created"`, `"pipelineARN"`, and `"updated"` fields\.

   For example, remove the following lines from the structure: 

   ```
   "metadata": {  
     "pipelineArn": "arn:aws:codepipeline:region:account-ID:pipeline-name",
     "created": "date",
     "updated": "date"
     }
   ```

   Save the file\.

1. To apply your changes, run the update\-pipeline command, specifying the pipeline JSON file, similar to the following:
**Important**  
Be sure to include `file://` before the file name\. It is required in this command\.

   ```
   aws codepipeline update-pipeline --cli-input-json file://pipeline.json
   ```

   This command returns the entire structure of the edited pipeline\.
**Note**  
The update\-pipeline command stops the pipeline\. If a revision is being run through the pipeline when you run the update\-pipeline command, that run is stopped\. You must manually start the pipeline to run that revision through the updated pipeline\. Use the start\-pipeline\-execution command to manually start your pipeline\.