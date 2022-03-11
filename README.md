## Architecture 
We leverage the AWS Fault Injection Simulator (FIS) and its integration with AWS Service Catalog and AWS Control Tower to induce failures in a multi-account setup. In this approach, we leverage CfCT to provision one AWS Service Catalog portfolio and a product using AWS fault injection service, which is then shared with other AWS accounts. 

Figure 2 illustrates three accounts of a multi-account environment. There is an AWS Control Tower Shared Services account where a AWS Service Catalog portfolio and a product leveraging Fault injection service is created. The portfolio is shared with the remaining account in the organization. There is a target-account (spoke) account where failures are injected. In this spoke account we have two fault injection experiments, one for web server instances, and another for Amazon RDS instances. Note that you can create other experiments in the target account as per your requirements.

![image](https://github.com/aws-samples/chaos-engineering-with-aws-fault-injection-simulator-cfct-and-aws-service-catalog/blob/main/ce11.png)


## Deployment


The deployment consists of five main steps :

Step 1 - S3 bucket in Control Tower Management Account that stores the code for AWS service catalog products used for FIS experiments.

Step 2 - Configuration changes to Control Tower Management Account

Step 3 - Changes to manifest file and parameter configurations in Management Account

Step 4 - Configuration changes to Spoke Accounts

Step 5 - Deploy the FIS experiments

These are explained below

Step 1 - S3 bucket in Management Account that stores the code for AWS service catalog products used for FIS experiments

- Log in to AWS console of Control Tower Management Account and select the Cloudformation service in the region of CfCt deployment.
- Launch a Cloudformation stack using s3-bucket-template.
- Upload the FIS-experiment-template as an object to the S3 bucket created above. Copy the object URL into your notepad as it will be used in further steps below.

Step 2 - Configuration changes to Control Tower Management Account

- First, deploy the CfCT framework in the AWS Control Tower Management Account. Follow the implementation guide. The framework can be deployed using the ‘Launch in the AWS Management Console’ button in the link provided. Use the default parameters when deploying the framework.
- Set up Amazon Simple Storage Service (S3) as the configuration source by following the steps outlined in appendix-a.
- Enable trusted access for AWS Service Catalog in your AWS Organizations organization. Navigate to AWS Organizations, Services, Service Catalog, and choose - Enable trusted access. Enabling trusted access lets AWS Service Catalog Portfolio be distributed within your organization.
  - Delegate the Shared Services account as a delegated administrator for AWS Service Catalog. This lets your Shared Services account act as an administrator when sharing Portfolios to spoke accounts.
  - From the Management Account, check your current list of delegated administrator accounts by running the following command: 
  ```aws organizations list-delegated-administrators```
  If this is your first time running this command, then you should receive an empty response.
  - To designate the Shared Services account as an administrator, run the following command:

```aws organizations register-delegated-administrator --account-id <YOUR_AWS_ACCOUNT_ID> --service-principal servicecatalog.amazonaws.com```

To verify the delegation, rerun the command:

```aws organizations list-delegated-administrators –service-principal servicecatalog.amazonaws.com```
The response will show the delegated account details

Step 3 : Changes to manifest file and parameter configurations in Management Account

- Download the custom-control-tower-configuration.zip file and unzip them.
- Update /parameters/sc-pf-pr-source.json for “OrganizationalUnit” with your Organization ID and “Provisioning Template” with the URL for the FIS source code object created in Step 1.
- Update the parameters for region, location path of resource file and parameter file and IDs of accounts in manifest.yml.
- Compress/zip the files back with name custom-control-tower-configuration.zip.

Step 4 : Configuration changes to Spoke Accounts

In the spoke account, you will require to designate users/groups/roles to the imported Portfolio. To do this, follow the below steps :

- Log in to the AWS console in spoke account in the region where CfCt is being deployed. Navigate to AWS Service Catalog.
- In the Imported portfolio details page, expand Users, groups and roles, and then choose Add user, group or role.
- Choose the Groups, Users, or Roles tab to add groups, users, or roles, respectively.
- Choose one or more users, groups, or roles, and then choose Add Access to grant them access to the current portfolio.
- Using the identity that you designated in the prior step, launch the Chaos Engineering product by navigating to ‘Products’ (above Administration grouping if visible) and launch the product.

Step 5 :Deploy the FIS Experiment
- From the same Spoke Account used above, navigate to the AWS FIS console. 
- On the left-hand menu, select ‘Experiment templates’. You should see two FIS Experiment templates: RebootRDSInstances, StopEC2Instances.
- Select one of the experiment templates to run (make sure that you have EC2/RDS instances that meet the requirements listed in the Prerequisites section above).
- Choose Actions, and Start experiment. When prompted, enter start, and choose Start experiment.
- Navigate to the EC2 and Amazon RDS console to observe the Experiment executions. You should observe EC2 instances being terminated and recreated via the Auto Scaler. For the Amazon RDS experiment, you should see the Amazon RDS primary DB being rebooted.

Note the following:

Additional actions can be added to existing Experiment Templates by selecting Update in the Actions dropdown in the desired Experiment Template. From there, Actions can be added and removed depending on the intent of what must be tested.
Experiment Targets can also be customized by updating the experiment and adding Targets. Targets are specified by Resource IDs or Resource tags and filters. In this post, we use the following tag key-value pair: ChaosTesting, ChaosReady.

## Authors

- Shiva Vaidyanthan - Sr.Cloud Infrastructure Architect - vaidys@amazon.com
- Benson Tanner - Cloud Infrastructure Architect - tanbenso@amazon.com

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

