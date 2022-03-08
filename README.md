# ansible-dockerfile-push-pull-via-git

Playbook to insatll Docker and build a simple html application image from a dockerfile and then it pushed to your  Docker Hub account.

## Ansible Modules used

- yum
- pip
- service
- git
- docker_image

## How to Use

```
git clone https://github.com/BetcyBabu/ansible-dockerfile-push-pull-via-git
cd ansible-dockerfile-push-pull-via-git
ansible-playbook main.yml
```


## Behind the code

```
---

- name: "Creating Docker Image"
  hosts: amazon
  become: yes
  vars:
    git_url: "https://github.com/jomyg/docker-git.git"
    docker_username: "jomyg"
    docker_password: "9495671831"
    image_name: jomyg/htmlapp

  tasks:
    - name: "Installing Docker and the required dependencies"
      yum:
        name:
          - docker
          - git
          - python2-pip
        state: present

    - name: "Installing Docker client for Python"
      pip:
        name: docker-py
        state: present

    - name: "Starting the docker now. Please wait .."
      service:
        name: docker
        state: restarted
        enabled: true

    - name: "Cloning the git repository"
      git:
        repo: "{{ git_url}}"
        dest: /var/htmlapp
      register: git_status

    - name: " Accessing into Image repo"
      when: git_status.changed == true
      docker_login:
        username: "{{ docker_username }}"
        password: "{{docker_password}}"

    - name: "Building the docker image via downloaded dockerfile"
      when: git_status.changed == true
      docker_image:
        build:
          path: /var/htmlapp/
        name: "{{image_name}}"
        source: build
        push: yes
        tag : "{{ item }}"
      with_items:
        - latest
        - "{{git_status.after}}"
      ```
