# Valorem Reply Modifications

## Updates to IAC

* Updated the build resources file (`\iac\createResources.bicep`) to include availability zones
    * Added the following to the agentPoolProfiles section of the AKS cluster
        * `availabilityZones: ['1', '2', '3']`

* Updated the YML file (`\.azurepipelines\contoso-traders-cloud-testing.yml`) to the full namespace for `replacetokens@5`
    * At Valorem Reply, we have 2 ADO Extensions for ReplaceTokens, so we had to fully qualify it like this: `qetza.replacetokens.replacetokens-task.replacetokens@5`

## Manual Updates

### Add Custom Autoscale to the VMSS for the AKS cluster
    
    Resource Group: contoso-traders-aks-nodes-rg{suffix}

* Select the Virtual Machine Scale Set (VMSS)
* Under the left menu section `Availability + scale`, select `Scaling`
* Choose `Custom autoscale`
    * Confirm default Name
    * Confirm default Resource Group
    * Predictive autoscale: Mode: `Disabled`
    * Scale mode: `Scale based on a metric`
    * Add 2 Rules:
        * Scale out
            * When Average CPU > 75%, Increase count by 1
        * Scale in
            * When Average CPU < 25%, Decrease count by 1
    * Instance limits
        * Minimum = 3
        * Maximum = 6
        * Default = 3
    * Save
  
    > This will create 3 Instances (Availability Zones) to the VMSS to support Chaos Experiment #1

## Chaos Studio

### Kubernetes and Availability Zone Outage

If you use a Managed User-assigned Identity when creating a Chaos Experiment with AKS and/or VMSS Availability Zones, you need to add the following Role Permissions

* Azure Kubernetes
    * In the AKS Resource, `contoso-traders-aks`{suffix}, navigate to `Access Control (IAM)`
    * Add `Role assignment`
    * For Job function roles, add `Azure Kubernetes Service Cluster Admin Role`
    * Assign access to `Managed Identity`, select Members, User-assigned managed identity
    * Choose `contoso-traders-aks`{suffix}`-agentpool`

    > See this link for more information on [Supported resource types and role assignments for Chaos Studio](https://learn.microsoft.com/en-us/azure/chaos-studio/chaos-studio-fault-providers)

* Availability Zone Auto Scale
    * At the Subscription or Resource Group level, navigate to `Access Control (IAM)`
    * Add `Role assignment`
    * For Job function roles, add `Web Plan Contributor`
    * Assign access to `Managed Identity`, select Members, User-assigned managed identity
    * Choose `contoso-traders-aks`{suffix}`-agentpool`

### AKS Chaos Mesh Stress Chaos

* When creating an experiment to stress the AKS cluster, you need to make sure the Target VMSS has the Stress Chaos capability selected.

    * In Chaos Studio, navigate to Targets
    * Select the AKS cluster (`contoso-traders-aks`{suffix})
    * Click `Manage Actions`
    * Under Service-direct capabilities
        * Select `AKS Chaos Mesh Stress Chaos`
    * Save
 