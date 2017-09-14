bamboo-agent
============

This role will install a bamboo agent (node) for a specific bamboo master server.

Requirements
------------

You already must have a bamboo server, since bamboo agents depends on it.
Before you use this role, make sure you have enough licenses for bamboo agents, otherwise they won't show up in bamboo.

Role Variables
--------------

These role variables are mandatory:
```yaml
openjdk_version: # OpenJDK is only used for running bamboo itself, not for building projects hosted on bamboo. Make sure that this matches' the bamboo agent version's supported platforms.

bamboo_agent: # Custom setings for the bamboo agent
  remote: # true/false. Installs remote agent binary when true
  npmrc: # Configure .npmrc file at bamboo agents home
  maven_settings: # Configures .m2/settings.xml file at bamboo agents home

  capabilities: # A list of capabilities whose are installed additionally
    - name: # Name of the package to install
      source: # repository/unarchive-remote/npm. Defines where to grab binaries to install
              # repository means Linux package manager, like yum or apt
      symlinks: # List of symlinks to create for this binary. Set it to
                # [] if you don't use it!
        - src: # Source path of symlink
          dest: # Destination path of symlink
      binary_path: # Add this binary to $PATH variable for all users
      extract_path: # Extract to this folder (only when source=unarchive-remote)
      properties: # List of properties which are set on (remote!) agents.
        - key: # Key of bamboo agent property
          value: # Value of bamboo agent property
```

Fill the variable "bamboo_agent" with your capabilities. A capability is defined in bamboo itself for each remote bamboo agent. It defines what the agent is able to build, ie. what build tools are installed on it. Like maven, g++, Oracle JDK and so on. Use these variables for each entry/capability:

```yaml
name: # Name of a yum package, a role's file or an URL to an RPM or TAR.GZ package which should be installed
type: # Capability type as it is called in bamboo itself. Only for your purpose, not used in the role itself.
source: This is either 'unarchive' for a role file's archive, 'unarchive-remote' for an archive which should be downloaded from the internet, and 'repository' if it should be installed by yum.
binary_path: Optional. If defined, this will be added to every Linux's users PATH environment variable
extract_path: Optional. The archive will be extracted in here after downloading.
symlink_src: Optional. If you want your software softlinked to a specific path, use this and 'symlink_dest'
symlink_dest: See symlink_src
note: Notes about this capability. Only for your purpose, not used in the role itself.
```

Other notes for npm dependencies
-----------
If you configure to install npm, you must make sure to install node by
yourself. Please note: If you install node to a non-default PATH, make
sure to define the "symlinks" sub-variable and point the destination of
npm and node to a default-path like /usr/bin/.

Simply setting "binary_path", which would add the path to $PATH variable
doesn't work since ansible npm_module ignores it!


Example:

```
- hosts: bamboo-agents
  vars:
    bamboo_agent:
      capabilities:
        - name: https://nodejs.org/dist/v7.7.4/node-v7.7.4-linux-x64.tar.gz
          source: unarchive-remote
          binary_path: /opt/nodejs/node-v7/bin/
          extract_path: /opt/nodejs/
          symlinks:
            - src: /opt/nodejs/node-v7.7.4-linux-x64
              dest: /opt/nodejs/node-v7

            - src: /opt/nodejs/node-v7.7.4-linux-x64/bin/npm
              dest: /usr/bin/npm

            - src: /opt/nodejs/node-v7.7.4-linux-x64/bin/node
              dest: /usr/bin/node

      - name: "@angular/cli"
        source: npm
        symlinks: []
        properties:
          - key: path.has.ng
            value: "true"
```

Dependencies
------------

The following mimacom roles should be applied in this order:
* common
