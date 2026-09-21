**Volume 08 Robot Database and Storage**

# 06. Data Lakes

## 06.01 Data Lake Architecture Overview for Robot Data

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 데이터(Robot Data)는 물리적 세계(Physical World)를 인식하고, 판단하며, 행동하는 기계에서 지속적으로 생성된다는 점에서 기존의 기업 데이터(Enterprise Data)와 근본적으로 다르다. 하나의 로봇은 카메라 프레임(Camera Frames), 라이다 포인트 클라우드(LiDAR Point Clouds), 깊이 맵(Depth Maps), 오디오(Audio), 관절 상태(Joint States), 모터 명령(Motor Commands), 위치 추정(Localization Estimates), 내비게이션 궤적(Navigation Trajectories), 진단 로그(Diagnostic Logs), AI 추론 결과(AI Inference Results)를 동시에 생성할 수 있다. 데이터 레이크(Data Lake)는 수집 단계에서 이러한 이기종 데이터 스트림(Heterogeneous Data Streams)을 하나의 경직된 스키마(Schema)에 강제로 맞추지 않고 저장할 수 있는 확장 가능한 기반을 제공한다.

로봇 데이터 레이크(Robot Data Lake)의 핵심 원칙은 원시 관측 데이터(Raw Observations)를 보존하면서 점진적으로 보다 구조화되고 재사용 가능한 표현(Representations)을 생성하는 것이다. 원본 센서 기록(Original Sensor Recordings)은 타임스탬프(Timestamps), 로봇 식별자(Robot Identifiers), 임무 정보(Mission Information), 보정 파라미터(Calibration Parameters), 소프트웨어 버전(Software Versions), 환경 맥락(Environmental Context)과 함께 보존할 수 있다. 이후 처리 데이터(Processed Data), 주석(Annotations), 임베딩(Embeddings), 궤적(Trajectories), 학습 샘플(Training Samples)을 원본을 대체하지 않고 파생함으로써 알고리즘이나 요구사항이 변경되더라도 실험을 재현하고 새로운 데이터셋(Datasets)을 생성할 수 있다.

실용적인 아키텍처(Architecture)는 데이터 레이크 전체를 하나의 거대한 저장 디렉터리(Storage Directory)로 취급하기보다 데이터의 수명주기(Data Lifecycle)에 따라 영역을 구분한다. 수집된 로봇 데이터는 먼저 인제션 또는 랜딩 영역(Ingestion or Landing Area)에 들어가며, 여기에서 파일과 스트림의 유효성을 검증하고 등록하며 우발적인 변경으로부터 보호한다. 검증된 정보는 이후 원시 영역(Raw Zone), 정제 영역(Curated Zone), 애플리케이션 지향 영역(Application-Oriented Zone)으로 이동할 수 있다. 이러한 계층적 구성은 데이터셋이 원본 증거(Original Evidence), 변환된 정보(Transformed Information), 특정 AI 워크로드(AI Workload)를 위해 준비된 버전 중 무엇을 의미하는지에 대한 모호성을 줄여준다.

인제션 계층(Ingestion Layer)은 로봇, 엣지 컴퓨터(Edge Computers), 게이트웨이(Gateways), 시뮬레이션 환경(Simulation Environments), 외부 데이터셋(External Datasets)을 데이터 레이크에 연결한다. 고대역폭 센서(High-Bandwidth Sensors)는 네트워크 연결이 지속적인 전송을 지원하지 못할 수 있으므로 데이터를 먼저 로컬 NVMe 저장장치(Local NVMe Storage)에 기록할 수 있다. 메타데이터(Metadata)와 중요 이벤트(Critical Events)는 즉시 전송하고 대규모 이미지 시퀀스(Image Sequences), 포인트 클라우드(Point Clouds), ROS 백 파일(ROS Bag Files)은 이후 동기화할 수 있다. 따라서 신뢰성 높은 데이터 수집에는 버퍼링(Buffering), 재시도 메커니즘(Retry Mechanisms), 체크섬(Checksums), 전송 매니페스트(Transfer Manifests), 불완전하거나 중복된 업로드를 탐지하는 기능이 필요하다.

저장 기술(Storage Technology)은 로봇 워크로드(Robot Workloads)의 규모와 접근 특성을 반영해야 한다. 객체 저장소(Object Storage)는 확장 가능한 네임스페이스(Namespaces)와 메타데이터 기능을 제공하므로 대규모 이미지, 비디오, 포인트 클라우드, 아카이브(Archives), 시뮬레이션 출력(Simulation Outputs), 모델 아티팩트(Model Artifacts)를 저장하는 데 특히 적합하다. NAS 시스템(NAS Systems)은 대화형 엔지니어링 워크플로(Interactive Engineering Workflows)와 공유 프로젝트 디렉터리(Shared Project Directories)에 여전히 유용하며, 고성능 로컬 저장장치(High-Performance Local Storage)는 활성 학습(Active Training)이나 전처리 작업(Preprocessing Jobs)을 지원할 수 있다. 데이터 레이크는 모든 워크로드가 하나의 물리적 저장 플랫폼(Physical Storage Platform)을 사용하도록 강제하는 대신 이러한 기술을 결합할 수 있다.

메타데이터(Metadata)는 대규모 파일 집합을 실제로 운영 가능한 로봇 데이터 레이크(Robot Data Lake)로 전환하는 핵심 요소이다. 각 데이터셋에는 로봇 유형(Robot Type), 센서 구성(Sensor Configuration), 수집 시간(Capture Time), 위치 범주(Location Category), 임무 식별자(Mission Identifier), 환경 조건(Environmental Conditions), 보정 버전(Calibration Version), 소프트웨어 빌드(Software Build), 처리 이력(Processing History) 등의 정보를 연결해야 한다. 카탈로그 서비스(Catalog Services)는 이러한 정보를 색인화하여 연구자가 수백만 개의 파일을 직접 탐색하는 대신 야간 내비게이션 시퀀스(Nighttime Navigation Sequences), 조작 실패(Manipulation Failures), 특정 카메라 구성(Camera Configurations), 희귀 장애물 상황(Rare Obstacle Encounters)과 같은 의미 있는 조건을 검색할 수 있도록 한다.

시간 동기화(Time Synchronization)는 피지컬 AI(Physical AI)가 관측(Observations)과 행동(Actions) 사이의 관계에 의존하기 때문에 특히 중요하다. 카메라 이미지(Camera Images), 라이다 스캔(LiDAR Scans), 관성측정장치 측정값(IMU Measurements), 관절 위치(Joint Positions), 제어 명령(Control Commands), 위치 추정값(Localization Estimates)은 공통 타임라인(Common Timeline)을 기준으로 재구성되어야 하는 경우가 많다. 따라서 아키텍처는 원본 타임스탬프(Source Timestamps), 동기화 품질(Synchronization Quality), 클록 도메인(Clock Domains), 시간 보정 정보(Timing Corrections)를 보존해야 한다. 사용 가능한 경우 정밀 시간 프로토콜(PTP, Precision Time Protocol)이나 하드웨어 타임스탬프(Hardware Timestamps)를 활용하면 시간 정렬(Temporal Alignment)을 개선하고 분산 센서와 컴퓨팅 노드(Computing Nodes)에서 발생한 이벤트를 더욱 정확하게 재구성할 수 있다.

정제 계층(Curated Layer)은 원시 로봇 기록(Raw Robot Recordings)을 분석과 머신러닝(Machine Learning)에 적합한 형태로 변환한다. 처리 파이프라인(Processing Pipelines)은 센서 로그 디코딩(Sensor Log Decoding), 프레임 추출(Frame Extraction), 좌표계 변환(Coordinate Transformation), 손상 샘플 제거(Corrupted Sample Removal), 멀티모달 스트림 동기화(Multimodal Stream Synchronization), 품질 지표 계산(Quality Metric Calculation), 민감 정보 익명화(Sensitive Data Anonymization), 표준화된 데이터셋 매니페스트(Standardized Dataset Manifests) 생성을 수행할 수 있다. 중요한 점은 정제 데이터가 원본 소스까지 명확한 데이터 계보(Data Lineage)를 유지해야 한다는 것이다. 엔지니어는 특정 학습 데이터셋을 생성한 정확한 기록, 변환 과정, 파라미터, 소프트웨어 버전을 확인할 수 있어야 한다.

로봇 데이터 레이크는 주석(Annotation)과 의미론적 보강(Semantic Enrichment)도 지원해야 한다. 사람에 의한 주석(Human Annotations), 자동 레이블(Automated Labels), 분할 마스크(Segmentation Masks), 바운딩 박스(Bounding Boxes), 객체 추적(Object Tracks), 장면 설명(Scene Descriptions), 실패 범주(Failure Categories), 작업 결과(Task Outcomes)를 변경 불가능한 원본 관측 데이터(Immutable Source Observations)에 연결된 독립적인 데이터 제품(Data Products)으로 저장할 수 있다. 이러한 분리는 대규모 센서 데이터셋을 반복적으로 복사하지 않고도 레이블(Label)을 발전시킬 수 있도록 한다. 여러 주석 버전(Annotation Versions)이 동시에 존재할 수 있으므로 팀은 레이블링 정책(Labeling Policies)을 비교하고 품질을 향상시키며 과거 AI 실험을 재현할 수 있다.

피지컬 AI(Physical AI) 개발에서 데이터 레이크는 실제 환경 운영(Real-World Operation), 시뮬레이션(Simulation), 모델 학습(Model Training)을 연결하는 다리 역할을 한다. 실제 로봇 경험(Real Robot Experiences)을 선택하여 학습 데이터셋을 구성할 수 있으며, 운영 과정에서 발견된 어려운 상황은 시뮬레이션에서 재현할 수 있다. 디지털 트윈(Digital Twins)에서 생성한 합성 데이터(Synthetic Data) 역시 호환 가능한 메타데이터와 스키마를 이용하여 저장할 수 있다. 아키텍처는 실제 데이터(Real Data), 시뮬레이션 데이터(Simulated Data), 재구성 데이터(Reconstructed Data), 증강 데이터(Augmented Data)를 구분하면서도 통합된 데이터셋 질의(Dataset Queries)와 학습 파이프라인에 함께 사용할 수 있도록 해야 한다.

잘 설계된 데이터 레이크(Data Lake)는 AI 인프라(AI Infrastructure)와 직접 통합되어야 한다. 학습 시스템(Training Systems)은 데이터셋 버전(Dataset Versions)을 선택하고, GPU 자원 근처에 샘플을 스테이징(Staging)하며, 자주 접근하는 파일을 캐싱(Caching)하고, 각 모델에 어떤 데이터가 사용되었는지를 기록할 수 있는 효율적인 방법이 필요하다. 임의의 디렉터리를 학습 서버에 복사하는 대신 데이터셋 매니페스트(Dataset Manifests)를 이용해 재현 가능한 객체 집합(Reproducible Object Collections)을 정의할 수 있다. 이러한 접근법은 분산 학습(Distributed Training), 실험 추적(Experiment Tracking), 모델 비교(Model Comparison), 배포된 모델이 사용한 정확한 데이터 환경(Data Environment)의 향후 재구성을 지원한다.

데이터 거버넌스(Data Governance)는 처음부터 아키텍처에 포함되어야 한다. 로봇 기록에는 얼굴(Faces), 음성(Voices), 차량 식별정보(Vehicle Identifiers), 시설 배치(Facility Layouts), 고객 자산(Customer Assets), 운영 정보(Operational Information)처럼 모든 사용자가 접근해서는 안 되는 정보가 포함될 수 있다. 따라서 데이터셋 분류(Dataset Classification), 프로젝트(Project), 역할(Role), 수명주기 단계(Lifecycle Stage)에 따라 접근 제어(Access Control)를 적용할 수 있다. 암호화(Encryption), 감사 로그(Audit Logs), 보존 정책(Retention Policies), 익명화 파이프라인(Anonymization Pipelines), 통제된 내보내기 메커니즘(Controlled Export Mechanisms)은 승인된 팀이 수집 데이터의 가치를 활용하면서 데이터 주권(Data Sovereignty)을 유지하도록 지원한다.

로봇 플릿(Robot Fleets)이 확대되면 지속적으로 생성되는 센서 데이터가 빠르게 페타바이트(Petabyte) 규모에 도달할 수 있으므로 수명주기 관리(Lifecycle Management)가 더욱 중요해진다. 모든 관측 데이터를 영구적인 고성능 저장장치에 유지할 필요는 없다. 자주 사용하는 데이터셋은 핫 스토리지(Hot Storage)에 유지하고, 사용 빈도가 낮아진 데이터는 비용이 낮은 저장 용량 계층으로 이동하며, 과거 아카이브(Historical Archives)는 콜드 스토리지(Cold Storage)로 이전할 수 있다. 보존 여부는 재현성(Reproducibility), 안전 사고 조사(Safety Investigations), 규제 요구사항(Regulatory Requirements), 데이터셋의 희소성(Dataset Uniqueness), 향후 모델 개선(Model Improvement)에 기여할 가능성을 고려하여 결정해야 한다.

품질 관리(Quality Management)는 데이터 레이크 전체에서 지속적으로 수행되어야 한다. 자동화된 파이프라인은 누락 프레임(Missing Frames), 타임스탬프 불연속(Timestamp Discontinuities), 손상된 아카이브(Corrupted Archives), 비정상 센서 값(Abnormal Sensor Values), 불완전한 메타데이터(Incomplete Metadata), 보정 불일치(Calibration Mismatches), 예상과 다른 파일 수(Unexpected File Counts)를 탐지할 수 있다. 체크섬(Checksums)은 로봇, 엣지 시스템(Edge Systems), NAS 장치(NAS Devices), 객체 저장소(Object Stores), 아카이브 사이의 전송 과정에서 무결성(Integrity)을 검증할 수 있다. 품질 결과 자체도 검색 가능한 메타데이터로 관리하여 학습 파이프라인이 비용이 높은 모델 학습을 수행한 후 문제를 발견하는 대신 신뢰할 수 없는 샘플을 자동으로 제외하도록 해야 한다.

보안 아키텍처(Security Architecture)는 로봇 데이터가 여러 신뢰 경계(Trust Boundaries)를 통과한다는 것을 전제로 설계해야 한다. 데이터는 임베디드 장치(Embedded Devices)에서 무선 네트워크(Wireless Networks), 엣지 게이트웨이(Edge Gateways), 온프레미스 인프라(On-Premise Infrastructure), 중앙 집중형 저장소(Centralized Storage)를 거쳐 이동할 수 있다. 인증(Authentication), 장치 신원(Device Identity), 암호화된 전송(Encrypted Transport), 최소 권한 인가(Least-Privilege Authorization), 변경 불가능한 로깅(Immutable Logging), 통제된 서비스 인터페이스(Controlled Service Interfaces)를 적용하면 무단 변경이나 데이터 유출 위험을 줄일 수 있다. 특히 가치가 높은 원시 기록과 검증된 데이터셋에는 한 번 쓰기(Write-Once) 또는 버전 관리 저장 정책(Versioned Storage Policies)을 적용하여 데이터 출처와 계보(Provenance)를 보호할 수 있다.

궁극적으로 로봇 데이터 레이크(Robot Data Lake)는 단순히 대용량 파일을 보관하는 저장소가 아니라 지속적인 피지컬 AI 개선(Continuous Physical AI Improvement)을 위한 데이터 기반(Data Foundation)으로 이해해야 한다. 운영 중인 로봇은 경험(Experiences)을 생성하고, 데이터 레이크는 그 경험을 보존하고 체계화하며, 처리 파이프라인은 이를 재사용 가능한 지식(Reusable Knowledge)으로 변환한다. AI 시스템은 선택된 데이터셋을 향상된 인식(Perception), 추론(Reasoning), 제어(Control) 모델로 변환한다. 이러한 모델은 다시 로봇에 적용되어 새로운 경험을 생성함으로써 로봇, 환경, 애플리케이션 전반으로 확장 가능한 폐쇄형 데이터-지능 수명주기(Closed Data-to-Intelligence Lifecycle)를 형성한다.

## 06.02 Apache Hadoop HDFS: Large-Scale Distributed Storage

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

아파치 하둡(Apache Hadoop)은 범용 서버(Commodity Servers)로 구성된 클러스터(Cluster)에서 매우 큰 데이터셋을 처리하기 위해 설계된 분산 컴퓨팅 및 저장 프레임워크(Distributed Computing and Storage Framework)이다. 하둡의 저장 구성요소인 하둡 분산 파일 시스템(HDFS, Hadoop Distributed File System)은 대용량 파일을 블록(Block)으로 나누어 여러 시스템에 분산 저장한다. 하나의 대용량 저장 서버에 의존하는 대신 HDFS는 여러 노드(Node)의 디스크와 처리 자원을 결합하여 하나의 통합된 저장 환경을 구성한다.

HDFS는 대규모 분산 시스템(Distributed Systems)에서 하드웨어 장애(Hardware Failures)가 정상적으로 발생할 수 있다는 전제를 기반으로 설계되었다. 수백 또는 수천 개의 디스크와 서버가 함께 동작하면 개별 구성요소는 결국 장애를 일으킬 수 있다. HDFS는 복제(Replication), 상태 모니터링(Health Monitoring), 자동 복구 메커니즘(Automatic Recovery Mechanisms)을 통해 이러한 문제를 처리한다. 데이터 블록은 일반적으로 여러 데이터노드(DataNodes)에 저장되므로 디스크, 서버 또는 네트워크 경로에 장애가 발생하더라도 데이터를 계속 제공할 수 있다.

HDFS 아키텍처(HDFS Architecture)는 메타데이터 관리(Metadata Management)와 실제 블록 저장(Physical Block Storage)을 분리한다. 네임노드(NameNode)는 파일 시스템 네임스페이스(Filesystem Namespace), 디렉터리 계층(Directory Hierarchy), 권한(Permissions), 파일 이름(Filenames), 파일과 분산 블록 사이의 매핑(Mappings)을 관리한다. 데이터노드(DataNode)는 실제 블록을 저장하고 클라이언트(Client)의 읽기 및 쓰기 요청에 응답한다. 이러한 분리를 통해 애플리케이션은 논리적 파일 시스템(Logical Filesystem)을 이용해 파일에 접근하고 HDFS는 클러스터 전체에서 데이터의 물리적 분산을 관리할 수 있다.

대용량 파일이 HDFS에 기록되면 독립적으로 분산할 수 있는 고정 크기의 블록(Fixed-Size Blocks)으로 분할된다. 현대적인 구축 환경에서는 HDFS가 수백만 개의 작은 임의 접근(Random Access) 작업보다 대용량 순차 데이터 접근(Large Sequential Data Access)에 최적화되어 있기 때문에 비교적 큰 블록 크기를 사용하는 것이 일반적이다. 예를 들어 수 기가바이트 규모의 로봇 센서 아카이브(Robot Sensor Archive)를 여러 데이터노드에 분산하여 저장 용량과 데이터 처리 워크로드(Data-Processing Workloads)를 클러스터 전체에 분배할 수 있다.

복제(Replication)는 HDFS의 기본적인 장애 허용(Fault Tolerance) 메커니즘을 제공한다. 각 블록은 서로 다른 데이터노드에 여러 개의 복제본(Replicas)을 저장할 수 있으며, 배치 정책(Placement Policies)은 하나의 하드웨어 장애로 모든 복제본이 동시에 손실되지 않도록 설계된다. 노드 장애로 특정 복제본이 사라진 것을 HDFS가 감지하면 다른 정상 복제본을 자동으로 복사하여 요구되는 복제 수준(Replication Level)을 복구할 수 있다. 이러한 메커니즘을 통해 특수한 고가 저장 하드웨어에 의존하지 않고도 분산 저장소의 복원력(Resilience)을 확보할 수 있다.

네임노드(NameNode)는 하트비트 메시지(Heartbeat Messages)와 블록 리포트(Block Reports)를 통해 데이터노드의 상태 정보를 지속적으로 수신한다. 하트비트는 해당 노드가 정상적으로 동작하고 있음을 나타내며, 블록 리포트는 해당 노드에 현재 저장되어 있는 블록을 알려준다. 데이터노드가 설정된 시간 동안 응답하지 않으면 네임노드는 해당 노드를 사용할 수 없는 상태로 판단하고 복구 작업을 조정하기 시작한다. 이러한 메커니즘을 통해 인프라 상태가 변화하더라도 클러스터 상태(Cluster Health)와 데이터 가용성(Data Availability)을 자동으로 유지할 수 있다.

