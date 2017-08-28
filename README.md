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
bamboo_agent: # A list of capabilities of this agent. See below.
bambooagent_user: # Linux user on which the bamboo agent should run
bambooagent_version: # Version which should be installed. This *must* be supported by the bamboo master server, because bamboo agent executable will be downloaded from it.
bambooagent_root_directory: # Linux's bambooagent_user home directory
bambooagent_home_directory: # Data folder for bamboo agent, should be underneath bambooagent_root_directory.
openjdk_version: # OpenJDK is only used for running bamboo itself, not for building projects hosted on bamboo. Make sure that this matches' the bamboo agent version's supported platforms.
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

Dependencies
------------

The following mimacom roles should be applied in this order:
* common
