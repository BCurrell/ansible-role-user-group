# Ansible Role - User + Group

Ansible role to control users and groups on non-Windows hosts.

- [Ansible Docs - ansible.builtin.user](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)
- [Ansible Docs - ansible.builtin.group](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/group_module.html)

Windows hosts require [ansible.windows.win_user](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_user_module.html) and [ansible.windows.win_group](https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_group_module.html), which this role does not use.

I've also added the ability to control a user's authorized_keys file without using [ansible.posix.authorized_key](https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html). There's no guarantee that collection is available, and I don't fully agree with the way it has been developed. However, this is just an initial attempt, and I would like to add more functionality in the future.

## Usage

**NOTE**: This role works by defining all of the Ansible module arguments within the inventory, with minimal parsing. This is unsafe (see [Ansible Docs - argsplat](https://docs.ansible.com/ansible/devel/reference_appendices/faq.html#argsplat-unsafe)). There is a way around this by disabling `INJECT_FACTS_AS_VARS` (see [Ansible Docs - config](https://docs.ansible.com/ansible/devel/reference_appendices/config.html#inject-facts-as-vars)). This prevents the host facts from being injected as variables. They are still accessible via `ansible_facts`.

This role is not currently on Ansible Galaxy. I'm not sure if it every will be. Instead, I recommend using a Git submodule to add this to your playbook:

```none
git submodule add https://github.com/BCurrell/ansible-role-user_group.git roles/user_group
```

To pull in new changes:

```none
git submodule update --remote --recursive --checkout
```

To remove the submodule:

```none
git rm roles/user_group
```

When the submodule is cloned, you can use it like any other Ansible role:

```yaml
---
- name: 'Example Playbook'
  hosts: 'all'

  roles:
    # Option 1 - Pass the name as a string
    - 'user_group'
    # Option 2 - Pass the name to they 'role' key with additional arguments
    - role: 'user_group'
      vars: {}

  tasks:
    # Option 3 - Use the ansible.builtin.include_role module
    - name: 'Include user_group role'
      ansible.builtin.include_role:
        name: 'user_group'
      vars: {}
```

## Development

To avoid installing global packages to my OS, I use Poetry (v2.0 with PEP508 support) to install my development requirements:

- `pre-commit` to automate formatting and linting
- `ansible-lint` to lint the Ansible role
- `black`, `isort`, `flake8`, etc to format / lint any Python (e.g. filter plugins)

To install the development virtual environment, run `poetry install -E dev`.

If you make a PR, please run `pre-commit` if possible in one of the following ways, in order of recommendation / preference:

- `poetry run pre-commit run` (after every `git add`)
    - Use the poetry environment to run pre-commit once.
- `poetry run pre-commit install` (once after `git clone`)
    - Use the poetry environment to install pre-commit hooks to .git, meaning pre-commit will automatically run when you stage and commit changes to Git.
- `pre-commit run` (after every `git add`)
    - Use a globally installed pre-commit to run pre-commit once.
- `pre-commit install` (once after `git clone`)
    - Use a globally installed pre-commit to install pre-commit hooks to .git, meaning pre-commit will automatically run when you stage and commit changes to Git.

## Notes

You can pop an item from a dict and maintain the updated dict by doing the following:

```yaml
- name: 'user_group:user:testing'
  ansible.builtin.set_fact:
    popped: "{{ user_instance.pop('popme', '') }}"
    user_instance: "{{ user_instance | dict2items | rejectattr('key', 'equalto', 'popme') | items2dict }}"
```

```none
TASK [user_group : user_group:user:debug] *****************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "user": {
        "name": "test-absent-02",
        "popme": "popped",
        "state": "absent"
    }
}

TASK [user_group : user_group:user:testing] ***************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [user_group : user_group:user:debug] *****************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "user_instance": {
        "force": false,
        "name": "test-absent-02",
        "remove": false,
        "state": "absent"
    }
}

TASK [user_group : user_group:user:debug] *****************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "popped": "popped"
}
```