파일 시스템 메타데이터(Filesystem Metadata)는 분산된 데이터의 위치를 파악하는 데 필수적이므로 고가용성(High Availability)은 특히 중요하다. 운영 환경의 하둡은 활성 및 대기 역할(Active and Standby Roles)을 사용하는 여러 네임노드를 구축할 수 있다. 메타데이터 변경 사항을 동기화하여 활성 서비스에 장애가 발생하면 대기 네임노드가 그 역할을 인계받을 수 있다. 추가적인 조정 메커니즘(Coordination Mechanisms)은 여러 노드가 동일한 파일 시스템 네임스페이스를 동시에 제어하려고 하는 스플릿 브레인(Split-Brain) 상태를 방지하는 데 도움을 준다.

HDFS는 초기 하둡 아키텍처의 핵심 원칙이 된 데이터 지역성(Data Locality)을 따른다. 매우 큰 데이터셋을 네트워크를 통해 반복적으로 이동하는 것은 필요한 데이터를 이미 보유하고 있는 시스템으로 연산(Computation)을 이동하는 것보다 비용이 클 수 있다. 따라서 하둡 처리 프레임워크(Hadoop Processing Frameworks)는 가능한 경우 관련 HDFS 블록과 가까운 위치에서 작업을 실행하도록 스케줄링할 수 있다. 이를 통해 네트워크 트래픽(Network Traffic)을 줄이고 여러 서버의 저장 대역폭(Storage Bandwidth)을 대규모 병렬 처리에 활용할 수 있다.

로봇 데이터(Robot Data)의 경우 이러한 아키텍처는 방대한 센서 기록을 병렬로 처리해야 할 때 유용할 수 있다. 카메라 시퀀스(Camera Sequences), 라이다 캡처(LiDAR Captures), 시뮬레이션 결과(Simulation Results), 텔레메트리 아카이브(Telemetry Archives), ROS 기록(ROS Recordings), 과거 로봇 플릿 로그(Historical Fleet Logs)는 전체적으로 수백 테라바이트에서 페타바이트 규모에 이를 수 있다. 분산 전처리 작업(Distributed Preprocessing Jobs)은 모든 원본 데이터를 하나의 저장 서버로 통과시키지 않고도 이러한 데이터에서 프레임을 추출하고, 통계를 계산하고, 이벤트를 식별하고, 형식을 변환하거나 학습 데이터셋을 생성할 수 있다.

HDFS는 대용량 파일(Large Files), 순차 읽기(Sequential Reads), 배치 처리(Batch Processing), 한 번 쓰고 여러 번 읽는 패턴(Write-Once-Read-Many Patterns)이 중심이 되는 워크로드에 특히 효과적이다. 이러한 특성은 많은 아카이브 및 오프라인 분석 워크로드(Offline Analytics Workloads)에 적합하지만 모든 로보틱스 애플리케이션(Robotics Applications)에 적합한 것은 아니다. 실시간 로봇 상태(Real-Time Robot State), 빠르게 변경되는 키-값 정보(Key-Value Information), 트랜잭션 레코드(Transactional Records), 매우 많은 소형 파일은 일반적으로 해당 접근 패턴에 최적화된 데이터베이스(Database), 캐시(Cache), 스트리밍 플랫폼(Streaming Platform), 저장 시스템(Storage System)을 사용하는 것이 적합하다.

소형 파일 문제(Small-File Problem)는 AI 및 로봇 데이터셋을 HDFS에 저장할 때 중요하게 고려해야 한다. 모든 파일과 블록에는 네임노드가 관리해야 하는 메타데이터가 필요하다. 따라서 수백만 개의 개별 이미지 파일을 저장하면 전체 저장 용량 자체는 관리 가능한 수준이라도 상당한 메타데이터 오버헤드(Metadata Overhead)가 발생할 수 있다. 작은 객체를 더 큰 컨테이너 파일(Container Files), 아카이브(Archives), 시퀀스 지향 형식(Sequence-Oriented Formats), 데이터셋 샤드(Dataset Shards)로 패키징하면 네임스페이스 부담(Namespace Pressure)을 줄이고 순차 입출력 효율(Sequential I/O Efficiency)을 향상시킬 수 있다.

HDFS는 전체 데이터 플랫폼(Data Platform) 자체가 아니라 더 광범위한 데이터 레이크 아키텍처(Data Lake Architecture)의 한 계층으로 구성할 수 있다. 원시 센서 아카이브(Raw Sensor Archives)를 분산 저장소에 저장하는 동시에 메타데이터 카탈로그(Metadata Catalogs), SQL 엔진(SQL Engines), NoSQL 데이터베이스(NoSQL Databases), 객체 저장소(Object Stores), 머신러닝 플랫폼(Machine-Learning Platforms)이 보완적인 기능을 제공할 수 있다. 아파치 스파크(Apache Spark)와 같은 처리 프레임워크는 분산 데이터셋에서 직접 동작하여 중앙 집중형 전처리 서버 없이 대규모 변환, 특징 추출(Feature Extraction), 집계(Aggregation), 데이터셋 준비(Dataset Preparation)를 수행할 수 있다.

HDFS로 데이터를 수집하는 과정(Data Ingestion)은 실제 로봇과 엣지 환경(Edge Environments)의 특성을 고려해야 한다. 로봇은 연결이 불안정한 환경에서 동작할 수 있으며 중앙 클러스터로 전송할 수 있는 속도보다 더 빠르게 데이터를 생성할 수도 있다. 따라서 엣지 시스템은 기록 데이터를 로컬에 버퍼링(Buffering)하고, 전송 매니페스트(Transfer Manifests)를 생성하고, 체크섬(Checksums)을 계산한 후 충분한 네트워크 연결이 확보되었을 때 완성된 데이터 세그먼트를 업로드할 수 있다. 중앙 수집 프로세스는 새로운 블록과 관련 메타데이터를 등록하기 전에 데이터 무결성(Data Integrity)을 검증할 수 있다.

보안(Security)은 운영 환경의 HDFS 구축에서 또 다른 필수 요소이다. 인증(Authentication)을 통해 사용자와 서비스의 신원을 확인할 수 있으며, 파일 시스템 권한(Filesystem Permissions)과 인가 정책(Authorization Policies)을 통해 민감한 데이터셋에 대한 접근을 제한할 수 있다. 암호화(Encryption)는 네트워크 전송 과정과 디스크 저장 상태의 정보를 보호할 수 있다. 감사 기록(Audit Records)은 중요한 로봇 기록에 대한 접근을 문서화하여 조직이 데이터 거버넌스(Data Governance)를 시행하고 비정상 활동을 조사하며 공유 인프라 전반에서 책임성(Accountability)을 유지하도록 지원한다.

저장 용량 계획(Storage Capacity Planning)에서는 복제(Replication)를 고려해야 한다. HDFS가 실제로 사용하는 물리적 저장 공간은 논리적 데이터셋 크기(Logical Dataset Size)보다 상당히 클 수 있기 때문이다. 수백 테라바이트 규모의 데이터셋은 설정된 복제 계수(Replication Factor)와 운영 여유 공간(Operational Reserve)에 따라 그보다 몇 배 큰 물리적 용량을 필요로 할 수 있다. 관리자는 현재 원시 데이터의 용량만을 기준으로 클러스터를 구성하는 것이 아니라 재균형화(Rebalancing), 임시 처리 결과(Temporary Processing Outputs), 노드 장애(Node Failures), 데이터셋 증가(Dataset Growth), 복구 작업(Recovery Operations)을 위한 충분한 여유 공간을 확보해야 한다.

새로운 서버가 추가되거나 기존 노드가 교체되거나 데이터 분포가 불균형해지면 클러스터 균형 조정(Cluster Balancing)이 필요하다. HDFS는 데이터노드 사이에서 저장 공간 사용률이 균형을 이루도록 블록을 재분배하는 메커니즘을 제공한다. 균형 잡힌 배치(Balanced Placement)는 특정 디스크나 서버가 용량 병목(Capacity Bottleneck)이 되는 것을 방지하고 병렬 워크로드가 클러스터 전체의 통합 대역폭(Aggregate Bandwidth)을 더욱 효과적으로 활용하도록 한다. 따라서 저장 공간 사용률과 네트워크 동작을 모니터링하는 것은 중요한 운영 관리 책임이다.

HDFS는 현대적인 객체 저장소(Object Storage)와 비교했을 때 장단점(Tradeoffs)을 가진다. 객체 저장소는 일반적으로 보다 단순한 스케일아웃 용량(Scale-Out Capacity), 유연한 메타데이터 통합(Metadata Integration), 클라우드 호환성(Cloud Compatibility), 현대적인 AI 플랫폼과의 편리한 인터페이스를 제공한다. 반면 HDFS는 특히 연산과 저장을 의도적으로 동일한 위치에 배치하는 온프레미스 클러스터(On-Premise Clusters)에서 강력한 데이터 지역성과 성숙한 하둡 기반 분석 환경(Hadoop-Oriented Analytics Environments)과의 통합을 제공할 수 있다. 따라서 아키텍처 선택은 특정 기술이 항상 우수하다고 가정하기보다 실제 워크로드 패턴(Workload Patterns)에 따라 결정해야 한다.

피지컬 AI 데이터 플랫폼(Physical AI Data Platform)에서 HDFS는 특정 대규모 워크로드를 위한 분산 저장 및 처리 기반(Distributed Storage and Processing Foundation)으로 이해하는 것이 적절하다. HDFS는 방대한 과거 로봇 데이터셋(Historical Robot Datasets)을 보존하면서 여러 컴퓨팅 노드가 서로 다른 데이터 영역을 동시에 처리하도록 할 수 있다. 메타데이터 카탈로그(Metadata Catalogs), 데이터셋 매니페스트(Dataset Manifests), AI 학습 인프라(AI Training Infrastructure), 거버넌스 제어(Governance Controls)와 결합하면 시간과 공간에 걸쳐 축적된 로봇 경험을 대규모 분석 및 모델 개발에 활용할 수 있는 자원으로 전환할 수 있다.

로봇 플릿(Robot Fleets)이 성장하면 분산 저장(Distributed Storage)은 단순한 저장 용량 확보 수단 이상의 의미를 갖게 된다. 이는 운영 경험(Operational Experience)을 재사용 가능한 지능(Reusable Intelligence)으로 전환하는 파이프라인의 일부가 된다. 로봇은 관측 데이터(Observations)를 생성하고, 엣지 시스템은 이를 버퍼링하여 전송하며, HDFS는 클러스터 전체에 내구성 있는 복제본(Durable Copies)을 분산한다. 병렬 처리(Parallel Processing)는 원시 기록을 정제 데이터셋(Curated Datasets)으로 변환하고 AI 시스템은 이를 학습에 활용한다. 향상된 모델은 다시 배치된 로봇에 적용되어 확장 가능한 데이터-학습 순환 구조(Scalable Data-to-Learning Cycle)를 완성한다.

## 06.03 Apache Parquet: Columnar Robot Sensor Storage [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

아파치 파케이(Apache Parquet)는 대규모 정형 및 반정형 데이터셋(Structured and Semi-Structured Datasets)의 효율적인 분석 처리를 위해 설계된 개방형 컬럼 기반 저장 형식(Open Columnar Storage Format)이다. 각 레코드의 모든 필드를 함께 저장하는 행 기반 형식(Row-Oriented Format)과 달리 파케이는 값을 주로 컬럼(Column) 단위로 구성한다. 이러한 방식은 수백 개의 속성을 포함하는 데이터셋에서 일부 변수만 사용하는 분석 작업이 많은 로봇 텔레메트리(Robot Telemetry)와 센서 메타데이터(Sensor Metadata)에 특히 유용하다.

로봇 시스템은 타임스탬프(Timestamps), 위치(Positions), 속도(Velocities), 가속도(Accelerations), 관절 각도(Joint Angles), 모터 전류(Motor Currents), 배터리 수준(Battery Levels), 온도(Temperatures), 위치 추정값(Localization Estimates), 객체 감지 결과(Object Detections), 진단 상태(Diagnostic States)와 같은 구조화된 측정값을 지속적으로 생성한다. 이러한 측정값을 개별 텍스트 레코드(Text Records)로 저장하면 상당한 저장 및 파싱 오버헤드(Storage and Parsing Overhead)가 발생할 수 있다. 파케이는 이를 명확한 스키마(Schema)를 유지하는 압축된 바이너리 표현(Compact Binary Representation)으로 변환하여 대규모 로봇 관측 데이터를 효율적으로 저장하고 질의할 수 있도록 한다.

컬럼 기반 저장(Columnar Storage)의 주요 장점은 분석 작업에서 일부 변수만 접근할 때 명확하게 나타난다. 예를 들어 로봇 데이터셋에 100개의 센서 및 상태 필드가 있지만 엔지니어가 타임스탬프, 배터리 전압(Battery Voltage), 모터 온도(Motor Temperature), 속도만 필요로 한다고 가정할 수 있다. 행 기반 표현에서는 각 레코드의 대부분을 읽어야 할 수 있지만 파케이는 주로 필요한 컬럼만 읽을 수 있다. 이를 통해 분석 워크로드에서 디스크 입출력(Disk I/O), 메모리 사용량(Memory Consumption), 데이터 전송량(Data Transfer)을 줄일 수 있다.

파케이 파일(Parquet Files)은 내부적으로 레코드 그룹을 포함하는 로우 그룹(Row Groups)으로 나뉘며, 이는 병렬 처리(Parallel Processing)의 중요한 단위가 된다. 각 로우 그룹 내부에서는 값이 컬럼 청크(Column Chunks)로 분리되고, 컬럼 청크는 다시 페이지(Pages)로 구성된다. 이러한 계층 구조(Hierarchical Structure)를 통해 처리 엔진은 대용량 파일의 필요한 부분만 효율적으로 읽을 수 있다. 따라서 로봇 데이터셋의 저장, 스캔(Scanning), 분산 연산(Distributed Computation)을 최적화할 때 로우 그룹 크기(Row-Group Size)는 중요한 설계 요소가 된다.

파케이에서는 동일한 컬럼의 값이 일반적으로 유사한 데이터 유형과 통계적 특성을 가지므로 압축(Compression)이 매우 효과적으로 동작한다. 타임스탬프 컬럼, 로봇 식별자(Robot Identifier) 컬럼, 동작 모드(Operating Mode) 컬럼, 반복되는 상태 필드(Status Field)는 서로 다른 유형의 값이 함께 저장되는 경우보다 효율적으로 인코딩할 수 있다. 파케이는 저장 용량을 줄이면서 분석 처리 과정에서 효율적인 디코딩(Decoding)을 유지할 수 있도록 압축 알고리즘(Compression Algorithms)과 인코딩 기법(Encoding Techniques)을 지원한다.

인코딩(Encoding)은 일반적인 압축 이외에도 추가적인 효율성을 제공한다. 로봇 모델(Robot Model), 임무 유형(Mission Type), 센서 상태(Sensor State), 오류 범주(Error Category)와 같이 반복되는 범주형 값(Categorical Values)은 사전 인코딩(Dictionary Encoding)의 이점을 얻을 수 있으며, 수치 시퀀스(Numerical Sequences)는 특화된 표현 방식을 활용할 수 있다. 실제 효과는 데이터 분포(Data Distribution)에 따라 달라지지만 컬럼 기반 구성은 로봇 텔레메트리에서 자주 나타나는 반복성과 예측 가능한 값 패턴을 저장 엔진이 효과적으로 활용할 수 있도록 한다.

파케이는 질의 엔진(Query Engines)이 불필요한 데이터 읽기를 피할 수 있도록 메타데이터(Metadata)와 통계 정보(Statistics)도 저장한다. 데이터 영역과 연결된 최솟값과 최댓값(Minimum and Maximum Values)을 이용하면 특정 로우 그룹이 필터 조건을 만족할 수 없는지 판단할 수 있다. 예를 들어 높은 모터 온도 또는 특정 시간 범위의 데이터를 검색하는 질의는 통계값이 요청 범위를 완전히 벗어나는 영역을 읽지 않을 수 있다. 이러한 기법은 일반적으로 프레디케이트 푸시다운(Predicate Pushdown) 또는 데이터 스키핑(Data Skipping)이라고 한다.

파티셔닝(Partitioning)은 파케이 내부의 컬럼 기반 구조를 보완한다. 대규모 로봇 데이터셋을 수집 날짜(Capture Date), 로봇 플릿(Robot Fleet), 로봇 식별자, 임무(Mission), 환경(Environment), 데이터셋 버전(Dataset Version) 등의 속성을 기준으로 디렉터리 구조에 배치할 수 있다. 특정 날짜의 특정 로봇 데이터를 요청하는 질의는 관련 없는 파티션(Partitions)을 전혀 스캔하지 않을 수 있다. 효과적인 파티션 설계는 성능을 크게 향상시키지만 지나치게 세분화된 파티션은 너무 많은 소형 파일과 메타데이터 작업을 발생시킬 수 있다.

따라서 소형 파일 문제(Small-File Problem)는 파케이 기반 로봇 저장소를 설계할 때 중요한 고려사항이다. 모든 센서 메시지나 짧은 기록 구간을 각각 별도의 파케이 파일로 저장하면 매우 많은 작은 파일이 생성되어 컬럼 기반 처리의 장점을 감소시킬 수 있다. 데이터 파이프라인(Data Pipelines)은 일반적으로 레코드를 적절한 크기의 파일과 로우 그룹으로 집계해야 한다. 압축 병합 프로세스(Compaction Processes)를 통해 작은 수집 결과 파일들을 주기적으로 결합하여 후속 분석과 머신러닝(Machine Learning)에 적합한 더 큰 최적화 파일을 생성할 수 있다.

스키마 관리(Schema Management)는 파케이의 또 다른 주요 장점이다. 로봇 플랫폼은 지속적으로 발전하며 새로운 센서, 펌웨어 버전(Firmware Versions), 인식 결과(Perception Outputs), 진단 변수(Diagnostic Variables)가 새로운 필드를 추가할 수 있다. 파케이는 데이터와 함께 스키마 정보를 저장하므로 처리 시스템이 각 데이터셋을 일관되게 해석할 수 있다. 그러나 필드 이름 변경, 호환되지 않는 데이터 유형, 단위 변경, 의미 변경은 파일 자체가 기술적으로 읽을 수 있는 상태라 하더라도 미묘한 오류를 발생시킬 수 있으므로 스키마 진화(Schema Evolution)에 대한 거버넌스(Governance)가 필요하다.

로보틱스(Robotics)에서는 파케이 스키마를 설계할 때 타임스탬프(Timestamps)에 특별한 주의를 기울여야 한다. 하나의 레코드에는 획득 시간(Acquisition Time), 하드웨어 타임스탬프(Hardware Timestamp), 시스템 시간(System Time), 동기화된 전역 시간(Synchronized Global Time), 처리 시간(Processing Time), 수집 시간(Ingestion Time)이 포함될 수 있다. 이러한 값을 의미가 모호한 하나의 일반적인 타임스탬프 필드로 통합해서는 안 된다. 명확한 타임스탬프 의미를 정의하면 센서 시퀀스를 재구성하고 처리 지연시간(Processing Latency)을 측정하며 여러 모달리티(Modality)를 정렬하고 로봇 및 분산 컴퓨팅 인프라에서 발생하는 시간 관련 문제를 분석할 수 있다.

파케이는 구조화된 센서 측정값과 파생 메타데이터(Derived Metadata)에 특히 적합하지만 모든 원시 로봇 데이터 유형을 저장하기 위한 최적의 컨테이너(Container)는 아니다. 대용량 JPEG 이미지, 비디오 스트림(Video Streams), 라이다 바이너리 캡처(LiDAR Binary Captures), 오디오 파일(Audio Files), ROS 백 아카이브(ROS Bag Archives)는 객체 저장소(Object Storage), NAS 또는 분산 파일 저장소(Distributed File Storage)에 유지할 수 있다. 파케이 테이블(Parquet Tables)은 이러한 객체를 참조하는 정보와 함께 타임스탬프, 레이블(Labels), 보정 정보(Calibration Information), 품질 지표(Quality Indicators), 기타 검색 가능한 속성을 저장할 수 있다.

