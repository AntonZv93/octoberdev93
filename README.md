# ansible-and-google-cloud
Заменить имя папки nginx на apache2
Отредактируйте файл var и подставьте значения со своей облачной платформы какую подробно я изложил в файле stepsforinstall.txt!!!!

----
#Create virtual machine, domain and install october:
ansible-playbook create.yml --extra-vars "domain=domain_name" --user=username --private-key=~/.ssh/ssh_private_key

#Destroy virtual machine and domain
ansible-playbook destroy.yml --extra-vars "domain=domain_name" --user=username --private-key=~/.ssh/ssh_private_key
 Подробная инструкция по установке находится в файле stepsforinstall.txt
