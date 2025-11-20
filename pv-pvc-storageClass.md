# 요약
- PV
  * 물리적인 볼륨을 지칭,의미하는 클러스터 수준의 객체
  * Cluster 수준에서 관리되며,
- PVC
  * Namespace 수준에서 PV를 소환(Claim)하는 객체
  * PVC = PV를 Namespace에서 요청하는 것
  * Pod는 PVC를 통해서 PV를 거쳐서 물리적인 스토리지에 연결됨
- StorageClass
  * 수동으로 PV를 미리 만들지 않고 PVC를 이용해 필요할 때 동적으로 볼륨을 할당하는 방식
- CSI(Container Storage Interface)
  * 쿠버네티스가 다양한 스토리지 벤더(AWS, GCP, Ceph 등)를 표준 방식으로 연결할 수 있도록 만든 인터페이스
  * CSI 드라이버가 실제 스토리지를 연결
  * 클라우드/온프레미스/외부 스토리지 모두 통합적으로 다룰 수 있음
    
# Life Cycle 4단계
### Provisioning
- 정적 프로비저닝 : 미리 관리자가 수동으로 생성하는 방식
- 동적 프로비저닝
  * 사용자가 PVC를 통해 PV를 요청하게 될 때 PV를 동적으로 생성함.
  * 어떤 종류의 Storage를 프로비저닝할지를 StorageClass로 정의한 후 PVC에서 StorageClass를 지정하면 PVC가 생성될 때 PV가 동적으로 프로비저닝됨
### Binding
- PV를 PVC에 연결하는 작업을 의미함 ( PV와 PVC는 1:1 관계 )
- PVC에서 스토리지의 용량과 액세스방법을 지정하여 PV를 요청하면 해당 요청에 부합하는 PVC가 바인딩됨
  * PVC가 요청하는 조건에 부합하는 PV가 없으면 PV가 생성될 때까지 Pending 상태로 대기함
### Using
- PVC를 Volume으로 지정하여 컨테이너에서 mount하여 사용함
- Pod에 연결된 PVC는 Pod가 실행된 동안 유지되며, 임의로 삭제될 수 없음(일종의 스토리지 보호 기능)
### Reclaiming
- 더이상 사용되지 않는 PVC를 삭제하거나 사용했던 PVC를 초기화하는 단계
- 초기화 정책
  * Retain
    - PV가 삭제되더라도 PC의 데이터는 클러스터에 유지됨.
    - 데이터의 보존,백업은 관리자가 직접해야 함.
    - PV를 다시 사용하려면 수동으로 다시 바인딩해야 함
  * Delete
    - PV가 삭제되는 PV와 관련된 데이터들도 함께 삭제되는 방식
    - 테스트 환경에서 주로 사용되는 방식
    - 데이터 보존이 필요하지 않을 때 사용함
    - 동적으로 프로비저닝된 PV의 Reclaiming 정책의 기본값은 Delete임.

## StorageClass를 이용한 동적 프로비저닝 예시
- StorageClass 정의
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata: 
  name: ebs-sc-sample
# WaitForFirstConsumer : Pod가 생성된 후에만 PV가 PVC에 바인딩된다는 의미
# Immediate : PV가 생성되고 PVC 생성 시 바인딩된다는 의미
volumeBindingMode: WaitForFirstConsumer
# EBS CSI Driver가 Storage를 프로비저닝함. 
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
```

- Storage Class를 요청하는 PVC
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-sc-sample
spec:
  storageClassName: ebs-sc-sample
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

- PV, PVC 연결 확인
  * kubectl get pvc,pv -n namespace
- PV status
  * Available
  * Bound
  * Released
  * Failed
