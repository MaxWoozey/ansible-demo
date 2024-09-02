# Ansible Demo

This repository is an Ansible demo for deploying a `Todo` application and a `MySQL` service. It contains Ansible Playbooks and related template files for configuring a Kubernetes cluster. The application includes two services: the main application and a database. A single-node Kubernetes cluster was applied in this project.

## The website
To visit the app, one can visit http://mytodoapp.westeurope.cloudapp.azure.com/
![image](https://github.com/user-attachments/assets/eee81704-4cf5-4c13-8e5e-58d3c06ac5c4)

## Components

```plaintext
ansible-demo/
├── inventory.yaml           # Ansible inventory, defined host
├── playbook.yaml            # Main Playbook, the file containing deployment roles
├── secrets.yaml             # Save sensitive data.
├── templates/               # Jinja2 templates dir
│   ├── todo-deployment.yaml.j2  # Template for `Todo` application Deployment
│   ├── todo-service.yaml.j2     # Template for `Todo` application Service
│   ├── mysql-deployment.yaml.j2 # Template for `MySQL` service Deployment
│   ├── mysql-service.yaml.j2    # Template for `MySQL` service Service
|   ├── mysql-pv.yaml.j2         # Template for `MySQL` Persistent Volume
|   ├── mysql-pvc.yaml.j2        # Template for `MySQL` Persistent Volume Claim
|   ├── ingress-service.yaml.j2  # Template for Ingress Service
│   ├── ingress.yaml.j2          # Template for Ingress configuration
└── roles/                  # Ansible roles dir
    ├── kubernetes/         # Kubernetes-related role
    │   └── tasks/
    │       └── main.yaml   # Tasks definition for the Kubernetes role
    ├── docker/             # Docker-related role
    │   └── tasks/
    │       └── main.yaml   # Tasks definition for the Docker role
    ├── mysql/              # `MySQL` service-related role
    │   └── tasks/
    │       └── main.yaml   # Tasks definition for the `MySQL` role
    ├── todo/               # `Todo` application-related role
    │   └── tasks/
    │       └── main.yaml   # Tasks definition for the `Todo` role
    └── ingress/            # Ingress-related role
        └── tasks/
            └── main.yaml   # Tasks definition for the Ingress role
```
## Setups
### Virtual Machines
For hosting Ansible, a WSL (Windows Subsystem for Linux) environment on Windows was used. For deploying Kubernetes, a cloud virtual machine on Azure was chosen.

### Inventory
In `inventory.yaml`, various parameters were set, including a namespace of "todo-app" and a replica count of 2.
![image](https://github.com/user-attachments/assets/c8916aaa-1bdd-45a2-aa32-67533cb451cb)

### Playbooks
The main playbook specifies several variables to make the project more organized. Roles are used to divide the deployment into 5 modules:
![image](https://github.com/user-attachments/assets/6fd3e53d-095a-4e03-b5a1-eeb4a4fc249f)
Each role contains a `main.yaml` file with detailed tasks for each module:
![image](https://github.com/user-attachments/assets/c567e77b-266f-4d31-97ac-791992b9c1ec)

### Templates
Jinja2 templates are used to deploy pods on the Kubernetes node. All required templates are stored in the templates/ directory.

## Usage
1. Configure Inventory File
Edit the `inventory.yaml` file to include information about your Kubernetes cluster hosts.
2. Edit Template Files
Modify the Jinja2 templates in the templates/ directory according to your needs for configuring the Todo application, MySQL service, and Ingress.

3. Run Playbook
Execute the following command to run the Ansible Playbook:
```bash
ansible-playbook -i inventory.yaml playbook.yaml --ask-vault-pass
```
![image](https://github.com/user-attachments/assets/6cf5ec0b-7d30-4627-861f-6db8d94f174f)
Input the password to pass secrets. In this project, `db_user` and `db_password` are stored in `secrets.yaml`.

4. Check Deployment Status
Verify that the application and service are deployed as expected on the Kubernetes cluster.
Run:
```bash
microk8s kubectl get node
microk8s kubectl get pods -n todo-app
```
You should see:
![image](https://github.com/user-attachments/assets/5a596748-7e6c-400b-a652-e61ee6f8ea98)
To test the persistence of the MySQL service, log in to the MySQL pod:
Run:
```bash
microk8s kubectl exec -it <mysql-pod-name> -n todo-app -- mysql -u <user> -p<password>
```
![image](https://github.com/user-attachments/assets/dae312d8-9c7e-41eb-8734-3ffef7f269ca)
Check the data manually. Record the data and then delete the MySQL pod. Wait for a new one to start:

Run:
```bash
microk8s kubectl delete pod <pod_name> -n todo-app
microk8s kubectl get pods -n todo-app
```
![image](https://github.com/user-attachments/assets/8d1c7865-89b4-45ce-ad14-db3421083fdb)
Check the data in tables again to confirm it is persistent if the data remains the same.

5. Ingress
In the demo, ingress was used to expose the port to the public.
![image](https://github.com/user-attachments/assets/f78eac13-b8cc-43fd-bc88-ef9c054122d2)


7. Visit the website
To visit the app, one can visit http://mytodoapp.westeurope.cloudapp.azure.com/
![image](https://github.com/user-attachments/assets/eee81704-4cf5-4c13-8e5e-58d3c06ac5c4)

## Automation process
In conclusion, assuming you have configured the Kubernetes cluster and the `inventory.yaml` file, running `playbook.yaml` will automatically:
- Set up Kubernetes
- Configure Docker
- Deploy the MySQL service
- Deploy the Todo application
- Configure Ingress to expose the Todo application
