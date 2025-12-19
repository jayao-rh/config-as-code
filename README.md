**Ansible Automation Platform - Configuration as Code (CaC)**
This repository serves as a template and comprehensive example for deploying and managing Red Hat Ansible Automation Platform (AAP) 2.5+ using Configuration as Code. It leverages the Red Hat Community of Practice (CoP) collections to provide a multi-environment (Dev/Test/Prod) framework for managing Controller, Automation Hub, and EDA configurations.

📂 Repository Structure
The project is organized to support global configurations (all) and environment-specific overrides.

Plaintext

.
├── changelogs/             # History of configuration changes
├── collections/            # requirements.yml for necessary Ansible collections
├── config/                 # YAML files defining AAP objects (Orgs, Users, Job Templates, etc.)
├── inventory/              # Inventory files defining the AAP infrastructure nodes
├── playbooks/              # Playbooks to trigger installation and configuration
│   ├── aap_config.yml      # Main playbook for pushing configuration to AAP
│   ├── install_aap.yml     # Playbook for automated AAP installation
│   └── install_configure.yml # Combined installer and config playbook
├── group_vars/             # Variables categorized by group (All, Dev, Test, Prod)
│   ├── all/                # Global settings applied to all environments
│   ├── dev/                # Overrides specific to Development
│   ├── test/               # Overrides specific to Testing
│   └── prod/               # Overrides specific to Production
├── .ansible-lint           # Rules for ansible-lint code quality
├── .yamllint               # Rules for YAML syntax validation
└── README.md               # Project documentation
📄 File Descriptions
🔧 Configuration Files (/config)
These files contain the desired state of your AAP components. Instead of clicking in the UI, you define objects here:

🔧 **config/all directory**: These files act as the "source of truth" for your Ansible Automation Platform (AAP) 2.5 setup.

    🎮 Automation Controller Configurations
    These files define the core automation objects within the Controller (formerly Tower).
    
    controller_credential_input_sources.yml: Configures external secret managers (like HashiCorp Vault, CyberArk, or AWS Secrets Manager) so Controller can pull credentials dynamically.
    
    controller_credential_types.yml: Defines custom metadata and input fields for unique credentials that aren't natively supported (e.g., custom API tokens).
    
    controller_credentials.yml: The primary list of credentials (SSH keys, Cloud tokens, SCM tokens) used to access managed nodes and external services.
    
    controller_execution_environments.yml: Registers the container images (EEs) that provide the Ansible runtime and specific collections/libraries needed for jobs.
    
    controller_groups.yml: Defines logical groupings of hosts within your inventories for targeted automation.
    
    controller_hosts.yml: The global list of managed nodes (IPs or FQDNs) and their specific host-level variables.
    
    controller_instance_groups.yml: Groups Controller execution nodes together, allowing you to dedicate specific hardware to certain tasks or organizations.
    
    controller_inventories.yml: Creates the top-level inventory objects that act as containers for your hosts and groups.
    
    controller_inventory_sources.yml: Configures "Dynamic Inventory" syncs from sources like AWS, Azure, VMware, or even a different Controller instance.
    
    controller_job_templates.yml: Defines the "How-To" for running a playbook: links a Project, Inventory, and Credential into an executable task.
    
    controller_labels.yml: Creates tags/labels used to organize and filter job templates and other objects across the UI.
    
    controller_notifications.yml: Sets up the communication channels (Slack, Email, PagerDuty, Webhooks) that receive status updates from jobs.
    
    controller_projects.yml: Links the Controller to your Git repositories (GitHub/GitLab) where the actual Ansible code is stored.
    
    controller_roles.yml: Manages RBAC (Role-Based Access Control) by assigning permissions to users or teams for specific objects.
    
    controller_schedule.yml: Automates the execution of job templates or project syncs at specific times (e.g., every Monday at 2 AM).
    
    controller_settings.yml: Controls global Controller parameters like LDAP integration, logging levels, and job isolation settings.
    
    controller_workflows.yml: Chinas multiple job templates together into a single visual flow with success/failure logic (DAGs).
    
    ⚡ Event-Driven Ansible (EDA)
    These files manage the newer Event-Driven capabilities of AAP.
    
    eda_credentials.yml: Stores credentials specifically needed for EDA to listen to event streams (e.g., Kafka auth or Webhook tokens).
    
    eda_decision_environments.yml: Similar to EEs, these define the containers that run the EDA rulebooks.
    
    eda_projects.yml: Links Git repositories containing the .yml rulebooks that define "If event X happens, run Job Y."
    
    eda_rulebook_activations.yml: Manages the active, running instances of rulebooks that are currently listening for events.
    
    📦 Private Automation Hub
    These files configure the local repository for Ansible Collections and Execution Environments.
    
    hub_collection_namespaces.yml: Defines the organizational namespaces (e.g., my_company.my_team) for internal collections.
    
    hub_collection_publish.yml: Manages settings for publishing and approving new collection versions.
    
    hub_collection_repositories.yml: Configures the local collection repositories and their distribution policies.
    
    hub_ee_images.yml: Manages the specific container images stored within the Hub's registry.
    
    hub_ee_registries.yml: Links Hub to external registries (like registry.redhat.io) for mirroring images.
    
    hub_ee_repositories.yml: Defines the actual image repositories within Hub where EEs are organized.
    
    🚪 AAP Gateway & General Utilities
    These files manage the unified interface (Gateway) and shared execution resources.
    
    gateway_applications.yml: Defines OAuth2 applications for external integrations with the AAP Gateway.
    
    gateway_organizations.yml: Top-level multi-tenancy structure that spans across Controller and Hub through the unified Gateway.
    
    gateway_teams.yml / gateway_users.yml: Manages global users and teams across the entire platform.
    
    ee_list.yml: A centralized master list used by automation scripts to build or manage multiple Execution Environments at once.

