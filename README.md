slm-ansible-role-wazuh-client
=============================

An Ansible role that provides reusable task entrypoints for interacting with the Wazuh Manager API.

This role currently behaves like a task library: consumers include specific task files with `tasks_from`.

What this role does
-------------------

- Authenticate against the Wazuh Manager API and expose a bearer token fact.
- Retrieve all agents from the manager.
- Retrieve one agent by IP.
- Wait until an agent reaches a desired status.
- Clear disconnected and never-connected agents.

Available task entrypoints
--------------------------

- `get_manager_access_token.yml`
- `get_agents.yml`
- `get_agent_by_ip.yml`
- `wait_for_agent_status.yml`
- `clear_agents.yml`

Requirements
------------

- Ansible controller with access to the Wazuh Manager API endpoint.
- Valid Wazuh API credentials (for token creation).
- HTTPS endpoint, for example `https://<wazuh-manager-ip>:55000`.

Notes:

- API calls in this role currently use `validate_certs: false`.
- `tasks/main.yml` is currently empty, so include explicit task files via `tasks_from`.

Role Variables
--------------

This role has no defaults yet. Provide variables where the task file is included.

### Common input variables

- `wazuh_client_manager_base_url` (required): Base URL of Wazuh API, for example `https://10.0.0.10:55000`.
- `wazuh_client_manager_access_token` (required by most tasks): Bearer token for Wazuh API.

### Authentication task inputs

- `wazuh_client_manager_username` (required by `get_manager_access_token.yml`)
- `wazuh_client_manager_password` (required by `get_manager_access_token.yml`)

### Agent-specific task inputs

- `wazuh_client_agent_ip` (required by `get_agent_by_ip.yml` and `wait_for_agent_status.yml`)
- `wazuh_client_agent_desired_status` (required by `wait_for_agent_status.yml`), for example `active`

Facts exported by this role
---------------------------

- `wazuh_client_manager_access_token`
  - Set by `get_manager_access_token.yml`

- `wazuh_client_manager_agents`
  - Set by `get_agents.yml`
  - Contains `json.data.affected_items` from `/agents`

- `wazuh_client_manager_agent_by_ip`
  - Set by `get_agent_by_ip.yml`
  - First element from `json.data.affected_items`

Usage Examples
--------------

### 1) Get manager access token

```yaml
- name: Get Wazuh API token
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Fetch token
      ansible.builtin.include_role:
        name: slm-ansible-role-wazuh-client
        tasks_from: get_manager_access_token.yml
      vars:
        wazuh_client_manager_base_url: "https://192.168.56.10:55000"
        wazuh_client_manager_username: "wazuh-wui"
        wazuh_client_manager_password: "<password>"
```

### 2) Get all agents

```yaml
- name: List Wazuh agents
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Fetch token
      ansible.builtin.include_role:
        name: slm-ansible-role-wazuh-client
        tasks_from: get_manager_access_token.yml
      vars:
        wazuh_client_manager_base_url: "https://192.168.56.10:55000"
        wazuh_client_manager_username: "wazuh-wui"
        wazuh_client_manager_password: "<password>"

    - name: Fetch agents
      ansible.builtin.include_role:
        name: slm-ansible-role-wazuh-client
        tasks_from: get_agents.yml
      vars:
        wazuh_client_manager_base_url: "https://192.168.56.10:55000"
        wazuh_client_manager_access_token: "{{ wazuh_client_manager_access_token }}"

    - name: Show agent count
      ansible.builtin.debug:
        msg: "Agents found: {{ wazuh_client_manager_agents | length }}"
```

### 3) Get one agent by IP

```yaml
- name: Get one Wazuh agent by IP
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Fetch token
      ansible.builtin.include_role:
        name: slm-ansible-role-wazuh-client
        tasks_from: get_manager_access_token.yml
      vars:
        wazuh_client_manager_base_url: "https://192.168.56.10:55000"
        wazuh_client_manager_username: "wazuh-wui"
        wazuh_client_manager_password: "<password>"

    - name: Fetch agent by IP
      ansible.builtin.include_role:
        name: slm-ansible-role-wazuh-client
        tasks_from: get_agent_by_ip.yml
      vars:
        wazuh_client_manager_base_url: "https://192.168.56.10:55000"
        wazuh_client_manager_access_token: "{{ wazuh_client_manager_access_token }}"
        wazuh_client_agent_ip: "192.168.56.20"

    - name: Show agent id
      ansible.builtin.debug:
        msg: "Agent ID: {{ wazuh_client_manager_agent_by_ip.id }}"
```

### 4) Wait for an agent status

```yaml
- name: Wait for Wazuh agent to become active
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Fetch token
      ansible.builtin.include_role:
        name: slm-ansible-role-wazuh-client
        tasks_from: get_manager_access_token.yml
      vars:
        wazuh_client_manager_base_url: "https://192.168.56.10:55000"
        wazuh_client_manager_username: "wazuh-wui"
        wazuh_client_manager_password: "<password>"

    - name: Wait for desired status
      ansible.builtin.include_role:
        name: slm-ansible-role-wazuh-client
        tasks_from: wait_for_agent_status.yml
      vars:
        wazuh_client_manager_base_url: "https://192.168.56.10:55000"
        wazuh_client_manager_access_token: "{{ wazuh_client_manager_access_token }}"
        wazuh_client_agent_ip: "192.168.56.20"
        wazuh_client_agent_desired_status: "active"
```

### 5) Clear disconnected or never connected agents

```yaml
- name: Remove stale Wazuh agents
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Fetch token
      ansible.builtin.include_role:
        name: slm-ansible-role-wazuh-client
        tasks_from: get_manager_access_token.yml
      vars:
        wazuh_client_manager_base_url: "https://192.168.56.10:55000"
        wazuh_client_manager_username: "wazuh-wui"
        wazuh_client_manager_password: "<password>"

    - name: Clear stale agents
      ansible.builtin.include_role:
        name: slm-ansible-role-wazuh-client
        tasks_from: clear_agents.yml
      vars:
        wazuh_client_manager_base_url: "https://192.168.56.10:55000"
        wazuh_client_manager_access_token: "{{ wazuh_client_manager_access_token }}"
```

Testing
-------

Molecule scenario is available in `molecule/default` and exercises all task entrypoints in `converge.yml`.

Run locally:

```bash
molecule test
```

Dependencies
------------

No runtime role dependencies are declared in `meta/main.yml` yet.

For Molecule, dependencies are defined in `molecule/default/requirements.yml` and include:

- `eclipse-slm/ansible-collection-slm` (git)
- `community.docker`
- `community.general`
- `ansible.posix`
- `fabos.molecule_kubevirt`
- `fabos.slm-ansible-role-docker`
- `slm.wazuh`

License
-------

Role files include SPDX headers with `MIT-0`. Align `meta/main.yml` license field accordingly before publishing.

Author
------

- Benjamin Goetz (Fraunhofer IPA)