이러한 분리는 로봇 데이터 레이크(Robot Data Lake)에 유용한 구조를 제공한다. 대용량 바이너리 센서 객체(Binary Sensor Objects)는 대용량 파일에 최적화된 저장 시스템에 유지하고 파케이는 해당 데이터의 맥락(Context)을 표현하는 분석용 인덱스(Analytical Index)와 구조화된 표현을 제공할 수 있다. 엔지니어는 수백만 개의 레코드를 질의하여 필요한 장면(Scene)을 식별하고 관련 이미지, 포인트 클라우드(Point Clouds), 기록 데이터만 가져올 수 있다. 이를 통해 실험에 필요한 데이터를 찾기 위해 거대한 바이너리 아카이브 전체를 반복적으로 스캔하는 작업을 방지할 수 있다.

파케이는 아파치 스파크(Apache Spark)와 같은 분산 처리 엔진(Distributed Processing Engines) 및 컬럼 기반 파일을 직접 읽을 수 있는 현대적인 분석 시스템(Analytical Systems)과 자연스럽게 통합된다. 따라서 대규모 텔레메트리 데이터셋을 여러 컴퓨팅 노드에서 필터링(Filtering), 조인(Join), 집계(Aggregation), 변환(Transformation)할 수 있다. 처리 파이프라인은 컬럼 프루닝(Column Pruning)과 분산 실행(Distributed Execution)의 이점을 활용하면서 로봇 플릿 통계(Fleet Statistics)를 계산하고, 이상 상태(Anomalies)를 탐지하며, 특징 테이블(Feature Tables), 학습 매니페스트(Training Manifests), 운영 행동 요약(Operational Behavior Summaries)을 생성할 수 있다.

머신러닝 워크플로(Machine-Learning Workflows)는 파케이를 원시 데이터 수집과 모델 학습(Model Training) 사이의 중간 표현(Intermediate Representation)으로 사용할 수도 있다. 전처리 파이프라인(Preprocessing Pipeline)은 원시 로봇 로그를 동기화된 레코드로 변환하고, 파생 특징(Derived Features)을 계산하고, 레이블을 연결한 후 결과로 생성된 구조화 샘플을 버전 관리된 파케이 데이터셋(Versioned Parquet Datasets)에 기록할 수 있다. 이후 학습 시스템은 모든 실험에서 전체 원시 기록을 반복적으로 디코딩하는 대신 필요한 컬럼과 파티션만 선택할 수 있다.

파케이 파일을 매니페스트(Manifests), 버전 식별자(Version Identifiers), 데이터 계보(Data Lineage)와 결합하면 데이터셋 재현성(Dataset Reproducibility)을 향상시킬 수 있다. 학습 데이터셋에는 어떤 원본 파일이 관측 데이터에 기여했는지, 어떤 변환이 적용되었는지, 어떤 스키마 버전이 사용되었는지, 최종 데이터셋이 어떤 파케이 객체로 구성되는지를 기록해야 한다. 변경 불가능하거나 버전 관리되는 데이터셋 릴리스(Immutable or Versioned Dataset Releases)를 사용하면 새로운 센서 기록과 처리 파이프라인이 추가된 이후에도 과거 실험을 재구성할 수 있다.

품질 관리(Quality Control)는 파케이 생성 파이프라인에 직접 통합할 수 있다. 레코드를 최종 저장하기 전에 처리 작업은 예상 컬럼(Expected Columns), 데이터 유형(Data Types), 단위(Units), 타임스탬프 순서(Timestamp Ordering), 허용 범위(Acceptable Ranges), 결측값(Missing Values), 동기화 품질(Synchronization Quality)을 검증할 수 있다. 품질 지표 자체를 컬럼으로 저장하면 이후 질의에서 손상되거나 신뢰도가 낮은 관측 데이터를 제외할 수 있다. 이를 통해 데이터 품질을 외부의 수동 처리 과정이 아니라 데이터셋 자체에서 검색 가능한 속성으로 전환할 수 있다.

파케이가 파일 형식(File Format)에 불과하더라도 보안(Security)과 거버넌스(Governance)는 여전히 필요하다. 접근 제어(Access Control)는 일반적으로 주변의 저장 플랫폼(Storage Platform), 카탈로그(Catalog), 질의 엔진(Query Engine), 데이터 레이크 인프라(Data-Lake Infrastructure)를 통해 적용된다. 민감한 컬럼(Sensitive Columns)은 제거하거나 변환하거나 익명화(Anonymization)하거나 별도의 통제된 데이터셋으로 분리할 수 있다. 저장 및 전송 암호화(Encryption at Rest and in Transit), 감사 로깅(Audit Logging), 데이터셋 분류(Dataset Classification), 보존 규칙(Retention Rules), 통제된 내보내기 정책(Controlled Export Policies)을 통해 구조화된 로봇 정보를 전체 수명주기에 걸쳐 보호할 수 있다.

피지컬 AI 아키텍처(Physical AI Architecture)에서 파케이는 방대한 원시 로봇 경험(Raw Robot Experience)과 확장 가능한 분석 지능(Scalable Analytical Intelligence)을 연결하는 효율적인 다리 역할을 한다. 로봇은 이기종 관측 데이터(Heterogeneous Observations)를 생성하고, 수집 파이프라인은 원본 증거(Original Evidence)를 보존하며, 처리 시스템은 선택된 정보를 구조화된 파케이 데이터셋으로 변환한다. 분산 엔진은 이러한 데이터셋을 효율적으로 분석하며, 그 결과 생성된 테이블은 데이터셋 탐색(Dataset Discovery), 플릿 분석(Fleet Analytics), 이상 탐지(Anomaly Detection), AI 학습(AI Training), 평가(Evaluation), 배치된 로봇 시스템의 지속적인 개선(Continuous Improvement)을 지원할 수 있다.

## 06.04 Delta Lake: ACID Transaction Data Lake [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

델타 레이크(Delta Lake)는 확장 가능한 파일 또는 객체 저장소(Object Storage)를 기반으로 구축된 데이터 레이크(Data Lake)에 신뢰성 높은 데이터 관리 기능을 제공하기 위해 설계된 개방형 테이블 형식 및 저장 계층(Open Table Format and Storage Layer)이다. 기존 데이터 레이크는 방대한 양의 원시 및 처리 데이터를 저장할 수 있지만 동시 쓰기(Concurrent Writes), 부분 업데이트(Partial Updates), 스키마 변경(Schema Changes), 처리 작업 실패(Failed Processing Jobs)로 인해 데이터셋이 일관성을 잃을 수 있다. 델타 레이크는 데이터 파일 주변에 트랜잭션 관리(Transaction Management)와 테이블 메타데이터(Table Metadata)를 추가하여 이러한 문제를 해결한다.

델타 레이크의 핵심 개념은 일반적으로 델타 로그(Delta Log)라고 하는 트랜잭션 로그(Transaction Log)이다. 파일 디렉터리를 관리되지 않는 데이터셋으로 취급하는 대신 델타 레이크는 테이블 변경 사항을 순서가 지정된 트랜잭션(Ordered Transactions)으로 기록한다. 로그에는 추가되거나 제거된 데이터 파일, 스키마 정보(Schema Information), 테이블 속성(Table Properties), 트랜잭션 메타데이터(Transaction Metadata)가 기록된다. 처리 엔진은 저장소에 현재 존재하는 파일을 단순히 나열하는 대신 이러한 변경 이력을 해석하여 일관된 테이블 상태(Consistent Table State)를 재구성할 수 있다.

ACID 트랜잭션(ACID Transactions)은 신뢰성 높은 업데이트를 위한 기반을 제공한다. 원자성(Atomicity)은 하나의 트랜잭션이 완전히 커밋(Commit)되거나 전혀 적용되지 않도록 하여 부분적으로 기록된 로봇 데이터셋이 정상적인 결과로 나타나는 것을 방지한다. 일관성(Consistency)은 정의된 테이블 규칙을 유지하고, 격리성(Isolation)은 동시 작업이 서로 간섭하지 않도록 보호하며, 지속성(Durability)은 성공적으로 커밋된 변경 사항이 계속 유지되도록 한다. 이러한 특성을 통해 데이터 레이크의 테이블을 관리형 데이터베이스 테이블(Managed Database Tables)과 유사하게 운영할 수 있다.

이러한 신뢰성은 로봇 데이터 파이프라인(Robot Data Pipelines)이 여러 작업을 동시에 수행하는 경우 특히 중요하다. 여러 로봇이 텔레메트리(Telemetry)를 업로드하는 동안 전처리 작업(Preprocessing Jobs)이 센서 레코드를 변환하고, 품질 관리 서비스(Quality-Control Services)가 손상된 샘플을 표시하며, 주석 시스템(Annotation Systems)이 레이블을 추가하고, 학습 파이프라인(Training Pipelines)이 기존 데이터셋을 읽을 수 있다. 트랜잭션 조정(Transactional Coordination)이 없으면 읽기 작업에서 불완전하거나 충돌하는 상태를 확인할 수 있다. 델타 레이크는 새로운 트랜잭션이 독립적으로 커밋되는 동안 이러한 작업들이 일관된 테이블 버전(Table Versions)을 사용하도록 한다.

델타 레이크는 일반적으로 아파치 파케이(Apache Parquet)와 같은 컬럼 기반 파일(Columnar Files)을 이용하여 테이블 데이터를 저장하면서 그 위에 트랜잭션 메타데이터를 추가한다. 파케이는 효율적인 압축(Compression), 컬럼 프루닝(Column Pruning), 분석 스캔(Analytical Scanning)을 제공하고, 델타 레이크는 특정 버전에서 어떤 파케이 파일이 유효한 테이블을 구성하는지를 관리한다. 이러한 분리는 확장 가능한 파일 기반 저장소(File-Based Storage)와 데이터베이스 수준의 일관성(Database-Like Consistency)을 결합하여 데이터 레이크의 경제성을 유지하면서 운영 신뢰성을 향상시킨다.

트랜잭션 로그는 스냅샷 격리(Snapshot Isolation)도 가능하게 한다. 다른 프로세스가 새로운 데이터를 기록하는 동안에도 하나의 질의(Query)는 안정적인 테이블 스냅샷(Table Snapshot)을 대상으로 동작할 수 있다. 예를 들어 AI 학습 파이프라인이 로봇 텔레메트리 데이터셋을 읽기 시작한 상태에서 데이터 수집이 동시에 계속될 수 있다. 학습 작업은 실험 도중 새롭게 추가된 파일을 예기치 않게 포함하는 대신 하나의 일관된 테이블 버전을 계속 사용할 수 있다.

시간 여행(Time Travel)은 테이블 버전 이력(Table Version History)을 통해 제공되는 또 다른 중요한 기능이다. 사용자는 보존된 이력과 실제 데이터 파일이 존재하는 범위 내에서 이전 버전의 델타 테이블을 질의하거나 데이터가 시간에 따라 어떻게 변경되었는지를 확인할 수 있다. 피지컬 AI(Physical AI) 워크플로에서는 과거 학습 데이터셋을 재현하거나, 잘못된 레이블이 언제 파이프라인에 유입되었는지 조사하거나, 처리 방식의 변경을 비교하거나, 이전 실험 당시 존재했던 데이터셋 상태를 이용하여 로봇 행동을 분석하는 데 활용할 수 있다.

스키마 강제 적용(Schema Enforcement)은 잘못 구성된 데이터가 공유 테이블을 조용히 오염시키는 것을 방지하는 데 도움을 준다. 입력되는 레코드는 예상되는 테이블 구조(Table Structure)를 기준으로 검증되므로 호환되지 않는 필드나 잘못된 데이터 유형이 감지되지 않은 상태로 저장될 위험을 줄일 수 있다. 로봇 플릿(Robot Fleets)은 서로 다른 하드웨어 세대와 소프트웨어 버전을 포함하는 경우가 많으므로 명시적인 스키마 검증(Schema Validation)을 통해 예상하지 못한 변경 사항이 분석, 모델 학습(Model Training), 안전 관련 평가(Safety-Related Evaluation) 과정으로 확산되기 전에 탐지할 수 있다.

동시에 제어된 스키마 진화(Controlled Schema Evolution)를 통해 로봇 플랫폼의 변화에 맞추어 테이블 구조를 발전시킬 수 있다. 새로운 센서, 인식 결과(Perception Outputs), 진단 필드(Diagnostic Fields), 임무 속성(Mission Attributes)이 추가되면서 새로운 컬럼이 필요할 수 있다. 스키마가 변경될 때마다 서로 관련 없는 데이터셋을 생성하는 대신 호환 가능한 변경 사항을 정의된 작업을 통해 관리할 수 있다. 그러나 단순히 필드를 추가하는 것과 해당 필드의 단위, 의미, 좌표 프레임(Coordinate Frame), 타임스탬프 의미(Timestamp Semantics), 해석 방법을 변경하는 것은 서로 다르므로 데이터 거버넌스(Data Governance)가 필요하다.

델타 레이크는 일반적인 변경 불가능한 파일 집합(Immutable File Collections)만으로는 안전하게 관리하기 어려운 업데이트 중심 작업(Update-Oriented Operations)도 지원한다. 애플리케이션 요구사항에 따라 레코드를 삽입(Insert), 업데이트(Update), 삭제(Delete), 병합(Merge)할 수 있다. 로보틱스 파이프라인은 병합 작업을 이용하여 수정된 주석(Corrected Annotations)을 반영하거나, 품질 플래그(Quality Flags)를 업데이트하거나, 늦게 도착한 텔레메트리(Late-Arriving Telemetry)를 통합하거나, 테이블의 트랜잭션 무결성(Transactional Integrity)을 유지하면서 현재 로봇 상태를 관리할 수 있다.

모바일 로보틱스(Mobile Robotics)에서는 네트워크 연결이 불안정할 수 있기 때문에 지연 도착 데이터(Late-Arriving Data)가 자주 발생한다. 로봇은 오프라인 상태로 동작하면서 텔레메트리를 로컬에 버퍼링하고 몇 시간 후 과거 데이터를 업로드할 수 있다. 트랜잭션 기반 수집(Transactional Ingestion)을 이용하면 전체 데이터셋을 다시 구축하지 않고도 지연된 레코드를 통합할 수 있다. 처리 로직은 적절한 파티션(Partitions)이나 키(Keys)를 식별하고 도착한 정보를 병합하여 새로운 일관된 테이블 버전을 생성하는 동시에 기존 읽기 작업은 이전 스냅샷을 계속 사용할 수 있다.

트랜잭션 기능이 제공되더라도 파티셔닝(Partitioning)은 성능 측면에서 여전히 중요하다. 로봇 데이터셋은 날짜(Date), 플릿(Fleet), 로봇 식별자(Robot Identifier), 임무(Mission), 사이트(Site), 환경(Environment) 또는 자주 필터링되는 다른 속성을 기준으로 구성할 수 있다. 질의 엔진은 필터 조건이 파티션 구조와 일치하면 관련 없는 파티션의 스캔을 피할 수 있다. 그러나 지나치게 세분화된 파티셔닝은 소형 파일(Small Files)과 메타데이터 오버헤드(Metadata Overhead)를 발생시킬 수 있으므로 실제 접근 패턴(Access Patterns)을 기준으로 파티션 전략을 설계해야 한다.

빈번한 스트리밍 또는 증분 쓰기(Incremental Writes)는 많은 수의 작은 파케이 파일을 생성할 수도 있다. 따라서 델타 레이크 환경에서는 분석 워크로드가 수많은 작은 객체를 여는 데 지나치게 많은 시간을 사용하지 않도록 파일 압축 병합(File Compaction)과 데이터 배치 최적화(Layout Optimization)가 유용하다. 주기적인 최적화 작업을 통해 테이블의 논리적 내용(Logical Contents)을 유지하면서 작은 출력 파일을 더 큰 파일로 통합할 수 있다. 이는 고주파 로봇 텔레메트리(High-Frequency Robot Telemetry)를 지속적으로 수집한 후 대규모 배치 분석을 수행하는 환경에서 특히 유용하다.

메달리온 아키텍처(Medallion Architecture)는 델타 레이크 데이터 플랫폼과 자주 함께 사용되는 구조이다. 브론즈 계층(Bronze Layer)은 최소한으로 처리된 로봇 레코드를 보존하고, 실버 계층(Silver Layer)은 검증, 동기화, 표준화된 정보를 포함하며, 골드 계층(Gold Layer)은 분석이나 AI를 위한 애플리케이션 지향 데이터셋(Application-Oriented Datasets)을 제공할 수 있다. 이러한 명칭은 델타 레이크의 필수 구성요소가 아니라 일반적으로 사용되는 설계 관례이지만, 계층형 모델은 원시 증거(Raw Evidence)와 점진적으로 정제된 데이터 제품(Data Products)을 분리하는 유용한 방법을 제공한다.

로봇 센서 시스템(Robot Sensor Systems)의 경우 브론즈 계층은 수집된 텔레메트리와 원시 이미지, 라이다(LiDAR), 비디오, 오디오, ROS 기록에 대한 참조를 보존할 수 있다. 실버 계층에서는 타임스탬프, 좌표계(Coordinate Systems), 로봇 식별자, 센서 상태, 품질 지표(Quality Indicators)를 정규화할 수 있다. 이후 골드 테이블(Gold Tables)은 플릿 통계(Fleet Statistics), 이상 탐지 특징(Anomaly Features), 학습 샘플(Training Samples), 임무 요약(Mission Summaries), 평가 지표(Evaluation Metrics)를 제공할 수 있다. 이러한 단계 사이의 트랜잭션 기반 데이터 계보(Transactional Lineage)는 재현성과 운영 신뢰도를 향상시킨다.

델타 레이크는 구조화된 테이블이 훨씬 큰 바이너리 센서 객체(Binary Sensor Objects)를 참조해야 할 때 특히 유용하다. 원시 비디오, 이미지 컬렉션(Image Collections), 포인트 클라우드(Point Clouds), 기록 아카이브(Recording Archives)는 객체 저장소나 다른 대용량 파일 저장소에 유지하고, 델타 테이블은 검색 가능한 메타데이터, 레이블, 타임스탬프, 품질 정보, 객체 위치(Object Locations)를 관리할 수 있다. 질의를 통해 관련 장면을 트랜잭션 기반으로 식별한 다음 분석이나 모델 학습에 필요한 대용량 센서 객체만 선택적으로 가져올 수 있다.

스트리밍 처리(Streaming Processing)와 배치 처리(Batch Processing)는 동일한 논리적 데이터 기반(Logical Data Foundation) 위에서 동작할 수 있다. 지속적인 텔레메트리 수집은 새로운 레코드를 추가하거나 병합하고, 예약된 분석 작업은 더 큰 과거 데이터 구간을 처리할 수 있다. 이를 통해 실시간 워크로드와 오프라인 워크로드를 위해 완전히 분리된 저장 시스템을 유지할 필요성을 줄일 수 있다. 통합된 테이블 이력(Unified Table History)은 특정 시점에 각 후속 처리 과정에서 어떤 데이터를 사용할 수 있었는지를 파악하기도 쉽게 한다.

머신러닝 재현성(Machine-Learning Reproducibility)은 버전 관리되는 델타 테이블(Versioned Delta Tables)을 통해 직접적인 이점을 얻을 수 있다. 학습 데이터셋을 변경 가능한 디렉터리 경로만으로 정의하는 대신 실험 과정에서 특정 테이블 버전을 모델 코드(Model Code), 파라미터(Parameters), 환경 정보(Environment Information)와 함께 기록할 수 있다. 과거 파일이 계속 보존되어 있다면 이후 동일한 논리적 데이터셋을 재구성할 수 있다. 이를 통해 모델 세대(Model Generations) 사이의 비교 신뢰성을 높이고 알고리즘 변경과 학습 데이터 변경의 영향을 구분하는 데 도움을 줄 수 있다.

