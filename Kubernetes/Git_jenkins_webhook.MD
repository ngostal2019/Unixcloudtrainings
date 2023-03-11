# GitHub hook trigger for GITScm polling settings between Git and Jenkins

- A *webhook* is an HTTP callback, an HTTP POST that occurs when something happens through a simple event-notification via HTTP POST.

- *GitHub webhooks* in Jenkins are used to trigger the build whenever a developer commits something to a target branch

1. On Jenkins
   - Manage Jenkins
   - Configure System
   - Scroll down to the `GitHub` section
      - Click on `Advanced`
        - Override Hook URL
          - Check `Specify another hook URL for GitHub configuration`
            - copy `http://jenkins-ip:8080/github-webhook/` to paste in Github webhook setting

1. Go to your GitHub project repository.
   - Go to `Settings` in the right corner.
   - Click on `webhooks`
     - On the right click on `Add webhook`
       - Write the Payload URL as `http://jenkins-ip:8080/github-webhook/` that was copied from Jenkins
       - Content type is `application/json`
       - Which events would you like to trigger this webhook?
         - *Just the push event*: Only send data when someone pushed into my repository.
         - *Send me everything*: If there is any pull or push event in our repository we will get notified.
         - *Let me select individual events*: We can configure for what events we want our data.
       - Make sure that `Active` is checked
       - Click `Create` and a push webhook will be created 