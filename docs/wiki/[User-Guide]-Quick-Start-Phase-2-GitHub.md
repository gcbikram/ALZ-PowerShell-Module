<!-- markdownlint-disable first-line-h1 -->
## 2.2.2 GitHub

You can choose to bootstrap with `bicep` or `terraform` skip to the relevant section below to do that.

Although you can just run `Deploy-Accelerator` and fill out the prompted inputs, we recommend creating an inputs file.  This will make it easier to run the accelerator more than once in order to refine your preferred configuration. In the following docs, we'll show that approach, but if you want to be prompted for inputs, just go ahead and run `Deploy-Accelerator` now.

### 2.2.2.1 GitHub with Bicep

1. Create a new folder on you local drive called `accelerator`.
1. Inside the accelerator create two folders called `config` and `output`. You'll store you input file inside config and the output folder will be the place that the accelerator stores files while it works.
1. Inside the `config` folder create a new file called `inputs.yaml`. You can use `json` if you prefer, but our examples here are `yaml`.

    ```pwsh
    # Windows
    New-Item -ItemType "file" c:\accelerator\config\inputs.yaml -Force
    New-Item -ItemType "directory" c:\accelerator\output

    # Linux/Mac
    New-Item -ItemType "file" /accelerator/config/inputs.yaml -Force
    New-Item -ItemType "directory" /accelerator/output
    ```

    ```plaintext
    📂accelerator
    ┣ 📂config
    ┃ ┗ 📜inputs.yaml
    ┗ 📂output
    ```

