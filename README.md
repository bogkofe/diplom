# Дипломный практикум в Yandex.Cloud

## Цели:

1. Подготовить облачную инфраструктуру на базе облачного провайдера Яндекс.Облако.
2. Запустить и сконфигурировать Kubernetes кластер.
3. Установить и настроить систему мониторинга.
4. Настроить и автоматизировать сборку тестового приложения с использованием Docker-контейнеров.
5. Настроить CI для автоматической сборки и тестирования.
6. Настроить CD для автоматического развёртывания приложения.

## Репозитории проекта
- [terraform](https://github.com/bogkofe/terraform-diplom/tree/master/src)

- [my-app](https://github.com/bogkofe/app-diplom)

- [kubernets](https://github.com/bogkofe/kubernetes-diplom)



## Этапы выполнения:

### Создание облачной инфраструктуры
- Создал сервисный аккаунт sagirov, дал ему права
![account](https://github.com/bogkofe/diplom/blob/main/files/2.png)

- Создал манифест main.tf в котором указал создание сети и подсетей
[main.tf](https://github.com/bogkofe/terraform-diplom/blob/master/src/main.tf)

- Отправил tfstate в backend s3 
![s3](https://github.com/bogkofe/diplom/blob/main/files/1.png)


### Создание Kubernetes кластера
- Создал 3 виртульные машины в YC с помощью манифеста k8s_instances.tf
[k8s_instance.tf](https://github.com/bogkofe/terraform-diplom/blob/master/src/k8s_instance.tf)

- Для разворачивания кластера я выбрал kubspray. Склонировал репозиторий себе на локальную машину.

- Выполнил некоторые подготовительные действия:

- Обновил python до 3.10

- Создал виртальное окружение
	* python3.10 -m venv venv
	* source venv/bin/activate

- Установил зависимости
	* pip install -r requirements.txt

- Скопировал inventory для ansible
	* cp -rfp inventory/sample inventory/kube_cluster

- Сгенерировал файл inventory (Адреса узнал командой terraform output vm_info), так как заранее указал output в terraform
	* declare -a IPS=(ip адреса)

- Установил зависимость ruamel.yaml, которая почему то не подтянулась
	* pip3 install ruamel.yaml

- Зашел и подправил полученный файл после команды
	* CONFIG_FILE=inventory/kube_cluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

- Выполнил ansible манифест который установит кластер kubernets
	* ansible-playbook -i inventory/kube_cluster/hosts.yaml cluster.yml -b -v &

- После этого я подключился по ssh к node1 и проверил работоспособность:
![kubespray](https://github.com/bogkofe/diplom/blob/main/files/3.png)

![kubespray](https://github.com/bogkofe/diplom/blob/main/files/4.png)


### Создание тестового приложения
- Создал приложение nginx и залил образ в DockerHub
![Docker_build](https://github.com/bogkofe/diplom/blob/main/files/5.png)

![Docker_hub](https://github.com/bogkofe/diplom/blob/main/files/6.png)


### Подготовка cистемы мониторинга и деплой приложения
- Зашел по ssh на master node

- Установил Helm
	* curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

- Добавил репозиторий Helm для kube-prometheus-stack
	* helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
	* helm repo update

- Создал namespace monitoring для системы мониторинга
	* kubectl create namespace monitoring

- Установил пакет с помощью Helm:
	* helm install prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring

- Проверил поды
![prometheus](https://github.com/bogkofe/diplom/blob/main/files/7.png)

- Создал service для графана, чтоб иметь доступ из вне (использовал NodePort) и задеплоил
[grafana-service.yaml](https://github.com/bogkofe/kubernetes-diplom/blob/master/grafana-service.yaml)

- Создал deployment и service для моего приложения и задеплоил
[nginx-deployment.yaml](https://github.com/bogkofe/kubernetes-diplom/blob/master/nginx-deployment.yaml)

[nginx-service.yaml](https://github.com/bogkofe/kubernetes-diplom/blob/master/nginx-service.yaml)

- Ссылка на Grafana поднятое в kubernets
	* http://89.169.152.93:31000/
![Grafana](https://github.com/bogkofe/diplom/blob/main/files/8.png)

- Ссылка на поднятое nginx приложение в kubernets 
	* http://89.169.152.93:32000/
![Grafana](https://github.com/bogkofe/diplom/blob/main/files/9.png)

- ВМки не отключал, может даже еще будут активны во время проверки

### Установка и настройка CI/CD
- Так как все репозитории у меня находятся в github, решил использовать github action

- Создал в репозитории моего приложения директорию .github/workflows

- Создал файл ci-cd.yml в котром описал что должно быть выполнено.

- Сделал изменение в репозитрии и запушил tag v1.1.1
![image](https://github.com/bogkofe/diplom/blob/main/files/10.png)

- Запустился pipeline и выполнился успешно
![image](https://github.com/bogkofe/diplom/blob/main/files/12.png)

- Создался образ и залился в dockerhub c тегом v1.1.1
![image](https://github.com/bogkofe/diplom/blob/main/files/11.png)

- Произошел деплой соответствующего Docker образа в кластер Kubernetes
![image](https://github.com/bogkofe/diplom/blob/main/files/13.png)







