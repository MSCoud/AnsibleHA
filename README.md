# 🛠️ Ansible HA Deployment: 2048 Game with NGINX & HAProxy

This project uses **Ansible** to automate the deployment of the [2048 game](https://github.com/ttwthomas/2048) on two web servers behind a **HAProxy** load balancer.

---

## 📦 Project Structure

```
.
├── inventory.yml              # Ansible inventory with web and loadbalancer hosts
├── playbook.yml              # Main playbook to set up NGINX and HAProxy
└── roles
    └── nginx_2048
        └── tasks
            └── main.yml     # Role for deploying the 2048 game and configuring NGINX
```

---

## 🚀 Features

- 📦 Installs and configures **NGINX** on `web1` and `web2`
- ⬇️ Clones the 2048 game from GitHub
- ⚙️ Updates NGINX root path to serve the 2048 app
- 🔁 Restarts NGINX after deployment
- ⚖️ Sets up **HAProxy** as a load balancer in front of the two web servers
- 🛠️ Uses community role: `geerlingguy.haproxy`

---

## 📋 Prerequisites

- Ansible 2.10+
- Python >= 3.6 on target hosts
- SSH access to all servers
- `geerlingguy.haproxy` Ansible role installed:
  ```bash
  ansible-galaxy install geerlingguy.haproxy
  ```

---

## 📁 Inventory Configuration (`inventory.yml`)

```yaml
all:
  children:
    web:
      hosts:
        web1:
          ansible_host: 10.141.17.100
        web2:
          ansible_host: 10.141.17.12

    loadbalancer:
      hosts:
        haproxy:
          ansible_host: 10.141.17.43
```

---

## 🧾 Playbook (`playbook.yml`)

```yaml
- name: Setup NGINX and Deploy APP
  hosts: web
  become: yes
  roles:
    - nginx_2048

- name: Setup HAProxy Load Balancer
  hosts: loadbalancer
  become: yes
  roles:
    - role: geerlingguy.haproxy
  vars:
    haproxy_backend_httpchk: ""
    haproxy_backend_servers:
      - name: web1
        address: "{{ hostvars['web1']['ansible_host'] }}:80"
      - name: web2
        address: "{{ hostvars['web2']['ansible_host'] }}:80"
```

---

## 📜 nginx_2048 Role Tasks

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Install git
  apt:
    name: git
    state: present

- name: Clone the App
  git:
    repo: https://github.com/ttwthomas/2048.git
    dest: /var/www/2048
    update: yes
    version: master
    force: yes

- name: Update the root of nginx
  replace:
    path: /etc/nginx/sites-enabled/default
    regexp: 'root /var/www/html;'
    replace: 'root /var/www/2048/app;'

- name: Change the owner of the web site directory
  file:
    path: /var/www/2048/app
    mode: 0755
    recurse: yes

- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

---

## ▶️ Running the Playbook

```bash
ansible-playbook -i ./inventory.yml playbook.yml
```

---

## ✅ Example Output

```
web1                       : ok=7    changed=5
web2                       : ok=7    changed=5
haproxy                    : ok=8    changed=4
```

---

## 📡 Access the Application

After running the playbook:

- Go to the IP of your **HAProxy server** (e.g., `http://10.141.17.43`)
- The load balancer will distribute traffic between `web1` and `web2`

---

## 🧑‍💻 Author

**Abdelilah AMRINI**  
📧 [amrini.abdelilah@gmail.com](mailto:amrini.abdelilah@gmail.com)

---

## 📜 License

This project is open-source and free to use under the MIT License.

