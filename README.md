# Ansible playbook to deploy ES docker-compose stack onto EC.


## Installation
  1. Clone repo
  2. Contact OPS Team to get `.vault` file - without this file you won't be able to use this playbook
  3. Rename `ansible.cfg.example` to `ansible.cfg` and alter:
      1. `ask_pass`      - set to `true` if you don't have key based login onto server, where things get deployed
      2. `cow_selection` - comment out this line if you don't have `cow` utility installed
      3. `library`       - to point to Custom Ansible modules you downloaded - without this one you won't be able to deploy Kibana Spaces


## Available tags
1.  `apm-server`                    - use this tag to deploy files & configurations for APM Server
2.  `elastic`                       - use this tag to deploy files & configurations for Elasticsearch
3.  `elastic-templates`             - use this tag to insert ILM Policies, Index Templates, Security Role Mappings & Security Roles into Elasticsearch
4.  `elasticsearch-full-upgrade`    - use this tag to initiate full upgrade of Elasticsearch cluster (restarts all Elasticsearch docker containers at once)
5.  `elasticsearch-rolling-upgrade` - use this tag to initiate rolling upgrade of Elasticsearch cluster (restarts Elasticsearch docker containers one by one, starting from data nodes)
6.  `elk-rolling-upgrade`           - use this tag to initiate rolling upgrade of Logstash/Kibana/Beats/APM Server (restarts mentioned docker containers one by one)
7.  `filebeat`                      - use this tag to deploy files & configurations for Filebeat
8.  `kibana`                        - use this tag to deploy files & configurations for Kibana
9.  `logstash`                      - use this tag to deploy files & configurations for Logstash
10. `metricbeat`                    - use this tag to deploy files & configurations for Metricbeat
11. `ssl`                           - use this tag to deploy shared SSL certificates (effectively all applications of ELK stack require those)


## Available inventories
1. `production/` - this inventory has 2 files and each contains nodes specific to each PROD cluster
    1. `tinsley_hosts.yml` - use this inventory file to deploy into production Tinsley environment
    2. `reigate_hosts.yml` - use this inventory file to deploy into production Reigate environment
2. `reference/`            - use this inventory to deploy things into reference/test Reigate environment


