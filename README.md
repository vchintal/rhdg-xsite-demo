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
git clone https://github.com/vchintal/rhdg-xsite-demo.git
cd rhdg-xsite-demo
```

### Step 2 - Update the Ansible variables file
Edit the `host_vars/localhost.yml` file and change the following variables (where `X` is either `1` or `2`):

1. `siteX_ocp_api_host` (hostname in the OpenShift API URL)
2. `ocp_api_token` (`token` created from the OpenShift web console, by clicking on the `Copy Login Command` under the username)

### Step 3 - Run the Ansible Playbook

```sh 
ansible-playbook setup.yml
```

> **Important Note!** Currently the playbook might fail because the service accounts are not created fast enough on the OpenShift side. When that happens re-run the above command again. 

### Step 4 - Ensure that Infinispan clusters report CrossSiteViewFormed  

1. Click on `Installed Operators` in the `rhdg-xsite-demo` namespace.
2. For the Data Grid Operator, click on the `Infinispan Cluster` link under the `Provided APIs`.
3. The Infinispan cluster `rhdg-cluster` should report `CrossSiteViewFormed` in the `Status` column. If you see it, you are ready for the demo. This step applies to both OpenShift clusters.
 
## Demo

1. For each OpenShift cluster get the Route URL for the `rhdg-xsite-demo-webapp` app in the `rhdg-xsite-demo` namespace. It should look like `http://rhdg-xsite-demo-webapp-rhdg-xsite-demo.apps.<cluster-name>.<domain-name>`.
2. Click-open such URL, one in its own browser tab.
3. There will be prompt to add a `username`, provide that as a single word, use the same exact username in both the tabs/applications.
4. Create a <K,V> pair by adding to a cache in one tab (application or cluster) and switch to the other tab to see the same <K,V> pair displayed. This is due to the XSite replication via RHDG under the covers.
5. Try to delete one or more <K,V> pairs on a site and ensure that deletes were synced over to the other site
   
## Disaster Recovery (DR) Simulation

### Delete Site

Simulate the deletion of a site completely removing the Data Grid and Webapp instances from a site. To do that run the following command and at the prompt give the name of the site (`site1` or `site2`) that you would like to delete.

```sh 
ansible-playbook delete_site.yml
```

Once deleted, demo that neither the webapp nor the RHDG cluster is available.

### Restore Site 

Now restore the same site with the following command:

```sh 
ansible-playbook restore_site.yml
```

> For the demo purposes you can choose to edit the `infinispan-cr.yml` file to have only 1 instance. This will facilitate faster restoration of the site (again, for demo only)

Once the playbook has finished, show the following:
1. On both the `Infinispan Cluster` CRs, the status is `CrossSiteViewFormed`. This means both the sites find each other 
2. Test the webapp on both sides. If you do not see the data synced over yet into the site that was rebooted
   1. On the site that was NOT rebooted, navigate to the route exposed by the `rhdg-cluster`, it should be of the form `	
http://rhdg-cluster-external-rhdg-xsite-demo.apps.<cluster-name>.<domain-name>`
   2. Click on the button `Open the Console` on the web page
   3. Click on the `user-xsite-cache` 
   4. Click on `Manage`
   5. Under `Backups Management`, under the `Action` column for the site-name that was just rebooted, click on `Start Transfer`
3. Once the above step (#2) is successful, check the web page for the webapps on both sites. Hopefully, 😉, you are lucky to see the state transfer generating the same results on the webapp on both the sites
