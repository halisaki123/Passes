#- include_vars: "{{ user_name }}.yml"
#  with_first_found:
#  - "{{ user_name }}.yml"
#  - default.yml

- name: create group {{ user_group }}
  group: name={{ user_group }} gid={{ user_id }} state={{ user_state }}
  when: '"present" in user_state'
  tags:
  - users
  ignore_errors: True

- name: create/delete {{ user_name }} with NOT defined home
  user: name={{ user_name }} uid={{ user_id }} createhome=yes group={{ user_group }} password={{ user_name }}_user_password state={{ user_state }} shell={{ user_shell }} force=yes
  tags:
  - users
  when: 'user_home is not defined'

- name: create/delete {{ user_name }} with defined home
  user: name={{ user_name }} uid={{ user_id }} createhome=yes group={{ user_group }} password={{ user_name }}_user_password state={{ user_state }} shell={{ user_shell }} home={{ user_home }} force=yes
  tags:
  - users
  when: 'user_home is defined'

- name: delete group {{ user_group }}
  group: name={{ user_group }} gid={{ user_id }} state={{ user_state }}
  when: '"absent" in user_state'
  tags:
  - users

- name: symlink back to /home
  file: src={{ user_home }} dest=/home/{{ user_name }} state=link owner={{ user_name }} group={{ user_name }}
  tags:
   - users
  when: 'user_home is defined'

- name: add {{ user_name }} to {{ user_groups }}
  user: name={{ user_name }} groups={{ user_groups }} state={{ user_state }}
  tags:
  - users
  when: user_groups is defined

- name: /home/{{ user_name }} to have mode 0755
  file: path=/home/{{ user_name }} state=directory mode=0755
  tags:
  - users

- name: create /home/{{ user_name }}/.ssh
  file: path=/home/{{ user_name }}/.ssh mode=0700 owner={{ user_name }} group={{ user_name }} state=directory
  when: user_state == 'present' and real_user is defined
  tags:
  - users
  - users-sshkey

- name: create /home/{{ user_name }}/.ssh/authorized_keys
  copy: src={{ user_name }}/authorized_keys dest=/home/{{ user_name }}/.ssh/authorized_keys mode=0600 owner={{ user_name }} group={{ user_name }}
  when: user_state == 'present' and real_user is defined and user_name != 'root'
  tags:
  - users
  - users-sshkey

- name: create /home/{{ user_name }}/.ssh/key-{{ user_name }}
  copy: src={{ user_name }}/{{ user_name }} dest=/home/{{ user_name }}/.ssh/key-{{ user_name }} owner={{ user_name }} group={{ user_name }} mode=0600
  tags:
  - users
  - users-sshkey
  when: user_ssh_key is defined

- name: create /home/{{ user_name }}/.ssh/id_rsa
  file: src=/home/{{ user_name }}/.ssh/key-{{ user_name }} dest=/home/{{ user_name }}/.ssh/id_rsa state=link owner={{ user_name }} group={{ user_name }}
  tags:
  - users
  - users-sshkey
  when: user_ssh_key is defined
