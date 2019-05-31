# ansible-and-google-cloud

Отредактируйте файлы var,hosts,inventory(/roles/mysql/tests/inventory и других roles) и подставьте значения со своей облачной платформы какую подробно я изложил в файле stepsforinstall.txt(это мануал - инструкция по установке)!!!!
После следования этому файлу запустите со своей ansible-master 3 следующие комманды
----
#Create virtual machine, domain and install october:
----
1 step :Create file on_ansible_master.yml on ansible-master in google cloud platform
and running this file :
-----
ansible-playbook /home/on_ansible_master.yml -i hosts
---
after successful copying go to that folder and execute the rest of the commands
----
2 step:
ansible-playbook /home/create.yml --extra-vars "domain=domain_name" --user=username --private-key=~/.ssh/ssh_private_key
------------
#Copy git to your gcp instance and running this playbook:
3 step:
ansible-playbook /home/octoberdev93.yml --extra-vars "domain=domain_name" --user=username --private-key=~/.ssh/ssh_private_key
-------
#Destroy virtual machine and domain
ansible-playbook destroy.yml --extra-vars "domain=domain_name" --user=username --private-key=~/.ssh/ssh_private_key
 Подробная инструкция по установке находится в файле stepsforinstall.txt
