# wcm_io_devops.aem_cms

This role installs Adobe Experience Manager (AEM) 6.x on Debian/Ubuntu or RHEL/CentOS servers.
> This role was developed as part of the
> [wcm.io DevOps Ansible Automation for AEM](http://devops.wcm.io/ansible-aem/)
> to integrate Ansible with [CONGA](http://devops.wcm.io/conga/) but can
> be used independently of it.

## Requirements

This role requires Ansible 2.7 or higher and works with AEM 6.1 or higher. Also required are an AEM quickstart JAR file and a valid AEM license file. The `license.properties` files needs be made accessible to the role, normally by copying it into the `files` folder in the playbook directory. The `AEM_*_Quickstart.jar` can be supplied in the same way or retrieved from a Maven/RPM/APT repository, an HTTP URL or a S3 bucket (see below).

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

	aem_cms_group: adobe
	aem_cms_user: adobe

The user and group AEM should be installed and executed as. User and Group will be created by the role if necessary.

	aem_cms_home: /opt/adobe/aem

The path AEM should be installed in.

	aem_cms_service_name: "{{ aem_cms_home | basename }}"

The role will install a service script or systemd unit for managing AEM, depending on the init system of the target host. The name of this service is controlled by this variable. 
> Note that this role doesn't start AEM after installation, although it enables the service. This is to be able to tweak the installation before the first startup or orchestrate the startup within a cluster outside of the role. The [wcm_io_devops.aem_service](https://github.com/wcm-io-devops/ansible-aem-service) role can be used to control startup and shutdown of AEM.

	aem_cms_version: 6.2.0
	aem_cms_version_short: "{{ aem_cms_version | regex_replace('(\\d+\\.\\d+).*', '\\1') }}"
	aem_cms_quickstart_name: "AEM_{{ aem_cms_version_short }}_Quickstart.jar"
	aem_cms_quickstart_sha1:

The version of AEM to install, filename of the Quickstart.jar and optional SHA1 checksum of the JAR for verification.

	aem_cms_install_source: file
	aem_cms_download_path: /tmp
	aem_cms_remove_download: false

The installation source, i.e. where the installation Quickstart.jar should be retrieved from. This can either be `file` for a local file, `package` for a distribution package , `url` for a generic URL, `s3` for an object from a S3 bucket or `maven_repository` for a Maven repository. If using a local file it needs to be copied someplace the Ansible `copy` module can find it. `aem_cms_download_path` controls where the installation file will be downloaded to on the target host, and `aem_cms_remove_download` whether the file will be deleted after installation.
	
	aem_cms_package: aem{{ aem_cms_version_short }}
	aem_cms_package_home: "/path/of/package/installation"

If using a distribution package, `aem_cms_package` must be set to the name of the package to install and `aem_cms_package_home` to the directory the package will install the AEM `Quickstart.jar` into. The role will use this to move the installation to `aem_cms_home`.

	aem_cms_maven_repository_coordinates:
	  - {
		  group_id: group.id,
		  artifact_id: artifact.id,
		  version: "{{ aem_cms_version }}",
		  repository_url: 'https://repo.url'
		}
		aem_cms_maven_repository_username: <username>
		aem_cms_maven_repository_password: <password>

Used to configure the Maven coordinates of the JAR artifact and repository credentials when using Maven repository as installation source.

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

    aem_cms_dependency_java: true

Controls if Java is installed by using [srsp.oracle-java](https://galaxy.ansible.com/srsp/oracle-java/) role for installing Java.

    aem_cms_in_place_upgrade: false

Enables / disables in-place-upgrade.

:exclamation: Warnings
* Use the in place upgrade functionality with care and test it in a
  staging environment before applying it to production!
* Do not use this mechanism when a content repository migration is
  necessary!
* Do not use this mechanism when upgrading from AEM 6.2 to AEM 6.3+

```
aem_cms_in_place_upgrade_paths:
  "6.3.0":
    - "6.4.0"
```

Format:

    "from_version":
        - "to_version" # list of versions that an upgrade is allowed for

Specifies from and to versions which are supported by the
in-place-upgrade mechanism.

    aem_cms_stop_timeout_seconds: 1200

Seconds to wait for instance to be stopped until process is killed.

    aem_cms_systemd_unit_template: "aem.service.j2"

Path to the systemd unit template. Use this variable to specify a custom
template.

    aem_cms_sysvinit_service_template: "aem.init.j2"

Path to the sysvinit service template. Path to the systemd unit
template. Use this variable to specify a custom template.

    aem_cms_stop_sync_template: "stop-sync.sh.j2"

Path to the synchronous stop script template. Use this variable to
specify a custom template.

    aem_cms_stop_sync_path: "{{ aem_cms_home }}/crx-quickstart/bin/stop-sync.sh"

Destination path of the synchronous stop script on the instance.

## Dependencies

This role depends on the
[srsp.oracle-java](https://galaxy.ansible.com/srsp/oracle-java/) role for
installing Java.

## Example Playbook

Installs AEM in `/opt/adobe/aem-author`: 

    - hosts: aem-author
      roles:
         - { role: wcm_io_devops.aem_cms,  aem_cms_home: /opt/adobe/aem-author }

## License

Apache 2.0