거버넌스(Governance)와 보안(Security)은 델타 레이크를 둘러싼 전체 플랫폼을 통해 구현된다. 접근 제어(Access Control), 암호화(Encryption), 감사 로깅(Audit Logging), 카탈로그 권한(Catalog Permissions), 보존 규칙(Retention Rules), 민감 데이터 정책(Sensitive-Data Policies)을 통해 테이블을 읽거나 수정할 수 있는 사용자를 결정한다. 트랜잭션 이력(Transaction History)은 유용한 운영 증거를 제공하지만 오래된 파일을 지나치게 빠르게 삭제하면 과거 테이블 버전에 접근할 수 있는 능력이 제한될 수 있으므로 보존 및 정리 정책(Retention and Cleanup Policies)을 신중하게 설계해야 한다.

피지컬 AI 데이터 아키텍처(Physical AI Data Architecture)에서 델타 레이크는 지속적으로 변화하는 로봇 경험(Robot Experience)을 신뢰할 수 있는 분석 및 학습 데이터셋과 연결할 수 있다. 로봇과 엣지 시스템(Edge Systems)은 관측 데이터를 생성하고, 수집 서비스(Ingestion Services)는 이를 트랜잭션 방식으로 커밋하며, 정제 파이프라인(Refinement Pipelines)은 검증된 테이블 버전을 생성한다. AI 시스템은 안정적인 스냅샷을 분석과 학습에 사용하고 새로운 모델은 다시 배치된 로봇으로 전달된다. 이후 로봇은 새로운 경험을 생성함으로써 거버넌스가 적용되고 재현 가능한 데이터-지능 순환 구조(Governed, Reproducible Data-to-Intelligence Cycle)를 형성한다.

## 06.05 Apache Hudi: Incremental Processing Data Lake [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

아파치 후디(Apache Hudi)는 시간에 따라 지속적으로 변화하는 대규모 데이터셋을 관리하도록 설계된 오픈소스 트랜잭션 데이터 레이크 플랫폼(Open-Source Transactional Data-Lake Platform)이다. 기존 파일 기반 데이터 레이크(File-Based Data Lake)는 변경되지 않는 과거 데이터를 저장하는 데 효율적이지만 업데이트(Update), 삭제(Delete), 지연 도착 레코드(Late-Arriving Records), 지속적인 데이터 수집(Continuous Ingestion)을 관리하는 것은 복잡해질 수 있다. 후디는 확장 가능한 저장 시스템 위에 테이블 관리(Table Management), 인덱싱(Indexing), 트랜잭션 쓰기(Transactional Writes), 증분 처리(Incremental Processing) 기능을 추가한다.

후디(Hudi)라는 이름은 원래 하둡 업서트 삭제 및 증분 처리(Hadoop Upserts Deletes and Incrementals)의 개념을 반영하며, 기존 레코드가 단순히 누적되는 것이 아니라 지속적으로 변화해야 하는 워크로드를 강조한다. 로봇 플릿(Robot Fleet)은 대표적인 사례로, 텔레메트리(Telemetry), 임무 상태(Mission States), 주석(Annotations), 품질 플래그(Quality Flags), 운영 메타데이터(Operational Metadata)가 지속적으로 도착하고 때로는 수정되어야 한다. 후디는 이러한 변화하는 레코드를 관리되지 않는 파일 집합이 아니라 논리적 테이블(Logical Tables)로 관리할 수 있도록 한다.

후디의 핵심 기능 중 하나는 레코드 수준 업서트 처리(Record-Level Upsert Processing)이다. 업서트(Upsert)는 입력되는 레코드를 새로운 데이터로 삽입(Insert)할 것인지 기존 레코드를 업데이트(Update)하는 데 사용할 것인지를 결정한다. 이는 로봇이 오프라인 동작 후 다시 연결되거나, 수정된 위치 추정 결과(Localization Results)가 나중에 도착하거나, 주석이 변경되거나, 중복 텔레메트리가 발견되는 경우 유용하다. 전체 과거 데이터셋을 다시 작성하지 않고도 이러한 변경 사항을 저장 계층에 통합할 수 있다.

후디는 테이블의 변경 사항을 작업 타임라인(Timeline of Actions)에 따라 구성한다. 커밋(Commits), 델타 커밋(Delta Commits), 컴팩션(Compactions), 클러스터링 작업(Clustering Operations), 기타 테이블 관리 활동을 기록하여 처리 시스템이 데이터셋이 어떻게 변화했는지 파악할 수 있도록 한다. 이러한 타임라인은 일관된 테이블 상태(Consistent Table State)를 제공하고 증분 소비(Incremental Consumption)를 지원한다. 전체 과거 파일을 반복적으로 스캔하는 대신 후속 애플리케이션은 이전에 처리한 시점 이후 어떤 변경 사항이 발생했는지 확인할 수 있다.

증분 처리(Incremental Processing)는 후디의 가장 중요한 특징 중 하나이다. 데이터 소비자(Consumer)는 전체 테이블을 읽는 대신 테이블 타임라인에서 선택한 두 시점 사이에 변경된 레코드만 요청할 수 있다. 대규모 로봇 플릿에서는 이를 통해 불필요한 연산을 크게 줄일 수 있다. 예를 들어 일일 분석 작업(Daily Analytics Job)은 수년간 축적된 전체 운영 이력을 다시 스캔하는 대신 이전의 성공적인 실행 이후 추가되거나 수정된 텔레메트리만 처리할 수 있다.

후디는 쓰기 성능(Write Performance)과 읽기 성능(Read Performance) 사이의 서로 다른 절충 관계를 제공하는 두 가지 주요 테이블 유형(Table Types)을 지원한다. 쓰기 시 복사(Copy-on-Write) 테이블은 변경 사항이 커밋될 때 업데이트된 베이스 파일(Base Files)을 생성하여 추가적인 쓰기 작업을 수행하는 대신 읽기에 최적화된 데이터셋을 제공한다. 읽기 시 병합(Merge-on-Read) 테이블은 변경 사항을 먼저 로그 파일(Log Files)에 기록한 후 베이스 데이터와 병합할 수 있으므로 수집 지연시간(Ingestion Latency)을 줄이는 대신 일부 작업을 컴팩션 또는 질의 시점의 병합으로 이동시킨다.

쓰기 시 복사(Copy-on-Write)는 주기적으로 업데이트되지만 분석, 보고(Reporting), 학습 데이터 준비(Training Preparation)를 위해 자주 읽히는 로봇 데이터셋에 적합할 수 있다. 업데이트가 새로운 베이스 파일에 반영되므로 읽기 작업은 최신 테이블 상태를 비교적 단순한 형태로 접근할 수 있다. 반면 일부 레코드만 수정하더라도 개별 바이트를 직접 변경하는 대신 더 큰 파일 그룹(File Groups)을 다시 작성해야 할 수 있으므로 쓰기 증폭(Write Amplification)이 발생한다.

읽기 시 병합(Merge-on-Read)은 고빈도 업데이트(High-Frequency Updates) 또는 낮은 지연시간의 데이터 수집(Low-Latency Ingestion)이 더 중요한 경우 유용하다. 로봇 텔레메트리, 변화하는 임무 상태, 이상 이벤트(Anomaly Events), 지속적으로 업데이트되는 인식 메타데이터(Perception Metadata)를 증분 변경 사항으로 빠르게 기록할 수 있다. 이후 컴팩션(Compaction)은 누적된 로그 레코드를 베이스 파일과 결합한다. 이를 통해 즉각적인 데이터 수집 요구사항과 효율적인 분석 저장 구조를 유지하는 데 필요한 무거운 처리 작업을 분리할 수 있다.

후디 인덱싱(Hudi Indexing)은 기존 레코드가 어디에 위치하는지를 식별하여 업데이트와 삭제를 효율적으로 적용할 수 있도록 한다. 레코드는 일반적으로 논리적 엔터티(Logical Entities)나 이벤트를 고유하게 식별하는 키(Keys)와 연결된다. 로보틱스에서는 로봇 식별자(Robot Identifier), 임무 식별자(Mission Identifier), 센서 스트림(Sensor Stream), 타임스탬프(Timestamp), 시퀀스 번호(Sequence Number) 또는 기타 안정적인 식별자를 조합하여 키를 구성할 수 있다. 잘못된 고유성 가정은 중복 레코드를 생성하거나 서로 관련 없는 관측 데이터를 덮어쓸 수 있으므로 신중한 키 설계가 중요하다.

동일한 논리적 레코드의 여러 버전이 도착하는 경우 사전 결합(Precombining)도 중요하다. 후디는 순서 지정 필드(Ordering Field)를 사용하여 데이터 수집 과정에서 어떤 버전이 우선되어야 하는지 결정할 수 있다. 예를 들어 처리 버전(Processing Version), 이벤트 시퀀스(Event Sequence), 업데이트 타임스탬프(Update Timestamp)를 이용하여 서로 경쟁하는 레코드 사이의 우선순위를 결정할 수 있다. 특히 네트워크 연결이 불안정한 환경에서는 데이터 도착 시간이 항상 가장 최신의 물리적 관측을 의미하지 않으므로 로보틱스 파이프라인은 이러한 순서를 의미론적으로 정의해야 한다.

지연 도착 데이터(Late-Arriving Data)는 자율 시스템(Autonomous Systems)과 모바일 시스템에서 일반적으로 발생한다. 로봇은 창고, 실외 환경, 지하시설, 원격 지역처럼 네트워크 접근이 일시적으로 불가능한 환경에서 동작할 수 있다. 데이터는 엣지 컴퓨터(Edge Computers)에 버퍼링된 후 나중에 업로드될 수 있다. 후디의 업데이트 및 증분 메커니즘을 이용하면 지연된 정보를 기존 테이블에 통합하면서 후속 처리 시스템이 사용할 수 있도록 관리 가능한 변경 이력(Change History)을 유지할 수 있다.

삭제(Delete)는 관리형 데이터 레이크 테이블(Managed Data-Lake Tables)을 단순한 추가 전용 저장소(Append-Only Storage)와 구분하는 또 다른 기능이다. 레코드가 중복되거나 유효하지 않거나 손상되었거나 보존 정책(Retention Policies)의 범위를 벗어나거나 개인정보 보호 요구사항(Privacy Requirements)의 적용을 받는 경우 삭제해야 할 수 있다. 후디는 관리자가 임의의 파일을 직접 찾아 다시 작성하지 않고도 삭제 작업을 트랜잭션 방식으로 처리할 수 있다. 다만 주변 거버넌스 시스템(Governance System)은 데이터가 삭제된 이유와 어떤 정책이 해당 작업을 승인했는지를 기록해야 한다.

파티셔닝(Partitioning)은 질의와 업데이트 과정에서 후디가 확인해야 하는 저장 영역을 제한하는 데 도움을 준다. 로봇 테이블은 날짜(Date), 사이트(Site), 플릿(Fleet), 임무(Mission), 환경(Environment) 또는 자주 접근하는 다른 차원을 기준으로 파티셔닝할 수 있다. 효과적인 파티셔닝은 프루닝(Pruning)과 데이터 관리 성능을 향상시키지만 지나치게 세분화된 파티션은 소형 파일 문제(Small-File Problem)를 발생시킬 수 있다. 따라서 파티션 설계는 질의 지역성(Query Locality), 업데이트 특성, 파일 크기, 입력되는 로봇 데이터의 분포를 균형 있게 고려해야 한다.

소형 파일 관리(Small-File Management)는 지속적으로 데이터가 수집되는 데이터셋에서 특히 중요하다. 수천 대의 로봇이 빈번한 업데이트를 생성하면 저장 및 질의 효율을 떨어뜨리는 많은 수의 작은 파일이 만들어질 수 있다. 후디는 파일 크기를 관리하고 데이터를 재구성하는 메커니즘을 제공한다. 클러스터링(Clustering)을 통해 애플리케이션에 제공되는 논리적 레코드를 변경하지 않으면서 테이블의 물리적 배치(Table Layout)를 다시 작성하여 파일 구성과 질의 성능을 향상시킬 수 있다.

컴팩션(Compaction)은 증분 로그 파일이 시간에 따라 누적되는 읽기 시 병합(Merge-on-Read) 테이블에서 특히 중요하다. 컴팩션 작업은 이러한 변경 사항을 새로운 베이스 파일로 병합하여 이후 읽기 작업에 필요한 처리량을 줄인다. 컴팩션을 자주 수행하면 컴퓨팅 및 저장 대역폭(Compute and Storage Bandwidth)을 소비하고, 지나치게 지연하면 읽기 증폭(Read Amplification)이 증가할 수 있다. 따라서 실제 워크로드 특성에 따라 적절한 컴팩션 전략을 결정해야 한다.

후디는 테이블 유형과 처리 요구사항에 따라 서로 다른 질의 관점(Query Perspectives)을 제공할 수 있다. 스냅샷 질의(Snapshot Queries)는 테이블의 현재 상태를 제공하고, 증분 질의(Incremental Queries)는 선택한 시간 구간에서 발생한 변경 사항을 제공한다. 읽기 시 병합 구성에서는 최근 기록된 데이터에 최적화된 뷰(View)와 컴팩션된 저장 구조를 기반으로 하는 뷰를 구분할 수도 있다. 이러한 기능을 통해 하나의 테이블 기반에서 현재 상태 분석(Current-State Analysis)과 변경 중심 처리(Change-Oriented Processing)를 모두 지원할 수 있다.

로봇 센서 아키텍처(Robot Sensor Architecture)에서 후디가 모든 원시 바이너리 객체(Raw Binary Objects)를 직접 포함할 필요는 없다. 대용량 이미지, 비디오, 포인트 클라우드(Point Clouds), 오디오 스트림(Audio Streams), ROS 기록은 객체 저장소(Object Storage)나 분산 파일 시스템(Distributed File Systems)에 유지할 수 있다. 후디 테이블은 타임스탬프, 로봇 식별자, 레이블(Labels), 품질 상태(Quality States), 처리 결과(Processing Results), 대용량 객체에 대한 참조(References)를 포함하는 검색 가능한 레코드를 관리할 수 있다. 이를 통해 효율적인 메타데이터 처리와 대용량 바이너리 저장을 분리할 수 있다.

이러한 구조는 피지컬 AI 데이터셋 생성(Physical AI Dataset Generation)에 강력하게 활용될 수 있다. 증분 질의를 이용하면 이전 학습 데이터 생성 이후 새롭게 수집되거나 수정된 장면(Scene)만 식별할 수 있다. 처리 파이프라인은 참조된 센서 객체를 가져와 동기화(Synchronization)와 품질 검사를 수행하고, 레이블 또는 특징(Features)을 생성한 후 결과 샘플을 새로운 데이터셋 버전에 추가할 수 있다. 업데이트가 발생할 때마다 비용이 높은 전체 과거 데이터를 다시 스캔할 필요가 없다.

후디는 대규모 SQL, 스트리밍(Streaming), 배치 워크로드(Batch Workloads)에 일반적으로 사용되는 엔진을 포함한 분산 데이터 처리 생태계(Distributed Data-Processing Ecosystem)와 통합할 수 있다. 로봇 플릿 분석(Robot Fleet Analytics)은 방대한 과거 데이터에서 운영 지표(Operational Metrics)를 집계하고, 이상 상태를 탐지하고, 임무 요약을 생성하거나, AI 학습 테이블을 준비할 수 있다. 증분 소비는 반복적인 처리를 줄이고 새로운 운영 데이터가 제공될 때 후속 파이프라인이 효율적으로 반응하도록 한다.

증분 테이블을 사용하더라도 재현성(Reproducibility)을 확보하려면 명확한 데이터셋 버전 관리(Dataset Versioning)와 데이터 계보(Data Lineage)가 필요하다. AI 실험에서는 관련 테이블 상태(Table State), 처리 구간(Processing Interval), 원본 객체(Source Objects), 스키마 버전(Schema Version), 변환 과정(Transformations), 출력 데이터셋 식별자(Output Dataset Identifiers)를 기록해야 한다. 증분 처리는 효율성을 높이지만 학습 데이터를 정의되지 않은 이동 목표(Moving Target)로 만들어서는 안 된다. 실험을 반복하거나 감사해야 하는 경우 안정적인 릴리스(Stable Releases) 또는 매니페스트(Manifests)가 여전히 필요하다.

보안(Security)과 거버넌스(Governance)는 후디를 둘러싼 전체 데이터 플랫폼을 통해 제공된다. 인증(Authentication), 인가(Authorization), 암호화(Encryption), 감사 로깅(Audit Logging), 보존 정책(Retention Policies), 카탈로그 통합(Catalog Integration), 통제된 데이터 내보내기(Controlled Data Export)를 통해 로봇 정보에 접근하고 수정할 수 있는 방식을 관리한다. 후디는 업데이트와 삭제를 지원하므로 운영 권한(Operational Permissions)은 데이터셋 읽기, 새로운 레코드 수집, 기존 정보 수정, 관리 목적의 테이블 작업을 서로 구분해야 한다.

피지컬 AI 데이터 레이크(Physical AI Data Lake)에서 아파치 후디는 로봇 정보가 지속적으로 변화하고 후속 시스템이 변경된 데이터만 처리해야 하는 환경에서 특히 유용하다. 로봇은 새로운 경험(New Experiences)을 생성하고, 엣지 시스템은 레코드를 버퍼링하고 업로드하며, 후디는 삽입, 업데이트, 삭제, 테이블 이력(Table History)을 관리한다. 증분 데이터 소비자는 최근 변경 사항을 분석 또는 학습 데이터로 변환하고, 개선된 AI 모델은 다시 로봇으로 전달된다. 이를 통해 효율적이고 지속적인 데이터-학습 순환 구조(Continuous Data-to-Learning Cycle)를 형성할 수 있다.

## 06.06 Apache Iceberg: Table Format Standard [w/Code]

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

아파치 아이스버그(Apache Iceberg)는 데이터 레이크(Data Lake)에 저장된 매우 큰 분석 데이터셋(Analytical Datasets)을 관리하도록 설계된 개방형 테이블 형식(Open Table Format)이다. 테이블을 느슨하게 관리되는 파일이 들어 있는 단순한 디렉터리로 취급하는 대신, 아이스버그는 각 테이블 상태(Table State)에 어떤 데이터 파일이 포함되는지를 명시적으로 정의하는 구조화된 메타데이터(Structured Metadata)를 도입한다. 이러한 방식은 객체 저장소(Object Store)와 분산 파일 시스템(Distributed File System)의 확장성을 유지하면서 데이터베이스와 유사한 테이블 관리(Database-Like Table Management)를 제공한다.

아이스버그는 논리적 테이블(Logical Table)을 데이터 파일의 물리적 배치(Physical Arrangement)와 분리한다. 애플리케이션은 디렉터리 구조에 직접 의존하는 대신 테이블, 스키마(Schema), 파티션(Partition), 스냅샷(Snapshot)을 대상으로 동작한다. 데이터 파일은 일반적으로 아파치 파케이(Apache Parquet)와 같은 컬럼 기반 형식(Columnar Format)으로 저장되며, 아이스버그는 파일의 구성을 설명하는 메타데이터를 관리한다. 이러한 추상화(Abstracted Layer)를 통해 애플리케이션이 내부 파일 관리의 모든 세부사항을 이해하지 않아도 저장 구조를 발전시킬 수 있다.

아이스버그의 핵심 개념 중 하나는 스냅샷(Snapshot)이다. 커밋된 각각의 테이블 변경 사항은 테이블의 일관된 버전을 나타내는 새로운 스냅샷을 생성한다. 스냅샷은 최종적으로 해당 버전에 포함되는 데이터 파일을 식별하는 메타데이터를 참조한다. 따라서 다른 쓰기 작업이 데이터를 추가하거나 수정하는 동안에도 읽기 작업은 안정적인 테이블 상태를 사용할 수 있다. 이러한 격리(Isolation)는 재현 가능한 입력 데이터가 필요한 장시간 분석 및 AI 파이프라인에서 중요하다.

아이스버그는 매우 큰 테이블을 효율적으로 추적하기 위해 여러 계층의 메타데이터(Metadata Layers)를 사용한다. 테이블 메타데이터(Table Metadata)는 현재 스키마, 파티션 명세(Partition Specification), 속성(Properties), 스냅샷 이력(Snapshot History) 등의 정보를 기록한다. 매니페스트 리스트(Manifest Lists)는 여러 매니페스트 그룹을 식별하고, 매니페스트 파일(Manifest Files)은 데이터 파일 집합과 관련 통계 정보를 설명한다. 이러한 계층적 메타데이터 구조는 어떤 파일이 테이블에 속하는지 확인하기 위해 질의 엔진이 거대한 저장 디렉터리를 반복적으로 나열하는 작업을 방지한다.

