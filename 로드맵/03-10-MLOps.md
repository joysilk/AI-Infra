# 03-10. MLOps (모델 배포, MLflow, Kubeflow, 데이터 파이프라인, GPU 인프라)

> **MLOps**: 머신러닝(ML) 모델을 "한 번 만들고 끝"이 아니라 데이터 수집 → 학습 → 배포 → 모니터링 → 재학습의 순환을 자동화하고 안정적으로 운영하는 일. DevOps + 데이터 + 모델 관리입니다. DevOps/클라우드 1~3년 경험 후 도전을 권장하지만, 개념과 작은 프로젝트는 0년차부터 시작할 수 있습니다.

## 1. 개념 설명

### 1-1. ML 기초 (인프라 엔지니어가 알아야 할 만큼)
- **학습(training)**: 데이터로 모델의 파라미터를 최적화(GPU 자원 대량 소비, 배치성). **추론(inference)**: 학습된 모델로 예측(지연시간/처리량이 중요, 온라인 서빙).
- **피처(feature)**: 모델 입력 변수. **하이퍼파라미터**: 학습 전 설정값. **실험 추적**: 어떤 데이터·코드·설정으로 어떤 성능이 나왔는지 기록.
- **드리프트**: 시간이 지나 데이터 분포가 변해(data drift) 또는 입력-정답 관계가 변해(concept drift) 모델 성능이 떨어지는 현상.
- **LLM 시대**: 모델 서빙(vLLM, TGI), 양자화, KV 캐시, GPU 메모리 관리, RAG(검색 증강 생성), 벡터 DB가 인프라 이슈로 부상 (LLMOps).

### 1-2. ML 생애주기와 MLOps 성숙도
1. 데이터 수집/검증/버전 관리 → 2. 피처 엔지니어링 → 3. 학습/실험 → 4. 평가/검증 → 5. 모델 레지스트리 등록 → 6. 배포(서빙) → 7. 모니터링 → 8. 재학습 트리거.
- 레벨 0(수동 노트북) → 레벨 1(학습 파이프라인 자동화) → 레벨 2(CI/CD/CT: 지속적 학습까지).
- **ML은 코드 + 데이터 + 모델** 세 가지가 모두 버전 관리 대상.

### 1-3. 주요 도구
- **실험 추적/모델 레지스트리: MLflow** — Tracking(파라미터·지표·아티팩트 기록), Projects, Models(포맷), **Model Registry**(Staging/Production 승격, 버전). 대안: Weights & Biases, Neptune, DVC(데이터 버전).
- **파이프라인/오케스트레이션**: **Airflow**(데이터 파이프라인 표준, DAG), **Kubeflow Pipelines**(쿠버네티스 기반 ML 워크플로), Argo Workflows, Prefect, Dagster, Flyte.
- **Kubeflow**: 쿠버네티스 위 ML 플랫폼(노트북, 파이프라인, 학습 오퍼레이터(Training Operator), KServe, Katib 하이퍼파라미터 튜닝). 설치/운영이 무거워 소규모는 MLflow+Argo/Airflow 조합도 많음.
- **모델 서빙**: **KServe**(쿠버네티스 표준, 서버리스 스케일링·카나리), **Seldon**, **BentoML**, **TorchServe**, **NVIDIA Triton**(다양한 프레임워크·배치·GPU 최적화), **vLLM**(LLM 고성능), Ray Serve. 클라우드: SageMaker, Vertex AI, Azure ML.
- **피처 스토어**: Feast. **데이터 품질**: Great Expectations. **모니터링**: Evidently, WhyLabs, Prometheus+Grafana.
- **데이터 파이프라인**: Kafka(스트리밍), Spark(대규모 처리), dbt(변환), 데이터 레이크(S3 + Parquet/Iceberg), 웨어하우스(BigQuery/Redshift/Snowflake).

### 1-4. 모델 배포 패턴
- **온라인(실시간) 서빙**: REST/gRPC API, 낮은 지연. **배치 추론**: 주기적으로 대량 예측. **스트리밍**. **엣지**.
- 안전 배포: 섀도 배포(실트래픽 복제해 비교), 카나리, A/B 테스트, 롤백. 모델 서명/검증, 재현 가능한 컨테이너 이미지.
- 최적화: ONNX, TensorRT, 양자화, 배치 처리(dynamic batching), 모델 캐시.

