# aapybr - the journey begins
Helping go from AAP 2.4 or 2.5 to 2.6 for all deployment types

This is focused on RPM and containerized hosts. It does not perform the actual upgrade — it collects inventory, verifies environment, performs backups (DB and filesystem), lists container images, finds custom plugins, checks for EDA presence, and fetches artifacts to the control node. It is intended to be run from your Ansible control host, with become: true enabled where needed.

Security note: This playbook runs potentially sensitive commands (DB dumps, tar of config dirs). Use Ansible Vault for secrets (DB password), restrict output storage to secure backup locations, and run in accordance with bank policies.

# How to run the playbook (example)
Create an inventory with controller hosts and DB host groups.
Create group_vars/all.yml with your values and place secrets in Ansible Vault (e.g., pg.pg_password).
From the control host run:

ansible-playbook -i inventory.yml playbooks/preupgrade_checks.yml
if using vault
ansible-playbook -i inventory.yml playbooks/preupgrade_checks.yml --ask-vault-pass

#Additional Informaiton
The playbook intentionally does not perform destructive operations (except creating tar and pg_dump). It will not delete any DBs or modify system repos.

All passwords and sensitive values should be stored in Ansible Vault (e.g., ansible-vault encrypt group_vars/all.yml) or supplied at runtime.

This playbook aims to be generic and safe across diverse environments. Adjust archive_dirs, pg parameters, and container.runtime for your environment.

You should run the playbook on a staging clone first to validate that pg_dump runs and that archives are created and fetched successfully.

For extremely large DBs, pg_dump might be slow; consider physical snapshots or pg_basebackup for large/replicated deployments. The playbook can be extended to run pg_basebackup when required.

The playbook includes EDA detection heuristics (rpm names, container names). EDA detection may require environment-specific amendments (custom package names or container naming conventions).

If your controllers are behind an HTTP proxy, ensure the control host environment variables are set or pass proxy vars to tasks where required.

For environments using OS images without subscription-manager (air-gapped), the repo checks will report subscription-manager not present — ensure the internal mirror is used instead and validate accessible repo files.
