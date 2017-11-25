# Ansible Role: bamboo-agent

[![Build Status](https://img.shields.io/travis/mimacom/ansible-role-bamboo-agent.svg)](https://travis-ci.org/mimacom/ansible-role-bamboo-agent)

Installs a local or remote Bamboo agent (node) for a specific Atlassian Bamboo
master server.

## Requirements

You already must have an Atlassian Bamboo server, since Bamboo agents depend on
it. Before using this role, make sure you have enough licenses remote Bamboo
agents (if wanted) otherwise they won't show up in Bamboo.

## Role Variables

Available variables are listed below, along with default values (see
`defaults/main.yml`):

    install_jdk: true

Whether to install JDK or not.

    openjdk_version: 1.8.0

The version of openJDK to install (remote agent only).

    bamboo_agent_remote: false

Whether this is a remote agent or not.

    bamboo_master_version: ""
    bamboo_master_fqdn: ""
    bamboo_master_https: false
    bamboo_master_port: ""

Bamboo master connection (remote agent only)

    bamboo_master_user: bamboo

Service user for Bamboo master node (local agent only)

    bamboo_agent_user: bambooagent
    bamboo_agent_uid: 5000

Service user name and ID for systemd (remote agent only).

    bamboo_agent_application_folder: "/home/{{ bamboo_agent_user }}"
    bamboo_agent_data_folder: "/home/{{ bamboo_agent_user }}/bamboo-agent-home"

Path where to store application binaries and application data (remote agent
only).

    bamboo_agent_jvm_memory: 768m

Java heap space for remote bamboo agent (remote agent only).

    bamboo_agent_npmrc: ""
    bamboo_agent_maven_settings: ""

Contents of the service user's setting files for npm and maven.

    bamboo_agent_capabilities: []

A list of Bamboo capabilities. For remote agents, the capabilities will be added
as actual capabilities in the Bamboo remote agent, when `properties` is set.
Unfortunately this is not possible for local agents.

If you specify the `source` with a proper `name`, an according package is
installed. You must install the used package manager by yourself!

A capability has the values:

  - `name`
  - `source` (One of `repository` (means yum/apt/...), `unarchive-remote`, `npm`)
  - `symlinks` (List of symlinks to create for this binary. Set it to `[]` if
    you don't use it!  Subkeys are `src` and `dest`)
  - `binary_path` (Value will be added to $PATH for all users and Bamboo agent
    service user)
  - `extract_path` (Path where to extract, when `source: unarchive-remote`)
  - `properties` (List of properties which are set on remote agents. Subkeys are
    `key` and `value`)

## Other notes for npm dependencies

If you configure to install npm, you must make sure to install node by
yourself. Please note: If you install node to a non-default PATH, make
sure to define the "symlinks" sub-variable and point the destination of
npm and node to a default-path like /usr/bin/.

Simply setting "binary_path", which would add the path to $PATH variable
doesn't work since ansible npm_module ignores it!

## Deprecation warning

  - dicts `bamboo_master` and `bamboo_agent` are deprecated and will be removed
    in future releases. Please consult README.
  - `bamboo_agent_capabilities` will not install packages anymore in the future.
    It will be used only for setting capability variables for remote agents, and
    set binary paths.
    Please use `pre_tasks` or `post_tasks` in your playbook, to install using
    your own tasks.

## Dependencies

Normally none. But if you use this role for a local bamboo agent, it is
recommended to use `mimacom.bamboo` role.

## Example playbook

This installs a Bamboo remote agent. The binary will be fetched from the Bamboo
master node (according to `bamboo_master_*` variables). The jar will be setup as
a systemd service.

Nodejs as a capability will be downloaded, extracted,
symlinked and the binary path will be added to the systemd service in order for
the agent to find nodejs.

Angular CLI as a second capability will be installed using npm. No symlinks are
created, and a custom capability will be set on the remote agent according to its
`properties`.

    - hosts: build-agents
      become: yes
      roles:
        - role: mimacom.bamboo-agent
          bamboo_agent_remote: true
          bamboo_master_version: 6.2.2
          bamboo_master_fqdn: "https://bamboo.company.example/
          bamboo_master_https: true
          bamboo_master_port: 443
          bamboo_agent_capabilities:
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
              properties:
                - key: system.builder.node.node-7
                  value: /opt/nodejs/node-v7/bin/node

            - name: "@angular/cli"
              source: npm
              symlinks: []
              properties:
                - key: path.has.ng
                  value: "true"

## Upgrade Bamboo remote agent

Bamboo remote agents will not be upgraded when you change the
`bamboo_master_version` variable. Each remote agent will update itself once it
has detected a new version on the master node. So, simply worry about
upgrading the Bamboo master :-)

## License

Apache License 2.0

## Author Information

This role was created by [Remo Wenger](http://www.remowenger.ch).
