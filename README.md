# observability

Grafana, Prometheus, Node Exporter, cAdvisor를 사용하여 컨테이너 및 노드 모니터링

## Getting started

1. nginx에서 사용할 도메인 등록

- [호스팅케이알](https://www.hosting.kr/)에 사용할 도메인 등록

2. `docker-compose.yml`파일에서 "<>"로 된 부분 수정
	```yaml
	...
	nginx:
	  ports:
	    - "<Port>:<Port>"
	  volumes:
	    - <Cert path>:/cert.pem
	    - <Key path>:/key.pem
	    - <DH parameters file path>:/dhparams4096.pem
	    - ${PWD}/nginx/conf.d:/etc/nginx/conf.d:ro
	    - /etc/localtime:/etc/localtime:ro
	...
	node-exporter:
	  expose:
	    - '<Port>'
	...
	```

3. nginx설정에서 "<>"로 된 부분 변경
	```
	...
	## deny other domains for grafana
	server {
		# The ssl parameter allows specifying that all connections accepted on this port should work in SSL mode.
		# The http2 parameter configures the port to accept HTTP/2 connections.
		listen <Port> ssl http2;
		# IPv6 addresses are specified in square brackets.
		listen [::]:<Port> ssl http2;
		# Sets names of a virtual server.
		server_name _;
	}

	server {
		# The ssl parameter allows specifying that all connections accepted on this port should work in SSL mode.
		# The http2 parameter configures the port to accept HTTP/2 connections.
		listen <Port> ssl http2;
		# IPv6 addresses are specified in square brackets.
		listen [::]:<Port> ssl http2;
		# Sets names of a virtual server.
		server_name <Domain name>;
	}
	...
	```

4. `prometheus.yml` 파일에서 node exporter가 실행된 노드의 hostname이나 IP 입력
	```YAML
	- job_name: 'node-exporter'
	  # Override the global default and scrape targets from this job every 5 seconds.
	  scrape_interval: 5s
	  # scheme defaults to 'http'.
	  static_configs:
	    - targets: ['<HOST name or IP>:<Port>']
	      labels:
	        container: 'node-exporter'
	```

5. (Optional) 컨테이너들이 실행되는 노드의 방화벽이 활성화 되어 있을 경우 아래 포트들을 허용해주어야 함. **만일 `docker-compose.yml`, nginx의 `default.conf`에서 포트를 수정했으면 수정한 포트를 방화벽에 허용해주어야 함**
	- 9100 (node exporter)
	- 43604 (nginx)

	- For Ubuntu:
		```bash
		$ sudo ufw allow 9100/tcp
		$ sudo ufw allow 43604/tcp
		```

6. 아래 명령어로 docker compose 시작
	```bash
	$ docker compose -p observability -f docker-compose.yml up -d
	```

7. nginx에 설정한 도메인:포트로 grafana 접속

8. admin 계정 초기 패스워드 변경

9. **"Home > Dashboards > General"** 아래에 2개의 대시보드가 정의되어 있음
	- Container Metrics
	- Node Metrics

## Usage

- compose up
```bash
$ docker compose -p observability -f docker-compose.yml up -d
```

- compose down
```bash
$ docker compose -p observability -f docker-compose.yml down
```

- list containers via `docker compose`
```bash
$ docker compose -p observability -f docker-compose.yml ps
```

- Fetch the logs of a container
```bash
$ docker logs -f <Container name>
```
