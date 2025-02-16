# Gitlab Deployment
> 단일 Node에서 Istio와 Gitlab 사용을 위한 가이드<br>
> 현재 환경에서 [Omnibus](https://docs.gitlab.com/omnibus/) 설치가 가장 쉽고 당장 할 수 있었으므로 적용. <br>
> - Pods: Gitlab, PostgreSQL, Redis
> - Service, VirtualService(Istio), PV, PVC, Configmap 설정
> - 개인 또는 5인 미만의 팀에서 사용 예정

#### about SSH port
- CloudFlare Proxy 무료 버전으로 Gitlab Web Service 호스팅 중<br>
  그로 인해 L4 Proxy가 지원되지 않아 ssh는 Proxying 하지 않는 별도 domain으로 접근한다.
- Istio Gateway의 default setting은 80, 443, 15021(Prometheus) 만 호스팅하므로,<br>
  별도의 tcp Gateway 를 추가하여 2222(기본 node ssh와 겹치므로.. 이는 제 환경이 단일 노드라 그렇습니다.) tcp 를 추가하여야 한다.

Helm Istio Chart 의 value.yaml 중 service - port 2222 추가 (또는 분리된 gateway 별도 설치)
```yaml
...
  service:
    # Type of service. Set to "None" to disable the service entirely
    type: LoadBalancer
    ports:
    - name: gitlab-ssh
      port: 2222
      protocol: TCP
      targetPort: 2222
...
```

이후 Istio Gateway Selector 에서 2222 지정
```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: tcp-gateway
  namespace: default
spec:
  selector:
    istio: tcp-ingress
  servers:
  - port:
      number: 2222
      name: gitlab
      protocol: TCP
    hosts:
    - "services.example.com"
```

### SMTP
- google - account - app password 에서 smtp 용 password 발행 후 사용

### Improvements
- [ ] Node ssh Port와 gitlab ssh Port 겹침, 추후에 Cluster - Control Node 환경 분리
- [ ] [Requirements](https://docs.gitlab.com/ee/install/requirements.html), [Prerequirements](https://docs.gitlab.com/charts/installation/tools.html): Gitaly(Backup, Replication), Minio(AWS 3S), Prometheus, Gitlab Runner 기능이 필요한 경우 활성화
- [ ] [outgoing-email-configuration](https://docs.gitlab.com/charts/installation/command-line-options.html#outgoing-email-configuration): Outgoing Email 활성화 필요
- [ ] [GitLab Requirement](https://docs.gitlab.com/ee/install/requirements.html), [Hybrid referenece architecture](https://docs.gitlab.com/charts/installation/index.html#use-the-reference-architectures): 요청 수가 늘어날 경우
- [ ] Cluster 환경에 GitLab을 설치 후 운용해야 하는 경우
  - [ ] Reliability, Availability, Observability, Scalability 관점 고민
  - [ ] Volume: NFS, AZURE Disk, AWS EBS, GCP PD
- [ ] Git secret 기반 환경 변수 추가하기

### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gitlab-config
  namespace: gitlab
data:
  gitlab.rb: |
    external_url 'http://gitlab.example.com'
    gitlab_rails['gitlab_shell_ssh_port'] = 2222
    gitlab_rails['initial_root_password'] = "PASSWORD"
    postgresql['enable'] = false
    gitlab_rails['db_host'] = "postgresql"
    gitlab_rails['db_port'] = "5432"
    gitlab_rails['db_username'] = "gitlab"
    gitlab_rails['db_password'] = "gitlab"
    gitlab_rails['db_database'] = "gitlabhq_production"
    gitlab_rails['db_adapter'] = 'postgresql'
    gitlab_rails['db_encoding'] = 'utf8'
    redis['enable'] = false
    gitlab_rails['redis_host'] = 'redis'
    gitlab_rails['redis_port'] = '6379'
    # smtp
    gitlab_rails['smtp_enable'] = true
    gitlab_rails['smtp_address'] = "smtp.gmail.com"
    gitlab_rails['smtp_port'] = 587
    gitlab_rails['smtp_user_name'] = "yourmail@example.com"
    gitlab_rails['smtp_password'] = "your-app-password"
    gitlab_rails['smtp_domain'] = "smtp.gmail.com"
    gitlab_rails['smtp_authentication'] = "login"
    gitlab_rails['smtp_enable_starttls_auto'] = true
    gitlab_rails['smtp_tls'] = false
    gitlab_rails['smtp_pool'] = false
```

#### Bugs
- [Setting.General.Sign-in restrictions.Home Page URL](https://gitlab.com/gitlab-org/gitlab-foss/-/issues/1020): 공란으로 비워두세요.