비용이 높은 파일 시스템 목록 조회(Filesystem Listing)를 피하는 것은 객체 저장소에서 특히 중요하다. 로보틱스 데이터 레이크(Robotics Data Lake)는 수년간의 플릿 운영(Fleet Operation)을 통해 생성된 수백만 개의 파일을 포함할 수 있다. 각 질의가 어떤 데이터를 읽어야 하는지 결정하기 전에 저장 객체 전체를 열거해야 한다면 질의 계획 오버헤드(Query Planning Overhead)가 크게 증가할 수 있다. 아이스버그의 메타데이터 구조는 매번 물리적 데이터셋을 다시 탐색하는 대신 관리되는 테이블 메타데이터를 통해 필요한 파일을 식별할 수 있도록 한다.

아이스버그는 원자적 테이블 변경(Atomic Table Changes)을 지원하여 읽기 작업에서 부분적으로 커밋된 업데이트를 확인하지 않도록 한다. 쓰기 작업은 새로운 테이블 상태를 공개하기 전에 새로운 데이터와 메타데이터를 준비한다. 커밋이 성공하면 읽기 작업에서 새로운 스냅샷을 사용할 수 있으며, 커밋이 실패하면 이전 스냅샷이 계속 유효하게 유지된다. 이러한 특성은 여러 처리 파이프라인에서 로봇 텔레메트리(Robot Telemetry), 주석(Annotations), 품질 결과(Quality Results), 파생 데이터셋(Derived Datasets)을 동시에 업데이트하는 환경에서 유용하다.

시간 여행(Time Travel)은 아이스버그의 스냅샷 아키텍처(Snapshot Architecture)에서 자연스럽게 제공되는 기능이다. 필요한 메타데이터와 데이터 파일이 보존되어 있다면 애플리케이션은 버전 또는 시간을 기준으로 이전 스냅샷에 접근할 수 있다. 피지컬 AI(Physical AI) 개발에서는 이전 실험에서 사용한 테이블 상태를 재구성하고, 데이터셋 세대(Dataset Generations)를 비교하며, 잘못된 데이터가 언제 파이프라인에 유입되었는지 조사하거나, 과거 시점의 일관된 데이터셋을 이용하여 모델을 평가하는 데 활용할 수 있다.

스키마 진화(Schema Evolution)는 컬럼 위치나 이름에만 취약하게 의존하지 않도록 설계되어 있다. 아이스버그는 필드에 지속적인 식별자(Persistent Identifiers)를 할당하여 스키마가 변경되더라도 컬럼을 추적할 수 있도록 한다. 테이블은 많은 경우 전체 과거 데이터를 다시 작성하지 않고도 필드를 추가(Add), 제거(Remove), 이름 변경(Rename), 순서 변경(Reorder)할 수 있다. 이는 새로운 센서와 소프트웨어 기능이 추가되면서 텔레메트리 스키마가 지속적으로 확장되는 로봇 플랫폼에서 유용하다.

그러나 스키마 진화에는 여전히 의미론적 거버넌스(Semantic Governance)가 필요하다. 기술적으로 유효한 이름 변경이나 데이터 유형 변경이 로봇 데이터의 의미까지 자동으로 보존하는 것은 아니다. 엔지니어는 단위(Units), 좌표 프레임(Coordinate Frames), 보정 정의(Calibration Definitions), 타임스탬프 의미(Timestamp Semantics), 센서 식별정보(Sensor Identities), 해석 규칙(Interpretation Rules)을 별도로 관리해야 한다. 아이스버그는 테이블 구조를 관리하는 메커니즘을 제공하지만 로보틱스 데이터 아키텍처는 하드웨어 및 소프트웨어 세대에 걸쳐 각 필드가 무엇을 의미하는지를 정의해야 한다.

파티션 진화(Partition Evolution)는 아이스버그의 또 다른 주요 기능이다. 기존의 디렉터리 기반 파티셔닝(Directory-Based Partitioning)은 파티션 구조를 파일 경로에 직접 포함하는 경우가 많아 이후 변경하기 어렵다. 아이스버그는 파티션 명세를 테이블 메타데이터로 관리하여 시간에 따라 파티션 전략을 변경할 수 있도록 한다. 과거 데이터는 기존 명세를 유지하면서 새롭게 기록되는 데이터에는 변경된 전략을 적용할 수 있으며, 질의 엔진은 동일한 논리적 테이블에서 두 방식을 모두 해석할 수 있다.

숨겨진 파티셔닝(Hidden Partitioning)은 애플리케이션이 물리적 저장 구조에 의존하는 정도를 더욱 줄여준다. 사용자는 타임스탬프와 같은 논리적 컬럼(Logical Columns)을 대상으로 질의할 수 있으며 아이스버그는 내부적으로 적절한 파티션 변환(Partition Transformations)을 적용한다. 따라서 애플리케이션이 수동으로 생성된 파티션 디렉터리 값을 이용해 필터를 구성할 필요가 없다. 이러한 분리를 통해 모든 데이터 소비 애플리케이션을 다시 작성하지 않고도 저장 최적화 전략을 변경할 수 있다.

파티션 프루닝(Partition Pruning)과 파일 수준 통계(File-Level Statistics)는 불필요한 데이터 스캔을 줄이는 데 도움을 준다. 메타데이터는 데이터 파일에 포함된 값의 범위와 특성을 설명할 수 있으며, 이를 지원하는 질의 엔진은 필터 조건을 만족할 수 없는 파일을 제외할 수 있다. 특정 시간 범위, 로봇 그룹(Robot Group), 운영 조건(Operational Condition)을 요청하는 플릿 분석(Fleet Analysis)은 전체 과거 테이블을 읽는 대신 대규모 테이블의 일부만 스캔할 수 있다.

아이스버그는 호환되는 처리 엔진(Processing Engines)을 통해 업데이트(Update), 삭제(Delete), 병합(Merge)과 같은 행 수준 변경(Row-Level Modification) 패턴을 지원한다. 엔진과 테이블 구성에 따라 변경 사항은 데이터 파일을 다시 작성하거나 기존 파일과 연결된 삭제 정보(Delete Information)를 이용하여 표현할 수 있다. 이러한 기능은 중복 로봇 텔레메트리를 제거하거나, 주석을 수정하거나, 품질 플래그(Quality Flags)를 변경하거나, 지연 도착 정보(Late-Arriving Information)를 기존 테이블에 통합해야 하는 경우 유용하다.

동시 데이터 파이프라인(Concurrent Data Pipelines)은 아이스버그의 커밋 모델(Commit Model)을 활용할 수 있다. 독립적인 쓰기 작업은 알려진 테이블 상태를 기준으로 작업한 후 원자적 메타데이터 업데이트(Atomic Metadata Update)를 시도할 수 있다. 충돌 탐지(Conflict Detection)와 재시도 동작(Retry Behavior)은 서로 다른 작업이 일관되지 않은 테이블을 생성하는 것을 방지하는 데 도움을 준다. 이는 데이터 수집, 주석, 데이터셋 정제(Dataset Curation), 분석, AI 데이터 준비 파이프라인이 동일한 공유 테이블과 동시에 상호작용하는 로보틱스 환경에서 유용하다.

아이스버그는 하나의 처리 프레임워크에 종속되지 않고 여러 분석 엔진(Analytical Engines) 사이의 상호운용성(Interoperability)을 제공하도록 설계되었다. 아파치 스파크(Apache Spark), 아파치 플링크(Apache Flink), 트리노(Trino) 및 기타 호환 시스템은 동일한 테이블 표현을 사용할 수 있다. 이를 통해 서로 다른 분석 도구마다 로봇 데이터셋의 별도 물리적 복사본을 생성해야 하는 필요성을 줄이고, 저장 계층은 안정적으로 유지하면서 컴퓨팅 기술을 독립적으로 발전시킬 수 있다.

이러한 엔진 독립성(Engine Independence)은 장기간 운영되는 로보틱스 플랫폼에서 중요하다. 현재 수집한 데이터는 수년 동안 가치가 유지될 수 있지만 처리 프레임워크, AI 시스템, 인프라는 훨씬 빠르게 변화할 수 있다. 표준화된 테이블 추상화(Standardized Table Abstraction)를 사용하면 장기간 유지해야 하는 데이터 자산(Durable Data Assets)을 특정 컴퓨팅 엔진과 분리할 수 있다. 따라서 조직이 데이터 분석 기술을 교체하거나 확장하더라도 과거의 로봇 경험(Historical Robot Experience)을 계속 활용할 수 있다.

대용량 바이너리 로봇 센서 객체(Large Binary Robot Sensor Objects)를 반드시 아이스버그 테이블 내부에 직접 포함할 필요는 없다. 이미지, 비디오, 라이다 포인트 클라우드(LiDAR Point Clouds), 오디오, ROS 기록은 확장 가능한 저장소에 대용량 객체로 유지할 수 있다. 아이스버그 테이블에는 구조화된 텔레메트리, 타임스탬프, 레이블(Labels), 보정 참조(Calibration References), 품질 지표(Quality Indicators), 처리 결과, 객체 위치(Object Locations)를 저장하여 훨씬 큰 물리적 센서 아카이브 위에 효율적으로 검색할 수 있는 계층을 제공할 수 있다.

AI 데이터셋 준비(AI Dataset Preparation) 과정에서 아이스버그 테이블은 후보 관측 데이터(Candidate Observations)를 관리하는 구조화된 카탈로그 역할을 할 수 있다. 질의를 통해 로봇 유형(Robot Type), 환경, 임무 결과(Mission Outcome), 센서 구성(Sensor Configuration), 이상 상태(Anomaly State), 레이블 품질(Label Quality), 시간 범위를 기준으로 장면을 선택할 수 있다. 이후 처리 파이프라인은 참조된 바이너리 객체를 가져와 버전 관리된 학습 데이터셋(Versioned Training Datasets)을 구성할 수 있다. 이를 통해 필요한 관측 데이터를 찾기 위해 대규모 미디어 아카이브 전체를 반복적으로 스캔할 필요가 없다.

스냅샷 기반 재현성(Snapshot-Based Reproducibility)은 머신러닝(Machine Learning)에 특히 유용하다. 하나의 실험에서 아이스버그 스냅샷 식별자(Snapshot Identifier)를 모델 코드(Model Code), 하이퍼파라미터(Hyperparameters), 소프트웨어 환경(Software Environment), 데이터셋 생성 로직(Dataset-Generation Logic)과 함께 기록할 수 있다. 스냅샷과 해당 스냅샷이 참조하는 파일이 계속 유지된다면 이후 동일한 입력 테이블을 일관되게 재구성할 수 있다. 이는 내용이 지속적으로 변경되는 디렉터리만 사용하는 것보다 더욱 강력한 재현성 경계(Reproducibility Boundary)를 제공한다.

데이터셋 규모가 증가하면 테이블 유지관리(Table Maintenance)가 필요하다. 빈번한 데이터 수집, 업데이트, 스트리밍 작업은 많은 소형 파일(Small Files)과 메타데이터 객체(Metadata Objects)를 생성할 수 있다. 데이터 파일 컴팩션(Data-File Compaction)을 통해 작은 파일을 결합하고, 메타데이터 유지관리(Metadata Maintenance)를 통해 질의 계획 오버헤드를 줄이며 보존 정책에 따라 오래된 정보를 제거할 수 있다. 이러한 작업은 논리적인 테이블 의미(Logical Table Semantics)를 유지하면서 저장 효율성과 질의 성능을 향상시켜야 한다.

거버넌스(Governance)와 보안(Security)은 아이스버그를 둘러싼 카탈로그(Catalog), 저장소(Storage), 컴퓨팅 환경(Compute Environment)을 통해 구현된다. 인증(Authentication), 인가(Authorization), 암호화(Encryption), 감사 로깅(Audit Logging), 보존 제어(Retention Controls), 데이터셋 분류(Dataset Classification)를 통해 로봇 정보에 접근하거나 수정할 수 있는 사용자를 결정한다. 스냅샷 만료(Snapshot Expiration)와 고아 파일 정리(Orphan-File Cleanup)는 지나치게 공격적으로 수행할 경우 시간 여행, 재현성, 사고 조사, 규정 준수에 필요한 과거 데이터를 제거할 수 있으므로 특히 신중하게 관리해야 한다.

피지컬 AI 데이터 레이크(Physical AI Data Lake)에서 아파치 아이스버그는 지속적으로 유지되는 저장소와 변화하는 분석 기술 사이에 표준화된 테이블 계층(Standardized Table Layer)을 제공한다. 로봇은 관측 데이터(Observations)를 생성하고, 수집 파이프라인은 구조화된 데이터와 참조 정보를 기록하며, 아이스버그는 파일을 일관된 스냅샷으로 구성한다. 여러 컴퓨팅 엔진은 동일한 테이블을 분석하고, AI 파이프라인은 안정적인 버전을 이용해 모델을 개발한다. 개선된 모델은 다시 로봇으로 전달되고 새로운 운영 경험이 지속적으로 추가되면서 거버넌스가 적용된 데이터 기반(Governed Data Foundation)이 확장된다.

## 06.07 Data Lakehouse Architecture: Databricks / AWS Lake Formation

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

데이터 레이크하우스(Data Lakehouse)는 데이터 레이크(Data Lake)의 확장 가능하고 비용 효율적인 저장 특성과 전통적으로 데이터 웨어하우스(Data Warehouse)가 제공해 온 구조화된 관리, 신뢰성, 질의 기능을 결합한 아키텍처이다. 로보틱스(Robotics)와 피지컬 AI(Physical AI)에서는 원시 센서 아카이브(Raw Sensor Archives), 구조화된 텔레메트리(Structured Telemetry), 운영 분석(Operational Analytics), 시뮬레이션 결과(Simulation Results), AI 학습 데이터셋(AI Training Datasets), 모델 관련 메타데이터(Model-Related Metadata)를 각 워크로드마다 분리된 플랫폼을 구축하지 않고 통합하여 관리할 수 있는 기반을 제공한다.

전통적인 데이터 레이크는 이미지, 비디오, 라이다 포인트 클라우드(LiDAR Point Clouds), ROS 기록(ROS Recordings), 로그(Logs), 테이블(Tables), 다양한 파일을 확장 가능한 객체 저장소(Object Storage)에 함께 저장할 수 있어 매우 유연하다. 그러나 관리되지 않는 데이터 집합은 일관되지 않은 스키마(Inconsistent Schemas), 중복 데이터(Duplicate Data), 취약한 트랜잭션 제어(Transaction Control), 어려운 데이터 탐색(Data Discovery) 문제를 발생시킬 수 있다. 레이크하우스는 기본 데이터 레이크를 영구 저장소로 유지하면서 테이블 형식(Table Formats), 메타데이터 카탈로그(Metadata Catalogs), 거버넌스(Governance), 트랜잭션 처리(Transactional Processing), 최적화된 컴퓨팅 엔진(Optimized Compute Engines)을 추가한다.

아키텍처는 일반적으로 저장소(Storage)와 컴퓨팅(Compute)을 분리한다. 로봇 데이터는 객체 저장소에 지속적으로 보존하면서 독립적인 컴퓨팅 자원을 이용하여 데이터 수집(Ingestion), SQL 분석(SQL Analytics), 분산 전처리(Distributed Preprocessing), 스트리밍(Streaming), 머신러닝(Machine Learning) 워크로드를 수행할 수 있다. 컴퓨팅 클러스터(Compute Clusters)는 기본 데이터셋을 이동하지 않고도 요구사항에 따라 확장할 수 있다. 이러한 분리는 방대한 과거 로봇 데이터셋을 유지하면서 서로 다른 팀과 워크로드에 서로 다른 처리 용량이 필요한 경우 유용하다.

일반적인 로보틱스 레이크하우스(Robotics Lakehouse)는 데이터 수집 계층(Ingestion Layer)에서 시작된다. 로봇, 엣지 컴퓨터(Edge Computers), 시뮬레이션 환경(Simulation Environments), 기업 애플리케이션(Enterprise Applications), 외부 데이터셋(External Datasets), 주석 시스템(Annotation Systems)은 지속적으로 정보를 생성한다. 고대역폭 바이너리 기록(High-Bandwidth Binary Recordings)은 배치 방식으로 업로드할 수 있고, 텔레메트리와 운영 이벤트는 스트리밍 파이프라인을 통해 전달할 수 있다. 검증(Validation), 체크섬 검증(Checksum Verification), 스키마 검사(Schema Checks), 타임스탬프 정규화(Timestamp Normalization), 메타데이터 등록(Metadata Registration)을 통해 관리형 분석 테이블에 데이터가 들어가기 전에 신뢰할 수 있는 입력을 확보한다.

대용량 바이너리 객체(Large Binary Objects)와 구조화된 정보(Structured Information)는 상호 보완적인 저장 경로를 사용할 수 있다. 이미지, 비디오, 오디오, 포인트 클라우드(Point Clouds), ROS 백 파일(ROS Bag Files)은 내구성 있는 객체(Durable Objects)로 유지하고, 구조화된 테이블에는 텔레메트리, 레이블(Labels), 품질 지표(Quality Indicators), 보정 참조(Calibration References), 임무 정보(Mission Information), 객체 위치(Object Locations)를 저장할 수 있다. 이러한 방식은 분석 엔진이 필요한 관측 데이터를 찾기 위해 방대한 바이너리 아카이브를 반복적으로 스캔하는 것을 방지하고 구조화된 질의를 통해 후속 처리에 필요한 대용량 센서 객체만 선택할 수 있도록 한다.

데이터브릭스(Databricks)는 확장 가능한 클라우드 저장소(Cloud Storage)를 중심으로 데이터 엔지니어링(Data Engineering), 분석(Analytics), 거버넌스, 스트리밍, 머신러닝 워크플로를 통합하는 레이크하우스 지향 플랫폼(Lakehouse-Oriented Platform)을 제공한다. 델타 레이크(Delta Lake)와 같은 기술은 트랜잭션 테이블 관리(Transactional Table Management)를 제공할 수 있으며, 분산 컴퓨팅 엔진(Distributed Compute Engines)은 대규모 데이터셋을 처리한다. 로보틱스 환경에서는 이러한 기능을 이용하여 플릿 분석(Fleet Analytics), 텔레메트리 변환(Telemetry Transformation), 데이터셋 정제(Dataset Curation), 특징 생성(Feature Generation), 모델 실험(Model Experimentation), 운영 모니터링(Operational Monitoring)을 하나의 조정된 데이터 플랫폼에서 지원할 수 있다.

메달리온 아키텍처(Medallion Architecture)는 레이크하우스 데이터 파이프라인을 구성할 때 일반적으로 사용된다. 브론즈 계층(Bronze Layer)은 원시 또는 최소한으로 변환된 정보를 보존하고, 실버 계층(Silver Layer)은 검증되고 표준화된 레코드를 포함하며, 골드 계층(Gold Layer)은 애플리케이션 지향 데이터 제품(Application-Oriented Data Products)을 제공한다. 이러한 계층 이름은 필수적인 기술 구성요소가 아니라 아키텍처 설계 관례이지만, 원본 로봇 증거(Original Robot Evidence), 재사용 가능한 신뢰 정보(Reusable Trusted Information), 특정 분석 또는 AI 워크로드에 최적화된 데이터셋을 분리하는 유용한 방법을 제공한다.

로봇 데이터의 경우 브론즈 저장소(Bronze Storage)는 업로드된 텔레메트리, 이벤트 스트림(Event Streams), 센서 메타데이터(Sensor Metadata), 원시 바이너리 기록에 대한 참조를 보존할 수 있다. 실버 처리(Silver Processing)는 타임스탬프, 좌표계(Coordinate Systems), 단위(Units), 로봇 식별자(Robot Identifiers), 센서 구성(Sensor Configurations), 품질 상태(Quality States)를 정규화하면서 중복 또는 유효하지 않은 레코드를 제거할 수 있다. 골드 데이터셋(Gold Datasets)은 특정 사용자를 위해 설계된 플릿 핵심성과지표(Fleet KPIs), 임무 요약(Mission Summaries), 이상 탐지 특징(Anomaly Features), 평가 테이블(Evaluation Tables), 정제된 AI 학습 인덱스(Curated AI Training Indexes)를 제공할 수 있다.