1. Open your `inputs.yaml` file in Visual Studio Code (or your preferred editor) and copy the content from [inputs-github-bicep-complete.yaml][example_powershell_inputs_github_bicep_complete] into that file.
1. Check through the file and update each input as required. It is mandatory to update items with placeholders surrounded by angle brackets `<>`:

    >NOTE: The following inputs can also be supplied via environment variables. This may be useful for sensitive values you don't wish to persist to a file. The `Env Var Prefix` denotes the prefix the environment variable should have. The environment variable is formatting is `<PREFIX>_<variable_name>`, e.g. `env:ALZ_iac_type = "bicep"` or `env:TF_VAR_github_personal_access_token = "*****..."`.

    | Input | Env Var Prefix | Placeholder | Description |
    | - | - | -- | --- |
    | `iac_type` | `ALZ` | `bicep` | This is the choice of `bicep` or `terraform`. Keep this as `bicep` for this example. |
    | `bootstrap_module_name` | `ALZ` | `alz_github` | This is the choice of Version Control System. Keep this as `alz_github` for this example. |
    | `starter_module_name` | `ALZ` | `complete` | This is the choice of [Starter Modules][wiki_starter_modules], which is the baseline configuration you want for your Azure landing zone. Keep this as `complete` for this example. |
    | `bootstrap_location` | `TF_VAR` | `<region>` | Replace `<region>` with the Azure region where you would like to deploy the bootstrap resources in Azure. This field expects the `name` of the region, such as `uksouth`. You can find a full list of names by running `az account list-locations -o table`. |
    | `starter_locations` | `TF_VAR` | `[<region-1>,<region-2>]` | Replace `<region-1>` and `<region-2>` with the Azure regions where you would like to deploy the starter module resources in Azure. This field expects the `name` of the regions in and array, such as `["uksouth", "ukwest"]`. You can find a full list of names by running `az account list-locations -o table`. |
    | `root_parent_management_group_id` | `TF_VAR` | `""` | This is the id of the management group that will be the parent of the management group structure created by the accelerator. If you are using the `Tenant Root Group` management group, you leave this as an empty string `""` or supply the tenant id. |
    | `subscription_id_management` | `TF_VAR` | `<management-subscription-id>` | Replace `<management-subscription-id>` with the id of the management subscription you created in the previous phase. |
    | `subscription_id_identity` | `TF_VAR` | `<identity-subscription-id>` | Replace `<identity-subscription-id>` with the id of the identity subscription you created in the previous phase. |
    | `subscription_id_connectivity` | `TF_VAR` | `<connectivity-subscription-id>` | Replace `<connectivity-subscription-id>` with the id of the connectivity subscription you created in the previous phase. |
    | `github_personal_access_token` | `TF_VAR` | `<token-1>` | Replace `<token-1>` with the `token-1` GitHub PAT you generated in a previous step. |
    | `github_runners_personal_access_token` | `TF_VAR` | `<token-2>` | Replace `<token-2>` with the `token-2` GitHub PAT you generated in the previous step specifically for the self-hosted runners. This only applies if you have `use_self_hosted_agents` set to `true`. You can set this to an empty string `""` if you are not using self-hosted runners. |
    | `github_organization_name` | `TF_VAR` | `<github-organization>` | Replace `<github-organization>` with the name of your GitHub organization. This is the section of the url after `github.com`. E.g. enter `my-org` for `https://github.com/my-org`. |
    | `use_separate_repository_for_templates` | `TF_VAR` | `true` | Determine whether to create a separate repository to store workflow templates as an extra layer of security. Set to `false` if you don't wish to secure your workflow templates by using a separate repository. This will default to `true`. |
    | `bootstrap_subscription_id` | `TF_VAR` | `""` | Enter the id of the subscription in which you would like to deploy the bootstrap resources in Azure. If left blank, the subscription you are connected to via `az login` will be used. In most cases this is the management subscription, but you can specifiy a separate subscription if you prefer. |
    | `service_name` | `TF_VAR` | `alz` | This is used to build up the names of your Azure and GitHub resources, for example `rg-<service_name>-mgmt-uksouth-001`. We recommend using `alz` for this. |
    | `environment_name` | `TF_VAR` | `mgmt` | This is used to build up the names of your Azure and GitHub resources, for example `rg-alz-<environment_name>-uksouth-001`. We recommend using `mgmt` for this. |
    | `postfix_number` | `TF_VAR` | `1` | This is used to build up the names of your Azure and GitHub resources, for example `rg-alz-mgmt-uksouth-<postfix_number>`. We recommend using `1` for this. |
    | `use_self_hosted_agents` | `TF_VAR` | `true` | This controls if you want to deploy self-hosted agents. This will default to `true`. |
    | `use_private_networking` | `TF_VAR` | `true` | This controls whether private networking is deployed for your self-hosted agents and storage account. This only applies if you have `use_self_hosted_agents` set to `true`. This defaults to `true`. |
    | `allow_storage_access_from_my_ip` | `TF_VAR` | `false` | This is not relevant to Bicep and we'll remove the need to specify it later, leave it set to `false`. |
    | `apply_approvers` | `TF_VAR` | `<email-address>` | This is a list of service principal names (SPN) of people you wish to be in the group that approves apply of the Azure landing zone module. This is an array of strings like `["abc@xyz.com", "def@xyz.com", "ghi@xyz.com"]`. You may need to check what the SPN is prior to filling this out as it can vary based on identity provider. Use empty array `[]` to disable approvals. Note if supplying via the user interface, use a comma separated string like `abc@xyz.com,def@xyz.com,ghi@xyz.com`. |
    | `create_branch_policies` | `TF_VAR` | `true` | This controls whether to create branch policies for the repository. This defaults to `true`. |

1. Now head over to your chosen starter module documentation to get the specific inputs for that module. Come back here when you are done.
    - [Bicep Complete Starter Module][wiki_starter_module_bicep_complete]
1. In your PowerShell Core (pwsh) terminal run the module:

    ```pwsh
    # Windows (adjust the paths to match your setup)
    Deploy-Accelerator -inputs "c:\accelerator\config\inputs.yaml" -output "c:\accelerator\output"

    # Linux/Mac (adjust the paths to match your setup)
    Deploy-Accelerator -inputs "/accelerator/config/inputs.yaml" -output "/accelerator/output"
    ```

1. You will see a Terraform `init` and `apply` happen.
1. There will be a pause after the `plan` phase you allow you to validate what is going to be deployed.
1. If you are happy with the plan, then type `yes` and hit enter.
1. The Terraform will `apply` and your environment will be bootstrapped.

### 2.2.2.2 GitHub with Terraform

