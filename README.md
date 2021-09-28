# X-Site Data Replication with Red Hat Data Grid on Cloud Platforms

## Prerequisites
* Two OpenShift clusters deployed on cloud platforms (AWS, GCP, IBM Cloud, Azure)
  * Operator catalog must be installed for installing Red Hat Data Grid Operator
  * Credentials for an user of the role of a `cluster-admin` for the ability to install the operator(s)
* Linux host  (Fedora 33+/RHEL 8) with the following installed 
  * Latest Ansible (2.9.x)
  * Git
  
## Setup

### Step 1 - Git-clone this repo on Linux Host 

```sh
git clone https://github.com/vchintal/rhdg-xsite-repl.git
cd rhdg-xsite-repl
```

### Step 2 - Update the Ansible variables file
Edit the `host_vars/localhost.yml` file and change the following variables (where `X` is either `1` or `2`):

1. `siteX_ocp_api_host` (hostname in the OpenShift API URL)
2. `ocp_api_token` (`token` created from the OpenShift web console, by clicking on the `Copy Login Command` under the username)

### Step 3 - Run the Ansible Playbook

```sh 
ansible-playbook rhdg-xsite-setup.yml
```

> **Important Note!** Currently the playbook might fail because 1) the service accounts are not created fast enough on the OpenShift side b) the operator install takes time and hence creation of CR might fail. When that happens re-run the above command again. 

### Step 4 - Ensure that Infinispan clusters report CrossSiteViewFormed  

1. Click on `Installed Operators` in the `rhdg-xsite-demo` namespace.
2. For the Data Grid Operator, click on the `Infinispan Cluster` link under the `Provided APIs`.
3. The Infinispan cluster `rhdg-cluster` should report `CrossSiteViewFormed` in the `Status` column. If you see it, you are ready for the demo. This step applies to both OpenShift clusters.
 
## Demo

1. For each OpenShift cluster get the Route URL for the `session-cache` app in the `rhdg-xsite-demo` namespace. It should look like `http://session-cache-rhdg-xsite-demo.apps.<cluster-name>.<domain-name>`.
2. Click-open such URL, one in its own browser tab.
3. There will be prompt to add a `username`, provide that as a single word, use the same exact username in both the tabs/applications.
4. Create a <K,V> pair by adding to a cache in one tab (application or cluster) and switch to the other tab to see the same <K,V> pair displayed. This is due to the XSite replication via RHDG under the covers.