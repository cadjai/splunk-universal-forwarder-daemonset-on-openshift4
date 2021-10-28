Role Name
=========

A utility role to deploy Splunk universal logforwarder as daemonset on OpenShift Container Platform 4.+ (OCP4).
This is not needed anymore given that out of box OpenShift provides several ways to forward logs to Splunk including using the fluentd forwarder to forward to Splunk using HEC. 
However, in some environments where everything has been built around using the universal forwarder, there are no other ways but to implement this. Luckily Splunk provides two container images to use for the universal forwarder (docker.io/splunk/splunk:latest and docker.io/splunk/universalforwarder:latest). The only missing thing was a lack of documentation on how to run and configure these containers on secured kubernetes platforms like OpenShift Container Platform. For example when one tries to run this container with podman as either a splunk user or a root user things tend to work but when one follows the same approach on OpenShift the containers fail to deploy. TO accomplish this I tried to use various SCC to until I finally decided to try privileged which the one that ultimately worked. I could have started there but didn't want to run this as privileged container. Anyway one has to use the privileged SCC in conjuction with a user and group of 0 (root) to successfully run this on OpenShift. Once that was figured out the rest was easy to deploy. 
This is just an Ansible playbook with an OpenShift Template to help easily deploy and configure the universal log forwarder as daemonset to run on all node within the cluster. 
In a later version a helm chart can be added if this is useful to others. 


Requirements
------------
A cluster with a valid registry containing  the appropriate forwarder image to be used if this is being deployed on a locked down or disconnected cluster that does not allow the use of images from docker.io or other public registry.
A running OCP 4 cluster with valid credentials provided through the variables described below.


Plays and Role Variables
------------------------

- ansible_name_module: The name of the module this role is part of. This is used to track context when this role is run in a playbook as part of other plays.  
- openshift_cli: Openshift client binary used to interact with the cluster api (default to 'oc')
- ocp_cluster_user: The name of the cluster-admin user used to perform the various actions against the cluster.
- ocp_cluster_user_password: The password of the cluster-admin user used to perform the various actions against the cluster.
- ocp_cluster_console_url: The URL of the API the cluster these actions are being applied to.
- ocp_cluster_console_port: The port on which the cluster api is listening (default to 6443)
- staging_dir: The directory where rendered yaml config are placed on the '/tmp'
- splunk_uf_namespace: The namespace to deploy the splunk forwarder daemonset into.
- splunk_uf_namespace_description: The description of the namespace above.
- splunk_uf_template_path: The path to the OpenShift template to use to deploy splunk forwarder.
- splunk_uf_application_name: The name of the splunk forwarder application (e.g. splunk-universalforwarder)
- splunk_uf_password: The password to use for this instance of splunk forwarder.
- splunk_uf_role: The role of this splunk forwarder. This goes with the image being used. If using docker.io/splunk/universalforwarder the value should be splunk_universal_forwarder. If using docker.io/splunk/splunk then the value should be splunk-heavyforwarder.
- splunk_uf_image: The image to use for the forwarder. It can either be docker.io/splunk/universalforwarder or docker.io/splunk/splunk.
- splunk_uf_output_cong_mount_path: The path where the outputs.conf file in mounted within the container. It depends on the image used. For docker.io/splunk/splunk the path should be /opt/splunk/etc/system/local/outputs.conf. For docker.io/splunk/universalforwarder the path is /opt/splunkforwarder/etc/system/local/outputs.conf.
- splunk_uf_indexer_server: The FQDN or IP of the Splunk instance the logs are to be forwarded to.
- splunk_uf_indexer_port: The Port of the Splunk instance the logs are to be for
warded to.

Dependencies
------------


Installation and Usage
-----------------------
Clone the repository to where you want to run this from and make sure you can connect to your cluster using the CLI .
Ensure you install all requirements using `ansible-galaxy install -r requirements.yml --force` before performing the next steps.
You will also need to have cluster-admin run in order to run the playbook since the cron job need to run privileged.
Finally before running the playbook make sure to set and update the variables as appropriate to your use case.

Playbooks
---------
To run the main playbook use the ansible-playbook command as follows
`ansible-plabook --ask-vault-pass install-and-configure-splunk-universalforwarder.yml -vvv`


License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
