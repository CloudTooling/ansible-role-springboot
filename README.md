# ansible-springboot-role
[![Build](https://github.com/CloudTooling/ansible-role-springboot/actions/workflows/workflow.yml/badge.svg)](https://github.com/CloudTooling/ansible-role-springboot/actions/workflows/workflow.yml)
[![Ansible Role](https://img.shields.io/ansible/role/d/cloudtooling/springboot)](https://galaxy.ansible.com/ui/standalone/roles/cloudtooling/springboot/)

Easy deployment of SpringBoot app on Linux

## Usage

Create a *requirements.yml* refering to this role and a *playbook.yml*, e.g.:
>**NOTE**: Make sure, that during CI the Distribution ZIP is available
```
  vars:
    extra_files:
      - ../src/main/resources/config
      - ../src/main/resources/idp

  roles:
    - role: springboot
      springboot_application_url: http://localhost:8080
      springboot_application_name: my-app
      springboot_dist_file: ../target/my-app.zip
      springboot_configuration_template: ../src/main/dist/application.yml.j2
```
You also need to provide a `springboot_configuration_template` which will be used as config template.

Make sure that the does not contain the base directory, e.g. use an `assembly.xml` like this:
```
<?xml version="1.0"?>
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3 http://maven.apache.org/xsd/assembly-1.1.3.xsd">
    <id>unix</id>

    <formats>
        <format>zip</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>

    <dependencySets>
        <dependencySet>
            <useProjectArtifact>true</useProjectArtifact>
            <outputDirectory>lib</outputDirectory>
        </dependencySet>
    </dependencySets>

...
</assembly>
```

Within your CI/CD pipeline just run the playbook then, e.g. via GH action:
```
-   name: Setup key for deployment
    uses: webfactory/ssh-agent@v0.9.0
    with:
        ssh-private-key: ${{ secrets.MY_SSH_KEY }}

-   name: Run Ansible playbook
    run: |
        cd ansible
        ansible-galaxy install -r requirements.yml -f
        ansible-playbook playbook.yml
```

## Advanced setup

For more advanced setup, you can add extra files via:
```
  vars:
    extra_files:
      - ../src/main/resources/config
      - ../src/main/resources/idp
```
For secrets create additional files, e.g. read EnvVars from GH Secrets and provide them as JSON files:
```
-   name: Setup key for deployment
    uses: webfactory/ssh-agent@v0.9.0
    with:
        ssh-private-key: ${{ secrets.SSHKEY_STAGING1 }}

-   name: Set Ansible vars from vars/secrets context JSON
    run: |
        # Pipe the JSON string into jq and convert to lower case keys
        echo "$VARS_CONTEXT" | jq -r 'with_entries( .key |= ascii_downcase )' > ansible/vars.json
        # same for secrets
        echo "$SECRETS_CONTEXT" | jq -r 'with_entries( .key |= ascii_downcase )' > ansible/secrets.json

-   name: Run Ansible playbook
    run: |
        cd ansible
        ansible-galaxy install -r requirements.yml -f
        ansible-playbook playbook.yml  --extra-vars "@vars.json"  --extra-vars "@secrets.json"
```