1. Create a new folder on you local drive called `accelerator`.
1. Inside the accelerator create two folders called `config` and `output`. You'll store you input file inside config and the output folder will be the place that the accelerator stores files while it works.
1. Inside the `config` folder create a new file called `inputs.yaml`. You can use `json` if you prefer, but our examples here are `yaml`.

    ```pwsh
    # Windows
    New-Item -ItemType "file" c:\accelerator\config\inputs.yaml -Force
    New-Item -ItemType "directory" c:\accelerator\output

    # Linux/Mac
    New-Item -ItemType "file" /accelerator/config/inputs.yaml -Force
    New-Item -ItemType "directory" /accelerator/output
    ```

    ```plaintext
    📂accelerator
    ┣ 📂config
    ┃ ┗ 📜inputs.yaml
    ┗ 📂output
    ```

1. Open your `inputs.yaml` file in Visual Studio Code (or your preferred editor) and copy the content from the relevant input file for your chosen starter module:
    1. Complete Multi Region - [inputs-github-terraform-complete-multi-region.yaml][example_powershell_inputs_github_terraform_complete_multi_region]
    1. Sovereign Landing Zone - [inputs-github-terraform-sovereign-landing-zone.yaml][example_powershell_inputs_github_terraform_sovereign_landing_zone]
    1. Basic - [inputs-github-terraform-basic.yaml][example_powershell_inputs_github_terraform_basic]
    1. Hub Networking - [inputs-github-terraform-hubnetworking.yaml][example_powershell_inputs_github_terraform_hubnetworking]
    1. Complete - [inputs-github-terraform-complete.yaml][example_powershell_inputs_github_terraform_complete]
