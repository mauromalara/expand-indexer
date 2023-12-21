## Scenario
1. **Wazuh version** -> 4.6.0
2. **Server 1** -> Will contain the W. Manager (master node)
3. **Server 2** -> Will contain the W. Manager (worker node)
4. **Server 3** -> Will contain the W. Dashboard and the W. Indexer (main node)
5. **Server 4** -> Will contain the W. Indexer (*extra node to be added*)

***

> [!IMPORTANT]
> Pay attention to the naming of the components.

***

> [!NOTE]
> This guide covers the installation, but if you've already installed the Wazuh components you can skip it.

***


Parts:
1. [Install the central components](#install-the-central-components-master-worker-dashboard-and-indexer)
2. [Add a new node to the Wazuh Indexer installation](#expand-the-installation-by-adding-a-new-w-indexer-node-on-server-3)


## Install the central components master, worker, dashboard, and indexer

<details></br>

  ### Install the W. Indexer and Dashboard on Server 3
  
  1. Start by downloading the necessary files:
  ```bash
  curl -sO https://packages.wazuh.com/4.6/wazuh-install.sh
  curl -sO https://packages.wazuh.com/4.6/config.yml
  nano config.yaml
  ```
  The content of the `config.yaml` file should look like this:
  ```yaml
  nodes:
    indexer:
      - name: main-node
        ip: "IP"
      #- name: extra-node (WE WILL ADD THIS LATER)
      #  ip: "<indexer-node-ip>"
  
    server:
      - name: wazuh-master
        ip: "IP"
        node_type: master
      - name: wazuh-worker
        ip: "IP"
        node_type: worker
  
    dashboard:
      - name: dashboard
        ip: "IP"
  ```
  2. Generate the certificates
  ```bash
  bash wazuh-install.sh --generate-config-files
  ```
  Output:
  ```console
  18/12/2023 19:25:53 INFO: Starting Wazuh installation assistant. Wazuh version: 4.6.0
  18/12/2023 19:25:53 INFO: Verbose logging redirected to /var/log/wazuh-install.log
  18/12/2023 19:26:20 INFO: --- Configuration files ---
  18/12/2023 19:26:20 INFO: Generating configuration files.
  18/12/2023 19:26:24 INFO: Created wazuh-install-files.tar. It contains the Wazuh cluster key, certificates, and passwords necessary for installation.
  ```
  3. Install the W. Indexer
  ```bash
  bash wazuh-install.sh --wazuh-indexer main-node
  ```
  Output:
  ```console
  18/12/2023 19:28:15 INFO: Starting Wazuh installation assistant. Wazuh version: 4.6.0
  18/12/2023 19:28:15 INFO: Verbose logging redirected to /var/log/wazuh-install.log
  18/12/2023 19:28:30 INFO: --- Dependencies ----
  18/12/2023 19:28:30 INFO: Installing apt-transport-https.
  18/12/2023 19:28:39 INFO: Wazuh repository added.
  18/12/2023 19:28:39 INFO: --- Wazuh indexer ---
  18/12/2023 19:28:39 INFO: Starting Wazuh indexer installation.
  18/12/2023 19:30:06 INFO: Wazuh indexer installation finished.
  18/12/2023 19:30:06 INFO: Wazuh indexer post-install configuration finished.
  18/12/2023 19:30:06 INFO: Starting service wazuh-indexer.
  18/12/2023 19:30:30 INFO: wazuh-indexer service started.
  18/12/2023 19:30:30 INFO: Initializing Wazuh indexer cluster security settings.
  18/12/2023 19:30:33 INFO: Wazuh indexer cluster initialized.
  18/12/2023 19:30:33 INFO: Installation finished.
  ```
  4. Initialize the cluster
  ```bash
  bash wazuh-install.sh --start-cluster
  ```
  Output:
  ```console
  18/12/2023 19:37:10 INFO: Starting Wazuh installation assistant. Wazuh version: 4.6.0
  18/12/2023 19:37:10 INFO: Verbose logging redirected to /var/log/wazuh-install.log
  18/12/2023 19:37:27 INFO: Wazuh indexer cluster security configuration initialized.
  18/12/2023 19:38:11 INFO: Wazuh indexer cluster started.
  ```
  5. Check if it is working
  ```json
  {
    "name" : "main-node",
    "cluster_name" : "wazuh-indexer-cluster",
    "cluster_uuid" : "aYrTM6tgR0O1SpBnNYGw9w",
    "version" : {
      "number" : "7.10.2",
      "build_type" : "rpm",
      "build_hash" : "db90a415ff2fd428b4f7b3f800a51dc229287cb4",
      "build_date" : "2023-06-03T06:24:25.112415503Z",
      "build_snapshot" : false,
      "lucene_version" : "9.6.0",
      "minimum_wire_compatibility_version" : "7.10.0",
      "minimum_index_compatibility_version" : "7.0.0"
    },
    "tagline" : "The OpenSearch Project: https://opensearch.org/"
  }
  ```
  
  ### Copy the certificates to the rest of the servers (except the server where we'll install the W. Indexer as an "extra-node" [Server 4])
  
  ```shell
  scp wazuh-install-files.tar user@server-ip:/path/to/store/file
  ```
  ### Install the W. Manger (master) on Server 1
  
  1. Download the necessary files
  ```shell
  curl -sO https://packages.wazuh.com/4.6/wazuh-install.sh
  ```
  2. Install the W. Manager
  ```shell
  bash wazuh-install.sh --wazuh-server wazuh-master
  ```
  Output:
  ```console
  18/12/2023 19:58:05 INFO: Starting Wazuh installation assistant. Wazuh version: 4.6.0
  18/12/2023 19:58:05 INFO: Verbose logging redirected to /var/log/wazuh-install.log
  18/12/2023 19:58:26 INFO: --- Dependencies ----
  18/12/2023 19:58:26 INFO: Installing apt-transport-https.
  18/12/2023 19:58:34 INFO: Wazuh repository added.
  18/12/2023 19:58:34 INFO: --- Wazuh server ---
  18/12/2023 19:58:34 INFO: Starting the Wazuh manager installation.
  18/12/2023 19:59:38 INFO: Wazuh manager installation finished.
  18/12/2023 19:59:38 INFO: Starting service wazuh-manager.
  18/12/2023 19:59:59 INFO: wazuh-manager service started.
  18/12/2023 19:59:59 INFO: Starting Filebeat installation.
  18/12/2023 20:00:06 INFO: Filebeat installation finished.
  18/12/2023 20:00:06 INFO: Filebeat post-install configuration finished.
  18/12/2023 20:00:12 INFO: Starting service filebeat.
  18/12/2023 20:00:13 INFO: filebeat service started.
  18/12/2023 20:00:13 INFO: Installation finished.
  ```
  ### Install the W. Manager (worker) on Server 2
  
  1. Download the necessary files
  ```shell
  curl -sO https://packages.wazuh.com/4.6/wazuh-install.sh
  ```
  2. Install the W. Manager
  ```shell
  bash wazuh-install.sh --wazuh-server wazuh-worker
  ```
  Output:
  ```console
  18/12/2023 20:48:08 INFO: Starting Wazuh installation assistant. Wazuh version: 4.6.0
  18/12/2023 20:48:08 INFO: Verbose logging redirected to /var/log/wazuh-install.log
  18/12/2023 20:48:29 INFO: --- Dependencies ----
  18/12/2023 20:48:29 INFO: Installing apt-transport-https.
  18/12/2023 20:48:36 INFO: Wazuh repository added.
  18/12/2023 20:48:36 INFO: --- Wazuh server ---
  18/12/2023 20:48:36 INFO: Starting the Wazuh manager installation.
  18/12/2023 20:49:43 INFO: Wazuh manager installation finished.
  18/12/2023 20:49:43 INFO: Starting service wazuh-manager.
  18/12/2023 20:50:04 INFO: wazuh-manager service started.
  18/12/2023 20:50:04 INFO: Starting Filebeat installation.
  18/12/2023 20:50:11 INFO: Filebeat installation finished.
  18/12/2023 20:50:11 INFO: Filebeat post-install configuration finished.
  18/12/2023 20:50:13 INFO: Starting service filebeat.
  18/12/2023 20:50:15 INFO: filebeat service started.
  18/12/2023 20:50:15 INFO: Installation finished.
  ```
  
  ### Install the W. Dashboard on Server 3
  
  1. The necessary files are already downloaded
  ```shell
  ls -l
  total 176
  -rw-r--r-- 1 root root  12937 Dec 18 19:26 wazuh-install-files.tar
  -rw-r--r-- 1 root root 161699 Dec 18 19:10 wazuh-install.sh
  ```
  2. Install the W. Dashboard
  ```shell
  bash wazuh-install.sh --wazuh-dashboard dashboard
  ```
  Output:
  ```console
  18/12/2023 20:54:37 INFO: Starting Wazuh installation assistant. Wazuh version: 4.6.0
  18/12/2023 20:54:37 INFO: Verbose logging redirected to /var/log/wazuh-install.log
  18/12/2023 20:54:52 INFO: Wazuh web interface port will be 443.
  18/12/2023 20:54:57 INFO: Wazuh repository added.
  18/12/2023 20:54:57 INFO: --- Wazuh dashboard ----
  18/12/2023 20:54:57 INFO: Starting Wazuh dashboard installation.
  18/12/2023 20:55:51 INFO: Wazuh dashboard installation finished.
  18/12/2023 20:55:51 INFO: Wazuh dashboard post-install configuration finished.
  18/12/2023 20:55:51 INFO: Starting service wazuh-dashboard.
  18/12/2023 20:55:52 INFO: wazuh-dashboard service started.
  18/12/2023 20:56:15 INFO: Initializing Wazuh dashboard web application.
  18/12/2023 20:56:16 INFO: Wazuh dashboard web application initialized.
  18/12/2023 20:56:16 INFO: --- Summary ---
  18/12/2023 20:56:16 INFO: You can access the web interface https://IP:443
      User: admin
      Password: VJuHB*8HuE*wzkwyW?5p0NgkQ+TJMyUO
  18/12/2023 20:56:16 INFO: Installation finished.
  ```
</details>



## Expand the installation by adding a new W. Indexer node on Server 3

<details></br>

  > [!CAUTION]
  > **Remember, Server 3 is the one that contains the W. Dashboard and the W. Indexer [main-node]**
  
  1. Get the necessary files
  ```shell
  curl -sO https://packages.wazuh.com/4.6/config.yml
  curl -sO https://packages.wazuh.com/4.6/wazuh-certs-tool.sh
  ```
  2. Extract the files into a separate directory, you will see a directory with the "old" certificates  
  ```shell
  mkdir tmp
  tar -xvf wazuh-install-files.tar -C tmp/
  ```
  Output:
  ```console
  wazuh-install-files/
  wazuh-install-files/dashboard.pem
  wazuh-install-files/admin-key.pem
  wazuh-install-files/admin.pem
  wazuh-install-files/clusterkey
  wazuh-install-files/main-node-key.pem
  wazuh-install-files/wazuh-passwords.txt
  wazuh-install-files/root-ca.key
  wazuh-install-files/wazuh-master.pem
  wazuh-install-files/dashboard-key.pem
  wazuh-install-files/config.yml
  wazuh-install-files/root-ca.pem
  wazuh-install-files/wazuh-master-key.pem
  wazuh-install-files/wazuh-worker.pem
  wazuh-install-files/wazuh-worker-key.pem
  wazuh-install-files/main-node.pem
  ```
  4. Edit the `config.yml` file leaving the new node info only:
  ```yaml
  nodes:
    indexer:
      - name: extra-node
        ip: "IP"
  ```
  5. Generate the certificates for the new node 
  ```shell
  bash wazuh-certs-tool.sh -wi /home/ubuntu/tmp/wazuh-install-files/root-ca.pem /home/ubuntu/tmp/wazuh-install-files/root-ca.key
  ```
  6. Move the certificates to the previously created folder
  ```shell
  mv wazuh-certificates/extra-node* tmp/wazuh-install-files/
  ```
  7. Edit the config.yml file inside that folder
  ```yaml
  nodes:
    indexer:
      - name: main-node
        ip: "IP"
      - name: extra-node
        ip: "IP"
  
    server:
      - name: wazuh-master
        ip: "IP"
        node_type: master
      - name: wazuh-worker
        ip: "IP"
        node_type: worker
  
    dashboard:
      - name: dashboard
        ip: "IP"
  ```
  8. Compress the previously mentioned directory (we'll "scp" this .tar file in a moment)
  ```shell
  tar -zcf 'wazuh-install-files.tar' -C 'tmp/' wazuh-install-files/
  ```
  > [!CAUTION]
  > Use the command as it is, because later the `wazuh-install.sh` script will run some validations on this .tar file
  
  9. Remove the following lines from the `/etc/wazuh-indexer/opensearch.yml` file
  ```yaml
  node.master: true
  node.data: true
  node.ingest: true
  .
  .
  .
  cluster.initial_master_nodes: main-node
  ```
  10. Modify the `/etc/wazuh-indexer/opensearch.yml` file to add the new node to the current installation
  ```yaml
  cluster.initial_master_nodes:
    - THIS-NODE-IP
    - NEW-EXTRA-NODE-IP
  
  discovery.seed_hosts:
    - THIS-NODE-IP
    - NEW-EXTRA-NODE-IP
  
  plugins.security.nodes_dn:
    - CN=main-node,OU=Wazuh,O=Wazuh,L=California,C=US
    - CN=extra-node,OU=Wazuh,O=Wazuh,L=California,C=US
  ```
  10. Copy the `wazuh-install-files.tar` file containing all the certificates [**IMPORTANT**]
  ```shell
  scp wazuh-install-files.tar user@IP:path
  ```
  
  ### Install the W. Indexer on Server 4 (extra-node)
  
  1. Start by downloading the necessary files:
  ```bash
  curl -sO https://packages.wazuh.com/4.6/wazuh-install.sh
  ```
  2. Install the W. Indexer
  ```bash
  bash wazuh-install.sh --wazuh-indexer extra-node
  ```
  Output:
  ```console
  18/12/2023 23:57:20 INFO: Starting Wazuh installation assistant. Wazuh version: 4.6.0
  18/12/2023 23:57:20 INFO: Verbose logging redirected to /var/log/wazuh-install.log
  18/12/2023 23:57:33 INFO: --- Dependencies ----
  18/12/2023 23:57:33 INFO: Installing apt-transport-https.
  18/12/2023 23:57:43 INFO: Wazuh repository added.
  18/12/2023 23:57:43 INFO: --- Wazuh indexer ---
  18/12/2023 23:57:43 INFO: Starting Wazuh indexer installation.
  18/12/2023 23:59:09 INFO: Wazuh indexer installation finished.
  18/12/2023 23:59:10 INFO: Wazuh indexer post-install configuration finished.
  18/12/2023 23:59:10 INFO: Starting service wazuh-indexer.
  19/12/2023 00:00:03 INFO: wazuh-indexer service started.
  19/12/2023 00:00:03 INFO: Initializing Wazuh indexer cluster security settings.
  19/12/2023 00:00:03 INFO: Wazuh indexer cluster initialized.
  19/12/2023 00:00:03 INFO: Installation finished.
  ```
  4. Remove the following lines from the `/etc/wazuh-indexer/opensearch.yml` file
  ```yaml
  node.master: true
  node.data: true
  node.ingest: true
  ```
  5. Initialize the cluster
  ```bash
  bash wazuh-install.sh --start-cluster -v
  ```
  Output:
  ```console
  19/12/2023 01:11:24 INFO: Starting Wazuh installation assistant. Wazuh version: 4.6.0
  19/12/2023 01:11:24 INFO: Verbose logging redirected to /var/log/wazuh-install.log
  Hit:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy InRelease
  Hit:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates InRelease
  Hit:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports InRelease
  Hit:4 http://security.ubuntu.com/ubuntu jammy-security InRelease
  Hit:5 https://packages.wazuh.com/4.x/apt stable InRelease
  Reading package lists...
  **************************************************************************
  ** This tool will be deprecated in the next major release of OpenSearch **
  ** https://github.com/opensearch-project/security/issues/1755           **
  **************************************************************************
  Security Admin v7
  Will connect to EXTRA-NODE-IP:9200 ... done
  Connected as "CN=admin,OU=Wazuh,O=Wazuh,L=California,C=US"
  OpenSearch Version: 2.8.0
  Contacting opensearch cluster 'opensearch' and wait for YELLOW clusterstate ...
  Clustername: wazuh-indexer-cluster
  Clusterstate: GREEN
  Number of nodes: 2
  Number of data nodes: 2
  .opendistro_security index already exists, so we do not need to create one.
  Populate config from /etc/wazuh-indexer/opensearch-security/
  Will update '/config' with /etc/wazuh-indexer/opensearch-security/config.yml 
     SUCC: Configuration for 'config' created or updated
  Will update '/roles' with /etc/wazuh-indexer/opensearch-security/roles.yml 
     SUCC: Configuration for 'roles' created or updated
  Will update '/rolesmapping' with /etc/wazuh-indexer/opensearch-security/roles_mapping.yml 
     SUCC: Configuration for 'rolesmapping' created or updated
  Will update '/internalusers' with /etc/wazuh-indexer/opensearch-security/internal_users.yml 
     SUCC: Configuration for 'internalusers' created or updated
  Will update '/actiongroups' with /etc/wazuh-indexer/opensearch-security/action_groups.yml 
     SUCC: Configuration for 'actiongroups' created or updated
  Will update '/tenants' with /etc/wazuh-indexer/opensearch-security/tenants.yml 
     SUCC: Configuration for 'tenants' created or updated
  Will update '/nodesdn' with /etc/wazuh-indexer/opensearch-security/nodes_dn.yml 
     SUCC: Configuration for 'nodesdn' created or updated
  Will update '/whitelist' with /etc/wazuh-indexer/opensearch-security/whitelist.yml 
     SUCC: Configuration for 'whitelist' created or updated
  Will update '/audit' with /etc/wazuh-indexer/opensearch-security/audit.yml 
     SUCC: Configuration for 'audit' created or updated
  Will update '/allowlist' with /etc/wazuh-indexer/opensearch-security/allowlist.yml 
     SUCC: Configuration for 'allowlist' created or updated
  SUCC: Expected 10 config types for node {"updated_config_types":["allowlist","tenants","rolesmapping","nodesdn","audit","roles","whitelist","internalusers","actiongroups","config"],"updated_config_size":10,"message":null} is 10 (["allowlist","tenants","rolesmapping","nodesdn","audit","roles","whitelist","internalusers","actiongroups","config"]) due to: null
  SUCC: Expected 10 config types for node {"updated_config_types":["allowlist","tenants","rolesmapping","nodesdn","audit","roles","whitelist","internalusers","actiongroups","config"],"updated_config_size":10,"message":null} is 10 (["allowlist","tenants","rolesmapping","nodesdn","audit","roles","whitelist","internalusers","actiongroups","config"]) due to: null
  Done with success
  19/12/2023 01:11:39 INFO: Wazuh indexer cluster security configuration initialized.
  {"error":"Content-Type header [] is not supported","status":406}
  [19/12/2023 01:12:05 DEBUG: Inserted wazuh-alerts template into the Wazuh indexer cluster.
  19/12/2023 01:12:05 DEBUG: Setting Wazuh indexer cluster passwords.
  19/12/2023 01:12:07 DEBUG: Generating password hashes.
  19/12/2023 01:12:14 DEBUG: Password hashes generated.
  19/12/2023 01:12:14 DEBUG: Creating password backup.
  **************************************************************************
  ** This tool will be deprecated in the next major release of OpenSearch **
  ** https://github.com/opensearch-project/security/issues/1755           **
  **************************************************************************
  Security Admin v7
  Will connect to EXTRA-NODE-IP:9200 ... done
  Connected as "CN=admin,OU=Wazuh,O=Wazuh,L=California,C=US"
  OpenSearch Version: 2.8.0
  Contacting opensearch cluster 'opensearch' and wait for YELLOW clusterstate ...
  Clustername: wazuh-indexer-cluster
  Clusterstate: GREEN
  Number of nodes: 2
  Number of data nodes: 2
  .opendistro_security index already exists, so we do not need to create one.
  Will retrieve '/config' into /etc/wazuh-indexer/backup/config.yml 
     SUCC: Configuration for 'config' stored in /etc/wazuh-indexer/backup/config.yml
  Will retrieve '/roles' into /etc/wazuh-indexer/backup/roles.yml 
     SUCC: Configuration for 'roles' stored in /etc/wazuh-indexer/backup/roles.yml
  Will retrieve '/rolesmapping' into /etc/wazuh-indexer/backup/roles_mapping.yml 
     SUCC: Configuration for 'rolesmapping' stored in /etc/wazuh-indexer/backup/roles_mapping.yml
  Will retrieve '/internalusers' into /etc/wazuh-indexer/backup/internal_users.yml 
     SUCC: Configuration for 'internalusers' stored in /etc/wazuh-indexer/backup/internal_users.yml
  Will retrieve '/actiongroups' into /etc/wazuh-indexer/backup/action_groups.yml 
     SUCC: Configuration for 'actiongroups' stored in /etc/wazuh-indexer/backup/action_groups.yml
  Will retrieve '/tenants' into /etc/wazuh-indexer/backup/tenants.yml 
     SUCC: Configuration for 'tenants' stored in /etc/wazuh-indexer/backup/tenants.yml
  Will retrieve '/nodesdn' into /etc/wazuh-indexer/backup/nodes_dn.yml 
     SUCC: Configuration for 'nodesdn' stored in /etc/wazuh-indexer/backup/nodes_dn.yml
  Will retrieve '/whitelist' into /etc/wazuh-indexer/backup/whitelist.yml 
     SUCC: Configuration for 'whitelist' stored in /etc/wazuh-indexer/backup/whitelist.yml
  Will retrieve '/allowlist' into /etc/wazuh-indexer/backup/allowlist.yml 
     SUCC: Configuration for 'allowlist' stored in /etc/wazuh-indexer/backup/allowlist.yml
  Will retrieve '/audit' into /etc/wazuh-indexer/backup/audit.yml 
     SUCC: Configuration for 'audit' stored in /etc/wazuh-indexer/backup/audit.yml
  19/12/2023 01:12:18 DEBUG: Password backup created in /etc/wazuh-indexer/backup.
  19/12/2023 01:12:18 DEBUG: Loading new passwords changes.
  **************************************************************************
  ** This tool will be deprecated in the next major release of OpenSearch **
  ** https://github.com/opensearch-project/security/issues/1755           **
  **************************************************************************
  Security Admin v7
  Will connect to EXTRA-NODE-IP:9200 ... done
  Connected as "CN=admin,OU=Wazuh,O=Wazuh,L=California,C=US"
  OpenSearch Version: 2.8.0
  Contacting opensearch cluster 'opensearch' and wait for YELLOW clusterstate ...
  Clustername: wazuh-indexer-cluster
  Clusterstate: GREEN
  Number of nodes: 2
  Number of data nodes: 2
  .opendistro_security index already exists, so we do not need to create one.
  Populate config from /home/ubuntu
  Force type: internalusers
  Will update '/internalusers' with /etc/wazuh-indexer/backup/internal_users.yml 
     SUCC: Configuration for 'internalusers' created or updated
  SUCC: Expected 1 config types for node {"updated_config_types":["internalusers"],"updated_config_size":1,"message":null} is 1 (["internalusers"]) due to: null
  SUCC: Expected 1 config types for node {"updated_config_types":["internalusers"],"updated_config_size":1,"message":null} is 1 (["internalusers"]) due to: null
  Done with success
  19/12/2023 01:12:22 DEBUG: Passwords changed.
  19/12/2023 01:12:22 INFO: Wazuh indexer cluster started.
  ```
  5. Check if the installation is working
  ```shell
  curl -k -u admin:VJuHB*8HuE*wzkwyW?5p0NgkQ+TJMyUO https://EXTRA-NODE-IP:9200/_cat/nodes?v
  ip            heap.percent ram.percent cpu load_1m load_5m load_15m node.role node.roles                                        cluster_manager name
  MAIN-NODE-IP           26          98   7    0.48    0.22     0.08 dimr      cluster_manager,data,ingest,remote_cluster_client *               main-node
  EXTRA-NODE-IP            6          94   8    0.03    0.24     0.18 dimr      data,ingest,master,remote_cluster_client          -               extra-node
  
  
  curl -k -u admin:VJuHB*8HuE*wzkwyW?5p0NgkQ+TJMyUO https://EXTRA-NODE-IP:9200
  {
    "name" : "extra-node",
    "cluster_name" : "wazuh-indexer-cluster",
    "cluster_uuid" : "aYrTM6tgR0O1SpBnNYGw9w",
    "version" : {
      "number" : "7.10.2",
      "build_type" : "rpm",
      "build_hash" : "db90a415ff2fd428b4f7b3f800a51dc229287cb4",
      "build_date" : "2023-06-03T06:24:25.112415503Z",
      "build_snapshot" : false,
      "lucene_version" : "9.6.0",
      "minimum_wire_compatibility_version" : "7.10.0",
      "minimum_index_compatibility_version" : "7.0.0"
    },
    "tagline" : "The OpenSearch Project: https://opensearch.org/"
  }
  ```
  
  ### Configure Filebeat on the W. Manager servers (Server 1 and 2)
  
  **Repeat the following steps on both Server 1 and 2:**
  
  1. Edit the `/etc/filebeat/filebeat.yml` file adding the new node IP (extra-node)
  ```yaml
  .
  .
  output.elasticsearch.hosts:
    - MAIN-NODE-IP:9200
    - EXTRA-NODE-IP:9200
  ```
  2. Restart Filebeat
  ```shell
  systemctl restart filebeat
  ```
  
  ### Configure the W. Dashboard on Server 3
  
  1. Edit the `/etc/wazuh-dashboard/opensearch_dashboards.yml` file by modifying the following line
  ```yaml
  opensearch.hosts: ["https://MAIN-NODE-IP:9200", "https://EXTRA-NODE-IP:9200"]
  ```
  2. Restart the W. Dashboard service
  ```shell
  systemctl restart wazuh-dashboard
  ```
</details>
