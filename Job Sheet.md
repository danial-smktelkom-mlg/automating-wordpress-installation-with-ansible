### **Job Sheet: Automating WordPress Installation with Ansible**

---

#### **Objective:**
This job sheet aims to help you:
1. Gradually set up inventory files and playbooks.
2. Install Apache, MySQL, and PHP on a remote server using Ansible.
3. Manage MySQL databases and users automatically.
4. Configure WordPress using dynamic templates.
5. Reinforce your understanding through multiple exercises for each step before the final practice.

---

### **Step 1: Preparing Your Environment**

#### **Exercise 1.1: Creating the Inventory File**

Create an inventory file named `inventory.ini` in your project directory:
```ini
[wordpress]
your_host_ip ansible_user=ubuntu ansible_ssh_private_key_file=/path/to/private/key.pem
```
- **Objective**: Define the remote server’s IP, host user, and SSH key.
- **Additional Exercise**: Try modifying the IP or SSH key file path to a different server, if available.

#### **Exercise 1.2: Testing Ansible Connectivity**

Verify the Ansible connection to the remote server:
```bash
ansible -i inventory.ini wordpress -m ping
```
- **Objective**: Ensure Ansible can connect to the server.
- **Additional Exercise**: Try using a different inventory file or modifying connection parameters.

---

### **Step 2: Creating the Playbook**

#### **Exercise 2.1: Basic Playbook Setup**

Create a basic playbook named `playbook.yml`:
```yaml
- hosts: wordpress
  become: yes

  tasks:
    - name: Update package repository
      apt:
        update_cache: yes
```
- **Objective**: Start with a basic task of updating the package repository.
- **Additional Exercise**: Add a task to install a simple package like `curl` and run the playbook.

```bash
ansible-playbook -i inventory.ini playbook.yml
```

#### **Exercise 2.2: Installing Apache**

Extend the playbook to install Apache:
```yaml
    - name: Install Apache
      apt:
        name: apache2
        state: present
```
- **Objective**: Automate Apache installation.
- **Additional Exercise**: Verify that Apache is installed by running `apache2 -v` on the server.

---

### **Step 3: Installing WordPress Dependencies**

#### **Exercise 3.1: Installing MySQL and PHP**

Extend your playbook to include MySQL and PHP:
```yaml
    - name: Install MySQL and PHP
      apt:
        name:
          - mysql-server
          - php
          - php-mysql
        state: present
```
- **Objective**: Automate the installation of MySQL and PHP.
- **Additional Exercise**: Check their versions using `mysql --version` and `php --version`.

#### **Exercise 3.2: Installing Python Dependencies for MySQL**

Ensure that the necessary Python dependencies for MySQL management are installed:
```yaml
    - name: Install Python MySQL dependencies
      apt:
        name: python3-pymysql
        state: present
```
- **Objective**: Install `python3-pymysql` to enable Ansible’s MySQL module.
- **Additional Exercise**: Verify the installation using `pip show pymysql` on the server.

---

### **Step 4: Managing MySQL**

#### **Exercise 4.1: Create MySQL Admin User**

Add this task to create an admin user with full privileges:
```yaml
    - name: Create MySQL admin user
      mysql_user:
        name: wordpress
        password: wordpress
        priv: "*.*:ALL,GRANT"
        host: "%"
        state: present
        login_unix_socket: /var/run/mysqld/mysqld.sock
```
- **Objective**: Create a MySQL admin user with permissions from any host (`%`).
- **Additional Exercise**: Check if the user was created by logging into MySQL and running `SELECT User FROM mysql.user;`.

#### **Exercise 4.2: Create WordPress Database**

Update the task to create the WordPress database with login credentials:
```yaml
    - name: Create WordPress database
      mysql_db:
        name: wordpress_db
        state: present
        login_user: wordpress
        login_password: wordpress
```
- **Objective**: Use the MySQL user created earlier to manage the database.
- **Additional Exercise**: Verify the database creation by running `SHOW DATABASES;`.

---

### **Step 5: Configuring WordPress**

#### **Exercise 5.1: Download and Extract WordPress**

Extend the playbook to download and extract WordPress:
```yaml
    - name: Download WordPress
      get_url:
        url: https://wordpress.org/latest.tar.gz
        dest: /tmp/wordpress.tar.gz

    - name: Extract WordPress
      unarchive:
        src: /tmp/wordpress.tar.gz
        dest: /var/www/
        remote_src: yes
```
- **Objective**: Automate the download and extraction of WordPress.
- **Additional Exercise**: Verify that WordPress was extracted to `/var/www/wordpress`.

#### **Exercise 5.2: Configuring wp-config.php**

Use dynamic templates to configure `wp-config.php`:
1. Create the `wp-config.php.j2` file with the necessary variables.
2. Add this task to your playbook:
```yaml
    - name: Configure WordPress (wp-config.php)
      template:
        src: wp-config.php.j2
        dest: /var/www/wordpress/wp-config.php
```
- **Objective**: Dynamically configure WordPress settings using Jinja2 templates.
- **Additional Exercise**: Check the contents of `/var/www/wordpress/wp-config.php` to confirm that the variables have been replaced.

---

### **Step 6: Apache Configuration**

#### **Exercise 6.1: Set Apache VirtualHost for WordPress**

Update the Apache configuration to point to the WordPress directory:
```yaml
    - name: Update Apache VirtualHost for WordPress
      replace:
        path: /etc/apache2/sites-available/000-default.conf
        regexp: 'DocumentRoot /var/www/html'
        replace: 'DocumentRoot /var/www/wordpress'
```
- **Objective**: Automate the configuration of Apache to serve WordPress.
- **Additional Exercise**: Verify that the Apache configuration is working by opening the server’s IP address in a browser. The WordPress installation page should appear.

---

### **Final Practice**

#### **Practice: Running the Full Playbook**

Combine all tasks into a single file named `wordpress-install.yml` and run the playbook:
```bash
ansible-playbook -i inventory.ini wordpress-install.yml
```
- **Objective**: Automate the entire WordPress installation process from start to finish.
- **Observation**: After running the playbook, WordPress should be fully installed and accessible via a browser.