1. Check through the file and update each input as required. It is mandatory to update items with placeholders surrounded by angle brackets `<>`:

    >NOTE: The following inputs can also be supplied via environment variables. This may be useful for sensitive values you don't wish to persist to a file. The `Env Var Prefix` denotes the prefix the environment variable should have. The environment variable is formatting is `<PREFIX>_<variable_name>`, e.g. `env:ALZ_iac_type = "terraform"` or `env:TF_VAR_github_personal_access_token = "*****..."`.

    | Input | Env Var Prefix | Placeholder | Description |
    | - | - | -- | --- |
    | `iac_type` | `ALZ` | `terraform` | This is the choice of `bicep` or `terraform`. Keep this as `terraform` for this example. |
    | `bootstrap_module_name` | `ALZ` | `alz_github` | This is the choice of Version Control System. Keep this as `alz_github` for this example. |
    | `starter_module_name` | `ALZ` | `complete_multi_region` | This is the choice of [Starter Modules][wiki_starter_modules], which is the baseline configuration you want for your Azure landing zone. Choose `complete_multi_region`, `complete`, `hubnetworking` or `basic` for this example. |
    | `bootstrap_location` | `TF_VAR` | `<region>` | Replace `<region>` with the Azure region where you would like to deploy the bootstrap resources in Azure. This field expects the `name` of the region, such as `uksouth`. You can find a full list of names by running `az account list-locations -o table`. |
    | `starter_locations` | `TF_VAR` | `[<region-1>,<region-2>]` | Replace `<region-1>` and `<region-2>` with the Azure regions where you would like to deploy the starter module resources in Azure. This field expects the `name` of the regions in and array, such as `["uksouth", "ukwest"]`. You can find a full list of names by running `az account list-locations -o table`. |
    | `root_parent_management_group_id` | `TF_VAR` | `""` | This is the id of the management group that will be the parent of the management group structure created by the accelerator. If you are using the `Tenant Root Group` management group, you leave this as an empty string `""` or supply the tenant id. |
    | `subscription_id_management` | `TF_VAR` | `<management-subscription-id>` | Replace `<management-subscription-id>` with the id of the management subscription you created in the previous phase. |
    | `subscription_id_identity` | `TF_VAR` | `<identity-subscription-id>` | Replace `<identity-subscription-id>` with the id of the identity subscription you created in the previous phase. |
    | `subscription_id_connectivity` | `TF_VAR` | `<connectivity-subscription-id>` | Replace `<connectivity-subscription-id>` with the id of the connectivity subscription you created in the previous phase. |
    | `github_personal_access_token` | `TF_VAR` | `<token-1>` | Replace `<token-1>` with the `token-1` GitHub PAT you generated in a previous step. |
    | `github_runners_personal_access_token` | `TF_VAR` | `<token-2>` | Replace `<token-2>` with the `token-2` GitHub PAT you generated in the previous step specifically for the self-hosted runners. This only applies if you have `use_self_hosted_agents` set to `true`. You can set this to an empty string `""` if you are not using self-hosted runners. |
    | `github_organization_name` | `TF_VAR` | `<github-organization>` | Replace `<github-organization>` with the name of your GitHub organization. This is the section of the url after `github.com`. E.g. enter `my-org` for `https://github.com/my-org`. |
    | `use_separate_repository_for_templates` | `TF_VAR` | `true` | Determine whether to create a separate repository to store workflow templates as an extra layer of security. Set to `false` if you don't wish to secure your workflow templates by using a separate repository. This will default to `true`. |
    | `bootstrap_subscription_id` | `TF_VAR` | `""` | Enter the id of the subscription in which you would like to deploy the bootstrap resources in Azure. If left blank, the subscription you are connected to via `az login` will be used. In most cases this is the management subscription, but you can specifiy a separate subscription if you prefer. |
    | `service_name` | `TF_VAR` | `alz` | This is used to build up the names of your Azure and GitHub resources, for example `rg-<service_name>-mgmt-uksouth-001`. We recommend using `alz` for this. |
    | `environment_name` | `TF_VAR` | `mgmt` | This is used to build up the names of your Azure and GitHub resources, for example `rg-alz-<environment_name>-uksouth-001`. We recommend using `mgmt` for this. |
    | `postfix_number` | `TF_VAR` | `1` | This is used to build up the names of your Azure and GitHub resources, for example `rg-alz-mgmt-uksouth-<postfix_number>`. We recommend using `1` for this. |
    | `use_self_hosted_agents` | `TF_VAR` | `true` | This controls if you want to deploy self-hosted agents. This will default to `true`. |
    | `use_private_networking` | `TF_VAR` | `true` | This controls whether private networking is deployed for your self-hosted agents and storage account. This only applies if you have `use_self_hosted_agents` set to `true`. This defaults to `true`. |
    | `allow_storage_access_from_my_ip` | `TF_VAR` | `false` | This controls whether to allow access to the storage account from your IP address. This is only needed for trouble shooting. This only applies if you have `use_private_networking` set to `true`. This defaults to `false`. |
    | `apply_approvers` | `TF_VAR` | `<email-address>` | This is a list of service principal names (SPN) of people you wish to be in the group that approves apply of the Azure landing zone module. This is an array of strings like `["abc@xyz.com", "def@xyz.com", "ghi@xyz.com"]`. You may need to check what the SPN is prior to filling this out as it can vary based on identity provider. Use empty array `[]` to disable approvals. Note if supplying via the user interface, use a comma separated string like `abc@xyz.com,def@xyz.com,ghi@xyz.com`. |
    | `create_branch_policies` | `TF_VAR` | `true` | This controls whether to create branch policies for the repository. This defaults to `true`. |
    | `architecture_definition_name` | `TF_VAR` | N/A | This is the name of the architecture definition to use when applying the ALZ archetypes via the architecture definition template. This is only relevant to some starter modules, such as the `sovereign_landing_zone` starter module. This defaults to `null`. |

1. Now head over to your chosen starter module documentation to get the specific inputs for that module. Come back here when you are done.
    - [Terraform Complete Multi Region Starter Module][wiki_starter_module_terraform_complete_multi_region]: Management groups, policies, Multi Region hub networking with fully custom configuration.
    - [Terraform Sovereign Landing Zone Starter Module][wiki_starter_module_terraform_sovereign_landing_zone]: Management groups, policies, hub networking for the Sovereign Landing Zone.
    - [Terraform Basic Starter Module][wiki_starter_module_terraform_basic]: Management groups and policies.
    - [Terraform Hub Networking Starter Module][wiki_starter_module_terraform_hubnetworking]: Management groups, policies and hub networking.
    - [Terraform Complete Starter Module][wiki_starter_module_terraform_complete]: Management groups, policies, hub networking with fully custom configuration.