AWS는 레이크하우스 아키텍처에 활용할 수 있는 클라우드 서비스를 제공하며, 아마존 S3(Amazon S3)는 일반적으로 확장 가능한 객체 저장소로 사용된다. AWS 레이크 포메이션(AWS Lake Formation)은 카탈로그에 등록된 데이터에 대한 접근을 조정하고 주변 AWS 분석 서비스와 통합하여 데이터 레이크를 구축하고 보호하며 관리하는 데 중점을 둔다. 레이크 포메이션 자체가 테이블 형식으로 동작하는 것이 아니라 광범위한 AWS 데이터 생태계에 저장된 데이터를 대상으로 거버넌스와 권한 관리(Permission Management) 기능을 제공한다.

중앙 집중형 카탈로그(Centralized Catalog)는 물리적 파일만으로는 로보틱스 데이터셋의 의미를 설명할 수 없기 때문에 필수적이다. 카탈로그 항목은 스키마, 테이블 위치(Table Locations), 소유권(Ownership), 분류(Classifications), 파티션(Partitions), 데이터 제품 사이의 관계를 설명할 수 있다. 추가적인 로보틱스 메타데이터에는 로봇 모델(Robot Models), 센서 구성, 소프트웨어 릴리스(Software Releases), 보정 버전(Calibration Versions), 임무 맥락(Mission Contexts), 환경 조건(Environmental Conditions), 처리 계보(Processing Lineage)를 포함하여 엔지니어가 파일 이름이 아니라 운영 의미를 기준으로 데이터셋을 탐색할 수 있도록 해야 한다.

개방형 테이블 형식(Open Table Formats)은 현대적인 레이크하우스에서 중요한 관리 계층을 제공한다. 델타 레이크(Delta Lake) 또는 아파치 아이스버그(Apache Iceberg)와 같은 기술은 파일 집합 위에 일관된 테이블 상태를 정의하고 원자적 커밋(Atomic Commits), 스키마 진화(Schema Evolution), 스냅샷(Snapshots), 업데이트(Updates), 확장 가능한 메타데이터 관리(Scalable Metadata Management)와 같은 기능을 지원할 수 있다. 정확한 기능은 선택한 테이블 형식과 처리 엔진에 따라 달라지지만, 핵심 목적은 분석 테이블이 서로 관련 없는 파일의 관리되지 않는 디렉터리가 되는 것을 방지하는 것이다.

여러 로보틱스 파이프라인이 동시에 동작하는 환경에서는 트랜잭션 관리(Transaction Management)가 중요하다. 플릿 데이터 수집은 텔레메트리를 추가하는 동시에 주석 시스템은 레이블을 수정하고, 품질 관리 서비스는 유효하지 않은 샘플을 표시하며, 전처리 작업은 파생 레코드(Derived Records)를 생성하고, AI 파이프라인은 학습 후보 데이터를 읽을 수 있다. 관리형 테이블 의미론(Managed Table Semantics)을 적용하면 쓰기 작업이 통제된 변경 사항을 게시하는 동안 읽기 작업은 일관된 상태에 접근할 수 있어 실험에서 부분적으로 업데이트되거나 내부적으로 불일치하는 데이터를 사용하는 위험을 줄일 수 있다.

스트리밍 워크로드(Streaming Workloads)와 배치 워크로드(Batch Workloads)는 동일한 레이크하우스 기반을 공유할 수 있다. 운영 텔레메트리는 지속적으로 도착하여 준실시간 모니터링(Near-Real-Time Monitoring)에 활용할 수 있고, 예약된 작업은 수개월 또는 수년간 축적된 과거 로봇 행동을 분석할 수 있다. 이러한 워크로드를 위해 완전히 분리된 데이터 복사본을 유지하는 대신 관리형 테이블과 객체 저장소를 공통 단일 진실 공급원(Common Source of Truth)으로 활용할 수 있다. 이후 처리 엔진은 각 워크로드에 적합한 접근 패턴을 선택할 수 있다.

레이크하우스는 모델 개발에 단순한 학습 파일 저장 이상의 기능이 필요하기 때문에 AI에 특히 유용하다. 엔지니어는 데이터셋 탐색(Dataset Discovery), 버전 관리(Versioning), 데이터 계보(Data Lineage), 품질 정보(Quality Information), 재현 가능한 변환(Reproducible Transformations), 실험 추적(Experiment Tracking), 원본 관측 데이터에 대한 통제된 접근이 필요하다. 구조화된 테이블을 통해 후보 장면(Candidate Scenes)을 식별하고 처리 파이프라인이 관련 이미지, 비디오, 라이다, 오디오를 가져와 인식(Perception), 월드 모델(World Model), 계획(Planning), 정책 학습(Policy Learning)을 위한 버전 관리 데이터셋으로 변환할 수 있다.

데이터셋 재현성(Dataset Reproducibility)은 명시적으로 설계해야 한다. AI 실험은 학습 및 평가 데이터를 구성할 때 사용한 정확한 테이블 버전 또는 스냅샷, 선택 로직(Selection Logic), 원본 객체(Source Objects), 처리 코드(Processing Code), 스키마 버전(Schema Version), 주석 릴리스(Annotation Release), 변환 파라미터(Transformation Parameters)를 기록해야 한다. 이러한 정보가 없으면 레이크하우스 역시 지속적으로 변하는 데이터 집합이 될 수 있다. 안정적인 데이터셋 매니페스트(Stable Dataset Manifests)와 데이터 계보는 지속적으로 변화하는 운영 데이터를 재현 가능한 과학 및 엔지니어링 자산으로 변환한다.

데이터 품질(Data Quality)은 레이크하우스의 각 단계에 통합할 수 있다. 수집 파이프라인은 손상된 파일(Corrupted Files), 누락된 메타데이터(Missing Metadata), 유효하지 않은 스키마(Invalid Schemas), 타임스탬프 불연속(Timestamp Discontinuities), 전송 실패(Transfer Failures)를 탐지할 수 있다. 정제 단계에서는 동기화(Synchronization), 보정 일관성(Calibration Consistency), 값의 범위(Value Ranges), 중복 레코드(Duplicate Records), 주석 품질(Annotation Quality)을 평가할 수 있다. 품질 결과 자체를 관리형 컬럼과 메타데이터로 저장하면 후속 애플리케이션에서 측정 가능한 신뢰성 요구사항에 따라 데이터를 선택할 수 있다.

레이크하우스가 여러 엔지니어링 팀, 프로젝트, 고객 또는 지리적 지역을 지원하게 되면 거버넌스(Governance)가 더욱 중요해진다. 접근 제어(Access Control)를 통해 원시 센서 아카이브, 운영 테이블, 개인정보가 포함된 민감 정보(Personally Sensitive Information), 학습 데이터셋, 외부 공유 가능 데이터 제품을 구분할 수 있다. 암호화(Encryption), 감사 로깅(Audit Logging), 보존 정책(Retention Policies), 데이터 분류(Data Classification), 카탈로그 권한(Catalog Permissions), 통제된 내보내기(Controlled Exports)는 편리한 중앙 집중형 접근이 통제되지 않은 접근으로 이어지지 않도록 지원한다.

클라우드 레이크하우스(Cloud Lakehouse)는 온프레미스(On-Premise) 및 엣지 인프라(Edge Infrastructure)와 함께 운영할 수 있다. 로봇은 먼저 로컬 NVMe 저장장치에 데이터를 기록하고, 이를 온프레미스 NAS 또는 객체 저장소로 전송한 후 선택된 정보를 클라우드 레이크하우스와 동기화할 수 있다. 민감하거나 지연시간에 민감한 워크로드(Latency-Critical Workloads)는 로컬에 유지하고 대규모 분석이나 학습에는 중앙 집중형 자원을 사용할 수 있다. 이러한 하이브리드 아키텍처(Hybrid Architecture)는 모든 로봇이 항상 고대역폭 네트워크에 연결되어 있다고 가정하는 것보다 실제 로보틱스 환경에 더 적합한 경우가 많다.

비용 관리(Cost Management)는 저장 비용뿐만 아니라 데이터 이동(Data Movement)도 고려해야 한다. 객체 저장소는 대규모 아카이브를 경제적으로 보존할 수 있지만 반복적인 데이터 검색, 네트워크 전송, 분산 스캔(Distributed Scans), 불필요한 데이터 복제는 비용을 증가시킬 수 있다. 수명주기 정책(Lifecycle Policies)을 통해 오래된 데이터셋을 콜드 스토리지(Cold Storage)로 이동하고, 파티션 프루닝(Partition Pruning), 캐싱(Caching), 컬럼 기반 형식(Columnar Formats), 데이터셋 선택(Dataset Selection)을 활용하여 처리량을 줄일 수 있다. 따라서 아키텍처는 단순히 로봇 데이터를 어디에 저장할 것인지가 아니라 대용량 데이터를 얼마나 자주 이동할 것인지까지 최적화해야 한다.

성숙한 레이크하우스는 분석 도구가 변경될 때마다 장기간 유지되는 데이터 기반(Durable Data Foundation)을 변경하지 않고도 여러 컴퓨팅 기술을 지원할 수 있다. SQL 엔진(SQL Engines), 분산 처리 프레임워크(Distributed Processing Frameworks), 스트리밍 시스템(Streaming Systems), 노트북(Notebooks), 머신러닝 환경(Machine-Learning Environments)은 호환 가능한 인터페이스를 통해 거버넌스가 적용된 데이터셋에서 동작할 수 있다. 이러한 분리를 통해 로보틱스 조직은 장기간 축적된 운영 경험을 보존하면서 그 경험에서 지능을 추출하는 소프트웨어를 지속적으로 발전시킬 수 있다.

피지컬 AI 아키텍처(Physical AI Architecture)에서 레이크하우스는 로봇 플릿 경험(Fleet Experience)과 지속적으로 개선되는 지능(Continuously Improving Intelligence)을 연결하는 기반이 된다. 로봇은 관측 데이터(Observations)를 생성하고, 엣지 및 클라우드 파이프라인은 이를 수집하며, 거버넌스가 적용된 저장소는 원본 증거(Raw Evidence)를 보존한다. 관리형 테이블은 재사용 가능한 정보를 구성하고 분석 시스템은 이를 인사이트(Insights)와 AI 데이터셋으로 변환한다. 이러한 데이터셋으로 학습된 모델은 다시 배치된 로봇으로 전달되어 추적 가능하고 확장 가능한 데이터-지능 피드백 순환 구조(Traceable and Scalable Data-to-Intelligence Feedback Cycle)를 형성한다.

## 06.08 Data Lake Metadata Management: Hive Metastore [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

데이터 레이크(Data Lake)는 방대한 규모의 로봇 텔레메트리(Robot Telemetry), 센서 기록(Sensor Recordings), 시뮬레이션 결과(Simulation Outputs), 주석(Annotations), 운영 로그(Operational Logs), AI 데이터셋(AI Datasets)을 포함할 수 있지만 물리적 파일만으로는 데이터가 무엇을 의미하는지 설명할 수 없다. 메타데이터 관리(Metadata Management)는 저장된 객체를 스키마(Schema), 위치(Location), 파티션(Partition), 소유권(Ownership), 형식(Format), 운영 의미(Operational Meaning)와 연결하는 설명 계층을 제공한다. 이러한 계층이 없으면 대규모 데이터 레이크는 점차 데이터를 탐색하고 해석하며 관리하기 어려운 환경으로 변할 수 있다.

메타데이터(Metadata)는 서로 보완적인 여러 범주로 구분할 수 있다. 기술 메타데이터(Technical Metadata)는 스키마, 컬럼 유형(Column Types), 파일 형식(File Formats), 파티션, 저장 위치(Storage Locations), 테이블 속성(Table Properties)을 설명한다. 운영 메타데이터(Operational Metadata)는 수집 시간(Ingestion Time), 처리 상태(Processing Status), 작업 이력(Job History), 데이터 품질(Data Quality)을 기록한다. 비즈니스 또는 의미 메타데이터(Business or Semantic Metadata)는 소유권, 목적, 분류(Classification), 의미를 설명한다. 로보틱스 시스템에서는 추가적으로 로봇, 센서, 임무(Missions), 보정(Calibration), 좌표 프레임(Coordinate Frames), 소프트웨어 버전(Software Versions)을 설명하는 도메인 메타데이터(Domain Metadata)가 필요하다.

메타데이터 카탈로그(Metadata Catalog)는 엔지니어가 저장 디렉터리를 직접 탐색하지 않아도 데이터 자산(Data Assets)을 검색할 수 있는 표현 체계를 제공한다. 하나의 테이블 항목(Table Entry)은 데이터가 어디에 저장되어 있고, 어떤 구조를 가지며, 처리 엔진이 이를 어떻게 해석해야 하는지를 정의할 수 있다. 이러한 추상화(Abstracted Layer)는 동일한 물리적 저장소에 파케이(Parquet) 테이블, 로그, 이미지, 비디오, 라이다 기록(LiDAR Recordings), ROS 백(ROS Bags), 시뮬레이션 데이터, 외부 관리 데이터셋에 대한 참조가 함께 존재하는 경우 특히 중요하다.

아파치 하이브 메타스토어(Apache Hive Metastore)는 아파치 하이브(Apache Hive) 생태계에서 시작된 널리 사용되는 메타데이터 저장소(Metadata Repository)이다. 실제 테이블 데이터와 독립적으로 데이터베이스, 테이블, 컬럼, 파티션, 저장 위치 및 관련 속성에 대한 메타데이터를 저장한다. 처리 엔진은 메타스토어(Metastore)를 조회하여 논리적 테이블(Logical Table)이 물리적 파일과 어떻게 연결되는지 파악할 수 있으며, 사용자는 매 작업마다 저장 경로와 스키마를 직접 지정하는 대신 이름이 지정된 테이블(Named Tables)을 질의할 수 있다.

하이브 메타스토어(Hive Metastore)는 일반적으로 여러 클라이언트에 메타데이터 서비스(Metadata Services)를 제공하면서 영구적인 카탈로그 정보(Persistent Catalog Information)를 백엔드 관계형 데이터베이스(Backing Relational Database)에 유지한다. 이러한 분리를 통해 컴퓨팅 엔진(Compute Engines)은 메타데이터 영속성(Metadata Persistence)으로부터 상당 부분 독립적으로 동작할 수 있다. 아파치 스파크(Apache Spark)와 기타 호환 가능한 분석 기술은 지원되는 통합 방식을 통해 카탈로그 정보와 상호작용하여 서로 다른 처리 워크로드가 테이블과 기본 저장소에 대한 일관된 정의를 공유할 수 있다.

테이블 메타데이터(Table Metadata)는 일반적으로 테이블 이름, 데이터베이스 또는 네임스페이스(Namespace), 컬럼, 데이터 유형(Data Types), 저장 위치, 파일 형식, 파티션 정보, 테이블 속성을 포함한다. 이러한 속성은 저장된 파일과 처리 엔진 사이의 계약(Contract)을 형성한다. 메타데이터에서 특정 컬럼을 타임스탬프(Timestamp), 정수(Integer), 문자열(String), 구조화된 값(Structured Value)으로 정의하면 호환되는 엔진은 질의를 실행하기 전에 모든 파일을 검사하지 않고도 적절한 논리적 표현(Logical Representation)을 구성할 수 있다.

파티션 메타데이터(Partition Metadata)는 대규모 과거 데이터셋에서 특히 중요하다. 로봇 텔레메트리는 날짜(Date), 사이트(Site), 플릿(Fleet), 로봇 그룹(Robot Group) 또는 다른 운영 차원을 기준으로 구성될 수 있다. 카탈로그가 파티션 정보를 기록하면 처리 엔진은 질의 조건을 만족할 가능성이 있는 데이터 하위 집합을 식별할 수 있다. 적절한 파티션 프루닝(Partition Pruning)을 적용하면 엔지니어가 특정 운영 기간이나 플릿의 일부만 필요로 할 때 스캔해야 하는 저장 데이터의 양을 크게 줄일 수 있다.

하이브 방식 파티션 관리(Hive-Style Partition Management)는 전통적으로 날짜나 사이트 값을 저장 경로에 포함하는 디렉터리 구조를 반영하는 경우가 많았다. 이러한 모델은 이해하기 쉽지만 파티션 전략이 변경되거나 매우 많은 파티션이 누적되면 관리가 어려워질 수 있다. 아파치 아이스버그(Apache Iceberg)와 같은 현대적인 테이블 형식(Table Formats)은 디렉터리 기반 파티션 탐색(Directory-Based Partition Discovery)에 대한 의존성을 줄이는 더욱 풍부한 메타데이터 모델을 제공하면서 지원되는 아키텍처에서는 카탈로그 서비스와 통합될 수 있다.

메타데이터 일관성(Metadata Consistency)은 매우 중요하다. 잘못된 카탈로그 정보는 정상적인 물리적 데이터에 접근하기 어렵게 만들거나, 더 위험하게는 데이터를 잘못 해석하게 만들 수 있다. 스키마 정의(Schema Definitions), 파티션 값(Partition Values), 저장 경로(Storage Paths), 파일 형식은 실제 데이터셋과 동기화되어야 한다. 따라서 자동화된 수집 파이프라인(Automated Ingestion Pipelines)은 파일이 도착한 이후 엔지니어가 수동으로 카탈로그 항목을 수정하는 방식보다 데이터 기록 과정의 일부로 메타데이터를 등록해야 한다.

로봇 플랫폼이 발전하면서 스키마 관리(Schema Management)는 더욱 복잡해진다. 새로운 센서가 도입되거나 제어기가 업그레이드되거나 추가적인 진단 정보(Diagnostic Information)가 제공되면 텔레메트리 테이블에 새로운 필드가 추가될 수 있다. 메타데이터 시스템은 이러한 변경 사항을 표현해야 하며, 후속 애플리케이션은 어떤 레코드에 어떤 스키마 버전(Schema Version)이 적용되는지 이해해야 한다. 스키마 진화(Schema Evolution)는 의미론적 문서화(Semantic Documentation)와 함께 관리하여 외형적으로 동일한 필드가 여러 세대에 걸쳐 동일한 의미를 가진다고 잘못 가정하지 않도록 해야 한다.

로보틱스 메타데이터(Robotics Metadata)는 기존 데이터베이스 스키마 이상의 정보를 설명해야 한다. 속도 필드(Velocity Field)는 단위와 좌표 프레임 규칙(Coordinate-Frame Conventions)이 없으면 의미가 모호하며, 카메라 이미지는 내부 보정(Intrinsic Calibration), 외부 보정(Extrinsic Calibration), 타임스탬프 출처(Timestamp Source), 센서 식별정보(Sensor Identity), 로봇 구성(Robot Configuration)이 필요할 수 있다. 라이다와 관성 측정 장치(IMU) 데이터 역시 프레임 관계(Frame Relationships)와 동기화 정보(Synchronization Information)에 의존한다. 따라서 유용한 로보틱스 카탈로그는 기술 메타데이터와 도메인별 의미 메타데이터(Domain-Specific Semantic Metadata)를 연결해야 한다.

시간 메타데이터(Time Metadata)는 피지컬 AI(Physical AI) 시스템에서 특별한 주의가 필요하다. 센서 타임스탬프(Sensor Timestamp), 하드웨어 캡처 시간(Hardware Capture Time), 로봇 시계(Robot Clock), 엣지 수신 시간(Edge Reception Time), 데이터 수집 시간(Ingestion Time), 클라우드 처리 시간(Cloud Processing Time)은 서로 다른 이벤트를 나타낼 수 있다. 메타데이터는 클록 도메인(Clock Domains), 동기화 방식(Synchronization Methods), 필요한 경우 시간대(Time Zones), 타임스탬프 출처(Timestamp Provenance)를 식별해야 한다. 이러한 구분이 없으면 멀티모달 관측 데이터(Multimodal Observations)가 구조적으로 정상으로 보이더라도 인식(Perception), 매핑(Mapping), 모델 학습(Model Training) 과정에서 잘못 정렬될 수 있다.

