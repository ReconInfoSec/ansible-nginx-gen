# ansible-nginx-gen

This role/playbook will generate TCP/UDP stream configuration files for NGINX based on the backend servers and ports provided, and update your AWS security groups to allow access to those ports.

This example uses 3 NGINX servers (nginx01/02/03) and 3 backend servers in 3 AZs (backend_servers_a/b/c) weighted with regards to their zone.

* Populate `hosts` file with your NGINX servers and connection params
* In `main.yml`:
  * Replace `172.20` addresses with the addresses of your backend servers, those are placeholders
  * Populate the TCP/TLS/UDP ports lists as needed
  * Populate `cert_path` and `key_path` to point to your cert and key
* If using AWS
  * Populate the CIDR you need to allow access from and security group IDs, if using AWS, otherwise set `aws` to `false`
  * Make sure you have the aws cli installed, `pip install awscli`, and configured with your credentials
* This generates TCP/UDP streams only, not HTTP, so you must have the following in your NGINX config somewhere:
  ```
  stream {
          include /etc/nginx/streams/*;
  }
  ```

### Example playbook, `main.yml`:

To run it:

```
ansible-playbook main.yml
```

Contents:
```
---

- name: Add new servers and ports to NGINX stream configs
  hosts:
    - nginx01
    - nginx02
    - nginx03
  vars:
    fail_timeout: 10s
    max_fails: 3
    cert_path: /etc/letsencrypt/live/mydomain.com/fullchain.pem
    key_path: /etc/letsencrypt/live/mydomain.com/privkey.pem
    tcp: [ "5044" ]
    tcp_tls: [ "5045" ]
    udp: [ "6514" ]
    backend_servers_a:
      172.20.1.11:
        weight: 5
      172.20.2.12:
        weight: 1
      172.20.3.13:
        weight: 1
    backend_servers_b:
      172.20.1.11:
        weight: 1
      172.20.2.12:
        weight: 5
      172.20.3.13:
        weight: 1
    backend_servers_c:
      172.20.1.11:
        weight: 1
      172.20.2.12:
        weight: 1
      172.20.3.13:
        weight: 5
    aws: true
    cidr: 0.0.0.0\0
    tcp_sg: "sg-xxxxxxxx"
    udp_sg: "sg-xxxxxxxx"
  roles:

    - { role: gen-nginx }
```
