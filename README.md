# wcm-io-devops.aem_cms

This role installs Adobe Experience Manager (AEM) 6.x on Debian/Ubuntu or RHEL/CentOS servers.
> This role was developed as part of the wcm.io set of roles to integrate Ansible with [CONGA](http://devops.wcm.io/conga/) but can be used independently of it.

## Requirements

This role requires Ansible 2.2 or higher and works with AEM 6.1 or higher. Also required are an AEM quickstart JAR file and a valid AEM license file. The `license.properties` files needs be made accessible to the role, normally by copying it into the `files` folder in the playbook directory. The `AEM_*_Quickstart.jar` can be supplied in the same way or retrieved from a Nexus/RPM/APT repository, an HTTP URL or a S3 bucket (see below).

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

	aem_cms_group: adobe
	aem_cms_user: adobe

The user and group AEM should be installed and executed as. User and Group will be created by the role if necessary.

	aem_cms_home: /opt/adobe/aem

The path AEM should be installed in.

	aem_cms_service_name: "{{ aem_cms_home | basename }}"

The role will install a service script or systemd unit for managing AEM, depending on the init system of the target host. The name of this service is controlled by this variable. 
> Note that this role doesn't start AEM after installation, although it enables the service. This is to be able to tweak the installation before the first startup or orchestrate the startup within a cluster outside of the role. The [wcm-io-devops.aem_service](https://github.com/wcm-io-devops/ansible-aem-service) role can be used to control startup and shutdown of AEM.

	aem_cms_version: 6.2.0
	aem_cms_version_short: "{{ aem_cms_version | regex_replace('(\\d+\\.\\d+).*', '\\1') }}"
	aem_cms_quickstart_name: "AEM_{{ aem_cms_version_short }}_Quickstart.jar"
	aem_cms_quickstart_sha1:

The version of AEM to install, filename of the Quickstart.jar and optional SHA1 checksum of the JAR for verification.

	aem_cms_install_source: file
	aem_cms_download_path: /tmp
	aem_cms_remove_download: false

The installation source, i.e. where the installation Quickstart.jar should be retrieved from. This can either be `file` for a local file, `package` for a distribution package , `url` for a generic URL, `s3` for an object from a S3 bucket or `Nexus` for a Maven repository. If using a local file it needs to be copied someplace the Ansible `copy` module can find it. `aem_cms_download_path` controls where the installation file will be downloaded to on the target host, and `aem_cms_remove_download` whether the file will be deleted after installation.
	
	aem_cms_package: aem{{ aem_cms_version_short }}
	aem_cms_package_home: "/path/of/package/installation"

If using a distribution package, `aem_cms_package` must be set to the name of the package to install and `aem_cms_package_home` to the directory the package will install the AEM `Quickstart.jar` into. The role will use this to move the installation to `aem_cms_home`.

	aem_cms_nexus_coordinates:
	  - {
		  group_id: group.id,
		  artifact_id: artifact.id,
		  version: "{{ aem_cms_version }}",
		  repository_url: 'https://repo.url'
		}
		aem_cms_nexus_username: <username>
		aem_cms_nexus_password: <password>

Used to configure the Maven coordinates of the JAR artifact and Nexus credentials when using Nexus as installation source.

	aem_cms_url: "http://host:port/path/{{ aem_cms_quickstart_name }}"
	aem_cms_url_username: <username>
	aem_cms_url_password: <password>

URL and optional authentication parameters to retrieve the AEM Quickstart.jar from when using the URL installation source.

	aem_cms_s3_bucket: <AWS bucket name>
	aem_cms_s3_object: "{{ aem_cms_quickstart_name }}"
	aem_cms_s3_access_key: <AWS access key>
	aem_cms_s3_secret_key: <AWS secret key>

Bucket and object name of the AEM Quickstart.jar and optional AWS credentials for retrieving the installation file from the S3 source. 

	aem_cms_quickstart_install_fileglob: []
	
Fileglob(s) of files to copy to the `crx-quickstart/install` directory during setup. This allows you to provision OSGi configuration before the first startup, e.g. for changing the repository type.

	aem_cms_limit_nofile: 20000

Sets the `nofile` limit for the AEM user.

## Dependencies

This role depends on the
[srsp.oracle-java](https://galaxy.ansible.com/srsp/oracle-java/) role for
installing Java.

## Example Playbook

Installs AEM in `/opt/adobe/aem-author`: 

    - hosts: aem-author
      roles:
         - { role: wcm-io-devops.aem_cms,  aem_cms_home: /opt/adobe/aem-author }

## License

Apache 2.0
