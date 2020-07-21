# Instruction - Kubernetes Practice

### Pre-requisites
  
1. <b>Spin AWS Instances:</b></br>
    Follow the below repository for deploying the instances which will be configured in Kubernetes Cluster Setup.</br>
    `No of Instances: 3`

      <a href="https://github.com/avasant21/terraform-practice"> https://github.com/avasant21/terraform-practice</a></br>

2. <b>OS Configurations:</b></br>
    Follow the below steps to configure required OS Hostname to be used in Ansible Inventory. Use the Terraform output private IPs to replace the IP Entries and update `/etc/hosts` file</br>

        cat <<EOF | sudo tee -a /etc/hosts
        <IP Instance 1>      master
        <IP Instance 2>      node01
        <IP Instance 3>      node02
        EOF

3. <b>Copy Private Key into Master:</b></br>
    Follow the below instructions to copy the private key to master which can be used for Ansible master & target communication. Copy the content of `test-key.pem`(key created for Terraform deployment) in below file.</br>

        cat <<EOF | sudo tee /etc/ansible_centos.pem
        <Content of test-key.pem>
        EOF
        
        $ sudo chmod 600 /etc/ansible_centos.pem

### Ansible Installation & Role configuration
  
1. Follow the below instruction to setup Ansible in `master`. As we are assuming the Kubernetes(k8s) Master as Ansible Master in this practice.</br>

      <a href="https://github.com/avasant21/anisible-practice/blob/master/README.md"> https://github.com/avasant21/ansible-practice</a></br>

2. Inventory Configuration</br>
    The inventory can be configured as below to enable ansible to know hosts.</br>

        $ cat <<EOF | sudo tee -a /etc/ansible/hosts
        [k8s]
        master
        node01
        node02
        
        [k8s:vars]
        ansible_user=centos
        ansible_ssh_private_key_file=/etc/ansible_centos.pem
        
        [master]
        master
        
        [master:vars]
        kubernetes_role=master
        
        [nodes]
        node01
        node02
        
        [nodes:vars]
        kubernetes_role=node       
        
        EOF
        
3. Below can be followed to verify the Ansible connection with the nodes. Hostkey verification can be done by giving `yes` when prompted.</br>

        $ sudo ansible k8s -m test.ping

3. Create the below playbook to install Docker and Kubernetes Cluster.</br>

        $ cat <<EOF | sudo tee /etc/ansible/roles/k8s_playbook.yml
      ```yml
      - hosts: k8s
        become: true

        roles:
          - docker
          - kubernetes
      ```
        EOF

4. Clone the role from `geerlingguy's` Repositories/Ansible galaxy. Install `git` if not installed with `sudo yum install git -y`.</br>
    
        $ cd /etc/ansible/roles/
        $ git clone https://github.com/geerlingguy/ansible-role-docker.git docker
        $ git clone https://github.com/geerlingguy/ansible-role-kubernetes.git kubernetes
    

### Kubernetes Cluster Deployment

1. Deploy the Kubernetes Cluster Envrironment using the below command</br>
    The below command will run the tasks defined in the roles and configure the docker and Kubernetes master & node environments.</br>

        $ cd /etc/ansible/roles/
        $ sudo ansible-playbook k8s_playbook.yml
    Incase of errors in joining the node, we can leave and follow the document to configure them with new token
    

2. Verify the Kubernetes environment and `kube-system` namespace pods with below commands.</br>

        $ kubectl get nodes
        $ kubectl get pods --namespace kube-system -o wide
        