1. In your PowerShell Core (pwsh) terminal run the module:

    >NOTE: The following examples include 2 input files. This is the recommended approach for the `complete_multi_region` starter module. However, all inputs can be combined into a single file if desired and other starter modules only require a single input file.

    ```pwsh
    # Windows (adjust the paths to match your setup)
    Deploy-Accelerator -inputs "c:\accelerator\config\inputs.yaml", "c:\accelerator\config\networking.yaml" -output "c:\accelerator\output"
    ```

    ```pwsh
    # Linux/Mac (adjust the paths to match your setup)
    Deploy-Accelerator -inputs "/accelerator/config/inputs.yaml", "/accelerator/config/networking.yaml" -output "/accelerator/output"
    ```

1. You will see a Terraform `init` and `apply` happen.
1. There will be a pause after the `plan` phase you allow you to validate what is going to be deployed.
1. If you are happy with the plan, then type `yes` and hit enter.
1. The Terraform will `apply` and your environment will be bootstrapped.

## Next Steps

Now head to [Phase 3][wiki_quick_start_phase_3].

 [//]: # (************************)
 [//]: # (INSERT LINK LABELS BELOW)
 [//]: # (************************)

[wiki_starter_modules]:                             %5BUser-Guide%5D-Starter-Modules "Wiki - Starter Modules"
[wiki_starter_module_bicep_complete]:               %5BUser-Guide%5D-Starter-Module-Bicep-Complete "Wiki - Starter Modules - Bicep Complete"
[wiki_starter_module_terraform_basic]:              %5BUser-Guide%5D-Starter-Module-Terraform-Basic "Wiki - Starter Modules - Terraform Basic"
[wiki_starter_module_terraform_hubnetworking]:      %5BUser-Guide%5D-Starter-Module-Terraform-HubNetworking "Wiki - Start Modules - Terraform Hub Networking"
[wiki_starter_module_terraform_complete]:           %5BUser-Guide%5D-Starter-Module-Terraform-Complete "Wiki - Starter Modules - Terraform Complete"
[wiki_starter_module_terraform_complete_multi_region]:           %5BUser-Guide%5D-Starter-Module-Terraform-Complete-Multi-Region "Wiki - Starter Modules - Terraform Complete Multi Region"
[wiki_starter_module_terraform_sovereign_landing_zone]:           %5BUser-Guide%5D-Starter-Module-Terraform-Sovereign-Landing-Zone "Wiki - Starter Modules - Terraform Sovereign Landing Zone"
[wiki_quick_start_phase_3]:                         %5BUser-Guide%5D-Quick-Start-Phase-3 "Wiki - Quick Start - Phase 3"
[example_powershell_inputs_github_bicep_complete]:     examples/powershell-inputs/inputs-github-bicep-complete.yaml "Example - PowerShell Inputs - GitHub - Bicep - Complete"
[example_powershell_inputs_github_terraform_basic]:     examples/powershell-inputs/inputs-github-terraform-basic.yaml "Example - PowerShell Inputs - GitHub - Terraform - Basic"
[example_powershell_inputs_github_terraform_hubnetworking]:     examples/powershell-inputs/inputs-github-terraform-hubnetworking.yaml "Example - PowerShell Inputs - GitHub - Terraform - Hub Networking"
[example_powershell_inputs_github_terraform_complete]:     examples/powershell-inputs/inputs-github-terraform-complete.yaml "Example - PowerShell Inputs - GitHub - Terraform - Complete"
[example_powershell_inputs_github_terraform_complete_multi_region]:     examples/powershell-inputs/inputs-github-terraform-complete-multi-region.yaml "Example - PowerShell Inputs - GitHub - Terraform - Complete Multi Region"
[example_powershell_inputs_github_terraform_sovereign_landing_zone]:     examples/powershell-inputs/inputs-github-terraform-sovereign-landing-zone.yaml "Example - PowerShell Inputs - GitHub - Terraform - Sovereign Landing Zone"
