## Ответы на вопросы и комментарии

# 1.
- Вместо копипаста который был, переделал манифест используя переменные и for_each с перечеслением
[main.tf](https://github.com/bogkofe/terraform-diplom/blob/master/src/main.tf)

# 2.
- Бэкенд был в файле main.tf, для наглядности вынес его в отдельный файл
[backend_s3.tf](https://github.com/bogkofe/terraform-diplom/blob/master/src/backend_s3.tf)

# 3.
- Чтобы у нод не было публичного ip, убрал у них нат, создал еще одну вм, через которую будет доступ на ноды и через которую ноды будут выходить в инет. 
[k8s_instance.tf](https://github.com/bogkofe/terraform-diplom/blob/master/src/k8s_instance.tf)

Так же создал load-balancer.
[nlb.tf](https://github.com/bogkofe/terraform-diplom/blob/master/src/loadbalancer.tf)

# 4.
- Была проблема как то с cloud-init на Centos, выкрутился таким способом. Использовал изначально тот же шаблон terraform. В текущем манифесте убрал его.

# 5.
- Для автоматического создания инвенторки, сделал скрипт питона, который автоматически выполняется после разворачивания виртульных машин. В итоге мы сразу получаем заполненный ip адерсами файл.
- Вот скрипт:
[kubespray_inventory.py](https://github.com/bogkofe/terraform-diplom/blob/master/src/kubespray_inventory.py)

- Добавил вызов скрипта в файл c outputs
[output.tf](https://github.com/bogkofe/terraform-diplom/blob/master/src/output.tf)

# 6.
- Обновленный DOCKERFILE. 
[DOCKERFILE](https://github.com/bogkofe/app-diplom/blob/master/Dockerfile)

# 7.
- Отредактировал файл k8s-cluster/addons.yml, добавил туда helm и функционал Nginx ingress controller

# 8.
- Есть проблемки с использованием ingress, не получется external_ip, всегда находится в состоянии pending. Пробовал назначать ip на машину на прямую, все равно не работает. При работе с microk8s проблем с ingress не было. Вот директория с новыми файлами.
[new_files](https://github.com/bogkofe/kubernets-diplom/blob/master/new_files)

# 9. 
- В переменных KUBECONFIG1…4 находиться разделенный конфиг .kube/config. Action не мог его обработать одним файлом, говорил что очень длинный.

# 10.
- Я указал runs-on: ubuntu-latest, там уже есть kubectl

# 11.
- Иправил pipeline, добавил job deploy. Теперь билд приложения происходил по любому тегу, а деплой только по тегу типа semver
[ci-cd.yml](https://github.com/bogkofe/app-diplom/blob/master/src/ci-cd.yml)

![k](https://github.com/bogkofe/diplom/blob/main/files/20.png)