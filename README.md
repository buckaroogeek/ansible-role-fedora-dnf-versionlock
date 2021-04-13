Ansible Role - Fedora DNF Versionlock Plugin
=========

Implements basic subcommends of the DNF versionlock plugin. Does not provide ability to add DNF options.

Requirements
------------

Fedora is the only platform tested at the moment. It should work for any linux system that uses DNF for package management.

Role Operation
--------------

The dnf versionlock plugin manages a list of rpm package names and versions that affect the list of packages that dnf will include in a transaction. The plugin will 'lock' the listed packages to the listed versions. See the playbook examples below for more details.

The plugin operates using commands and a package list (specification). The role operates with a command (fdv_cmd) and a package specification (fdv_package_spec). The default command without a package spec is `list`. The default command with a package spec is `add`.

The role will install the plugin if not present. Set `fdv_present` to `false` to remove the plugin.

The role can be disabled, but not removed by setting `fdv_enabled` to false.

The exclude command 'adds' the package spec to the list versionlock manages, but is used to exclude all matching package names from any dnf transaction.

Role Variables
--------------

Variables that can be set in a playbook are listed below, along with default values if any. See `defaults/main.yaml` for more information. None are mandatory.

| Variable   | Default Value | Type | Notes |
| ---------- | ------------- | ----- | ----- |
| fdv_cmd | See Operation above  | string | One of add, delete, exclude, list, clear |
| fdv_package_spec | none    | string | Required when fdv_cmd is one of add, delete, exclude. The package specification for versionlock to act on |
| fdv_raw | false    | boolean | Enables --raw option if true |
| fdv_enabled | true | boolean | Enables (true) or disables (false) use of local repository |
| fdv_present | true | boolean | If false, removes dnf versionlock plugin rpm from system |

Technical Notes
---------------

Be sure to read and be familiar with the online documentation at https://dnf-plugins-core.readthedocs.io/en/latest/versionlock.html. From the description: "versionlock is a plugin that takes a set of names and versions for packages and excludes all other versions of those packages. This allows you to protect packages from being updated by newer versions. Alternately, it accepts a specific package version to exclude from updates, e.g. for when it’s necessary to skip a specific release of a package that has known issues."

The specific impetus for this role is to lock upstream kubernetes packages (kubectl, kubeadm, kubelet) to the same version as cri-o from the Fedora repositories. Version control of cri-o in Fedora is handled by modularity. This plugin can lock a package to a specific minor version (e.g. 1.20) but allow patch releases to be installed without manual intervention (e.g. 1.20.1, 1.20.2, etc.).

Wildcard characters (?, \*) are allowed in the `fdv_package_spec names`. The versionlock plugin will do a pattern match and add all packages which match the pattern into the versionlock configuration. If the 'raw' option is enabled, pattern matching is bypassed. For example, without 'raw', the pattern 'kube???-1.20.\*' will add kubectl-1.20.5, kubeadm-1.20.5, and kubelet-1.20.5 to the configuration. This will lock these 3 packages to the specific patch version (.e. 1.20.5). I assume the upstream kubernetes repo is enabled. If the 'raw' option is enabled, then the configuration will contain 'kube???-1.20.\*'. This allows dnf to install the current patch level (5), and when available update to future patches such as 1.20.6 or 1.20.7 when they are added to the upstream repository.

Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

Bradley G Smith
