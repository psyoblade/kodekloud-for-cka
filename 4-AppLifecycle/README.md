# Application Lifecycle Management
> 배포 시에 어떻게 스케일링 할 것인 지에 대한 이야기를 합니다

## 1. Rolling Updates and Rollbacks

* Recreate Strategy
  - 모든 서비스를 일괄 내리고, 다시 올리는 전략을 말하는데 Application Downtown 이 발생하는 단점이 있습니다
* Rolling Update (Default)
  - 하나씩 종료 및 시작함으로써 서비스 다운 타임이 발생하지 않도록 하는 전략입니다

* How to update
  - 직접 이미지를 변경하는 것도 가능하지만, yaml 파일과 서버와 달라지는 점을 유의해야 합니다
```bash
bash> kubectl apply -f deployment.yaml
bash> kubect set image deployment/my-deployment nginx=nginx:1.9.1
```
