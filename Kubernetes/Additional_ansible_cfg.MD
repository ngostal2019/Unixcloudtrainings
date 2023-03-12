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
    ```
1. Now run the playbook
   ```sh
   ansible-playbook -i hosts create-car-brand-choice-image.yml
   ```
#### NOTE: Due to the difficulty Jenkins has to retreive the vault password during the pipeline execution, I have deleted that instruction from the current file

