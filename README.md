# Example Ansible Alfresco Setup

This project is an example of one way to automate Alfresco installation and
configuration management.

## Setup

Here are the high-level steps you'll need to perform before running any of the
playbooks with Ansible:

1. Setup SSH Keys
2. Clone this project to your local machine.
3. Edit the inventory file. Add your Alfresco and SOLR hosts to the appropriate
groups. It is okay if the Alfresco and SOLR machine are the same.
4. Edit the group_vars for the groups you just populated in the inventory file
to override the variables declared in group_var/all with values specific to the
group.
5. Setup Ansible Vault to encrypt passwords
6. Collect dependencies such as the ACS distribution zip, ASS distribution zip,
Tomcat, ActiveMQ, and database driver.

Let's look at some of these setup steps in further detail.

### Setup SSH keys

The easiest way to authenticate with each of the machines in your inventory is
to use SSH keys. For example, if the user you will run Alfresco and SOLR with
is "alfresco" make sure you can ssh to each box in your inventory as the
alfresco user without providing a password.

The steps to set this up are well-documented online, but it typically involves:

    ssh-keygen -t rsa
    ssh-copy-id alfresco@198.51.100.0

And you may want to edit ~/.ssh/config on your local machine so that you don't
have to specify the SSH key when using ssh or scp:

    Host alfresco.someco.com
      IdentityFile ~/.ssh/someco_id_rsa

### Edit the inventory file

The inventory file tells Ansible about your environments, including each host
name and the role it plays in your stack.

In this project, the ansible.cfg file specifies that our default inventory file
is named "inventory" and lives in the root of the project.

The inventory file is pre-populated with sample groups that assume you have
three environments, dev, test, and prod. There is one group each for Alfresco
and SOLR per environment for a total of six groups.

The "local" group is used for testing the playbook using local virtual machines
managed by Vagrant and can be ignored. If you want to learn more about how that
works, see the README.md file in the local directory.

### Edit group variables

Ansible Templates have variables and the value of those variables can differ by
group. The group_vars directory has one folder per group. Within that there is
a vars file and a vault file (see next section on Vault).

There is a special group named "all". The vars file in the group_vars/all
directory defines default values for all variables. If you introduce new
variables, make sure you add them to group\_vars/all/vars.

Each group can optionally override the value declared in all by specifying a
new value for that variable in its own group\_vars vars file.

For example, the default database name is declared as "alfresco" in all:

    alf_db_name: alfresco

Suppose that in the dev environment we want to name the database something else.
To do that, we'd edit group\_vars/alfresco\_dev/vars with:

    alf_db_name: alfrescodev

In this example, all templates rendered for all servers in the alfresco_dev
group will have "alfrescodev" set as the value for alf\_db\_name.

Here are some key variables you should review before running:

* alf_home: The Alfresco home directory. Typically should be a soft link to a
version specific directory.
* alf_install_dir: A version specific directory where Alfresco is installed.
* alf_archive: The file name of the ACS distribution zip.
* tomcat_archive: The file name of the Apache Tomcat tar file.
* search_archive: The file name of the Alfresco Search Services distribution zip.
* alf_db_driver: The hibernate dialect.
* alf_db_driver_file: The file name of the database driver.
* activemq_archive: The file name of the Apache ActiveMQ tar file.
* alf_java_home: The path to the Java home directory.

It's best to rely on group_vars as much as possible, but if you need to you can
also have host-specific variables by creating a directory called "host_vars"
and creating directories named for specific hosts below that.

### Managing secrets using Ansible Vault

