﻿# 1. Giới thiệu

Có thể hiểu một cách cơ bản, docker là ảo hóa ở tầng ứng dụng. Nghĩa là nó đóng gói mọi thứ (thư viện, môi trường) để một ứng dụng có thể chạy 
bình thường mà không cần cài đặt thêm bất cứ thứ gì. Các gói được đóng này gọi là Container. Trên một HĐH đang sử dụng, bạn có thể chạy cùng lúc vào 
Container về MySQL với các version khác nhau để so sánh tính năng. Việc này rất khó thực hiện nếu bạn cài trực tiếp MySQL trên OS đang sử dụng.

# 2. Cài đặt

Để có thể chạy được docker, chúng ta cần cài đặt docker trên máy tính của mình. Môi trường tôi thử nghiệm là Ubuntu Server 14.04 

- Đầu tiên ta cài đặt OS và cập nhật hệ điều hành
```sh
sudo apt-get update -y
sudo apt-get -y upgrade
sudo apt-get -y dist-upgrade
sudo apt-get autoremove -y
```

- Kiểm tra aufs hỗ trợ có sẵn chưa
```sh
sudo apt-get install linux-image-extra-`uname -r`
```

- Thêm key để xác thực các gói docker sẽ cài đặt
```sh
sudo apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

- Thêm repository để lấy các gói cài đặt
```sh
echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
```

- Chạy lệnh update để cập nhật repository mới.
```sh
sudo apt-get update -y
```

- Tiến hành cài đặt docker
```sh
sudo apt-get install docker-engine -y
```

- Mặc định firewall từ chối các lưu lượng chuyển tiếp, ta cần cho phép chuyển tiếp trên firewall 
	- Mở file cấu hình firewall `/etc/default/ufw` và tìm dòng bắt đầu với `DEFAULT_FORWARD_POLICY` để chuyển giá trị từ `DROP` thành `ACCEPT`
```sh
DEFAULT_FORWARD_POLICY="ACCEPT"
```
	Press CTRL+X and approve with Y to save and close.
	- reload lại UFW
```sh
sudo ufw reload
```

*NOTE*: Nhớ kiểm tra các port trên firewall để đảm bảo enable thì vẫn ssh được
```sh
ufw status
ufw allow 22
ufw enable
```

# 3. Sử dụng docker

Sau khi cài đặt xong, ta có thể kiểm tra tiến trình docker daemon đã chạy chưa. nếu chưa chạy thì cần khởi động lên.
```sh
root@ELKServer2:~# ps -ef |grep docker
root      29466      1  0 09:08 ?        00:00:01 /usr/bin/dockerd --raw-logs
root      29476  29466  0 09:08 ?        00:00:00 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc
```

Câu lệnh thao tác docker có dạng như sau:
```sh
sudo docker [option] [command] [arguments]
```

Ta có thể liệt kê một loạt các tham số trong câu lệnh thao tác docker.
```sh
root@ELKServer2:~# sudo docker

Usage:	docker COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/root/.docker")
  -D, --debug              Enable debug mode
      --help               Print usage
  -H, --host list          Daemon socket(s) to connect to (default [])
  -l, --log-level string   Set the logging level ("debug", "info", "warn", "error", "fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  container   Manage containers
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  volume      Manage volumes

Commands:
  attach      Attach to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.
```

Kiểm tra các thông tin hệ thống cùng phiên bản docker cài đặt theo cách sau:
```sh
root@ELKServer2:~# sudo docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.03.0-ce
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 0
 Dirperm1 Supported: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins: 
 Volume: local
 Network: bridge host macvlan null overlay
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 977c511eda0925a723debdc94d09459af49d082a
runc version: a01dafd48bc1c7cc12bdb01206f9fea7dd6feb70
init version: 949e6fa
Security Options:
 apparmor
Kernel Version: 4.4.0-62-generic
Operating System: Ubuntu 14.04.5 LTS
OSType: linux
Architecture: x86_64
CPUs: 1
Total Memory: 1.936 GiB
Name: ELKServer2
ID: GPMR:7P3S:QM42:GYV5:HVSZ:ZZCI:BFIW:VCZ6:23XI:5PVR:CB2Y:PWCL
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
WARNING: No swap limit support
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false

root@ELKServer2:~# sudo docker version
Client:
 Version:      17.03.0-ce
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   60ccb22
 Built:        Thu Feb 23 10:57:47 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.03.0-ce
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   60ccb22
 Built:        Thu Feb 23 10:57:47 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

## Làm việc với Images

Image là các container có sẵn được người dùng chia sẻ trên mạng. Ta có thể tự làm hoặc lấy các Image có sẵn về sử dụng.

- Tìm kiếm một Image
```sh
sudo docker search ubuntu
sudo docker search mysql
sudo docker search mariadb
```

- Downloading (PULLing) an image mà bạn cần, nên lấy các bản được cộng đồng đánh giá cao.
```sh
sudo docker pull mysql
```

*NOTE*: ở bước này nếu download xuất hiện lỗi `Error response from daemon` và không download được, bạn cần mở port 443 trên firewall
```sh
# ufw allow 443
```

- Liệt kê các Images được download về
```sh
sudo docker images
```

- Running a container, liên kết một thư mục trên máy tính ngoài vào container, chỉ định image chạy, và chạy lệnh gì trong container.
```sh
docker run -v <thư mục trên máy tính>:<thư mục trong container> -it <image name> /bin/bash
```

- Khi cần gán port (NAT 1-1) giữa máy tính ngoài và docker ta làm như sau:
```sh
docker run -v /abc:/abc -p 8080:8080 -it ubuntu /bin/bash
```

# Test

## MySQL

Tôi thực hiện thử nghiệm docker với mysql để kiểm tra cách hoạt động.

- Đầu tiên tôi kéo 1 image mysql về bằng lệnh `pull`.

- Tôi thực hiện tạo một container từ image mysql vừa lấy về bằng lệnh
```sh
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=tan124 -d mysql
```

- Sau khi chạy xong lệnh trên, kiểm tra lại container xem đã chạy chưa
```sh
root@ELKServer2:~/code# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
2ebbbddc9fdc        mysql               "docker-entrypoint..."   10 seconds ago      Up 9 seconds        0.0.0.0:3306->3306/tcp   mysql
```

- Thực hiện login vào database mysql vừa chạy trên container
```sh
root@ELKServer2:~/code# mysql -uroot -ptan124 --host=172.16.68.17 --port=3306
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
mysql>
```

*Note*: để login được vào mysql thì máy tính ngoài cần cài đặt mysql client bằng lệnh sau:
```sh
root@ELKServer2:~/code# apt-get install mysql-client-core-5.5
```

	
# Tham khảo
- [https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-getting-started](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-getting-started)
- [https://kipalog.com/posts/Toi-da-dung-Docker-nhu-the-nao](https://kipalog.com/posts/Toi-da-dung-Docker-nhu-the-nao)