대용량 바이너리 객체(Large Binary Objects)는 관계형 형태의 테이블 내부에 직접 저장하지 않더라도 카탈로그 기반 관리(Catalog-Based Management)에 포함할 수 있다. 구조화된 메타데이터 테이블에는 객체 URI(Object URI), 체크섬(Checksum), 크기(Size), 미디어 유형(Media Type), 캡처 시간(Capture Time), 로봇 식별자(Robot Identifier), 센서 식별자(Sensor Identifier), 처리 상태(Processing Status)를 저장할 수 있다. 질의를 통해 먼저 이러한 경량 메타데이터를 검색한 후 분석 또는 AI 학습에 필요한 이미지, 비디오, 포인트 클라우드, 오디오 파일, ROS 백만 선택적으로 가져올 수 있다.

데이터 계보(Data Lineage)는 메타데이터를 단순히 데이터셋이 무엇인지를 설명하는 수준에서 데이터셋이 어떻게 생성되었는지를 설명하는 수준으로 확장한다. 데이터 계보는 원시 로봇 관측 데이터(Raw Robot Observations)를 검증 작업(Validation Jobs), 정제된 테이블(Cleaned Tables), 주석, 특징 생성 파이프라인(Feature-Generation Pipelines), 학습 데이터셋, 실험(Experiments), 배포된 모델(Deployed Models)과 연결할 수 있다. 모델이 예상과 다르게 동작하면 엔지니어는 데이터 계보를 추적하여 어떤 원본 관측 데이터와 변환 과정이 해당 모델 버전에 영향을 주었는지 확인할 수 있다.

운영 메타데이터(Operational Metadata)는 데이터 레이크가 정상적으로 동작하는지를 모니터링하는 데 도움을 준다. 데이터 수집 작업은 시작 및 완료 시간, 레코드 수(Record Counts), 전송 바이트(Transferred Bytes), 거부된 파일(Rejected Files), 검증 결과(Validation Results), 처리 오류(Processing Errors)를 기록할 수 있다. 품질 관리 시스템은 누락된 값(Missing Values), 타임스탬프 간격(Timestamp Gaps), 중복 레코드, 보정 문제(Calibration Problems), 손상된 객체(Corrupted Objects)를 등록할 수 있다. 이러한 기록은 메타데이터를 정적인 카탈로그에서 데이터 플랫폼의 운영 관측성 계층(Operational Observability Layer)으로 확장한다.

메타데이터는 데이터 탐색(Data Discovery)도 지원한다. 엔지니어는 로봇 플랫폼(Robot Platform), 센서 유형(Sensor Type), 사이트, 임무, 환경 조건(Environmental Condition), 주석 상태(Annotation Status), 품질 수준(Quality Level), 시간 범위와 같은 의미 있는 특성을 기준으로 데이터셋을 찾을 수 있어야 한다. 검색 가능한 태그(Searchable Tags)와 의미 설명(Semantic Descriptions)은 조직 내부의 암묵적 지식(Institutional Knowledge)에 대한 의존도를 줄인다. 이는 로보틱스 조직이 수년간 운영 경험을 축적하고 인력이나 프로젝트 구조가 변화할수록 더욱 중요해진다.

거버넌스(Governance)는 보호 대상 데이터가 무엇인지 알아야 보안 정책을 적용할 수 있으므로 신뢰할 수 있는 메타데이터에 크게 의존한다. 카탈로그 분류(Catalog Classifications)는 공개 정보(Public Information), 내부 운영 기록(Internal Operational Records), 민감한 센서 데이터(Sensitive Sensor Data), 고객별 데이터셋(Customer-Specific Datasets), 제한된 AI 자산(Restricted AI Assets)을 구분할 수 있다. 접근 제어 시스템(Access-Control Systems)은 사용자 신원(Identity), 역할(Roles), 분류, 자원 소유권(Resource Ownership)을 결합하여 특정 데이터 제품을 누가 탐색하고 읽고 수정하거나 외부로 내보낼 수 있는지를 결정할 수 있다.

운영 환경의 메타스토어(Production Metastore)는 자체적인 신뢰성 전략(Reliability Strategy)이 필요하다. 많은 분석 워크로드가 메타데이터 탐색에 의존하기 때문에 메타스토어를 사용할 수 없으면 정상적인 데이터 파일이 존재하더라도 사용자가 접근하지 못할 수 있다. 따라서 백엔드 데이터베이스는 적절한 백업(Backup), 복구(Recovery), 모니터링(Monitoring), 고가용성(High Availability) 방식을 통해 보호해야 한다. 또한 카탈로그 정의의 실수로 인한 삭제나 변경은 여러 종속 파이프라인에 영향을 줄 수 있으므로 메타데이터 변경 사항을 감사 가능(Auditable)하게 관리해야 한다.

데이터 플랫폼이 발전하면서 조직은 하이브 메타스토어를 계속 사용하거나 새로운 카탈로그 및 개방형 테이블 형식 카탈로그(Open Table-Format Catalogs)와 함께 사용하거나 이들로 전환할 수 있다. 아파치 아이스버그, 델타 레이크(Delta Lake), 클라우드 카탈로그(Cloud Catalogs), 거버넌스 플랫폼(Governance Platforms)은 서로 다른 메타데이터 기능과 통합 모델을 제공한다. 아키텍처의 목표는 특정 카탈로그 구현에 종속되는 것이 아니라 안정적인 논리적 식별자(Logical Identities), 상호운용 가능한 메타데이터(Interoperable Metadata), 통제된 진화(Controlled Evolution), 장기 보존 데이터와 교체 가능한 컴퓨팅 기술 사이의 명확한 분리를 확보하는 것이다.

메타데이터 관리는 머신러닝(Machine Learning)의 재현성(Reproducibility)도 향상시킨다. 학습 데이터셋은 물리적 파일뿐만 아니라 테이블 버전(Table Versions), 스키마, 필터(Filters), 주석, 품질 상태(Quality States), 변환 이력(Transformation History), 원본 출처(Source Provenance)를 참조해야 한다. 이러한 관계를 기록하면 엔지니어는 모델이 학습된 당시의 조건을 재구성할 수 있다. 따라서 데이터셋 식별성(Dataset Identity)은 데이터 내용, 메타데이터 상태, 처리 로직(Processing Logic), 버전 정보(Version Information)의 조합으로 구성된다.

피지컬 AI 데이터 레이크(Physical AI Data Lake)에서 메타데이터는 저장된 경험(Stored Experience)을 활용 가능한 지능(Usable Intelligence)과 연결하는 탐색 및 제어 계층(Navigation and Control Layer)을 형성한다. 로봇은 관측 데이터(Observations)를 생성하고 저장 시스템은 물리적 바이트(Physical Bytes)를 보존하며, 카탈로그는 이러한 데이터가 무엇을 의미하고 어디에 존재하며 어떻게 생성되었고 누가 사용할 수 있는지를 설명한다. 하이브 메타스토어와 관련 카탈로그 기술은 이러한 논리적 연결을 제공하여 분석 및 AI 시스템이 대규모 로봇 아카이브를 탐색 가능하고 거버넌스가 적용되며 재현 가능한 데이터셋으로 변환할 수 있도록 한다.

## 06.09 Data Lake Security: Column / Row Level Access Control

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

로보틱스 데이터 레이크(Robotics Data Lake)는 운영 텔레메트리(Operational Telemetry), 지도(Maps), 이미지, 비디오, 오디오, 라이다 기록(LiDAR Recordings), 유지보수 정보(Maintenance Information), 고객 데이터(Customer Data), 주석(Annotations), AI 학습 데이터셋(AI Training Datasets)을 포함할 수 있다. 모든 사용자에게 이러한 정보 전체에 대한 무제한 접근을 허용하면 불필요한 보안 및 개인정보 위험(Security and Privacy Risk)이 발생한다. 따라서 데이터 레이크 보안(Data Lake Security)은 저장 객체뿐만 아니라 각 사용자가 접근할 수 있는 테이블(Table), 행(Row), 열(Column), 파생 데이터 제품(Derived Data Products)을 제어하는 계층화된 인가(Layered Authorization)가 필요하다.

인증(Authentication)은 누가 또는 어떤 시스템이 접근을 요청하는지를 확인하고, 인가(Authorization)는 해당 주체가 무엇을 수행할 수 있는지를 결정한다. 엔지니어, 분석가, AI 연구자, 로봇, 엣지 시스템(Edge Systems), 데이터 수집 서비스(Ingestion Services), 자동화된 학습 파이프라인(Automated Training Pipelines)은 일반적으로 식별 가능한 계정 또는 서비스 아이덴티티(Service Identities)를 사용해야 한다. 중앙 집중형 아이덴티티 관리(Central Identity Management)를 적용하면 공유 자격 증명이나 수동으로 배포된 저장소 키에 의존하는 대신 조직의 역할에 따라 권한을 관리할 수 있다.

역할 기반 접근 제어(Role-Based Access Control)는 데이터 엔지니어(Data Engineer), 플릿 운영자(Fleet Operator), AI 연구자(AI Researcher), 보안 관리자(Security Administrator), 외부 협력자(External Collaborator)와 같은 책임에 따라 권한을 할당한다. 속성 기반 접근 제어(Attribute-Based Access Control)는 프로젝트(Project), 사이트(Site), 조직(Organization), 데이터 분류(Data Classification), 장치 식별정보(Device Identity), 환경(Environment) 등의 속성을 고려하여 이를 확장할 수 있다. 역할과 속성을 결합하면 여러 고객, 임무, 지역, 보안 도메인(Security Domains)에 걸쳐 로보틱스 데이터셋이 존재하는 환경에서 더욱 정밀한 인가를 구현할 수 있다.

최소 권한 원칙(Principle of Least Privilege)은 모든 접근 결정의 기본 원칙이 되어야 한다. 레이블이 지정된 카메라 데이터가 필요한 인식 연구자(Perception Researcher)가 고객 정보, 유지보수 기록 또는 모든 원시 플릿 로그에 자동으로 접근할 필요는 없다. 마찬가지로 데이터 수집 서비스는 데이터를 추가(Append)할 권한은 필요하지만 과거 데이터셋을 삭제할 권한은 필요하지 않을 수 있다. 각 아이덴티티에 필요한 최소한의 작업만 허용하면 실수로 인한 손상과 자격 증명 탈취의 영향을 모두 줄일 수 있다.

테이블 수준 접근 제어(Table-Level Access Control)는 사용자 또는 서비스가 전체 논리적 테이블(Logical Table)을 읽거나 수정할 수 있는지를 결정한다. 공개 참조 데이터(Public Reference Data), 내부 플릿 텔레메트리(Internal Fleet Telemetry), 제한된 프로젝트 정보(Restricted Project Information)처럼 데이터셋에 명확한 보안 경계가 존재할 때 유용하다. 사용하는 카탈로그(Catalog), 질의 엔진(Query Engine), 거버넌스 플랫폼(Governance Platform)에 따라 읽기(Read), 삽입(Insert), 업데이트(Update), 삭제(Delete), 테이블 관리(Administration) 등의 작업별로 권한을 구분할 수 있다.

열 수준 접근 제어(Column-Level Access Control)는 민감한 속성과 비민감한 속성이 동일한 테이블에 함께 존재할 때 더욱 세밀한 보호 기능을 제공한다. 로봇 임무 테이블(Robot Mission Table)에는 운영 지표와 함께 고객 식별자(Customer Identifiers), 정확한 위치(Exact Locations), 장치 일련번호(Device Serial Numbers), 내부 진단 필드(Internal Diagnostic Fields)가 포함될 수 있다. 허가된 애플리케이션은 작업에 필요한 열에만 접근하고 보호 대상 열은 숨기거나 접근하지 못하도록 하여 별도의 정제 테이블(Sanitized Tables)을 불필요하게 복제하는 것을 방지할 수 있다.

열 마스킹(Column Masking)은 원본 값에 대한 완전한 접근 차단이 적절하지 않은 경우 추가적인 보호 계층을 제공할 수 있다. 원본 값을 그대로 노출하는 대신 정책에 따라 삭제된 값(Redacted Value), 해시 값(Hashed Value), 일반화된 값(Generalized Value), 일부가 가려진 값(Partially Obscured Value)을 반환할 수 있다. 분석가는 정확한 위치 대신 지역 식별자(Regional Identifier)를 받거나 고객과 연결된 값 대신 가명 식별자(Pseudonymous Identifier)를 받을 수 있다. 다른 질의 경로를 통해 보호된 정보를 쉽게 복원하지 못하도록 마스킹 정책을 일관되게 적용해야 한다.

행 수준 접근 제어(Row-Level Access Control)는 사용자가 접근 가능한 테이블에서도 조회할 수 있는 레코드를 제한한다. 예를 들어 특정 고객은 자신의 배치 환경(Deployment)에 연결된 로봇 데이터만 볼 수 있고, 사이트 운영자는 자신에게 할당된 시설의 레코드만 볼 수 있다. 질의 시스템은 정책 조건식(Policy Predicate)을 자동으로 적용하여 허가되지 않은 행을 필터링하므로 모든 애플리케이션 개발자가 동일한 필터링 로직을 개별적으로 정확하게 구현할 필요가 없다.

행 수준 정책(Row-Level Policies)과 열 수준 정책(Column-Level Policies)은 함께 적용할 수 있다. 외부 프로젝트 파트너는 승인된 프로젝트에 속하는 행만 접근하고, 해당 행에서도 민감하지 않은 일부 열만 사용할 수 있다. 내부 엔지니어에게는 더 넓은 접근 범위를 제공하고 보안 관리자는 특별 권한(Privileged Capabilities)을 유지할 수 있다. 이러한 계층화된 접근 방식을 사용하면 각각의 접근 패턴마다 통제되지 않은 복사본을 생성하지 않고도 하나의 거버넌스가 적용된 논리적 데이터셋을 여러 사용자 그룹이 사용할 수 있다.

데이터 분류(Data Classification)는 확장 가능한 정책 결정을 위한 기반을 제공한다. 카탈로그 메타데이터(Catalog Metadata)는 데이터 자산을 공개(Public), 내부용(Internal), 기밀(Confidential), 제한(Restricted), 고객별(Customer-Specific), 개인정보 민감(Personally Sensitive), 수출 통제(Export-Controlled) 또는 조직 자체의 분류 모델에 따라 지정할 수 있다. 이후 정책은 각 파일마다 독립적으로 권한을 정의하는 대신 이러한 분류를 참조할 수 있다. 민감성이 계속 유지되는 경우에는 파생 테이블과 데이터셋에도 해당 분류가 이어지도록 관리해야 한다.

메타데이터 카탈로그(Metadata Catalog)는 인가 정책이 테이블 식별자(Table Identities), 스키마(Schemas), 소유권(Ownership), 태그(Tags), 분류(Classifications)에 의존하는 경우가 많기 때문에 접근 제어의 핵심 구성요소가 된다. 카탈로그 권한(Catalog Permissions)은 사용자가 데이터셋을 질의할 수 있는지뿐만 아니라 해당 데이터셋의 존재 자체를 탐색할 수 있는지도 결정한다. 일부 환경에서는 테이블 이름, 스키마, 위치, 설명 태그만으로도 민감한 운영 정보가 노출될 수 있으므로 제한된 데이터셋에 대한 메타데이터를 숨기는 것 자체가 중요하다.

물리적 저장소 권한(Physical Storage Permissions)과 논리적 데이터 권한(Logical Data Permissions)은 서로 일관되게 구성되어야 한다. 질의 엔진에서 행 수준 제한을 적용하더라도 사용자가 저장소의 원본 객체 파일을 직접 다운로드할 수 있다면 논리적 정책을 우회할 수 있다. 따라서 민감한 테이블은 허가되지 않은 직접 접근을 차단할 수 있는 저장 아키텍처와 아이덴티티 정책을 사용해야 한다. 거버넌스가 적용된 질의 경로(Governed Query Paths), 카탈로그 인가(Catalog Authorization), 객체 저장소 권한(Object-Store Permissions)이 일관된 보안 경계(Security Boundary)를 형성해야 한다.

암호화(Encryption)는 인가 제어만으로 충분하지 않은 상황에서 데이터를 보호한다. 데이터는 일반적으로 안전한 통신 프로토콜을 이용하여 전송 중 암호화(Encryption in Transit)를 적용하고 적절한 저장 메커니즘을 이용하여 저장 시 암호화(Encryption at Rest)를 적용해야 한다. 암호화 키(Encryption Keys)는 통제된 수명주기 관리(Lifecycle Management), 접근 정책, 교체(Rotation), 모니터링, 복구 절차가 필요하다. 허가된 처리 서비스가 데이터를 복호화할 수 있더라도 어떤 레코드를 노출할 수 있는지에 대한 제한은 여전히 필요하므로 암호화가 인가를 대체하지는 않는다.

클라우드 자격 증명(Cloud Credentials), 데이터베이스 비밀번호(Database Passwords), API 토큰(API Tokens), 인증서(Certificates), 암호화 키와 같은 비밀정보(Secrets)는 로봇 로그, 노트북(Notebooks), 소스 코드 또는 데이터 처리 스크립트에 포함해서는 안 된다. 비밀정보 관리 시스템(Secret-Management Systems)은 자격 증명의 노출을 제한하면서 발급과 교체를 수행할 수 있다. 단기 자격 증명(Short-Lived Credentials)과 워크로드 아이덴티티(Workload Identities)는 개발 컴퓨터, 로봇, 공유 컴퓨팅 환경에 복사될 경우 특히 위험한 장기 정적 키(Long-Lived Static Keys)에 대한 의존성을 더욱 줄여준다.

감사 로깅(Audit Logging)은 보호된 정보가 어떻게 사용되고 있는지를 확인할 수 있는 증거를 제공한다. 보안 로그(Security Logs)는 중요한 인증 이벤트(Authentication Events), 권한 변경(Permission Changes), 데이터 접근(Data Accesses), 정책 결정(Policy Decisions), 관리 작업(Administrative Actions), 지원되는 경우 데이터 내보내기(Exports)를 기록해야 한다. 민감한 로보틱스 데이터셋에서는 감사 추적(Audit Trails)을 통해 특정 정보에 누가 접근했는지, 언제 접근했는지, 어떤 서비스를 통해 접근했는지를 확인할 수 있다. 감사 로그 자체에도 민감한 운영 메타데이터가 포함될 수 있으므로 별도의 보호가 필요하다.

보안 모니터링(Security Monitoring)은 감사 이벤트를 단순히 저장하는 것에서 나아가 비정상적인 동작을 식별해야 한다. 반복적인 인가 실패(Authorization Failures), 예상보다 큰 규모의 데이터 내보내기, 비정상적인 아이덴티티의 접근, 민감한 정책 변경, 서비스의 정상 워크로드 범위를 벗어난 데이터셋 접근 등이 대표적인 예이다. 경고(Alert)와 조사 워크플로(Investigation Workflows)를 구축하면 비정상 이벤트가 더 큰 보안 사고로 발전하기 전에 대응할 수 있다. 과도한 오탐(False Alarms)을 줄이기 위해 정상적인 자동화 로보틱스 및 AI 워크로드의 특성을 기준선(Baseline)에 반영해야 한다.

AI 파이프라인(AI Pipelines)은 학습 과정에서 여러 출처의 정보를 결합하는 경우가 많기 때문에 추가적인 보안 고려사항이 필요하다. 데이터셋 구성(Dataset Construction) 과정에서는 접근 제한, 분류, 출처(Provenance), 승인된 사용 조건(Approved-Use Constraints)을 유지해야 한다. 파생 학습 테이블(Derived Training Table)을 생성했다고 해서 원본 데이터에 적용된 보호 정책이 자동으로 제거되어서는 안 된다. 또한 민감한 정보가 유지될 수 있는 데이터셋, 모델 아티팩트(Model Artifacts), 임베딩(Embeddings), 특징(Features), 중간 표현(Intermediate Representations)을 누가 외부로 내보낼 수 있는지도 통제해야 한다.

