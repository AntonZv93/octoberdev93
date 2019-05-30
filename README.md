# ansible-and-google-cloud

Отредактируйте файлы var,hosts,inventory(/roles/mysql/tests/inventory и других roles) и подставьте значения со своей облачной платформы какую подробно я изложил в файле stepsforinstall.txt(это мануал - инструкция по установке)!!!!
После следования этому файлу запустите со своей ansible-master 2 следующие комманды
----
#Create virtual machine, domain and install october:
ansible-playbook create.yml --extra-vars "domain=domain_name" --user=username --private-key=~/.ssh/ssh_private_key
------------
#Copy git to your gcp instance and running this playbook:
ansible-playbook octoberdev93.yml --extra-vars "domain=domain_name" --user=username --private-key=~/.ssh/ssh_private_key
-------
#Destroy virtual machine and domain
ansible-playbook destroy.yml --extra-vars "domain=domain_name" --user=username --private-key=~/.ssh/ssh_private_key
 Подробная инструкция по установке находится в файле stepsforinstall.txt
