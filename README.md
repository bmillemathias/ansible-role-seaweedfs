egeneralov.seaweedfs
====================

Provision installation for [seaweedfs](https://github.com/chrislusf/seaweedfs)

Requirements
------------

Debian.

Role Variables
--------------

- **domain**: `weed.domain.tld`
- **weed**:
  - **version**: `0.76`
  - **bind**: `127.0.0.1`
  - **ip**: `127.0.0.1`
  - **destination**: `/usr/local/sbin`
  - **master**:
    - **dir**: `/opt/seaweedfs/{{ domain }}/master`
  - **volume**:
    - **dir**: `/opt/seaweedfs/{{ domain }}/volume`
    - **dataCenter**: `DefaultDataCenter`
    - **rack**: `DefaultRack`
- **download_url**: `https://github.com/chrislusf/seaweedfs/releases/download/{{ weed.version }}/{{ ansible_system|lower }}_{{ arch[ansible_architecture] }}.tar.gz`
- **manage_iptables**: `false`

Example Playbook
----------------

    ---
    
    - hosts: weed.domain.tld
      gather_facts: no
    #   no_log: true
      vars:
        domains:
          - weed.domain.tld
          - master-weed.domain.tld
      tasks:
    
        - sysctl:
            name: "{{ item }}"
            value: 1
            sysctl_set: yes
            state: present
            reload: yes
          with_items:
            - net.ipv6.conf.lo.disable_ipv6
            - net.ipv6.conf.all.disable_ipv6
            - net.ipv6.conf.default.disable_ipv6
    
        - file:
            path: /etc/nginx/ssl/
            state: directory
    
        - apt:
            name:
              - python-pip
              - python-setuptools
              - openssl
            update_cache: yes
            cache_valid_time: 3600
    
        - pip:
            name:
              - setuptools
              - pyopenssl
    
        - openssl_privatekey:
            path: "/etc/nginx/ssl/{{ item }}.key"
          with_items: "{{ domains }}"
    
        - openssl_csr:
            path: "/etc/nginx/ssl/{{ item }}.csr"
            privatekey_path: "/etc/nginx/ssl/{{ item }}.key"
            common_name: "{{ item }}"
          with_items: "{{ domains }}"
    
        - openssl_certificate:
            path: "/etc/nginx/ssl/{{ item }}.crt"
            privatekey_path: "/etc/nginx/ssl/{{ item }}.key"
            csr_path: "/etc/nginx/ssl/{{ item }}.csr"
            provider: selfsigned
          with_items: "{{ domains }}"
    
        - file:
            path: /usr/local/share/ca-certificates/extra
            state: directory
    
        - copy:
            remote_src: yes
            src: "/etc/nginx/ssl/{{ item }}.crt"
            dest: "/usr/local/share/ca-certificates/extra/{{ item }}.crt"
          with_items: "{{ domains }}"
          register: copy
    
        - shell: update-ca-certificates
          changed_when: copy is changed
    
    
    
    - hosts: weed.domain.tld
    #   no_log: true
      vars:
        manage_iptables: yes
        vhosts:
          - domain: "weed.domain.tld"
            ssl: true
            rewrite_ssl: true
            rewrite_www: false
            ssl_certificate: /etc/nginx/ssl/weed.domain.tld.crt
            ssl_certificate_key: /etc/nginx/ssl/weed.domain.tld.key
            root: /opt/seaweedfs/weed.ccbh.tf
            client_max_body_size: 100m
            locations:
              - path: /
                type: proxy
                schema: "http://"
                proxy_to:
                  - "127.0.0.1:8080"
          - domain: "master-weed.domain.tld"
            ssl: true
            rewrite_ssl: true
            rewrite_www: false
            ssl_certificate: /etc/nginx/ssl/master-weed.domain.tld.crt
            ssl_certificate_key: /etc/nginx/ssl/master-weed.domain.tld.key
            root: /opt/seaweedfs/weed.ccbh.tf
            client_max_body_size: 100m
            locations:
              - path: /
                type: proxy
                schema: "http://"
                proxy_to:
                  - "127.0.0.1:9333"
        domain: weed.domain.tld
        weed:
          version: 0.76
          destination: /usr/local/sbin
          master:
            dir: "/opt/seaweedfs/{{ domain }}/master"
          volume:
            dir: "/opt/seaweedfs/{{ domain }}/volume"
            dataCenter: "fra1"
            rack: "rack1"
      roles:
        - egeneralov.sshd_config
        - egeneralov.iptables
        - egeneralov.seaweedfs
        - egeneralov.nginx

License
-------

MIT

Author Information
------------------

Eduard Generalov <eduard@generalov.net>
