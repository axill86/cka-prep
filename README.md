# CKA Exam Preparation
I'll try to collect notes and tasks which have been made during prepration to [CKA exam](https://www.cncf.io/certification/cka/).


## General information about the exam
* Practice exam (not multi-select)
* Current version is 1.8 (May 2020) [Curriculum](https://github.com/cncf/curriculum)

## Materials used for preparation
During my prepration I used following courses and guides:

### K8S
* [LinuxAcademy course](https://linuxacademy.com/cp/modules/view/id/327)
* [K8S the hard way LA course](https://linuxacademy.com/cp/modules/view/id/221)
* [K8S the hard way guide](https://github.com/kelseyhightower/kubernetes-the-hard-way)
* [CKA prepration Udemy course](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)

## Tasks in this repo

* [Setup ghost blog](./tasks/ghost-blog/README.md) - setup simple ghost-blog with my-sql persistenc
* [Grant access to new developer](./tasks/security/README.md) - create CSR and grant permission to accesss the cluster 
* [Schedule on master](./tasks/master-only-node/README.md) - Use taints, toleratoins, selectors
* [Use network policies](./tasks/network-policies/README.md) - Use network policies to access k8s dns
* [API access from pod](./tasks/access-api-from-pod/README.md) - Direct access api from pod 