**🏗️ Environment-Specific Configuration (Prod & Test)**
The following files under config/prod/, config/test/ or config/<any environment> allow you to customize settings for each stage of your SDLC:

  
  📂 Folder Logic: How it Works
  When you run the configuration playbooks, the infra.aap_configuration collection uses a "Merge" strategy:
  
  Load all: It first loads the global standards and baseline objects from config/all/.
  
  Apply env: It then layers the files from config/prod/ or config/test/ on top.
  
  Conflict Resolution: If an object (like a specific Job Template name) exists in both, the version in the environment folder takes precedence. Otherwise, the lists are combined.


🖥️ **Inventory Files (/inventory)**
inventory_dev.yml: Defines the hostnames/IPs for the Development AAP cluster (Controller, Hub, Database, etc.).

inventory_prod.yml: Defines the Production infrastructure.

🚀 **Playbooks (/playbooks)**
install_aap.yml: Uses the infra.aap_utilities roles to automate the installation of the platform using a Red Hat manifest.

aap_config.yml: The "Sync" playbook. It reads the files in /config and ensures the AAP Controller/Hub UI matches the code.

install_configure.yml: A "Day 0" playbook that performs both the installation and the initial configuration in one run.


**🚀 Usage and Execution**
This repository is designed to be executed using the ansible-playbook command. The logic relies on the env variable to determine which configuration files and variables to load.

🔑 Prerequisites before Running
Decrypt your Vault: Ensure your vault.yml is populated with the required tokens and passwords.

Update Inventories: Ensure the HERE placeholders in inventory/inventory_dev.yml and inventory_prod.yml are replaced with your actual AAP hostnames.

🛠️ Scenario 1: Provisioning a New Environment (Day 0)
If you are installing AAP for the first time and want to apply the configuration immediately after the install:

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
Use this when you have modified files in the config/ folder (e.g., added a new Job Template or User) and want to push those changes to the existing AAP instance.

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
It is highly recommended to run a "Check Mode" (Dry Run) before pushing changes to Production to see what Ansible would change without actually making the changes.


ansible-playbook -i inventory/inventory_prod.yml \
                 -l prod \
                 playbooks/aap_config.yml \
                 --ask-vault-pass \
                 -e "env=prod" \
                 --check
📈 Execution Logic
When you run the commands above, the automation follows this precedence:

Load Global Defaults: It reads every file in config/all/.

Load Environment Overrides: It looks in config/{{ env }}/. If a file exists there, it will append to or override the global list.