The [Ansible Vault docs](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
are worth reading, but here's how it works at a high-level:

* Each group folder has a file called vars for that group's plain-text
variables, and, optionally, a file called vault for that group's encrypted
variables.
* The ansible.cfg file points to the Ansible Vault password file location which
is .vault-passwd. This password should be shared with your team, but don't
check it in to source code control. NOTE: This file is not distributed as part
of this project. You'll need to create your own. To do that, just create a file
named ".vault-passwd" and put it in the root of your project (adjacent to this
README.md file). The file should contain one line with the value of your vault
password.
* Variables that need to be encrypted will have a plain variable name and a
vault variable name. For example, in group_vars/all/vars the following will use
the value from the vault for the Alfresco DB password:

      alf_db_password: "{{ vault_alf_db_password }}"

* From the root of the project, use `ansible-vault edit group_vars/all/vault`
to edit a vault file (in this example, it is the all group, but each group can
have its own vault file).
* Ansible Vault will launch a configurable editor. On save, the value will be
encrypted using the Ansible Vault password.
* To set values for the vault variables that are used in this project, copy and
paste the following into your vault file (use whatever values you want, these
mimic what used to be set by the installer):

      vault_alf_db_password: admin
      vault_alf_initial_admin_password: "209c6174da490caeb422f3fa5a7ae634"
      vault_alf_jmx_monitor_password: monitor_password
      vault_alf_jmx_control_password: control_password
      vault_alf_keystore_password: kT9X6oe68t
      vault_alf_truststore_password: kT9X6oe68t

* When setting up a new vault for the first time use `ansible-vault encrypt group_vars/all/vault` to encrypt the file. After that, just use `ansible-vault edit group_vars/all/vault`.
* You should now have an encrypted vault file that uses your own vault password
that is stored in .vault-passwd. When you do `ansible-vault edit` you should
see the values for the database password, JMX passwords, and key/truststore
passwords.

Remember that secrets are only encrypted on your local machine. Ansible
decrypts the value in memory when the playbook runs, and then delivers the
value to the target in plain text.

### Gathering Dependencies

If you are using the playbook to install Alfresco Content Services and/or
Alfresco Search Services you will need to gather a few dependencies. Here is
a list of what you need and where they go:

|File Description|Example|Source|Put It Here|
|----------------|-------|------|-----------|
|ACS Distribution|alfresco-content-services-community-distribution-6.2.0-ga.zip|alfresco.com (or support for Enterprise)|files/third-party|
|Apache Tomcat|apache-tomcat-8.5.34.tar.gz|tomcat.apache.com|files/third-party|
|ASS Distribution|alfresco-search-services-1.4.0.zip|alfresco.com|files/third-party|
|Database driver|postgresql-42.2.1.jar|Database vendor|files/third-party|
|Apache ActiveMQ|apache-activemq-5.15.6-bin.tar.gz|activemq.apache.com|files/third-party|

You can optionally deploy AMPs as part of installation or as a separate
playbook. Put AMPs in files/third-party/amps or files/third-party/amps_share
depending on what type of AMP it is.

Note that AOS and Share Services AMPs are shipped with the distribution ZIP so
there is no need to put those in the amps/amps_share directory with custom or
third-party AMPs.

## Installing Alfresco and Search

### Prepare the target machines

The installation playbook assumes you do not have root access to the target
machines. It assumes certain tasks have already been performed. You'll need to
perform these tasks before running the install playbook:

* Create an alfresco user and group
* Create an install directory owned by the alfresco user and group
* Increase file handles
* Install Java
* Install a database (if it is going to be local)
* Install LibreOffice
* Install ImageMagick
* Open appropriate ports in the firewall
* Create init.d/systemctl scripts for Alfresco and ActiveMQ
* Attach and mount any needed NFS mounts or other storage volumes

If you have root access to the server (either directly or via sudo), you can
modify the playbook to perform all of these tasks for you.

### Run the playbook to install Alfresco Content Services

Once all pre-requisites are in place, install Alfresco by running:

    ansible-playbook installAlfresco.yml --extra-vars="hosts=alfresco_dev"

Where "alfresco_dev" is the desired group.

SSH to each Alfresco server and start up ActiveMQ and ACS:

    cd /opt/alfresco
    ./activemq/bin/activemq start
    ./tomcat/bin/startup.sh

The startup commands might look a little different if you have init.d/systemctl
scripts in place.

### Run the playbook to install Alfresco Search Services

Now install Alfresco Search Services by running:

    ansible-playbook installSearch.yml --extra-vars="hosts=solr_dev"

Next, SSH to each SOLR server and start it up so it can init the cores:

    cd /opt/alfresco/alfresco-search-services
    ./solr/bin/solr start -a "-Dcreate.alfresco.defaults=alfresco,archive"

Let it populate the core configurations, then:

    ./solr/bin/solr stop

Back on your local machine, run a playbook to remove any indices that were
created:

    ansible-playbook removeSolrIndexMetadata.yml --extra-vars="hosts=solr_dev"

Then, run a playbook to copy the SOLR core config into place:

    ansible-playbook copySolrCoreConfig.yml --extra-vars="hosts=solr_dev"

Finally, SSH to each SOLR server and start it up:

    cd /opt/alfresco/alfresco-search-services
    ./solr/bin/solr start

## Copying Configuration Files

The installation playbooks copy Alfresco and SOLR configuration files to the
servers in the target group for you. Outside of an installation or upgrade you
may need to deploy configuration updates. To do that, run one of the "copy"
playbooks:

    ansible-playbook copyAllAlfrescoConfig.yml --extra-vars="hosts=alfresco_dev"
    ansible-playbook copySolrCoreConfig.yml --extra-vars="solr_dev"

You might consider creating additional playbooks for doing other types of
maintenance such as deploying AMPs.

### Dry-run

Use the `--check` flag to tell Ansible to do a dry-run, like this:

    ansible-playbook copyAllAlfrescoConfig.yml --check --extra-vars="hosts=alfresco_dev"

The output will show you what *would have* been changed had you not used the
flag.

Use the `--diff` flag along with `--check` to see the difference between the
configuration in your current project and what is currently on the server,
like this:

    ansible-playbook copyAllAlfrescoConfig.yml --diff --check --extra-vars="hosts=alfresco_dev"

The output will show you a difference between what is currently on the target
and what would have been deployed without the flags.

## Security

Running the installation playbook as provided in this example setup will result
in an Alfresco server with no SSL certificates configured, either for ACS or
for SOLR. It also uses a "admin" as the admin user password. You will need to
make changes appropriate for your environment to properly secure your install.
