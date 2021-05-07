na_oracle19c_deploy
=========

This is an archive of Ansible tasks used to configure a new ONTAP SVM along with provisioning the storage, and configuration for Oracle Linux 8.2 for an NFS Oracle19c database deployment.

    Automated solutions built over these tasks are available in the na_oracle19c_deploy playbook.
    Refer to individual solutions READme.md file for instructions for use of respective solutions.

License
-------

By accessing, downloading, installing or using the content in this repository, you agree the terms of the License laid out in [License](license.txt) file.

Note that there are certain restrictions around producing and/or sharing any derivative works with the content in this repository. Please make sure you read the terms of the [License](license.txt) before using the content. If you do not agree to all of the terms, do not access, download or use the content in this repository.

Requirements
------------

    Ansible version: 2.10 and above
    Python 3
    Python libraries:
    netapp-lib
    xmltodict
    jmespath

    ONTAP Cluster: 9.3-9.7

    Oracle Linux 7/8.2
    RHEL 7/8

Variables
---------

For detailed instructions for running the playbook refer to NetAppDocs https://docs.netapp.com/us-en/netapp-solutions/ent-db/getting_started_requirements.html

    ######################################################################
    ###### Oracle 19c deployment user configuration variables       ######
    ###### Consolidate all variables from ontap, linux and oracle   ######
    ######################################################################

    ###########################################
    ### Ontap env specific config variables ###
    ###########################################

    # CA_signed_certificates (ONLY CHANGE to 'true' IF YOU ARE USING CA SIGNED CERTIFICATES
    ca_signed_certs: false

    #AWX/Tower Inventory Group name
    hosts_group: ontap

    # Names of the Nodes in the ONTAP Cluster
    nodes:
      - AFF-01
      - AFF-02

    # Storage VLANs
    # Add additional rows for vlans as necessary
    storage_vlans:
      - {vlan_id: "100", name: "ORA_NFS", protocol: "NFS"}

    # Details of the Data Aggregates that need to be created
    # If Aggregate creation takes longer, subsequent tasks of creating volumes may fail.
    # There should be enough disks already zeroed in the cluster, otherwise aggregate create will zero the disks and will take long time
    data_aggregates:
      - {aggr_name: aggr01_node01}
      - {aggr_name: aggr01_node02}

    svm_name: ora_svm

    # SVM Management LIF Details
    svm_mgmt_details:
      - {address: 172.21.91.100, netmask: 255.255.255.0, home_port: "e0M"}

    # NFS storage parameters when data protocol set to NFS. Volume named after Oracle hosts name identified by mount point as follow for oracle host 1.
    # Add addtional volumes for addtional hosts and also add the volumes list to host_vars as host_datastores_nfs variable.
    # u01 - Oracle binary
    # u02 - Oracle data
    # u03 - Oracle redo
    volumes_nfs:
      - {vol_name: "{{groups.oracle[0]}}_u01", aggr_name: aggr01_node01, lif: 172.21.126.200, size: 50}
      - {vol_name: "{{groups.oracle[0]}}_u02", aggr_name: aggr01_node01, lif: 172.21.126.200, size: 100}
      - {vol_name: "{{groups.oracle[0]}}_u03", aggr_name: aggr01_node01, lif: 172.21.126.200, size: 100}

    # NFS LIFs IP address and netmask
    nfs_lifs_details:
      - address: 172.21.126.200 #for node-1
        netmask: 255.255.255.0
      - address: 172.21.126.201 #for node-2
        netmask: 255.255.255.0

    client_match: 172.21.126.0/24

    ###########################################
    ### Linux env specific config variables ###
    ###########################################

    mount_points:
      - /u01
      - /u02
      - /u03

    # Up to 75% of node memory size divided by 2mb. Consider how many databases to be hosted on the node and how much ram to be allocated to each DB. 
    # Leave it black if hugepage is not configured on the host.
    hugepages_nr: 3072

    redhat_sub_username: username
    redhat_sub_password: password

    ####################################################
    ### DB env specific install and config variables ###
    ####################################################

    # Database domain name
    db_domain: yourdomain.com

    # Set initial password for all required Oracle passwords. Change them after installation.
    initial_pwd_all: pw123

Host_vars
---------
    ansible_host: 10.10.10.10

    # Oracle db log mode: true or false
    log_archive_mode: true

    # Number of pluggable database per container instance identified by sid
    oracle_sid: cdb2
    pdb_num: 3
    pdb_name: "{{ oracle_sid }}_pdb"

    # CDB listener port, use different listener port for additional CDB on same host
    listener_port: 1523

    # CDB is created with SGA at 75% of memory_limit, MB. Consider how many databases to be hosted on the node and how much ram to be allocated to each DB. The grand total SGA should not exceed 75% available RAM on node.
    memory_limit: 5464

    # Set "em_configuration: DBEXPRESS" to install enterprise manager express and choose a unique port from 5500 to 5599 for each sid on the host.
    # Leave them black if em express is not installed.
    em_configuration: DBEXPRESS
    em_express_port: 5501

    host_datastores_nfs:
      - {vol_name: "{{groups.oracle[0]}}_u01", aggr_name: aggr01_node01, lif: 172.21.126.200, size: 25}
      - {vol_name: "{{groups.oracle[0]}}_u02", aggr_name: aggr01_node01, lif: 172.21.126.200, size: 100}
      - {vol_name: "{{groups.oracle[0]}}_u03", aggr_name: aggr01_node01, lif: 172.21.126.200, size: 100}

Tags Info
---------


Author Information
------------------

NetApp Solutions Automation Team
