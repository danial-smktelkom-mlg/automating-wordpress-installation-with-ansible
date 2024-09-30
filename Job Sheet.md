### **Job Sheet: Automating WordPress Installation with Ansible**

---

#### **Objective:**
This job sheet will guide you through the process of:
1. Setting up an inventory file and playbook.
2. Installing Apache, MySQL, and PHP.
3. Managing MySQL databases and users.
4. Automatically installing and configuring WordPress.
5. Configuring `wp-config.php` from a sample file.
6. Reinforcing your understanding through repeated exercises for each key step.

---

### **Step 1: Preparing Your Environment**

#### **Exercise 1.1: Creating the Inventory File**

Create an inventory file named `inventory.ini` with the following content:
```ini
[wordpress]
your_host_ip ansible_user=your_host_user ansible_ssh_private_key_file=/path/to/private/key.pem
```
- **Objective**: Define the server’s IP, host user, and SSH key.
- **Additional Exercise**: Modify the file to connect to a different server, if available.

#### **Exercise 1.2: Testing Ansible Connectivity**

Verify the Ansible connection using the command:
```bash
ansible -i inventory.ini wordpress -m ping
```
- **Objective**: Ensure Ansible can connect to the server.
- **Additional Exercise**: Use a different inventory or modify the connection parameters.

---

### **Step 2: Creating the Playbook**

#### **Exercise 2.1: Basic Playbook Setup**

Create a playbook named `playbook.yml`:
```yaml
- hosts: wordpress
  become: yes

  tasks:
    - name: Update package repository
      apt:
        update_cache: yes
```
- **Objective**: Start with updating the package repository.
- **Additional Exercise**: Add a task to install a simple package like `curl`.

#### **Exercise 2.2: Installing Apache**

Extend the playbook to install Apache:
```yaml
    - name: Install Apache
      apt:
        name: apache2
        state: present
```
- **Objective**: Automate the installation of Apache.
- **Additional Exercise**: Verify that Apache is installed by running `apache2 -v`.

---

### **Step 3: Installing WordPress Dependencies**

#### **Exercise 3.1: Installing MySQL and PHP**

Extend the playbook to include MySQL and PHP:
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
- **Additional Exercise**: Check the versions using `mysql --version` and `php --version`.

#### **Exercise 3.2: Installing Python Dependencies for MySQL**

Install Python dependencies for MySQL:
```yaml
    - name: Install Python MySQL dependencies
      apt:
        name: python3-pymysql
        state: present
```
- **Objective**: Ensure MySQL management modules work in Ansible.
- **Additional Exercise**: Verify by running `pip show pymysql`.

---

### **Step 4: Managing MySQL**

#### **Exercise 4.1: Create MySQL Admin User**

Create a MySQL admin user:
```yaml
    - name: Create MySQL admin user
      mysql_user:
        name: mysql_admin_user
        password: mysql_admin_password
        priv: "*.*:ALL,GRANT"
        host: "%"
        state: present
        login_unix_socket: /var/run/mysqld/mysqld.sock
```
- **Objective**: Automate the creation of an admin user for MySQL.
- **Additional Exercise**: Verify by logging into MySQL and running `SELECT User FROM mysql.user;`.

#### **Exercise 4.2: Create WordPress Database**

Create the WordPress database:
```yaml
    - name: Create WordPress database
      mysql_db:
        name: wordpress_db_name
        state: present
        login_user: mysql_admin_user
        login_password: mysql_admin_password
```
- **Objective**: Automate database creation.
- **Additional Exercise**: Verify by running `SHOW DATABASES;`.

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
- **Objective**: Automate downloading and extracting WordPress.
- **Additional Exercise**: Verify by checking if WordPress was extracted to `/var/www/wordpress`.

---

### **Step 6: WordPress Configuration**

#### **Exercise 6.1: Copy and Configure wp-config.php**

##### **Step 6.1.1: Copy wp-config-sample.php**

Add a task to copy the sample configuration file:
```yaml
    - name: Copy wp-config-sample.php to wp-config.php
      command: cp /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php
      args:
        creates: /var/www/wordpress/wp-config.php
```
- **Objective**: Copy the sample WordPress configuration file.
- **Additional Exercise**: Check if `wp-config.php` exists after running the playbook.

##### **Step 6.1.2: Configure wp-config.php**

Use the `lineinfile` module to dynamically configure the WordPress database settings:
```yaml
    - name: Configure wp-config.php
      lineinfile:
        path: /var/www/wordpress/wp-config.php
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
      loop:
        - { regexp: "DB_NAME", line: "define('DB_NAME', 'wordpress_db_name');" }
        - { regexp: "DB_USER", line: "define('DB_USER', 'mysql_admin_user');" }
        - { regexp: "DB_PASSWORD", line: "define('DB_PASSWORD', 'mysql_admin_password');" }
        - { regexp: "DB_HOST", line: "define('DB_HOST', 'localhost');" }
```
- **Objective**: Replace the database settings in `wp-config.php`.
- **Additional Exercise**: Manually check the file to ensure the changes were made.

---

### **Step 7: Apache Configuration**

#### **Exercise 7.1: Configure Apache VirtualHost**

Update Apache configuration for WordPress:
```yaml
    - name: Update Apache VirtualHost for WordPress
      replace:
        path: /etc/apache2/sites-available/000-default.conf
        regexp: 'DocumentRoot /var/www/html'
        replace: 'DocumentRoot /var/www/wordpress'
```
- **Objective**: Automate Apache VirtualHost configuration for WordPress.
- **Additional Exercise**: Open the server IP in a browser to verify that WordPress is accessible.

#### **Exercise 7.2: Enable Site Configuration and Restart Apache**

Add tasks to enable the new site configuration and restart Apache:
```yaml
    - name: Enable the new site configuration
      command: a2ensite 000-default.conf

    - name: Restart Apache to apply changes
      service:
        name: apache2
        state: restarted
```
- **Objective**: Enable the new site configuration and restart Apache.
- **Additional Exercise**: Ensure that Apache is running by checking its status.

---

### **Final Practice: Full Playbook Execution**

To execute the full automation, combine all tasks and run:
```bash
ansible-playbook -i inventory.ini playbook.yml
```
- **Objective**: Automate the complete WordPress installation and configuration process.
- **Observation**: After running the playbook, WordPress should be fully installed and configured.

---

### **Summary**

This job sheet provides a comprehensive guide for automating WordPress installation using Ansible. Each section is designed to build your understanding through practical exercises, ensuring you gain hands-on experience. Review the objectives and complete the additional exercises to enhance your learning further.

---

### **Additional Recommendations:**

- Ensure that you have all necessary permissions to execute the tasks outlined in this job sheet.
- Familiarize yourself with Ansible documentation for any specific commands or modules you may not understand.
- Backup any existing configurations before making changes to your server.
