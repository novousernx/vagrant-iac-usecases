# Ansible AWX Server - Using Vaagrant IaaC 
(Infra as a Code)

AWX provides a web-based user interface, REST API, and task engine built on top of Ansible. It is the upstream project for Tower, a commercial derivative of AWX.
[Learn More](https://github.com/ansible/awx)

Any questions : [Email](mailto:net.gini@gmail.com) | [LinkedIn](http://bit.ly/gineesh) | [www.techbeatly.com](www.techbeatly.com)

## Pre-Req
Make sure you have read [Vagrant & Provider Setup](https://github.com/ginigangadharan/vagrant-iaac-usecases/blob/master/README.md) before start.

## Build AWX Sandbox
- Clone [master repo](https://github.com/ginigangadharan/vagrant-iaac-usecases)
- Switch to `gcp-iaac-awx-server` directory
- `vagrant up --provider=google`