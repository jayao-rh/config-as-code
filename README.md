# Ansible Automation Platform - Configuration as Code (CaC)

This repository serves as a template and comprehensive example for deploying and managing **Red Hat Ansible Automation Platform (AAP) 2.5+** using Configuration as Code. It leverages the Red Hat Community of Practice (CoP) collections to provide a multi-environment (Dev/Test/Prod) framework for managing Controller, Automation Hub, and EDA configurations.

---

## 📂 Repository Structure

The project is organized to support global configurations (`all`) and environment-specific overrides.

```text
.
├── changelogs/               # History of configuration changes
├── collections/              # requirements.yml for necessary Ansible collections
├── config/                   # YAML files defining AAP objects (Orgs, Users, Job Templates, etc.)
├── inventory/                # Inventory files defining the AAP infrastructure nodes
├── playbooks/                # Playbooks to trigger installation and configuration
│   ├── aap_config.yml        # Main playbook for pushing configuration to AAP
│   ├── install_aap.yml       # Playbook for automated AAP installation
│   └── install_configure.yml # Combined installer and config playbook
├── group_vars/               # Variables categorized by group (All, Dev, Test, Prod)
│   ├── all/                  # Global settings applied to all environments
│   ├── dev/                  # Overrides specific to Development
│   ├── test/                 # Overrides specific to Testing
│   └── prod/                 # Overrides specific to Production
├── .ansible-lint             # Rules for ansible-lint code quality
├── .yamllint                 # Rules for YAML syntax validation
└── README.md                 # Project documentation


📄 File Descriptions**

🔧 Configuration Files (/config)
These files contain the desired state of your AAP components. Instead of clicking in the UI, you define objects here.

🔧 config/all directory
These files act as the "source of truth" for your global baseline setup.

🎮 Automation Controller Configurations

controller_credential_input_sources.yml: Configures external secret managers (HashiCorp Vault, CyberArk, etc.).

controller_credential_types.yml: Defines custom metadata and input fields for non-native credentials.

controller_credentials.yml: Primary list of auth for SCM, Cloud, and SSH.

controller_execution_environments.yml: Registers container images (EEs) providing the Ansible runtime.

controller_groups.yml: Defines logical groupings of hosts within your inventories.

controller_hosts.yml: Global list of managed nodes and host-level variables.

controller_instance_groups.yml: Groups execution nodes to dedicate hardware to certain tasks.

controller_inventories.yml: Creates top-level inventory objects.

controller_inventory_sources.yml: Configures dynamic syncs (AWS, Azure, VMware, etc.).

controller_job_templates.yml: Defines the "How-To" for running a playbook.

controller_labels.yml: Metadata tags for filtering job templates.

controller_notifications.yml: Communication channels (Slack, Email, PagerDuty).

controller_projects.yml: Links Controller to Git repositories where code is stored.

controller_roles.yml: Manages RBAC permission assignments.

controller_schedule.yml: Automates execution at specific times.

controller_settings.yml: Global system parameters (LDAP, logging, etc.).

controller_workflows.yml: Chains multiple job templates into visual flows.

⚡ Event-Driven Ansible (EDA)

eda_credentials.yml: Auth for event streams (Kafka, Webhooks, etc.).

eda_decision_environments.yml: Defines containers that run EDA rulebooks.

eda_projects.yml: Links Git repos containing rulebooks.

eda_rulebook_activations.yml: Manages active instances listening for events.

📦 Private Automation Hub

hub_collection_namespaces.yml: Defines organizational namespaces for internal collections.

hub_collection_publish.yml: Settings for publishing and approving collections.

hub_collection_repositories.yml: Configures local collection repositories.

hub_ee_images.yml: Manages container images in the Hub registry.

hub_ee_registries.yml: Links to external registries for mirroring.

hub_ee_repositories.yml: Defines image repositories within Hub.

🚪 AAP Gateway & General Utilities

gateway_applications.yml: Defines OAuth2 applications for integrations.

gateway_organizations.yml: Top-level multi-tenancy structure for the Gateway.

gateway_teams.yml / gateway_users.yml: Manages global identity across the platform.

ee_list.yml: Master list used to build/manage multiple Execution Environments.

🏗️ Environment-Specific Configuration (Prod & Test)
The files under config/prod/, config/test/ or config/<env>/ allow you to customize settings for each stage of your SDLC.

📂 Folder Logic: How it Works
When you run the configuration playbooks, the infra.aap_configuration collection uses a "Merge" strategy:

Load all: Loads global standards and baseline objects from config/all/.

Apply env: Layers files from config/prod/ or config/test/ on top.

Conflict Resolution: If an object exists in both, the environment-specific version takes precedence.

🖥️ Inventory Files (/inventory)
inventory_dev.yml: Hostnames/IPs for the Development cluster.

inventory_prod.yml: Hostnames/IPs for the Production infrastructure.

🚀 Playbooks (/playbooks)
install_aap.yml: Automated platform installation using infra.aap_utilities.

aap_config.yml: Syncs /config files to the AAP Controller/Hub UI.

install_configure.yml: "Day 0" playbook performing installation and configuration in one run.

🚀 Usage and Execution
The logic relies on the env variable to determine which configuration files and variables to load.

🔑 Prerequisites
Decrypt Vault: Ensure vault.yml is populated with required tokens.

Update Inventories: Replace HERE placeholders in inventory files with actual hostnames.

🛠️ Scenario 1: Provisioning a New Environment (Day 0)
For Development:



ansible-playbook -i inventory/inventory_dev.yml \
                 playbooks/install_configure.yml \
                 --ask-vault-pass \
                 -e "env=dev"
For Production:



ansible-playbook -i inventory/inventory_prod.yml \
                 playbooks/install_configure.yml \
                 --ask-vault-pass \
                 -e "env=prod"
🔄 Scenario 2: Syncing Configuration (Day 2 Operations)
Sync Development:



ansible-playbook -i inventory/inventory_dev.yml \
                 -l dev \
                 playbooks/aap_config.yml \
                 --ask-vault-pass \
                 -e "env=dev"
Sync Production:



ansible-playbook -i inventory/inventory_prod.yml \
                 -l prod \
                 playbooks/aap_config.yml \
                 --ask-vault-pass \
                 -e "env=prod"
🧪 Scenario 3: Testing Configuration (Dry Run)


ansible-playbook -i inventory/inventory_prod.yml \
                 -l prod \
                 playbooks/aap_config.yml \
                 --ask-vault-pass \
                 -e "env=prod" \
                 --check
📈 Execution Logic
When you run the commands above, the automation follows this precedence:

Load Global Defaults: Reads every file in config/all/.

Apply Environment Overrides: Layers files from config/{{ env }}/.

Variable Merging: group_vars/all/ is loaded, followed by group_vars/{{ env }}/.
