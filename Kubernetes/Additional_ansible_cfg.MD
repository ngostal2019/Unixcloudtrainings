# Setups and Security concerns with ansible

## While writing the final playbook

#### DRY(Do Not Repeat Yourself): It's important for us to reduce manual interventions to mitigate risks
1. In the *YAML* file, let's construct the docker tag image version
    ```yaml
    #Build the tag variables that will be used across the playbook but be able to be changed from a single place
    
    major_v: 1
    minor_v: 0
    build_num: 0
    ```
1. Create the *vault file* `docker_vars.yaml` at the location `/opt/ansible/` where we will keep our `docker login credentials` with the below content
    ```yaml
    ---
    docker_username: your_dockerhub_username
    docker_passwd: your_dockerhub_password

    # NOTE:
    ______

    #An ansible *vault file* is a file that is encrypted with password and only accessible if the vault password corresponds to the encryption key 
    # To encrypt the file with a password, use the command --> ansible-vault encrypt docker_vars.yaml
    # To view the content of the encrypted file by providing the original password --> ansible-vault view docker_vars.yaml
    # To decrypt the file with your password --> ansible-vault encrypt docker_vars.yaml
    ```
1. Before running the playbook
   #### While running the playbook, and giving the fact that the docker_vars.yaml file is encrypted and is being called inside the playbook, ansible vault will require a password, To avoid such an issue, let's tell our system ow to retrieve the vault password
    - Export the password that you used to create the vault with as a system variable using the below command:
      ```sh
      export VAULT_PASSWD=myvaultpasswd
      
      # To persist this, you can put it into the ~/.bash_profile file
      export VAULT_PASSWD=myvaultpasswd

      source ~/.bash_profile
      ``` 
    - Create a simple script `vault.sh` in `/opt/ansible/` that will read the file as below:
      ```sh
      #!/bin/bash
      echo $VAULT_PASSWD

      chmod +x vault.sh
      ```
    - Ansible comes pre-installed with the `ansible.cfg` file, but when installing it using `pip3`, you have to create and populate the file yourself just like the hosts file. So, let's do it:
      ```yaml
      vim /opt/ansibleansible.cfg
      # Write the below inside to tell ansible where to find the vault password that will be used to read the docker_vars.yaml file:

      [defaults]
      vault_password_file=/opt/ansible/vault.sh
      ```
1. Now run the playbook
   ```sh
   ansible-playbook -i hosts create-car-brand-choice-image.yml
   ```

