# Gitlab Chart
> - Pods: Gitlab, PostgreSQL, Redis
> - ~~Service, VirtualService(Istio), PV, PVC, Configmap 설정~~ -> Helm Chart 로 구조 변경
> - 개인 또는 5인 미만의 팀에서 사용 예정

##### PreRequirement
- `gitlab` Namespace 생성, istio-injection 활성화
```bash
$ kubectl create namespace gitlab
$ kubectl label namespace gitlab istio-injection=enabled --overwrite
```
- gitlab
  - `domain`: 공개할 도메인 연결, `gitlab.{domain}` 으로 연결됩니다.
  - `ssh_sub_domain`: CloudFlare Proxy 사용 시 별도 사용할 Non Proxy Domain
  - `initial_root_password`: gitlab 최초 root 비밀번호
  -  `tls_credential_name`: `gitlab` namespace에 배포된 [cert-manager TSL secret](https://cert-manager.io/docs/usage/certificate/)의 secret name
- smtp
  - `user`: 메일 발송자 명
  - `password`: [Google App Password](https://support.google.com/accounts/answer/185833?hl=en)

```yaml
gitlab:
  domain: "{example.com}"
  protocol: "http://"
  ssh_sub_domain: "services"
  initial_root_password: "{your-first-root-password}"
  tls_credential_name: "{your-tls-secret-name}"

smtp:
  server: "smtp.gmail.com"
  user: "{smtp outmail user name}"
  password: "{smtp password}"
  ```

##### Installing
```bash
$ helm install gitlab ./gitlab-omnibus-with-Istio -n gitlab --values ./gitlab-omnibus-with-Istio/values.yaml
```

##### Uninstalling
```bash
$ helm uninstall gitlab -n gitlab
```

### SMTP
- google - account - app password 에서 smtp 용 password 발행 후 사용

#### Gitlab Bugs
- [Setting.General.Sign-in restrictions.Home Page URL](https://gitlab.com/gitlab-org/gitlab-foss/-/issues/1020): 공란으로 비워두세요.


<details><summary>Etc...</summary>

### Improvements
- [x] ~~Node ssh Port와 gitlab ssh Port 겹침, 추후에 Cluster - Control Node 환경 분리~~ Port 겹침의 문제가 아닌, CloudFlare Proxy CE 의 L4 Proxy 미지원 문제
- [x] ~~[Requirements](https://docs.gitlab.com/ee/install/requirements.html), [Prerequirements](https://docs.gitlab.com/charts/installation/tools.html): Gitaly(Backup, Replication), Minio(AWS 3S), Prometheus, Gitlab Runner 기능이 필요한 경우 활성화~~ => 단일 노드 사용 중이므로 Gitaly 사용 의미 x, Istio Prometheus 연계하여 사용
- [x] [outgoing-email-configuration](https://docs.gitlab.com/charts/installation/command-line-options.html#outgoing-email-configuration): Outgoing Email 활성화 필요
- [x] [GitLab Requirement](https://docs.gitlab.com/ee/install/requirements.html), [Hybrid referenece architecture](https://docs.gitlab.com/charts/installation/index.html#use-the-reference-architectures): 요청 수가 늘어날 경우
- [x] Cluster 환경에 GitLab을 설치 후 운용해야 하는 경우 -> 다른 Repo에서 진행
  - [x] Reliability, Availability, Observability, Scalability 관점 고민
  - [x] Volume: NFS, AZURE Disk, AWS EBS, GCP PD
- [x] ~~Git secret 기반 환경 변수 추가하기~~ Chart Template 사용

</details>


