# pmg-inversion_microservices
pmg-inversion microservices repository

## ДЗ - 12
### Технология контейнеризации. Введение в Docker

#### Что сделано
- Устанавливаем Docker
	for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

	# Add Docker's official GPG key:
	sudo apt-get update
	sudo apt-get install ca-certificates curl
	sudo install -m 0755 -d /etc/apt/keyrings
	sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
	sudo chmod a+r /etc/apt/keyrings/docker.asc

	# Add the repository to Apt sources:
	echo \
	  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
	  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
	  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	sudo apt-get update

	sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

- Развлекаемся
1. Docker hello-world
	docker run hello-world

2. Docker ps
	#Список запущенных контейнеров
	docker ps

	#Список всех контейнеров
	docker ps -a

3. Docker images
	#Список сохранненных образов
	docker images

4. Docker run
	docker run -it ubuntu:18.04 /bin/bash
		root@de60cf43969d:/# echo 'Hello world!' > /tmp/file
		root@de60cf43969d:/# exit

	docker run -it ubuntu:18.04 /bin/bash
		root@6357653e55da:/# cat /tmp/file
		cat: /tmp/file: No such file or directory
		root@6357653e55da:/# exit

5. Docker exec
	docker exec -it 6357653e55da bash
		root@6357653e55da:/# ps axf
		root@6357653e55da:/# exit

6. Docker commit
	docker commit de60cf43969d pesin/ubuntu-tmp-file
	docker images
		REPOSITORY TAG IMAGE ID CREATED SIZE
		pesin/ubuntu-tmp-file   latest    a45561ae0ec1   About a minute ago   63.2MB


- Docker-контейнеры

1. Установка Yandex Cloud CLI
	curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash

2. Подключение к YC
	yc init -> ввод токена -> выбор пространства (default)

3. Генерация ключей
	ssh-keygen -t rsa

4. Создание Docker хост в Yandex Cloud
	yc compute instance create --name docker-host --zone ru-central1-a --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 --ssh-key ~/.ssh/id_rsa.pub

5. Инициализация окружение Docker в YC
	docker-machine create --driver generic --generic-ip-address=<IP_СОЗДАНОГО_ИНСТАНСА_YC> --generic-ssh-user yc-user --generic-ssh-key ~/.ssh/id_rsa docker-host

6. Переключение на созданный Docker-host
	eval $(docker-machine env docker-host)

7. Сравнение вывода команд docker run --rm -ti tehbilly/hto и docker run --rm --pid host -ti tehbilly/htop
	
	docker run --rm -ti tehbilly/htop:
	Эта команда запускает контейнер с утилитой htop.
	Опция --rm указывает Docker удалить контейнер после его остановки, что позволяет избежать накопления неиспользуемых контейнеров.
	Опция -ti обеспечивает интерактивный режим (TTY), который позволяет взаимодействовать с процессом внутри контейнера.
	tehbilly/htop это образ Docker, который содержит утилиту htop.
	docker run --rm --pid host -ti tehbilly/htop:

	Эта команда также запускает контейнер с утилитой htop.
	Опция --pid host позволяет контейнеру видеть все процессы хостовой системы. Это означает, что htop внутри контейнера будет отображать процессы хостовой системы, а не только процессы контейнера.
	Опция --rm также указывает Docker удалить контейнер после его остановки.
	Опция -ti также обеспечивает интерактивный режим (TTY).
	
	Сравнение:
	Первая команда запустит htop внутри изолированной среды контейнера. htop будет видеть только процессы, запущенные в этом контейнере.
	Вторая команда также запустит htop, но он будет видеть все процессы на хостовой системе благодаря опции --pid host.

8. Сборка образа
	подготовил 4 файла: 
		Dockerfile - текстовое описание нашего образа (нужно указать версию bundler 'RUN gem install bundler -v 2.3.27')
		mongod.conf - подготовленный конфиг для mongodb
		db_config - содержит переменную окружения со ссылкой на mongodb
		start.sh - скрипт запуска приложения Вся работа происходит в папке

	docker build -t reddit:latest .

9. Запуск контейнера
	docker run --name reddit -d --network=host reddit:latest

10. Проверка и подключение к вебморде
	docker-machine ls
	http://<IP_адрес>:9292

11. Публикация на Docker hub
	docker login
 	docker tag reddit:latest grooou/otus-reddit:1.0
 	docker push grooou/otus-reddit:1.0

12. Проверка скачивания образа из Docker hub
	docker run --name reddit -d -p 9292:9292 grooou/otus-reddit:1.0

13. Ещё проверка
	docker logs reddit -f
		about to fork child process, waiting until server is ready for connections.
		forked process: 9
		child process started successfully, parent exiting
		Puma starting in single mode...
		* Puma version: 6.4.2 (ruby 2.5.1-p57) ("The Eagle of Durango")
		*  Min threads: 0
		*  Max threads: 5
		*  Environment: development
		*          PID: 32
		/reddit/helpers.rb:4: warning: redefining `object_id' may cause serious problems
		* Listening on http://0.0.0.0:9292

	docker exec -it reddit bash
		root@54d50d54f3bd:/# ps aux
			USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
			root           1  0.0  0.0  18388  2904 ?        Ss   12:50   0:00 /bin/bash /start.sh
			root           9  0.6  1.6 1015668 59056 ?       Sl   12:50   0:01 /usr/bin/mongod --fork --logpath /var/log/mongod.log --config /etc/mongodb.conf
			root          32  0.2  0.9 593764 36332 ?        Sl   12:50   0:00 puma 6.4.2 (tcp://0.0.0.0:9292) [reddit]
			root          43  0.1  0.0  18520  3320 pts/0    Ss   12:54   0:00 bash
			root          57  0.0  0.0  34416  2820 pts/0    R+   12:55   0:00 ps aux
	
	killall5 1
	docker start reddit
	docker stop reddit && docker rm reddit
	docker run --name reddit --rm -it grooou/otus-reddit:1.0 bash
		root@c5d441e69d67:/# ps aux
			USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
			root           1  0.1  0.0  18520  3348 pts/0    Ss   13:00   0:00 bash
			root          16  0.0  0.0  34416  2796 pts/0    R+   13:00   0:00 ps aux
		root@c5d441e69d67:/# exit

	docker inspect grooou/otus-reddit:1.0
	docker inspect grooou/otus-reddit:1.0 -f '{{.ContainerConfig.Cmd}}'
	docker run --name reddit -d -p 9292:9292 grooou/otus-reddit:1.0
	docker exec -it reddit bash
		root@09176e675f16:/# mkdir /test1234
		root@09176e675f16:/# touch /test1234/testfile
		root@09176e675f16:/# rmdir /opt
		root@09176e675f16:/# exit

	docker diff reddit
	docker stop reddit && docker rm reddit
	docker run --name reddit --rm -it grooou/otus-reddit:1.0 bash
		root@73654b356d12:/# ls /
