Role Name
=========

# Ansible apache solr server playbook
[![AnsibleTest](https://github.com/spy86/ansible-apache-solr/actions/workflows/main.yml/badge.svg)](https://github.com/spy86/ansible-apache-solr/actions/workflows/main.yml) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Requirements
------------

 - CentOS 6/7
 - Debian 
 - Ubuntu 16.04

Role Variables
--------------

Configuration file is splited to 4 sections:

- `download_dir` : The directory into which Solr will be downloaded for setup.
- `solr_dir` : The directory inside which Solr will be installed.
- `solr_version` : Solr version.
- `solr_checksum` : Solr download information.

Dependencies
------------

None

Example Playbook
----------------
```YAML
---
- hosts: all

  vars_files:
    - vars.yml

  pre_tasks:
    - name: Update apt cache if needed.
      apt: update_cache=yes cache_valid_time=3600

  handlers:
    - name: restart solr
      service: name=solr state=restarted

  tasks:
    - name: Install Java.
      apt: name=openjdk-8-jdk state=present

    - name: Download Solr.
      get_url:
        url: "https://archive.apache.org/dist/lucene/solr/{{ solr_version }}/solr-{{ solr_version }}.tgz"
        dest: "{{ download_dir }}/solr-{{ solr_version }}.tgz"
        checksum: "{{ solr_checksum }}"

    - name: Expand Solr.
      unarchive:
        src: "{{ download_dir }}/solr-{{ solr_version }}.tgz"
        dest: "{{ download_dir }}"
        copy: no
        creates: "{{ download_dir }}/solr-{{ solr_version }}/README.txt"

    - name: Run Solr installation script.
      shell: >
        {{ download_dir }}/solr-{{ solr_version }}/bin/install_solr_service.sh
        {{ download_dir }}/solr-{{ solr_version }}.tgz
        -i /opt
        -d /var/solr
        -u solr
        -s solr
        -p 8983
        creates={{ solr_dir }}/bin/solr

    - name: Ensure solr is started and enabled on boot.
      service: name=solr state=started enabled=yes
```