### 1-5. GPU 인프라
- **GPU**: 행렬 연산 병렬 처리 하드웨어. NVIDIA(A100/H100/L4/T4 등). 핵심 지표: GPU 메모리(VRAM), 활용률, 대역폭, NVLink(GPU 간 연결).
- 쿠버네티스에서: **NVIDIA GPU Operator**/Device Plugin(드라이버·런타임 설치와 `nvidia.com/gpu` 리소스 노출), 노드 라벨/테인트로 GPU 노드 분리, **MIG**(GPU 분할), **time-slicing**(공유), 스케줄러(Kueue, Volcano — 대기열/갱 스케줄링), 분산 학습(PyTorch DDP, Horovod, Ray).
- 비용: GPU는 비쌈 → Spot/Preemptible, 자동 스케일(Karpenter), 유휴 방지, 오프피크 학습, 활용률 모니터링(DCGM exporter + Prometheus), 작은 모델/양자화로 대체 가능성 검토.
- 스토리지/네트워크: 대용량 데이터셋 접근(FSx for Lustre, S3 + 캐시), 고속 네트워크(EFA, InfiniBand).
- 클라우드 GPU 서비스: AWS(P/G 인스턴스, SageMaker), GCP(A3, TPU), Azure(ND), 국내(네이버클라우드, KT, NHN, 가비아 GPU 서비스).

## 2. 공부 순서
(선수) Linux, Docker, K8s, 클라우드, CI/CD
→ ML 기초(scikit-learn 모델 하나 학습, 지표 이해) → MLflow 추적/레지스트리 → 모델 API 서빙(FastAPI) 컨테이너화 → 파이프라인(Airflow 또는 Kubeflow Pipelines) → KServe/Triton 서빙 → 모니터링/드리프트 → GPU 노드/GPU Operator → 분산 학습/LLM 서빙(vLLM) → 비용/스케줄링 최적화.
- 기간: 입문 6~8주(DevOps 선수 지식 있다는 가정). 진짜 역량은 업무에서 형성.

## 3. 실습 과제
1. scikit-learn/XGBoost로 간단한 분류 모델 학습 → MLflow로 파라미터/지표/모델 기록, 실험 3개 비교.
2. MLflow Model Registry에서 모델 버전 승격(Staging→Production).
3. 모델을 FastAPI로 서빙 → Docker → 쿠버네티스 배포 + HPA + 헬스체크 → 부하 테스트(k6)로 지연 측정.
4. GitHub Actions: 코드 변경 시 학습 → 평가 임계치 통과 시 이미지 빌드·레지스트리 푸시 → 스테이징 배포(CI/CD for ML).
5. Airflow DAG: 데이터 수집→검증(Great Expectations)→학습→등록. 실패 알림.
6. KServe로 InferenceService 배포, 카나리 트래픽 10% 분할.
7. Evidently로 드리프트 리포트 생성, 임계치 초과 시 재학습 트리거.
8. GPU: (비용 주의) 클라우드 GPU 1대 또는 로컬 GPU로 GPU Operator/DCGM exporter → Grafana GPU 활용률 대시보드.
9. (심화) vLLM으로 오픈소스 LLM 서빙, 처리량·지연 측정, 양자화 전후 비교.
10. (심화) Kubeflow를 kind/minikube나 클라우드에 설치해 파이프라인 1개 실행 (리소스 요구가 큼 → 짧게 실습 후 삭제).

## 4. 추천 자료
- 공식(무료): MLflow 문서, Kubeflow 문서, KServe 문서, Airflow 문서, NVIDIA GPU Operator 문서, Triton 문서, vLLM 문서, Google "MLOps: Continuous delivery and automation pipelines in ML".
- 한국어: 「머신러닝 시스템 디자인 패턴」, 「MLOps 입문」 계열 번역서, 카카오/네이버/당근/토스 ML 플랫폼 기술 블로그, 모두의연구소/AI 커뮤니티(MLOps KR 페이스북 그룹).
- 영어: 「Designing Machine Learning Systems」(Chip Huyen), 「Machine Learning Engineering」(Andriy Burkov), Made With ML, DataTalks.Club MLOps Zoomcamp(무료), Full Stack Deep Learning, Coursera MLOps Specialization(DeepLearning.AI).
- 자격: AWS Machine Learning Engineer Associate/ML Specialty, Google Professional ML Engineer, CKA(필수 기반), NVIDIA 교육.

## 5. 면접 단골 질문
1. MLOps와 DevOps의 차이?
2. 모델 배포 시 코드/데이터/모델 버전을 어떻게 관리?
3. 모델 드리프트란? 어떻게 감지하고 대응하나?
4. 온라인 서빙과 배치 추론의 선택 기준?
5. MLflow가 해결하는 문제(추적·레지스트리)?
6. Kubeflow vs 직접 구성(MLflow + Argo/Airflow), 어떤 기준으로?
7. GPU를 쿠버네티스에서 어떻게 스케줄링·공유하나?
8. GPU 비용을 줄이는 방법?
9. 모델 서빙 지연을 줄이려면?
10. 새 모델을 안전하게 배포하는 방법(섀도/카나리)?
(모범 답변은 [06-3](06-3-기술면접-질문50.md))
