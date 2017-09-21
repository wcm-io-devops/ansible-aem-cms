# aem-cms

This role installs and Adobe Experience Manager (AEM) 6.x on Debian/Ubuntu or RHEL/CentOS servers.
> This role was developed as part of the wcm.io set of roles to integrate Ansible with [CONGA](http://devops.wcm.io/conga/) but can be used independently of it.

## Requirements

This role requires Ansible 2.2 or higher and works with AEM 6.1 or higher. Also required are an AEM quickstart JAR file and a valid AEM license file. The `license.properties` files needs be made accessible to the role, normally by copying it into the `files` folder in the playbook directory. The `AEM_*_Quickstart.jar` can be supplied in the same way or retrieved from a Nexus or RPM/APT repository (see below).

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

	aem_group: adobe
	aem_user: adobe

The user and group AEM should be installed and executed as. User and Group will be created by the role if necessary.

	aem_home: /opt/adobe/aem

The path AEM should be installed in.

	aem_instance_name: "{{ aem_home | basename }}"

The role will install a service script or systemd unit for managing AEM, depending on the init system of the target host. The name of this service is controlled by this variable. 
> Note that this role doesn't start AEM after installation, although it enables the service. This is to be able to tweak the installation before the first startup or orchestrate the startup within a cluster outside of the role. The [aem-service](https://github.com/wcm-io-devops/ansible-aem-service) role can be used to control startup and shutdown of AEM.

	aem_version: 6.2.0
	aem_version_short: "{{ aem_version | regex_replace('(\\d+\\.\\d+).*', '\\1') }}"
	aem_quickstart_name: "AEM_{{ aem_version_short }}_Quickstart.jar"

The version of AEM to install.

	aem_install_source: file
	aem_download_path: /tmp
	aem_remove_download: false

The installation source, i.e. where the installation Quickstart.jar should be retrieved from. This can either be `file` for a local file, `package` for a distribution package or `Nexus` for a Maven repository. If using a local file it needs to be copied someplace the Ansible `copy` module can find it. `aem_download_path` controls where the installation file will be downloaded to on the target host, and `aem_remove_download` whether the file will be deleted after installation.
	
	aem_package: aem{{ aem_version_short }}
	aem_package_home: "/path/of/package/installation"

If using a distribution package, `aem_package` must be set to the name of the package to install and `aem_package_home` to the directory the package will install AEM into. The role will use this to move the installation to `aem_home`.

	aem_mvn_coordinates:
	  - {
	  group_id: group.id,
	  artifact_id: artifact.id,
	  version: "{{ aem_version }}",
	  repository_url: 'https://repo.url'
	  }

Used to configure the Maven coordinates of the JAR artifact when using Nexus as installation source.

	aem_quickstart_install_fileglob: []
	
Fileglob(s) of files to copy to the `crx-quickstart/install` directory during setup. This allows you to provision OSGi configuration before the first startup, e.g. for changing the repository type.

	aem_limit_nofile: 20000

Sets the `nofile` limit for the AEM user.

## Dependencies

This role depends on the [williamyeh.oracle-java](https://galaxy.ansible.com/williamyeh/oracle-java/) role for installing Java.

## Example Playbook

Installs AEM in `/opt/adobe/aem-author`: 

    - hosts: aem-author
      roles:
         - { role: aem-cms,  aem_home: /opt/adobe/aem-author }

## License

Apache 2.0