로봇과 엣지 장치(Edge Devices)는 플릿에 속한다는 이유만으로 신뢰해서는 안 되며 독립적인 보안 주체(Security Principals)로 취급해야 한다. 장치 아이덴티티(Device Identities), 인증서, 통제된 자격 증명, 안전한 통신, 제한된 서비스 권한을 사용하여 각 로봇이 업로드하거나 가져올 수 있는 데이터를 제한할 수 있다. 하나의 장치가 침해되더라도 조직 전체의 과거 플릿 데이터, 클라우드 저장소, 메타데이터 카탈로그 또는 모델 저장소(Model Repositories)에 무제한으로 접근할 수 없어야 한다.

네트워크 아키텍처(Network Architecture)는 추가적인 방어 계층(Defensive Layer)을 제공한다. 저장 서비스(Storage Services), 메타데이터 카탈로그, 처리 클러스터(Processing Clusters), 관리 인터페이스(Administrative Interfaces)를 신뢰 경계(Trust Boundaries)에 따라 분리하고 필요한 네트워크 경로를 통해서만 접근하도록 구성할 수 있다. 프라이빗 연결(Private Connectivity), 방화벽(Firewalls), 엔드포인트 제어(Endpoint Controls), 서비스 간 인증(Service-to-Service Authentication)은 불필요한 노출을 줄인다. 네트워크 제한은 아이덴티티 기반 인가(Identity-Based Authorization)를 보완하지만 세밀한 데이터 권한을 대체해서는 안 된다.

데이터 보존 및 삭제 정책(Data Retention and Deletion Policies)은 더 이상 필요하지 않은 정보도 계속 존재하는 한 위험을 발생시키기 때문에 보안의 일부이다. 원시 센서 아카이브(Raw Sensor Archives), 파생 데이터셋(Derived Datasets), 임시 처리 결과(Temporary Processing Outputs), 감사 기록(Audit Records), 모델 개발 아티팩트(Model-Development Artifacts)는 서로 다른 보존 기간이 필요할 수 있다. 또한 삭제 시 스냅샷(Snapshots), 백업(Backups), 복제본(Replicas), 캐시(Caches), 아카이브 사본(Archived Copies)까지 고려해야 하며, 보이는 테이블을 삭제한 것만으로 관련 데이터 전체가 사라졌다고 잘못 판단해서는 안 된다.

백업 및 복구 절차(Backup and Recovery Procedures)는 기존의 보안 속성을 유지해야 한다. 제한된 로봇 데이터가 포함된 백업에는 기본 데이터셋과 동일한 수준의 암호화, 접근 통제, 보존 정책, 감사 기능을 적용해야 한다. 복구 절차는 오래된 권한(Obsolete Permissions)을 실수로 다시 적용하거나 복구된 정보를 더 넓은 사용자 그룹에 노출해서는 안 된다. 따라서 보안 구성(Security Configuration)과 카탈로그 메타데이터도 기본 데이터와 함께 백업하고 복구 시험을 수행해야 할 수 있다.

성숙한 피지컬 AI 데이터 레이크(Physical AI Data Lake)는 보안을 하나의 권한 설정이 아니라 엔드투엔드 아키텍처(End-to-End Architecture)로 취급한다. 아이덴티티 관리(Identity Management)는 신뢰할 수 있는 보안 주체를 정의하고, 카탈로그는 자원을 분류하며, 행 및 열 수준 접근 제어(Row- and Column-Level Access Control)는 데이터 가시성을 제한한다. 저장소 정책은 우회를 방지하고, 암호화는 데이터를 보호하며, 감사 기능은 책임 추적성(Accountability)을 제공한다. 이러한 메커니즘을 결합하면 로봇 경험을 분석과 AI에 폭넓게 활용하면서도 운영 필요성, 민감도, 거버넌스 정책에 따라 접근 범위를 제한할 수 있다.

## 06.10 Robot AI Training Data Lake Build Case

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 AI 학습 데이터 레이크(Robot AI Training Data Lake)는 지속적으로 축적되는 로봇 경험(Robot Experience)을 재사용 가능한 학습 및 평가 자산(Training and Evaluation Assets)으로 변환하기 위해 구축된다. 단순히 파일을 보존하는 기존 아카이브(Archive)와 달리 센서 관측 데이터(Sensor Observations), 텔레메트리(Telemetry), 임무 맥락(Mission Context), 주석(Annotations), 시뮬레이션 결과(Simulation Outputs), 처리 이력(Processing History)을 체계적으로 구성하여 AI 엔지니어가 데이터셋을 탐색하고 재현할 수 있도록 한다. 이러한 설계는 배치된 로봇과 반복적인 데이터-모델 학습 순환(Data-to-Model Learning Cycle)을 연결한다.

구축 사례(Build Case)는 로보틱스 시스템 전체에서 데이터 생산자(Data Producers)를 식별하는 것부터 시작한다. 모바일 로봇(Mobile Robots), 매니퓰레이터(Manipulators), 자율주행 차량(Autonomous Vehicles), 엣지 컴퓨터(Edge Computers), 시뮬레이션 환경(Simulation Environments), 주석 도구(Annotation Tools), 유지보수 시스템(Maintenance Systems), 외부 데이터 소스(External Sources)가 모두 정보를 제공할 수 있다. 일반적인 관측 데이터에는 카메라 이미지, 비디오, 라이다(LiDAR), 레이더(Radar), 깊이(Depth), 오디오, 관성 측정 장치(IMU), 위성항법시스템(GNSS), 관절 상태(Joint States), 제어 명령(Control Commands), 내비게이션 상태(Navigation States), 시스템 로그, 임무 결과, 운영자 개입(Operator Interventions)이 포함된다.

데이터 획득(Data Acquisition)은 가능한 경우 원본 관측 데이터(Original Observation)를 보존해야 한다. 향후 AI 작업에서 현재의 전처리 과정이 제거한 정보가 필요할 수 있기 때문이다. 로봇은 먼저 고대역폭 데이터(High-Bandwidth Data)를 로컬 NVMe 저장소에 기록하면서 경량 텔레메트리(Lightweight Telemetry)를 지속적으로 스트리밍할 수 있다. 네트워크 연결이 가능한 경우 엣지 소프트웨어는 완료된 세션이나 선택된 이벤트를 중앙 저장소로 업로드한다. 체크섬(Checksums)과 전송 매니페스트(Transfer Manifests)를 사용하면 대용량 기록이 감지되지 않은 손상 없이 전송되었는지 검증할 수 있다.

시간 동기화(Time Synchronization)는 멀티모달 로봇 학습(Multimodal Robot Learning)의 기본 요소이다. 카메라 프레임(Camera Frames), 라이다 스캔(LiDAR Scans), IMU 샘플, 위치추정 결과(Localization Estimates), 제어 명령, 액추에이터 상태(Actuator States)는 신뢰할 수 있는 시간 기준과 연결되어야 한다. 데이터 수집 파이프라인은 캡처 타임스탬프(Capture Timestamps), 클록 도메인(Clock Domains), 동기화 상태(Synchronization Status), 타임스탬프 출처(Timestamp Provenance)를 보존해야 한다. 하드웨어 아키텍처가 지원하는 경우 정밀 시간 프로토콜(PTP), 하드웨어 타임스탬프(Hardware Timestamps) 또는 다른 동기화 메커니즘을 이용하여 데이터 정렬 정확도를 향상시킬 수 있다.

물리적 저장 계층(Physical Storage Layer)은 배치 요구사항에 따라 확장 가능한 객체 저장소(Object Storage), NAS 또는 분산 저장소(Distributed Storage)를 사용할 수 있다. 비디오, 포인트 클라우드(Point Clouds), ROS 백(ROS Bags), 원시 센서 시퀀스(Raw Sensor Sequences)와 같은 대용량 바이너리 객체(Large Binary Objects)는 변경 불가능하거나 엄격하게 버전 관리되는 파일로 유지한다. 구조화된 메타데이터와 텔레메트리는 파케이(Parquet)와 같은 컬럼 기반 형식(Columnar Formats)으로 저장할 수 있다. 대용량 객체와 검색 가능한 구조화 인덱스(Searchable Structured Indexes)를 분리하면 페타바이트 규모(Petabyte-Scale)의 아카이브를 더욱 효율적으로 질의하고 관리할 수 있다.

원시 영역(Raw Zone)은 최소한의 변환만 적용하여 원본 증거(Original Evidence)를 보존한다. 업로드된 각 세션에는 안정적인 식별자(Stable Identifiers)를 부여하고 로봇 식별정보(Robot Identity), 플랫폼 버전(Platform Version), 센서 구성(Sensor Configuration), 소프트웨어 릴리스(Software Release), 임무, 사이트, 타임스탬프, 저장 위치를 설명하는 메타데이터를 연결한다. 원시 영역은 파일을 임의로 수정하는 작업 공간으로 사용해서는 안 된다. 원본 관측 데이터를 보존하면 알고리즘이나 요구사항이 변경되었을 때 향후 파이프라인에서 데이터셋을 다시 생성할 수 있다.

검증 영역(Validated Zone)은 자동화된 품질 검사(Automated Quality Checks)를 수행한 이후 생성된다. 파이프라인은 손상된 파일(Corrupted Files), 누락된 센서 스트림(Missing Sensor Streams), 타임스탬프 불연속(Timestamp Discontinuities), 불완전한 전송(Incomplete Transfers), 유효하지 않은 스키마(Invalid Schemas), 보정 불일치(Calibration Mismatches), 비정상적인 기록 시간(Unexpected Recording Durations)을 탐지한다. 불완전한 데이터를 단순히 삭제하는 대신 품질 결과(Quality Results)를 메타데이터로 저장한다. 일반적인 학습에 적합하지 않은 기록이라도 장애 분석(Failure Analysis), 강건성 연구(Robustness Research), 이상 탐지(Anomaly Detection)에는 가치가 있을 수 있다.

표준화(Standardization)는 서로 다른 플릿 데이터(Fleet Data)를 일관된 표현으로 변환한다. 로봇 식별자, 센서 이름, 단위(Units), 좌표 프레임(Coordinate Frames), 타임스탬프, 임무 상태(Mission States), 이벤트 정의(Event Definitions)를 통제된 스키마(Controlled Schemas)에 따라 정규화한다. 보정 참조(Calibration References)와 소프트웨어 버전은 관측 데이터와 연결하여 엔지니어가 과거 데이터를 정확하게 해석할 수 있도록 한다. 표준화를 통해 여러 세대의 로봇 데이터를 공통 데이터셋 선택 및 분석 워크플로에서 사용할 수 있다.

메타데이터(Metadata)는 학습 데이터 레이크의 검색 가능한 인덱스(Searchable Index)가 된다. 엔지니어는 수백만 개의 센서 파일을 직접 스캔하는 대신 관측 데이터가 어디에서, 언제, 어떤 조건에서 수집되었는지를 설명하는 구조화된 정보를 질의한다. 검색 조건에는 로봇 유형(Robot Type), 센서 구성, 환경(Environment), 날씨(Weather), 임무 결과(Mission Outcome), 개입(Intervention), 이상 상태(Anomaly), 객체 클래스(Object Class), 품질 수준(Quality Level), 주석 상태(Annotation State), 소프트웨어 버전이 포함될 수 있다. 선택된 메타데이터는 이후 필요한 물리적 객체의 위치를 연결한다.

데이터 정제(Data Curation)는 어떤 경험을 학습 후보(Training Candidates)로 사용할 것인지 결정한다. 모든 데이터를 무작위로 저장한다고 해서 유용한 학습 데이터가 확보되는 것은 아니며, 반복되는 정상 운영 데이터가 드물지만 중요한 상황을 압도할 수 있다. 정제 파이프라인은 실패(Failures), 아차 사고(Near Misses), 비정상 객체(Unusual Objects), 위치추정 성능 저하(Localization Degradation), 어려운 조명 조건(Difficult Lighting), 희귀 환경(Rare Environments), 사람의 개입(Human Interventions), 모델 불확실성(Model Uncertainty)을 식별할 수 있다. 이러한 이벤트를 우선적으로 선택하면서 정상 운영 상황을 대표하는 샘플도 함께 유지할 수 있다.

주석 워크플로(Annotation Workflows)는 선택된 관측 데이터에 지도 학습 정보(Supervised Information)를 추가한다. AI 작업에 따라 바운딩 박스(Bounding Boxes), 분할 마스크(Segmentation Masks), 추적 정보(Tracks), 자세(Pose), 행동(Actions), 궤적(Trajectories), 장면 설명(Scene Descriptions), 객체 관계(Object Relationships), 성공 상태(Success States), 사람의 수정(Human Corrections)을 포함할 수 있다. 주석 버전(Annotation Versions), 주석 작업자 정보(Annotator Information), 품질 검토(Quality Reviews), 원본 관측 데이터는 서로 연결하여 레이블을 수정하더라도 학습 데이터셋이 생성된 이력을 유지해야 한다.

시뮬레이션 데이터(Simulation Data)는 희귀하거나 위험하거나 물리적으로 수집하기 어려운 시나리오에서 실제 데이터(Real-World Recordings)를 보완할 수 있다. 디지털 트윈(Digital Twins)과 합성 환경(Synthetic Environments)은 객체, 조명, 날씨, 기하 구조(Geometry), 센서 노이즈(Sensor Noise), 로봇 행동을 통제된 방식으로 변화시켜 데이터를 생성할 수 있다. 합성 데이터(Synthetic Data)는 메타데이터를 통해 명확하게 식별하여 실제 데이터와 시뮬레이션 데이터의 구성 비율을 측정하고 시뮬레이션 경험으로 학습한 모델이 실제 로봇으로 성공적으로 전이되는지를 평가할 수 있어야 한다.

데이터셋 생성(Dataset Generation)은 임의의 파일을 출처 정보 없이 임시 디렉터리에 복사하는 대신 변경 불가능하거나 버전 관리되는 매니페스트(Versioned Manifests)를 생성해야 한다. 매니페스트에는 선택된 관측 데이터, 객체 참조(Object References), 레이블, 스키마 버전, 품질 기준(Quality Criteria), 변환(Transformations), 학습-검증-시험 분할(Train-Validation-Test Assignments)을 기록할 수 있다. 이렇게 생성된 데이터셋 버전은 실험, 벤치마크(Benchmarks), 모델 카드(Model Cards), 배포 기록(Deployment Records)에서 참조할 수 있는 재현 가능한 엔지니어링 자산이 된다.

학습, 검증, 시험 데이터의 분리(Train, Validation, and Test Separation)는 데이터 상관관계(Correlation)를 신중하게 고려해야 한다. 동일한 임무에서 연속적으로 촬영된 프레임은 거의 동일할 수 있으므로 개별 이미지를 무작위로 분할하면 실제보다 과도하게 좋은 평가 결과를 만들 수 있다. 애플리케이션에 따라 세션(Session), 경로(Route), 사이트, 로봇, 환경 또는 시간 구간(Time Period)을 기준으로 데이터셋을 분리해야 할 수 있다. 메타데이터를 활용하면 파일 이름이나 수작업 구성에 의존하지 않고 이러한 제약조건을 체계적으로 적용할 수 있다.

데이터 레이크는 모든 중간 표현(Intermediate Representations)을 영구적으로 복제하지 않으면서 분산 전처리(Distributed Preprocessing)를 지원해야 한다. 컴퓨팅 클러스터(Compute Clusters)는 선택된 원본 객체에서 기록을 디코딩하고, 여러 센서 모달리티를 동기화하고, 크롭(Crops)을 생성하고, 포인트 클라우드를 변환하고, 특징(Features)을 계산하거나 모델 입력용 샘플(Model-Ready Samples)을 생성할 수 있다. 자주 재사용되는 출력은 구체화(Materialized)하고 캐시할 수 있으며, 임시 변환 결과는 필요할 때 다시 생성하여 저장 비용, 처리 시간, 재현성 사이의 균형을 맞출 수 있다.

모델 학습(Model Training)은 명시적으로 버전 관리된 데이터셋을 사용하고 해당 데이터셋의 식별정보를 코드(Code), 하이퍼파라미터(Hyperparameters), 소프트웨어 환경(Software Environments), 체크포인트(Checkpoints), 평가 결과(Evaluation Results)와 함께 기록한다. 실험 추적(Experiment Tracking)은 모델을 해당 모델을 생성하는 데 사용된 정확한 데이터와 연결한다. 이후 모델의 성능이 향상되거나 저하될 경우 엔지니어는 데이터셋 버전을 비교하여 그 차이가 아키텍처, 최적화(Optimization), 레이블, 데이터 구성(Data Composition), 전처리(Preprocessing) 중 어디에서 발생했는지를 분석할 수 있다.

평가 데이터(Evaluation Data)는 일반적인 학습 데이터보다 더 강력한 안정성이 필요하다. 안전 중요 시나리오(Safety-Critical Scenarios), 희귀 이벤트(Rare Events), 어려운 환경 조건(Difficult Environmental Conditions), 정상적인 플릿 운영을 대표하는 벤치마크 세트(Benchmark Sets)는 신중하게 버전 관리하고 의도하지 않은 학습 데이터 오염(Training Contamination)으로부터 보호해야 한다. 접근 제어와 데이터 계보(Lineage)를 이용하면 평가 샘플이 학습 파이프라인에 포함되었는지를 확인할 수 있다. 안정적인 벤치마크를 사용하면 여러 모델 세대 사이의 성능 변화를 일관된 기준으로 해석할 수 있다.

보안 및 거버넌스(Security and Governance)는 데이터 레이크 전체에 적용된다. 아이덴티티 기반 권한(Identity-Based Permissions)을 통해 정보를 업로드, 주석 처리, 질의, 수정, 내보내기 또는 삭제할 수 있는 사용자를 제어한다. 민감한 열이나 행에는 더욱 세밀한 접근 제한을 적용할 수 있으며, 암호화(Encryption)를 통해 전송 중 데이터와 저장 데이터를 보호한다. 로봇 기록에 고객, 시설, 지리 정보 또는 사람과 관련된 정보가 포함되는 경우 보존 정책(Retention Policies), 감사 로그(Audit Logs), 데이터셋 분류(Dataset Classifications), 통제된 공유(Controlled Sharing)가 특히 중요하다.

수명주기 관리(Lifecycle Management)는 학습 데이터 레이크가 통제 없이 계속 증가하는 것을 방지한다. 자주 사용하는 데이터셋과 최근 기록은 고성능 저장소(High-Performance Storage)에 유지하고 오래된 원시 아카이브는 저비용 저장 계층(Lower-Cost Storage Tiers)으로 이동할 수 있다. 중복 데이터나 임시 출력은 정책에 따라 제거할 수 있지만 삭제 과정에서는 데이터셋 매니페스트, 스냅샷(Snapshots), 실험, 법적 보존 요구사항(Legal Retention Requirements)을 고려해야 한다. 저장소 최적화(Storage Optimization)가 중요한 모델의 재현 가능성을 조용히 파괴해서는 안 된다.

가장 중요한 운영 패턴은 폐쇄형 학습 순환(Closed Learning Loop)이다. 배포된 모델은 예측(Predictions), 신뢰도(Confidence Measures), 실패, 개입, 새로운 관측 데이터를 생성한다. 이러한 신호는 데이터 레이크로 다시 전달되고 정제 시스템은 주석과 재학습(Retraining)에 가치가 높은 경험을 식별한다. 새로운 데이터셋 버전을 생성하고 모델을 학습 및 평가한 후 승인된 모델을 다시 플릿에 배포한다. 따라서 배포(Deployment)는 개발의 끝이 아니라 새로운 학습 데이터를 생성하는 또 하나의 단계가 된다.

성숙한 로봇 AI 학습 데이터 레이크(Robot AI Training Data Lake)는 피지컬 AI 시스템(Physical AI System)의 장기 기억(Long-Term Memory)이 된다. 원시 관측 데이터는 로봇이 경험한 것을 보존하고, 메타데이터는 그 맥락을 설명하며, 품질 파이프라인(Quality Pipelines)은 데이터의 신뢰성을 확보하고, 주석은 학습 신호(Learning Signals)를 추가하며, 버전 관리된 매니페스트는 재현 가능한 데이터셋을 정의한다. 학습과 평가는 축적된 경험을 향상된 모델로 변환하고, 배포된 로봇은 다시 새로운 증거를 지속적으로 제공함으로써 거버넌스가 적용된 플릿 규모 지능 향상 순환(Governed Cycle of Fleet-Scale Intelligence Improvement)을 형성한다.
