
### **Job Sheet: Automating WordPress Installation with Ansible**

---

#### **Objective:**
Through this job sheet, you will learn to:
1. Gradually set up inventory files and playbooks.
2. Install Apache, MySQL, and PHP on a remote server using Ansible.
3. Manage MySQL databases automatically.
4. Configure WordPress using dynamic templates.
5. Practice each step with multiple exercises before moving on to full practice.

---

### **Step 1: Preparing Your Environment**

#### **Exercise 1.1: Creating the Inventory File**

1. **Objective**: Create an inventory file to define the remote server.
2. **Instructions**:
   - Create a new file called `inventory.ini` in the project directory.
   - Add the following content:
     ```ini
     [wordpress]
     your_host_ip ansible_user=ubuntu ansible_ssh_private_key_file=/path/to/private/key.pem
     ```
3. **Observation**:
   - The inventory file must include the server’s IP address and login details, using the correct format for defining the user and private key file.

4. **Additional Exercise**:
   - Change the IP and SSH key file path to a different server if available.
   - Explain the difference between `ansible_ssh_private_key_file` and `ansible_user`.

#### **Exercise 1.2: Testing Ansible Connectivity**

1. **Objective**: Ensure Ansible can connect to the remote server.
2. **Instructions**:
   - Run the command:
     ```bash
     ansible -i inventory.ini wordpress -m ping
     ```
   - Check if the result is `pong`.
3. **Additional Exercise**:
   - Perform the ping command with a different inventory file, for instance, using a different server or without an SSH key, and observe the results.

---

### **Step 2: Creating the Playbook**

#### **Exercise 2.1: Creating a Basic Playbook**

1. **Objective**: Write a basic playbook to update the package repository.
2. **Instructions**:
   - Create a new file called `playbook.yml`.
   - Add the following task:
     ```yaml
     - hosts: wordpress
       become: yes

       tasks:
         - name: Update package repository
           apt:
             update_cache: yes
     ```
3. **Additional Exercise**:
   - Add a new task to install a package, such as `curl`.
   - Run the playbook using:
     ```bash
     ansible-playbook -i inventory.ini playbook.yml
     ```
   - Check if `curl` is installed on the server.

#### **Exercise 2.2: Creating a Playbook to Install Apache**

1. **Objective**: Extend the playbook to install Apache.
2. **Instructions**:
   - Add the following task to the previous playbook:
     ```yaml
         - name: Install Apache
           apt:
             name: apache2
             state: present
     ```
3. **Additional Exercise**:
   - After running the playbook, verify Apache is installed by running `apache2 -v` on the server.

#### **Exercise 2.3: Verifying Apache Installation**

1. **Objective**: Test if Apache is running correctly.
2. **Instructions**:
   - After running the playbook, open a browser and enter the server’s IP address, such as `http://your_host_ip`.
   - Check if the default Apache page appears.
3. **Additional Exercise**:
   - Add a task to ensure that the Apache service is started:
     ```yaml
         - name: Ensure Apache is running
           service:
             name: apache2
             state: started
     ```

---

### **Step 3: Installing WordPress Dependencies**

#### **Exercise 3.1: Adding Tasks to Install MySQL and PHP**

1. **Objective**: Add MySQL and PHP installation to the playbook.
2. **Instructions**:
   - Add the following tasks:
     ```yaml
         - name: Install MySQL and PHP
           apt:
             name:
               - mysql-server
               - php
               - php-mysql
             state: present
     ```
3. **Additional Exercise**:
   - Test with `mysql --version` and `php --version` on the server to confirm they are installed.

#### **Exercise 3.2: Installing Python Dependencies for MySQL**

1. **Objective**: Install Python dependencies to manage MySQL using Ansible.
2. **Instructions**:
   - Add the following task:
     ```yaml
         - name: Install Python MySQL dependencies
           apt:
             name: python3-pymysql
             state: present
     ```
3. **Additional Exercise**:
   - Verify that `python3-pymysql` is installed by running `pip show pymysql` on the server.

---

### **Step 4: Managing MySQL Database**

#### **Exercise 4.1: Creating the WordPress Database**

1. **Objective**: Create a MySQL database for WordPress.
2. **Instructions**:
   - Add the following task to the playbook:
     ```yaml
         - name: Create MySQL database for WordPress
           community.mysql.mysql_db:
             name: wordpress_db
             state: present
     ```
3. **Additional Exercise**:
   - Verify the database was created by logging into MySQL and running `SHOW DATABASES;`.

#### **Exercise 4.2: Creating MySQL User for WordPress**

1. **Objective**: Create a MySQL user for WordPress.
2. **Instructions**:
   - Add the following task:
     ```yaml
         - name: Create MySQL user for WordPress
           community.mysql.mysql_user:
             name: wordpress_user
             password: "your_secure_password"
             priv: 'wordpress_db.*:ALL'
             host: localhost
             state: present
     ```
3. **Additional Exercise**:
   - Check if the MySQL user was created by running `SELECT User FROM mysql.user;`.

---

### **Step 5: WordPress Configuration**

#### **Exercise 5.1: Downloading and Extracting WordPress**

1. **Objective**: Download and extract WordPress automatically using Ansible.
2. **Instructions**:
   - Add the following tasks:
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
3. **Additional Exercise**:
   - Verify that WordPress was extracted to `/var/www/wordpress`.

#### **Exercise 5.2: Configuring wp-config.php**

1. **Objective**: Use a template to configure `wp-config.php` with dynamic variables.
2. **Instructions**:
   - Create the `wp-config.php.j2` file with the provided content.
   - Add the following task to use the template:
     ```yaml
         - name: Update WordPress configuration (wp-config.php)
           template:
             src: wp-config.php.j2
             dest: /var/www/wordpress/wp-config.php
     ```
3. **Additional Exercise**:
   - Check the contents of `/var/www/wordpress/wp-config.php` to ensure the variables have been replaced correctly.

---

### **Step 6: Apache Configuration**

#### **Exercise 6.1: Setting Apache VirtualHost**

1. **Objective**: Modify the Apache VirtualHost to point to the WordPress directory.
2. **Instructions**:
   - Add the following task:
     ```yaml
         - name: Update Apache VirtualHost for WordPress
           replace:
             path: /etc/apache2/sites-available/000-default.conf
             regexp: 'DocumentRoot /var/www/html'
             replace: 'DocumentRoot /var/www/wordpress'
     ```
3. **Additional Exercise**:
   - Verify the Apache configuration by opening the server’s IP address in a browser. If successful, the WordPress installation page should appear.

---

### **Final Practice**

#### **Practice: Running the Full Playbook**

1. **Objective**: Run the full playbook and automate the WordPress installation.
2. **Instructions**:
   - Combine all the tasks above into a single file called `wordpress-install.yml`.
   - Run the playbook:
     ```bash
     ansible-playbook -i inventory.ini wordpress-install.yml
     ```
3. **Observation**:
   - All tasks should complete successfully, and WordPress should be accessible via the browser.