## Node specific attributes
Due to the size of this playbook (it's really massive) there are some node specific attributes you may want to tweak before deployment
1. Each inventory has
    1. `group_vars/` directory, where you can specify role shared and specific attributes, ie
        1. `all`           - folder contains attributes that would be applied to all roles
        2. `elastic_nodes` - folder contains attributes specific to `elastic` role and such would be applied to all nodes that are part of this role
    2. `*hosts.yml` - this file has a list of all nodes together with any NODE specific attributes
2. `all` has following attributes you may wish to alter before the deployment
    1. `elk_version: 7.17.4` - update this one when you are upgrading ELK version
    2. `bump_version: false` - set this to `true` if you only want to update the ELK version (really useful when there are no changes to configurations and just need to update ELK)
3. `*hosts.yml` also has some attributes one may want to alter prior to deployment
    1. Under `kibana_nodes` section
        1. `install_kibana:       true`  - set this to `false` if you do not need to update configuration files
        2. `deploy_spaces:        false` - set this to `true`  if you need to deploy Kibana Spaces
        3. `import_saved_objects: false` - set this to `true`  if you need to Import Saved Objects into Kibana
    2. Under `elk_rolling_upgrade` section there are 6 nodes and each has at least one of below
        1. `apm_server_rolling_upgrade: false` - set this to `true` if you need to deploy APM Server docker container
        2. `filebeat_rolling_upgrade:   false` - set this to `true` if you need to deploy Filebeat docker container
        3. `logstash_rolling_upgrade:   false` - set this to `true` if you need to deploy Logstash docker container
        4. `kibana_rolling_upgrade:     false` - set this to `true` if you need to deploy Kibana docker container
        5. `metricbeat_rolling_upgrade: false` - set this to `true` if you need to deploy Metricbeat docker container 
    3. Under `elastic_templates` section
        1. `import_ilm_policies:           false` - set this to `true` if you need to Import Index Life Management Policy files into Elasticsearch
        2. `import_index_templates:        false` - set this to `true` if you need to Import Index Templates files into Elasticsearch
        3. `import_security_role_mappings: false` - set this to `true` if you need to Import Security Role Mapping files into Elasticsearch
        4. `import_security_roles:         false` - set this to `true` if you need to Import Security Role files into Elasticsearch
        5. `import_watchers:               false` - set this to `true` if you need to Import Watchers files into Elasticsearch


## Usage:
0. If you plan to deploy SSL certificates, make sure to place them into **SSL** role directory under folder `files/ssl_<environment>`, where **<environment>** is either
    1. **production** - production environment
    2. **reference**  - reference (aka test) environment
1. You must use tags, otherwise everything gets deployed at the same time at it may get really messy (but shouldn't break anything, I hope)
2. Use following command selecting required tags and inventory
```
ansible-playbook -i inventories/<INVENTORY> site.yml --tags='<TAGS>' -v --vault-password-file .vault

For example, to deploy Elasticsearch and initiate rolling upgrade of it in Tinsley production environment run
ansible-playbook -i inventories/production/tinsley_hosts.yml site.yml --tags='elastic,elasticsearch-rolling-upgrade' -v --vault-password-file .vault
```


## Notes
  1. Be aware that before this playbook could be run against new EC server, you would need to run [this playbook first][ec_setup_playbook]
  2. To do a **dry-run** you need to add **--check** to the deployment command, ie
      1. `ansible-playbook -i inventories/production site.yml --tags='elastic' -v --vault-password-file .vault --check`
  3. Because Ansible Vault is used, some of the .yml files are locked down - those containing credentials:
      1. To edit them from CLI, run `EDITOR=vim ansible-vault edit <PATH_TO_SECRETS.YML> --vault-password-file .vault`
      2. To edit them from within Vim, when encrypted file is open
        ```
        !ansible-vault decrypt --vault-password-file .vault % // to make change
        !ansible-vault encrypt --vault-password-file .vault % // encrypt file back after changes are made
        ```
  4. Private key should be downloaded from [here][ssl-storage] and placed in the file named **elk.key** in a role which is to be deployed (i.e **roles/elasticsearch/**) under path **files/ssl_production/**
  5. When launching completely new ES cluster, follow steps:
      1. Deploy cluster with all security features in **elasticsearch.yml** set to **false**, ie: http ssl, transport ssl and etc.
      2. Start up the cluster
      3. Once the whole cluster is up, submit the license key, which unlocks security features, ie `curl -XPUT 'http://es_hostname:9200/_xpack/license' -H "Content-Type: application/json" -d @license.json`
      4. After license key is submitted to cluster, enable security features in **elasticsearch.yml** on all nodes
      5. Restart cluster's nodes one-by-one starting with the master enabled nodes that aren't current masters
  6. Kibana role uses custom Ansible modules which must be `git clone`-ed [from here][custom-ansible-modules] somewhere onto the machine (**from where deployment is run**) and then that path needs to be assigned to `library` variable in `ansible.cfg` file


[ilm_docs]: https://www.elastic.co/guide/en/elasticsearch/reference/6.7/index-lifecycle-management.html
[ssl-storage]: https://wiki.nat.bt.com/radius/secrets/ssl-certs/elasticsearch
[ec_setup_playbook]: https://agile.nat.bt.com/gitlab/RADIUS/ansible-playbooks/ec-init-setup
[custom-ansible-modules]: https://gitlab.agile.nat.bt.com/RADIUS/ansible-playbooks/custom-ansible-modules


# Deployment-specific notes

## Logstash deployment (as of 26/05/2023)
1. Add pipeline config template into `/roles/logstash/templates/pipelines` (check existing templates to pick up required variables, ie kafka brokers, Elasticsearch credentials)
2. Reference this template within `inventories/<environment>/*hosts.yml` under `logstash_nodes >> logstash_pipelines_list` (there also examples, so should be easy)
3. Depending on the environment & data center, add pipeline details to `pipelines_<data_center>_<environment>_cluster.yml`, following the format of the other pipelines
4. Copy the `roles/logstash/files/logstash-docker/config_example` directory into `roles/logstash/files/logstash_docker/config` and remove .example from the end of all the files in this new directory
5. Run the ansible playbook as described above. Note, for the pipeline to take effect the tag `elk-rolling-upgrade` must also be used in order to restart logstash by rebuilding the docker container. Read the details on this in the readme above; only the `logstash_rolling_upgrade` property needs to be set to true within the reference cluster's `hosts.yml` file
