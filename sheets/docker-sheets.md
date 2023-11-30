# 🛳 Docker Sheets

<figure><img src="../.gitbook/assets/docker-containers-wallpaper-preview.jpeg" alt=""><figcaption></figcaption></figure>

| Komut                           | Açıklama                                                                   | Komut Kullanım Örneği                            |
| ------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------ |
| `docker --version`              | Docker sürümünü gösterir.                                                  | `docker --version`                               |
| `docker info`                   | Docker sistem bilgisini gösterir.                                          | `docker info`                                    |
| `docker run`                    | Yeni bir konteyner başlatır.                                               | `docker run nginx`                               |
| `docker ps`                     | Çalışan konteynerleri listeler.                                            | `docker ps`                                      |
| `docker ps -a`                  | Tüm konteynerleri listeler.                                                | `docker ps -a`                                   |
| `docker stop <container_id>`    | Bir konteyneri durdurur.                                                   | `docker stop 1a2b3c4d`                           |
| `docker start <container_id>`   | Durdurulmuş bir konteyneri başlatır.                                       | `docker start 1a2b3c4d`                          |
| `docker restart <container_id>` | Bir konteyneri yeniden başlatır.                                           | `docker restart 1a2b3c4d`                        |
| `docker rm <container_id>`      | Bir konteyneri siler.                                                      | `docker rm 1a2b3c4d`                             |
| `docker images`                 | Yereldeki Docker imajlarını listeler.                                      | `docker images`                                  |
| `docker rmi <image_id>`         | Bir Docker imajını siler.                                                  | `docker rmi abc123`                              |
| `docker pull <image_name>`      | Docker Hub'dan bir imaj indirir.                                           | `docker pull nginx`                              |
| `docker push <image_name>`      | Docker imajını Docker Hub'a yükler.                                        | `docker push myimage`                            |
| `docker build`                  | Dockerfile'dan bir Docker imajı oluşturur.                                 | `docker build -t myimage:latest .`               |
| `docker exec`                   | Çalışan bir konteynerde komut çalıştırır.                                  | `docker exec -it 1a2b3c4d /bin/bash`             |
| `docker logs <container_id>`    | Bir konteynerin loglarını gösterir.                                        | `docker logs 1a2b3c4d`                           |
| `docker volume create`          | Yeni bir Docker volume oluşturur.                                          | `docker volume create myvolume`                  |
| `docker volume ls`              | Docker volumelerini listeler.                                              | `docker volume ls`                               |
| `docker volume rm`              | Bir Docker volume siler.                                                   | `docker volume rm myvolume`                      |
| `docker network create`         | Yeni bir Docker ağı oluşturur.                                             | `docker network create mynetwork`                |
| `docker network ls`             | Docker ağlarını listeler.                                                  | `docker network ls`                              |
| `docker network rm`             | Bir Docker ağını siler.                                                    | `docker network rm mynetwork`                    |
| `docker login`                  | Docker Hub'a giriş yapar.                                                  | `docker login`                                   |
| `docker logout`                 | Docker Hub'dan çıkış yapar.                                                | `docker logout`                                  |
| `docker-compose up`             | docker-compose.yml dosyasını kullanarak servisleri başlatır.               | `docker-compose up -d`                           |
| `docker-compose down`           | docker-compose.yml dosyasındaki servisleri durdurur.                       | `docker-compose down`                            |
| `docker-compose logs`           | docker-compose.yml dosyasındaki servislerin loglarını gösterir.            | `docker-compose logs`                            |
| `docker-compose build`          | docker-compose.yml dosyasını kullanarak imajları oluşturur.                | `docker-compose build`                           |
| `docker-compose ps`             | docker-compose.yml dosyasındaki servisleri listeler.                       | `docker-compose ps`                              |
| `docker-compose restart`        | docker-compose.yml dosyasındaki servisleri yeniden başlatır.               | `docker-compose restart`                         |
| `docker-compose pull`           | docker-compose.yml dosyasındaki imajları Docker Hub'dan indirir.           | `docker-compose pull`                            |
| `docker-compose push`           | docker-compose.yml dosyasındaki imajları Docker Hub'a yükler.              | `docker-compose push`                            |
| `docker-compose rm`             | docker-compose.yml dosyasındaki durdurulmuş servisleri siler.              | `docker-compose rm`                              |
| `docker-compose config`         | docker-compose.yml dosyasının yapılandırmasını doğrular ve görüntüler.     | `docker-compose config`                          |
| `docker-compose top`            | docker-compose.yml dosyasındaki servisler için çalışan işlemleri listeler. | `docker-compose top`                             |
| `docker-compose version`        | docker-compose sürümünü gösterir.                                          | `docker-compose version`                         |
| `docker-compose scale`          | docker-compose.yml dosyasındaki servislerin örnek sayısını ayarlar.        | `docker-compose scale web=3`                     |
| `docker-compose pause`          | docker-compose.yml dosyasındaki servisleri duraklatır.                     | `docker-compose pause`                           |
| `docker-compose unpause`        | docker-compose.yml dosyasındaki duraklatılmış servisleri devam ettirir.    | `docker-compose unpause`                         |
| `docker-compose port`           | docker-compose.yml dosyasındaki servis için yayınlanan portu görüntüler.   | `docker-compose port web 80`                     |
| `docker-compose kill`           | docker-compose.yml dosyasındaki servisleri zorla durdurur.                 | `docker-compose kill`                            |
| `docker-compose events`         | docker-compose.yml dosyasındaki servisler için olayları görüntüler.        | `docker-compose events`                          |
| `docker-compose images`         | docker-compose.yml dosyasındaki servisler için imajları listeler.          | `docker-compose images`                          |
| `docker swarm init`             | Yeni bir Docker Swarm başlatır.                                            | `docker swarm init`                              |
| `docker swarm join`             | Bir Docker Swarm'a katılır.                                                | `docker swarm join --token TOKEN`                |
| `docker swarm leave`            | Bir Docker Swarm'dan ayrılır.                                              | `docker swarm leave`                             |
| `docker swarm update`           | Docker Swarm ayarlarını günceller.                                         | `docker swarm update --cert-expiry 48h`          |
| `docker node ls`                | Docker Swarm'daki düğümleri listeler.                                      | `docker node ls`                                 |
| `docker service create`         | Yeni bir Docker Swarm servisi oluşturur.                                   | `docker service create --name web nginx`         |
| `docker service ls`             | Docker Swarm servislerini listeler.                                        | `docker service ls`                              |
| `docker service update`         | Bir Docker Swarm servisini günceller.                                      | `docker service update web --image nginx:latest` |
| `docker service rm`             | Bir Docker Swarm servisini siler.                                          | `docker service rm web`                          |

