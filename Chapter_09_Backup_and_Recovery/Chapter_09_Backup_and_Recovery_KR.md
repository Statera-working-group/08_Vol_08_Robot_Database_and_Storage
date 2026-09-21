**Volume 08 Robot Database and Storage**

# 09. Backup and Recovery

## 09.01 Backup Strategy: Full, Incremental, Differential

![](images/image1.png){width="7.268055555555556in" height="7.268055555555556in"}

백업 전략(Backup Strategy)은 운영 데이터(Operational Data)의 복사본을 생성하고 보존하며, 기본 데이터(Primary Data)가 사용할 수 없거나 손상·삭제 또는 침해되었을 때 이를 복구하는 방법을 정의한다. 로봇 데이터베이스(Database) 및 스토리지(Storage) 환경에서는 기존 데이터베이스뿐만 아니라 센서 기록(Sensor Recordings), 지도(Maps), 구성 파일(Configuration Files), AI 데이터셋(AI Datasets), 객체 스토리지(Object Storage), 로봇 로그(Robot Logs), 시스템 메타데이터(System Metadata)까지 백업 계획에 포함해야 한다. 따라서 신뢰할 수 있는 전략은 복구 능력(Recovery Capability), 저장 공간 사용량(Storage Consumption), 백업 시간(Backup Duration), 네트워크 대역폭(Network Bandwidth), 운영 복잡성(Operational Complexity) 사이의 균형을 고려해야 한다.

전체 백업(Full Backup)은 선택된 모든 데이터를 특정 시점(Point in Time)에 완전하게 복사하기 때문에 가장 단순한 백업 모델(Backup Model)이다. 백업 범위(Backup Scope)에 포함된 모든 데이터베이스 파일(Database Files), 데이터셋 객체(Dataset Objects), 구성 레코드(Configuration Records) 및 기타 보호 대상은 이전 백업 이후 변경 여부와 관계없이 복사된다. 복구 과정에서는 일반적으로 선택한 전체 백업만으로 보호된 데이터셋을 재구성할 수 있으므로 복원 절차(Restoration Process)를 비교적 예측 가능하게 운영할 수 있다.

전체 백업(Full Backup)의 주요 장점은 운영의 단순성(Operational Simplicity)이다. 각각의 백업 세트(Backup Set)는 독립적인 복구 시점(Recovery Point)을 나타낼 수 있으므로 이전 백업 작업에 대한 의존성을 줄이고 복원 절차를 단순화한다. 이러한 특성은 예측 가능한 복구가 백업 크기 최소화보다 중요한 핵심 로봇 구성 저장소(Robot Configuration Repository), 데이터베이스 스냅샷(Database Snapshot), 지도 데이터베이스(Map Database), 소규모 운영 데이터셋(Operational Dataset)에 특히 유용하다. 또한 전체 백업은 이후 증분 또는 차등 백업을 위한 기준 복사본(Baseline Copy)을 제공한다.

전체 백업의 가장 큰 단점은 자원 소비(Resource Consumption)이다. 수 테라바이트(Terabyte) 규모의 로봇 데이터셋 전체를 반복적으로 복사하면 데이터의 일부만 변경되었더라도 상당한 저장 용량(Storage Capacity), 네트워크 대역폭(Network Bandwidth), 백업 시간(Backup Time)이 필요하다. 대규모 이미지 컬렉션(Image Collections), LiDAR 기록(LiDAR Recordings), MCAP 파일(MCAP Files), 시뮬레이션 데이터셋(Simulation Datasets), 파운데이션 모델 학습 저장소(Foundation-Model Training Repositories)에서는 이러한 문제가 특히 중요하다. 따라서 일반적으로 전체 백업의 실행 빈도를 낮추고 그 사이에는 보다 효율적인 백업 방식을 함께 사용한다.

증분 백업(Incremental Backup)은 가장 최근의 백업 작업 이후 변경된 데이터만 복사한다. 이때 직전 작업은 전체 백업(Full Backup)일 수도 있고 다른 증분 백업일 수도 있다. 예를 들어 일요일에 전체 백업을 생성했다면 월요일 증분 백업은 일요일 이후의 변경 사항을 저장하고, 화요일 증분 백업은 월요일 이후의 변경 사항만 저장한다. 수요일에는 다시 화요일 이후의 변경 사항만 저장한다. 이 방식은 기준 전체 백업 이후 보호 데이터가 어떻게 변화했는지를 나타내는 비교적 작은 백업 세트들의 연속적인 체인(Backup Chain)을 형성한다.

증분 백업은 비교적 작은 변경 집합(Change Set)만 저장하므로 일상적인 백업 시간, 저장 공간 요구량, 네트워크 트래픽(Network Traffic)을 크게 줄일 수 있다. 이러한 특성은 텔레메트리(Telemetry), 로그(Logs), 센서 관측 데이터(Sensor Observations), 데이터베이스 트랜잭션(Database Transactions), 운영 기록(Operational Records)을 지속적으로 생성하는 로봇 시스템에서 유용하다. 변경되지 않은 정보를 반복 전송하는 대신 새롭게 생성되거나 수정된 데이터에 자원을 집중하므로 전체 운영 데이터셋이 매우 큰 경우에도 높은 빈도의 데이터 보호가 가능하다.

그러나 증분 백업의 효율성은 복구 의존성 체인(Recovery Dependency Chain)을 발생시킨다. 수요일 상태를 복원하려면 최초의 전체 백업과 월요일, 화요일, 수요일의 증분 백업을 정확한 순서로 적용해야 할 수 있다. 필요한 백업 중 하나가 손실되거나 손상되면 완전한 복구가 어려워질 수 있다. 긴 증분 체인은 복원 시간(Restoration Time)과 운영 복잡성도 증가시킨다. 따라서 신뢰할 수 있는 백업 카탈로그(Backup Catalog), 무결성 검증(Integrity Verification), 보존 관리(Retention Management), 정기적인 신규 전체 백업 또는 합성 기준 백업(Synthetic Baseline Backup)이 필요하다.

차등 백업(Differential Backup)은 전체 백업과 증분 백업의 중간적인 방식을 제공한다. 차등 백업은 직전 백업 이후가 아니라 가장 최근의 전체 백업 이후 변경된 모든 데이터를 복사한다. 일요일에 전체 백업을 수행했다면 월요일 차등 백업에는 월요일 변경 사항이 저장된다. 화요일 차등 백업에는 월요일과 화요일의 누적 변경 사항이 저장되며, 수요일 차등 백업에는 일요일 이후 수요일까지 변경된 모든 내용이 포함된다. 따라서 새로운 전체 백업을 생성할 때까지 차등 백업의 크기는 점차 증가한다.

차등 백업의 복구는 긴 증분 백업 체인을 사용하는 것보다 일반적으로 단순하다. 수요일의 보호 상태를 복원하려면 보통 일요일 전체 백업과 수요일 차등 백업만 필요하다. 해당 복구 시점에서는 월요일과 화요일의 차등 백업이 필요하지 않다. 따라서 복원 과정에서 사용되는 백업 구성 요소의 수를 줄일 수 있으며 중간 백업 손실에 따른 복구 실패 가능성도 낮출 수 있다. 다만 백업 주기가 진행될수록 차등 백업의 크기가 증가하기 때문에 증분 백업보다 많은 저장 공간을 사용할 수 있다.

이러한 백업 방식의 실질적인 선택은 백업 윈도(Backup Window), 복구 요구사항(Recovery Requirements), 데이터 변경률(Change Rate), 데이터셋 크기(Dataset Size), 인프라 용량(Infrastructure Capacity)의 관계에 따라 결정된다. 전체 백업은 복원 단순성을 제공하지만 더 많은 백업 자원이 필요하고, 증분 백업은 빈번하고 저장 효율적인 보호에 적합하지만 긴 복구 체인이 발생한다. 차등 백업은 그 중간에 위치하며 증분 백업보다 많은 백업 용량을 요구하지만 일반적으로 복원이 더 단순하다. 따라서 하나의 방식이 모든 로봇 데이터 유형과 운영 환경에 최적인 것은 아니다.

로봇 시스템(Robot Systems)에서는 데이터의 복구 특성이 서로 크게 다르기 때문에 백업 방식 선택이 더욱 복잡해진다. PostgreSQL 운영 데이터베이스(Operational Database)는 지속적으로 변경될 수 있지만 보정된 지도(Calibrated Map)는 수주 동안 변경되지 않을 수 있다. 센서 아카이브(Sensor Archive)는 기존 파일을 수정하지 않고 새로운 데이터만 빠르게 증가할 수 있는 반면, 로봇 구성 저장소(Robot Configuration Repository)는 크기가 작더라도 운영상 매우 중요할 수 있다. AI 학습 데이터셋(AI Training Dataset)은 수백만 개의 객체를 포함하면서도 새로운 데이터셋 버전이 배포될 때만 변경될 수 있다. 따라서 백업 정책은 데이터 유형을 먼저 분류한 후 적절한 백업 방식을 할당해야 한다.

실용적인 백업 아키텍처(Backup Architecture)는 정기적인 전체 백업과 빈번한 증분 또는 차등 보호를 결합할 수 있다. 핵심 데이터베이스와 구성 저장소에는 빈번한 변경 기반 백업(Change-Based Backup)을 적용하고, 대규모 불변 센서 아카이브(Immutable Sensor Archive)는 새롭게 생성된 객체나 스토리지 수준 복제 정책(Storage-Level Replication Policy)에 따라 보호할 수 있다. 지도 릴리스(Map Releases)와 검증된 AI 데이터셋은 버전이 지정된 복구 시점(Versioned Recovery Point)으로 보존할 수 있다. 결과적으로 백업 아키텍처는 모든 스토리지 시스템에 하나의 일률적인 일정을 적용하기보다 각 데이터 클래스(Data Class)의 특성과 중요도를 반영해야 한다.

백업 빈도(Backup Frequency)는 백업 보존 정책(Backup Retention)과 구분해야 한다. 백업을 자주 생성하더라도 오래된 복구 시점을 즉시 삭제한다면 장기적인 복구 가능성을 보장할 수 없다. 보존 정책(Retention Policy)은 일간(Daily), 주간(Weekly), 월간(Monthly), 장기 보관용(Archival) 백업을 얼마나 유지할지를 결정한다. 손상, 실수에 의한 변경, 랜섬웨어(Ransomware), 애플리케이션 오류(Application Error)는 일정 시간이 지난 후 발견될 수 있으므로 여러 세대의 백업을 보존하는 것이 중요하다. 유지 중인 모든 백업에 이미 잘못된 변경이 포함되어 있다면 기술적으로 성공한 백업이라도 유용한 복구 시점을 제공하지 못할 수 있다.

백업 무결성(Backup Integrity)도 동일하게 중요하다. 백업 작업이 완료되었다는 사실은 데이터가 복사되었다는 것을 의미할 뿐, 실제로 정상적인 복원이 가능하다는 것을 보장하지 않는다. 백업 시스템은 체크섬(Checksum), 백업 카탈로그, 객체 수(Object Count), 데이터베이스 일관성(Database Consistency), 암호화 메타데이터(Encryption Metadata), 필요한 의존성 체인을 검증해야 한다. 정기적인 복원 테스트(Restoration Test)는 격리된 환경에서 데이터베이스, 파일 또는 데이터셋을 실제로 재구성하고 애플리케이션이 복원된 정보를 정상적으로 사용할 수 있는지 확인함으로써 보다 강력한 검증 수단을 제공한다.

백업의 저장 위치(Storage Location)는 백업 방식과 독립적으로 설계해야 한다. 운영 데이터와 동일한 물리 시스템에 저장된 전체, 증분 또는 차등 백업은 하드웨어 장애(Hardware Failure), 스토리지 손상(Storage Corruption), 도난(Theft), 파괴적인 관리 오류(Administrative Error)가 발생할 경우 운영 데이터와 함께 사라질 수 있다. 따라서 백업 아키텍처는 적절하게 분리된 스토리지 도메인(Storage Domain)을 사용하고 필요하면 지리적 또는 논리적으로 격리된 복사본을 유지해야 한다. 오프라인(Offline) 또는 불변형 보호(Immutable Protection)는 운영 시스템과 백업 시스템에 동시에 영향을 주는 장애나 공격에 대한 노출을 더욱 줄일 수 있다.

보안 제어(Security Controls)는 백업 데이터의 전체 수명주기(Lifecycle)를 보호해야 한다. 백업 저장소(Backup Repository)에는 데이터베이스 자격 증명(Database Credentials), 로봇 구성, 운영 이력, 지도, 센서 기록, 독점 AI 데이터셋(Proprietary AI Datasets)과 같은 민감한 정보가 포함될 수 있다. 따라서 전송 및 저장 데이터 암호화(Encryption in Transit and at Rest), 제한된 관리자 접근(Restricted Administrative Access), 자격 증명 분리(Credential Separation), 감사 로그(Audit Logging), 통제된 삭제(Controlled Deletion)를 백업 설계에 포함해야 한다. 복구 자격 증명과 암호화 키(Encryption Keys) 역시 재해 상황에서 사용할 수 있어야 하면서 비인가 접근의 경로가 되지 않도록 보호해야 한다.

로봇 플릿(Robot Fleet)과 스토리지 시스템의 규모가 확대될수록 자동화(Automation)는 필수 요소가 된다. 백업 소프트웨어는 작업 예약(Scheduling), 변경 데이터 식별, 보존 정책 관리, 백업 완료 검증, 장애 탐지, 복구 시점 상태 보고를 자동화할 수 있다. 모니터링(Monitoring)은 단순한 작업 성공 여부와 실제 데이터 보호 상태를 구분해야 하며, 예상 데이터가 정상적으로 백업되었는지와 최신 유효 복구 시점이 운영 목표를 충족하는지 확인해야 한다. 또한 백업 누락, 비정상적인 백업 크기, 무결성 오류, 저장소 용량 문제, 손상된 증분 의존성 체인에 대한 경고(Alert)가 필요하다.

궁극적으로 전체 백업(Full Backup), 증분 백업(Incremental Backup), 차등 백업(Differential Backup)은 서로 경쟁하는 기술이 아니라 상호 보완적인 백업 메커니즘(Backup Mechanisms)이다. 전체 백업은 신뢰할 수 있는 기준점(Baseline)을 제공하고, 증분 백업은 효율적인 고빈도 보호를 가능하게 하며, 차등 백업은 일정 수준의 저장 효율성을 유지하면서 복원 의존성을 줄인다. 견고한 로봇 스토리지 아키텍처(Robot Storage Architecture)는 데이터 특성과 복구 우선순위에 따라 이러한 방식을 조합하고, 보존 정책, 무결성 검증, 격리(Isolation), 보안 제어, 모니터링, 정기적인 복구 테스트를 함께 적용해야 한다.

## 09.02 3.2.1 Backup Rule and Robot System Application

![](images/image2.png){width="7.268055555555556in" height="7.268055555555556in"}

3-2-1 백업 규칙(3-2-1 Backup Rule)은 하드웨어 장애(Hardware Failure), 소프트웨어 손상(Software Corruption), 사용자 오류(Human Error), 사이버 공격(Cyberattack), 물리적 재해(Physical Disaster)로 인해 핵심 정보가 손실될 위험을 줄이기 위해 널리 사용되는 복원력 원칙(Resilience Principle)이다. 이 규칙은 중요한 데이터의 복사본을 최소 3개 유지하고, 이를 최소 2개의 서로 다른 저장 매체(Storage Media) 또는 독립적인 스토리지 시스템(Storage Systems)에 저장하며, 최소 1개의 복사본을 원격(Offsite) 또는 격리된 위치(Isolated Location)에 보관하도록 권장한다. 로봇 시스템에서는 이러한 원칙이 복구 가능한 데이터 인프라(Recoverable Data Infrastructure)를 설계하는 실용적인 기반이 된다.

첫 번째 요소인 "3"은 중요한 정보가 3개의 복사본으로 존재해야 한다는 의미이며,

여기에는 기본 운영 데이터(Primary Production Data)와 최소 2개의 추가 백업 복사본(Backup Copies)이 포함된다. 로봇 플릿(Robot Fleet)은 활성 운영 데이터베이스(Operational Database)를 운영 서버에 저장하면서 전용 백업 스토리지(Dedicated Backup Storage)에 로컬 백업(Local Backup)을 유지하고, 별도의 원격 환경(Remote Environment)에 또 하나의 보호된 복사본을 유지할 수 있다. 여러 복사본을 확보하면 단일 스토리지 장치, 서버, 파일시스템 또는 관리 작업에 대한 의존성을 줄이고 하나의 복사본을 사용할 수 없을 때 대체 복구 경로를 확보할 수 있다.

동일한 스토리지 시스템에 단순히 3개의 복사본을 생성하는 것만으로는 충분한 보호가 이루어지지 않는다. 운영 데이터베이스, 백업 디렉터리, 추가 복사 디렉터리가 모두 동일한 물리적 스토리지 어레이(Storage Array)에 존재한다면 컨트롤러 장애, 파일시스템 손상, 랜섬웨어(Ransomware) 공격 또는 파괴적인 관리 명령이 세 복사본 모두에 동시에 영향을 줄 수 있다. 따라서 3-2-1 규칙은 단순히 복제된 파일의 수를 계산하는 것이 아니라 장애 도메인 분리(Failure-Domain Separation)를 중요하게 다룬다. 각각의 복사본은 전체 복구 아키텍처(Recovery Architecture)에 실질적인 독립성을 제공해야 한다.

"2"는 최소 2개의 서로 다른 저장 매체(Storage Media) 또는 독립적인 스토리지 플랫폼(Storage Platforms)을 사용한다는 의미이다. 로봇 인프라에 따라 운영 데이터는 NVMe 또는 SSD 스토리지에 저장하고, 백업 복사본은 NAS(Network Attached Storage), 객체 스토리지(Object Storage), 전용 백업 장치(Dedicated Backup Appliance), 이동식 매체(Removable Media) 또는 독립적으로 관리되는 다른 스토리지 시스템에 저장할 수 있다. 목적은 공통적인 하드웨어, 소프트웨어 또는 관리 장애가 보호된 모든 데이터 복사본을 동시에 파괴하는 것을 방지하는 것이다.

현대적인 구현에서는 반드시 디스크와 테이프 같은 전통적인 저장 매체 조합을 요구하기보다 장애 독립성(Failure Independence)을 기준으로 스토리지 다양성(Storage Diversity)을 해석한다. 예를 들어 로컬 SSD 스토리지에서 운영되는 로봇 데이터베이스를 독립적인 NAS에 백업하고, 별도의 인프라 도메인(Infrastructure Domain)에서 관리되는 객체 스토리지로 다시 복제할 수 있다. 중요한 것은 백업 아키텍처가 운영 환경을 지원하는 동일한 호스트, 컨트롤러, 파일시스템, 자격 증명(Credentials), 관리 인터페이스 또는 물리적 장치에 지나치게 의존하지 않는 것이다.

"1"은 최소 하나의 백업 복사본을 기본 운영 환경에서 떨어진 원격 위치 또는 충분히 격리된 환경에 보관해야 한다는 의미이다. 원격 데이터센터(Remote Data Center), 클라우드 객체 스토리지(Cloud Object Storage), 지리적으로 분리된 시설 또는 안전하게 연결이 차단된 저장소(Disconnected Repository)를 활용할 수 있다. 화재, 침수, 도난, 전기적 손상, 사이트 전체의 스토리지 장애와 같은 물리적 사고가 발생할 경우 원격 분리는 특히 중요하다. 운영 서버 옆에 위치한 백업은 논리적인 데이터베이스 장애에는 대응할 수 있지만 모든 사이트 수준 재해(Site-Level Disaster)를 방어할 수는 없다.

격리(Isolation)는 랜섬웨어와 파괴적인 사이버 공격에 대한 보호에서도 점점 중요해지고 있다. 운영 환경과 동일한 관리자 자격 증명을 사용하여 항상 쓰기가 가능한 원격 백업(Remote Backup)은 공격에 함께 노출될 수 있다. 따라서 백업 아키텍처는 불변 객체 버전(Immutable Object Versions), 쓰기 보호 스냅샷(Write-Protected Snapshots), 오프라인 매체(Offline Media), 분리된 자격 증명, 제한된 삭제 권한 또는 논리적으로 격리된 백업 저장소를 통해 3-2-1 원칙을 강화할 수 있다. 목표는 운영 관리 영역이 침해되더라도 최소 하나의 복구 경로가 생존하도록 만드는 것이다.

로봇 시스템에는 3-2-1 규칙을 구현할 때 서로 다르게 다루어야 하는 여러 데이터 클래스(Data Classes)가 존재한다. 운영 데이터베이스에는 로봇 상태(Robot States), 작업 이력(Task Histories), 플릿 할당(Fleet Assignments), 충전 정보(Charging Information), 유지보수 기록(Maintenance Records), 임무 결과(Mission Results)가 포함될 수 있다. 파일 또는 객체 스토리지에는 ROS Bag, MCAP 기록, 카메라 이미지, LiDAR 포인트 클라우드(Point Clouds), 진단 로그(Diagnostic Logs), AI 학습 샘플(AI Training Samples)이 저장될 수 있다. 지도 데이터베이스, 보정 파라미터(Calibration Parameters), 로봇 구성 파일, 배포 패키지(Deployment Packages), AI 모델 아티팩트(Model Artifacts) 역시 저장 용량이 작더라도 중요한 보호 대상이 될 수 있다.

실내 또는 실외 자율이동로봇(AMR) 플릿에서는 기본 복사본을 로봇이 지속적으로 운영 정보를 교환하는 플릿 관리 서버(Fleet-Management Server) 또는 엣지 인프라(Edge Infrastructure)에 둘 수 있다. 두 번째 복사본은 외부 네트워크 연결에 의존하지 않고 빠르게 복원할 수 있도록 로컬 NAS 또는 전용 백업 스토리지에 유지할 수 있다. 세 번째 복사본은 원격 객체 스토리지(Remote Object Storage) 또는 물리적으로 분리된 다른 사이트로 전송할 수 있다. 이러한 아키텍처는 빠른 로컬 복구(Local Recovery)와 전체 운영 사이트에 영향을 미치는 장애에 대한 보호를 동시에 제공한다.

대규모 센서 데이터셋(Sensor Datasets)은 모든 객체를 반복적으로 복제할 경우 상당한 저장 공간과 네트워크 대역폭을 소비하기 때문에 조금 다른 구현 방식이 필요하다. 카메라 기록, LiDAR 스캔, 레이더 데이터(Radar Data), 멀티모달 로봇 데이터셋(Multimodal Robot Datasets)은 테라바이트(Terabyte) 또는 페타바이트(Petabyte) 규모에 이를 수 있다. 따라서 3-2-1 원칙을 수명주기 관리(Lifecycle Management), 증분 전송(Incremental Transfer), 중복 제거(Deduplication), 압축(Compression), 버전 관리(Versioning), 적절한 보존 정책(Retention Policies)과 결합해야 한다. 핵심 데이터셋에는 완전한 3개 복사본 보호를 적용하고, 임시 데이터나 재생성 가능한 중간 데이터에는 비용이 낮은 보존 정책을 적용할 수 있다.

AI 데이터셋(AI Datasets)과 피지컬 AI 학습 저장소(Physical AI Training Repositories)는 재현성(Reproducibility)이 단순히 원본 파일을 보존하는 것 이상에 의존하기 때문에 추가적인 요구사항이 존재한다. 데이터셋 버전(Dataset Versions), 어노테이션(Annotations), 메타데이터(Metadata), 체크섬(Checksums), 전처리 구성(Preprocessing Configurations), 학습 매니페스트(Training Manifests), 모델 연계 정보(Model Associations)를 함께 복구할 수 있어야 한다. 이미지의 여러 복사본을 유지하더라도 정확한 학습 데이터셋을 식별하는 메타데이터를 잃으면 운영적으로 불완전한 백업이 될 수 있다. 따라서 백업 범위는 실험을 재현하고 학습 파이프라인을 재구성하는 데 필요한 논리적 데이터셋 구조(Logical Dataset Structure)를 보존해야 한다.

로봇 지도(Robot Maps)와 구성 데이터(Configuration Data)는 일반적으로 센서 아카이브보다 크기가 작지만 단위 저장 용량당 운영 중요도는 훨씬 높을 수 있다. 내비게이션 지도(Navigation Maps), 시맨틱 지도(Semantic Maps), 지오펜스(Geofences), 도킹 위치(Docking Positions), 센서 보정 파라미터, 네트워크 구성(Network Configuration), 안전 설정(Safety Settings), 플릿 구성(Fleet Configuration)은 시스템 장애 이후 로봇이 다시 운용될 수 있는지를 결정할 수 있다. 이러한 데이터는 상대적으로 크기가 작아 저장 공간이나 네트워크에 큰 부담을 주지 않으면서 빈번한 버전 기반 백업(Versioned Backup)을 적용하기에 적합하다.

데이터베이스 보호(Database Protection)는 3-2-1 스토리지 원칙과 데이터베이스 인식형 복구 메커니즘(Database-Aware Recovery Mechanisms)을 결합해야 한다. 트랜잭션 일관성(Transaction Consistency)을 고려하지 않고 데이터베이스 파일만 복사하면 물리적으로 백업 파일은 존재하지만 신뢰성 있게 복원할 수 없는 상태가 될 수 있다. 따라서 데이터베이스 기술에 따라 데이터베이스 스냅샷(Database Snapshots), 트랜잭션 로그(Transaction Logs), 선행 기록 로그(Write-Ahead Logs), 일관성 있는 덤프(Consistent Dumps), 특정 시점 복구 데이터(Point-in-Time Recovery Data)를 포함해야 한다. 생성된 백업 세트는 독립적인 로컬 및 원격 스토리지에 분산하여 전체적인 복원력 요구사항을 충족할 수 있다.

백업 빈도(Backup Frequency)와 복사본 수(Number of Copies)는 서로 다른 문제를 해결한다. 백업을 드물게 수행한다면 3개의 복사본이 존재하더라도 충분히 최근의 복구 시점(Recovery Point)을 보장하지 못한다. 지속적으로 운영 데이터를 생성하는 로봇 시스템에서는 허용 가능한 데이터 손실량에 따라 백업 간격을 정의해야 한다. 빈번하게 변경되는 데이터베이스에는 연속적 또는 짧은 주기의 보호가 필요할 수 있지만 안정적인 지도나 배포된 AI 모델은 버전이 변경될 때를 중심으로 백업할 수 있다. 3-2-1 아키텍처가 위치적 다양성(Location Diversity)을 제공한다면 백업 스케줄링(Scheduling)은 시간적 복구 범위(Temporal Recovery Coverage)를 결정한다.

각 복사본을 어디에 저장할 것인지 결정할 때는 복구 속도(Recovery Speed)도 고려해야 한다. 원격 백업은 뛰어난 재해 보호 기능을 제공하지만 수 테라바이트의 로봇 데이터를 다시 가져오는 데 상당한 시간이 필요할 수 있다. 로컬 복구 복사본(Local Recovery Copy)을 유지하면 자주 필요한 데이터베이스, 구성 정보, 운영 파일을 빠르게 복원할 수 있고, 원격 복사본은 재해 수준의 보호를 담당한다. 따라서 백업 설계에서는 복원력뿐만 아니라 실제 복원 대역폭(Restoration Bandwidth), 데이터셋 크기, 서비스 의존성(Service Dependencies), 예상 복구 시간(Expected Recovery Time)을 함께 고려해야 한다.

3개의 복사본이 존재한다는 사실이 3개의 사용 가능한 복사본이 존재한다는 것을 의미하지 않으므로 무결성 검증(Integrity Verification)이 필요하다. 백업 작업에서는 체크섬, 파일 수(File Counts), 데이터베이스 일관성, 메타데이터, 암호화 정보(Encryption Information), 백업 카탈로그를 검증해야 한다. 자동화된 모니터링(Automated Monitoring)은 전송 실패, 불완전한 데이터셋, 손상된 아카이브(Corrupted Archives), 비정상적인 백업 크기, 누락된 복구 시점을 탐지할 수 있다. 이후 정기적인 복원 테스트(Restoration Tests)를 통해 보호된 복사본으로부터 로봇 데이터베이스, 지도, 구성, 센서 아카이브, AI 데이터셋을 실제로 재구성할 수 있는지 확인해야 한다.

가능한 경우 백업 도메인(Backup Domains) 간의 보안(Security) 역시 독립적으로 유지해야 한다. 운영 서버와 모든 백업 저장소가 동일한 특권 자격 증명(Privileged Credentials)을 공유한다면 하나의 계정이 침해되는 것만으로 전체 3-2-1 설계가 무력화될 수 있다. 분리된 관리자 역할(Administrative Roles), 제한된 서비스 계정(Service Accounts), 암호화(Encryption), 적용 가능한 환경에서의 다중요소 인증(Multifactor Authentication), 감사 로깅(Audit Logging), 삭제 보호(Deletion Protection), 통제된 키 관리(Key Management)를 통해 이러한 위험을 줄일 수 있다. 원격 및 불변 복사본은 공격자나 실수에 의한 작업이 운영 환경을 통해 쉽게 수정할 수 없을 때 가장 높은 보호 효과를 제공한다.

3-2-1 규칙은 완전한 재해 복구 솔루션(Disaster-Recovery Solution)이 아니라 아키텍처의 기본 원칙(Architectural Baseline)으로 이해해야 한다. 로봇 조직은 여전히 보존 정책, 복구 목표(Recovery Objectives), 백업 자동화(Backup Automation), 무결성 검증, 보안 제어, 복원 절차(Restoration Procedures), 정기적인 재해 복구 훈련(Disaster-Recovery Exercises)을 구축해야 한다. 이러한 메커니즘을 결합하면 3개의 복사본은 중복성(Redundancy)을 제공하고, 2개의 스토리지 도메인은 공통 원인 장애(Common-Mode Failure)를 줄이며, 1개의 격리 또는 원격 복사본은 사이트 수준 또는 관리 영역 전체의 재해로부터 데이터를 보호한다.

3-2-1 원칙을 올바르게 적용하면 로보틱스(Robotics) 및 피지컬 AI(Physical AI) 시스템을 위한 계층형 복구 아키텍처(Layered Recovery Architecture)를 구축할 수 있다. 운영 스토리지(Production Storage)는 정상적인 로봇 운용을 지원하고, 독립적인 로컬 백업 스토리지(Local Backup Storage)는 신속한 복구를 제공하며, 격리된 원격 스토리지(Isolated Remote Storage)는 심각한 사고가 발생했을 때 최종 복구 경로를 제공한다. 데이터베이스, 지도, 구성, 센서 아카이브, 로그, AI 데이터셋, 모델 아티팩트 전체에 이 규칙을 확장 적용하면 로봇 운영의 연속성(Operational Continuity)과 자율 시스템 개발에 필요한 장기 데이터 자산(Long-Term Data Assets)을 함께 보호할 수 있다.

## 09.03 PostgreSQL PITR Backup Configuration [w/Code]

![](images/image3.png){width="7.268055555555556in" height="7.268055555555556in"}

PostgreSQL 특정 시점 복구(Point-in-Time Recovery), 일반적으로 PITR로 줄여 부르는 이 기능은 기존 백업이 생성된 시점으로만 복원하는 것이 아니라 데이터베이스 클러스터(Database Cluster)를 특정 시점으로 복구할 수 있도록 하는 백업 및 복구 메커니즘(Backup and Recovery Mechanism)이다. PITR은 물리적 기본 백업(Physical Base Backup)과 보관된 선행 기록 로그(Write-Ahead Log, WAL)를 결합한다. PostgreSQL은 데이터베이스 변경 사항이 영구적으로 반영되기 전에 WAL에 기록하므로, 알려진 백업 상태에서 이러한 기록을 다시 재생하면 이후의 데이터베이스 활동을 재구성할 수 있다.

선행 기록 로깅(Write-Ahead Logging)은 PostgreSQL의 데이터 내구성(Durability)과 PITR의 핵심 기반이다. WAL 레코드(WAL Records)는 데이터베이스 페이지와 관련 구조의 변경 사항을 재현하는 데 필요한 저수준 변경 정보를 기록한다. 정상 운영 중 PostgreSQL은 순차적인 WAL 세그먼트 파일(WAL Segment Files)을 사용하여 pg_wal 디렉터리에 WAL을 기록한다. PITR은 완료된 WAL 세그먼트를 활성 데이터베이스 클러스터 외부에 보존함으로써 이 메커니즘을 확장하며, 원래 데이터베이스 스토리지가 손실되더라도 기본 백업 이후에 발생한 변경 사항을 사용할 수 있도록 한다.

따라서 PITR 백업 아키텍처(PITR Backup Architecture)는 서로 보완적인 두 가지 구성 요소를 포함한다. 기본 백업(Base Backup)은 데이터베이스 파일과 필요한 클러스터 메타데이터(Cluster Metadata)를 포함하여 PostgreSQL 클러스터의 일관된 물리적 상태를 저장한다. WAL 아카이빙(WAL Archiving)은 해당 백업 이후 생성되는 트랜잭션 이력(Transaction History)을 지속적으로 보존한다. 필요한 WAL 이력이 없는 기본 백업은 복구 범위를 백업 시점으로 제한하며, 유효한 시작 기본 백업이 없는 WAL 아카이브는 독립적으로 전체 클러스터를 재구성할 수 없다. 신뢰할 수 있는 PITR을 위해서는 두 구성 요소를 하나의 복구 체인(Recovery Chain)으로 관리해야 한다.

WAL 아카이빙은 PostgreSQL의 아카이브 모드(Archive Mode)를 활성화하고 완료된 WAL 세그먼트를 내구성 있는 백업 스토리지(Durable Backup Storage)에 복사하는 방법을 정의하는 것에서 시작한다. archive_mode 파라미터(Parameter)는 아카이빙을 활성화하며, archive_command 또는 적절한 아카이브 라이브러리(Archive Library)는 PostgreSQL이 완료된 각 세그먼트를 전송하는 방식을 결정한다. 아카이브 대상은 기본 데이터베이스 스토리지와 독립되어야 한다. 동일하게 장애가 발생한 디스크나 서버에 WAL을 저장하면 복구 설계의 의미가 약화되기 때문이다. 또한 아카이브 작업에서는 기존 WAL 파일을 실수로 덮어쓰지 않도록 해야 한다.

아카이브 프로세스(Archive Process)는 WAL 세그먼트가 안전하게 보존된 이후에만 성공 상태를 반환해야 한다. PostgreSQL은 성공적으로 아카이브되지 않은 WAL을 유지하므로 아카이브 메커니즘에 지속적인 장애가 발생하면 pg_wal의 사용 가능한 저장 공간이 결국 소진될 수 있다. 따라서 모니터링(Monitoring)은 아카이브 실패, 증가하는 아카이브 적체(Archive Backlog), 스토리지 용량 문제, 비정상적인 WAL 생성량, 대상 저장소의 사용 불가 상태를 탐지해야 한다. 로봇 플릿 데이터베이스(Robot Fleet Database)에서 지속적인 텔레메트리(Telemetry)나 작업 이벤트(Task Event)가 발생하면 상당한 WAL 트래픽이 생성될 수 있으므로 아카이브 처리량과 용량 계획이 운영상 중요하다.

물리적 기본 백업(Physical Base Backup)은 pg_basebackup과 같은 PostgreSQL 메커니즘을 사용하여 생성할 수 있다. PITR은 물리적 클러스터 수준에서 동작하므로 선택된 SQL 객체만이 아니라 전체 데이터베이스 클러스터를 백업해야 한다. 백업 절차에서는 PostgreSQL이 유효한 복구 시작점을 설정하는 데 필요한 파일과 메타데이터를 보존해야 한다. 관리자는 각각의 기본 백업을 올바른 WAL 아카이브와 연결할 수 있도록 백업 시간, PostgreSQL 버전, 클러스터 식별 정보(Cluster Identity), WAL 요구사항, 체크섬(Checksum) 정보, 보존 관계(Retention Relationships)를 기록해야 한다.

WAL이 지속적으로 아카이브되더라도 기본 백업은 주기적으로 생성해야 한다. 매우 오래된 하나의 기본 백업만 유지하면 복구 시 매우 많은 양의 WAL을 재생해야 하므로 복원 시간이 증가하고 장기간의 WAL 보존에 대한 의존성이 커질 수 있다. 기본 백업을 더 자주 생성하면 잠재적인 WAL 재생 구간을 줄일 수 있지만 추가적인 백업 용량이 필요하다. 따라서 적절한 백업 주기는 데이터베이스 크기, WAL 생성률(WAL Generation Rate), 복구 시간 목표(Recovery-Time Objectives), 사용 가능한 저장 공간, 네트워크 성능, 로봇 서비스의 운영 중요도에 따라 결정해야 한다.

복구(Recovery)는 적절한 기본 백업을 PostgreSQL 데이터 디렉터리(Data Directory)에 복원하고 서버가 아카이브된 WAL을 검색하도록 구성하는 과정에서 시작한다. PostgreSQL 복구 구성(Recovery Configuration)은 필요한 WAL 세그먼트를 아카이브 스토리지에서 가져오는 명령이나 메커니즘을 지정한다. 복구가 시작되면 PostgreSQL은 복원된 클러스터 상태를 읽고 WAL 레코드를 순차적으로 재생한다. 이 과정은 기본 백업 상태에서 원하는 복구 목표(Recovery Target) 방향으로 데이터베이스를 진행시키면서 백업 이후 기록된 커밋 변경 사항(Committed Changes)을 재구성한다.

PITR은 사용 가능한 모든 WAL 레코드를 끝까지 재생하는 대신 정의된 목표에 따라 복구를 중지할 수 있다. 복구 목표는 타임스탬프(Timestamp), 트랜잭션 식별자(Transaction Identifier), 명명된 복원 지점(Named Restore Point) 또는 기타 지원되는 복구 기준을 사용하여 지정할 수 있다. 이러한 기능은 실수에 의한 삭제, 잘못된 애플리케이션 업데이트, 파괴적인 트랜잭션 또는 손상된 운영 변경이 발생하기 직전의 시점으로 데이터베이스를 복구해야 할 때 유용하다. 다만 타임스탬프와 트랜잭션 경계(Transaction Boundary)가 사람이 예상하는 시점과 정확하게 일치하지 않을 수 있으므로 복구 목표를 신중하게 선택해야 한다.

명명된 복원 지점(Named Restore Points)은 잠재적으로 위험한 변경 작업을 계획할 때 운영 복구를 더욱 쉽게 만들 수 있다. 주요 데이터베이스 마이그레이션(Database Migration), 플릿 관리 소프트웨어 배포(Fleet-Management Software Deployment), 스키마 변경(Schema Modification), 관리 작업을 수행하기 전에 복원 지점을 생성하여 WAL 스트림(WAL Stream)의 알려진 위치를 표시할 수 있다. 이후 작업에서 문제가 발생하면 해당 복원 지점이나 다른 적절한 복구 위치를 PITR 대상으로 지정할 수 있다. 그러나 복원 지점은 단순한 WAL 마커(WAL Marker)이므로 백업을 대체하지 않으며, 여전히 유효한 기본 백업과 보존된 WAL 이력에 의존한다.

로봇 플릿 시스템(Robot Fleet Systems)은 운영 데이터베이스가 하루 동안 지속적으로 변경되기 때문에 PITR을 통해 상당한 이점을 얻을 수 있다. 로봇 작업 할당(Robot Task Assignments), 임무 상태(Mission States), 충전 이벤트(Charging Events), 유지보수 정보(Maintenance Information), 구성 변경(Configuration Changes), 사용자 작업(User Actions), 플릿 관리 기록(Fleet-Management Records)이 PostgreSQL에 지속적으로 커밋될 수 있다. 야간 백업만 사용하는 경우 장애 발생 시 수 시간의 운영 이력을 잃을 수 있지만, 지속적인 WAL 아카이빙을 사용하면 예약된 물리적 기본 백업 사이에서 발생하는 데이터베이스 변경 사항을 보존하여 잠재적인 복구 공백(Recovery Gap)을 줄일 수 있다.

그러나 PITR은 고가용성(High Availability) 및 복제(Replication)와 구분해야 한다. 스트리밍 복제본(Streaming Replica)은 기본 데이터베이스 서버 장애 시 빠른 장애 조치(Failover)를 제공할 수 있지만 기본 서버에서 발생한 실수에 의한 삭제나 애플리케이션 수준의 손상도 그대로 복제할 수 있다. PITR은 시간에 따른 복구 가능한 변경 기록을 보존하여 과거 상태로 복구할 수 있도록 한다. 따라서 운영 로봇 시스템에서는 하나의 메커니즘이 다른 하나를 완전히 대체한다고 보기보다 서비스 연속성을 위한 복제와 과거 상태 복원을 위한 PITR을 함께 사용할 수 있다.

아카이브된 WAL과 기본 백업은 서로 연계된 보존 관리(Retention Management)가 필요하다. WAL 세그먼트를 너무 일찍 삭제하면 유지 중인 기본 백업의 복구 체인이 손상될 수 있고, 모든 세그먼트를 무기한 보존하면 과도한 저장 공간을 소비할 수 있다. 백업 소프트웨어 또는 관리 절차는 WAL 파일을 삭제하기 전에 현재 사용 가능한 복구 시점을 위해 어떤 WAL 파일이 필요한지 판단해야 한다. 보존 설계에서는 여러 기본 백업, 복구 목표, 규정 준수 요구사항(Compliance Requirements), 재해 복구 복사본(Disaster-Recovery Copies), 손상이 장기간 발견되지 않을 가능성도 함께 고려해야 한다.

백업 스토리지(Backup Storage)는 운영 PostgreSQL 서버와 독립적으로 보호해야 한다. 기본 백업과 아카이브된 WAL은 전체 백업 아키텍처에 따라 전용 백업 스토리지, NAS, 객체 스토리지(Object Storage) 또는 원격 인프라(Remote Infrastructure)에 저장할 수 있다. 3-2-1 원칙(3-2-1 Principle)을 적용하면 여러 독립적인 장애 도메인(Failure Domains)에 복사본을 유지하여 PITR을 더욱 강화할 수 있다. 특히 원격 또는 불변 스토리지(Immutable Storage)는 데이터베이스 관리자, 랜섬웨어 또는 파괴적인 스크립트가 하나의 침해된 환경을 통해 운영 데이터와 모든 복구 이력을 동시에 삭제하지 못하도록 하는 데 유용하다.

PostgreSQL 백업에는 로봇 플릿의 전체 운영 이력이 포함될 수 있으므로 보안(Security)이 중요하다. 백업 저장소는 필요한 경우 암호화(Encryption), 제한된 서비스 자격 증명(Restricted Service Credentials), 통제된 파일시스템 또는 객체 권한, 감사 로깅(Audit Logging), 보호된 암호화 키(Encryption Keys)를 사용해야 한다. WAL 아카이빙을 수행하는 프로세스에는 백업 객체를 기록하는 데 필요한 최소한의 권한만 부여해야 한다. 물리적 기본 백업과 해당 WAL 파일을 확보하면 민감한 데이터베이스 내용에 접근할 수 있으므로 복구 접근 권한 역시 통제해야 한다.

무결성 검증(Integrity Verification)은 기본 백업과 WAL 아카이브 모두를 대상으로 해야 한다. 백업 작업이 성공한 것처럼 보여도 파일이 불완전하거나 손상되었거나 잘못 연결되어 있거나 실제 복구 시 접근할 수 없다면 충분하지 않다. 체크섬, 백업 매니페스트(Backup Manifest), WAL 연속성 검사(WAL Continuity Checks), 아카이브 모니터링, 스토리지 수준 무결성 메커니즘(Storage-Level Integrity Mechanisms)은 재해가 발생하기 전에 문제를 탐지하는 데 도움을 준다. PostgreSQL 백업 도구와 외부 백업 시스템을 모니터링하여 누락된 세그먼트나 손상된 백업 세트를 대체 복구 방법이 존재하는 동안 발견해야 한다.

정기적인 복구 테스트(Recovery Testing)는 PITR 구성의 가장 강력한 검증 방법이다. 테스트 환경(Test Environment)에서 선택된 기본 백업을 복원하고, 아카이브된 WAL을 가져와 정의된 복구 목표까지 변경 사항을 재생한 후 운영 환경에 영향을 주지 않고 PostgreSQL을 시작할 수 있다. 이후 관리자는 데이터베이스 일관성, 예상 테이블, 애플리케이션 연결성(Application Connectivity), 로봇 플릿 기록, 실제 복원 소요 시간을 확인할 수 있다. 이러한 훈련은 일반적인 백업 성공 메시지만으로는 발견할 수 없는 구성 오류를 식별하고 복구 절차가 실제로 동작한다는 근거를 제공한다.

자동화(Automation)는 기본 백업 스케줄링(Base-Backup Scheduling), WAL 아카이빙, 보존 관리, 모니터링, 무결성 검사, 복구 문서화(Recovery Documentation)를 통합해야 한다. 경고(Alert)는 아카이브 실패, 누락된 WAL 세그먼트, 오래된 기본 백업(Stale Base Backups), 예상하지 못한 WAL 증가, 부족한 저장소 용량, 실패한 검증 작업을 보고해야 한다. 지속적으로 운영되는 로봇 시스템에서는 백업 상태(Backup Health)를 가끔 수행하는 관리 작업이 아니라 관찰 가능한 운영 서비스(Observable Production Service)로 다루어야 한다. 복구 준비 상태(Recovery Readiness)는 전체 백업 체인이 계속 유효하다는 지속적인 증거에 의존한다.

잘 설계된 PostgreSQL PITR 구성은 서로 분리된 백업 파일의 집합이 아니라 연속적인 복구 타임라인(Continuous Recovery Timeline)을 형성한다. 주기적인 물리적 기본 백업은 신뢰할 수 있는 시작 상태를 제공하고, 아카이브된 WAL은 이후 변경 사항을 보존하며, 복구 목표는 원하는 과거 시점을 지정하고, 보호된 스토리지는 전체 복구 체인을 인프라 장애로부터 보호한다. 복제, 독립적인 백업 도메인, 모니터링, 보존 관리, 정기적인 복원 테스트와 결합된 PITR은 PostgreSQL 기반 로보틱스(Robotics) 및 피지컬 AI(Physical AI) 서비스를 위한 견고한 복구 기반을 제공한다.

## 09.04 MongoDB Backup: Atlas Backup / mongodump [w/Code]

![](images/image4.png){width="7.268055555555556in" height="7.268055555555556in"}

MongoDB 백업 설계(MongoDB Backup Design)는 장애 또는 실수에 의한 변경 이후 운영 데이터베이스를 복원하는 데 필요한 문서 데이터(Document Data), 인덱스(Indexes), 메타데이터(Metadata), 일관성 관계(Consistency Relationships)를 보호해야 한다. 대표적인 두 가지 접근 방식은 관리형 MongoDB Atlas 백업(Managed MongoDB Atlas Backup)과 mongodump 같은 MongoDB 데이터베이스 도구(MongoDB Database Tools)를 이용한 논리적 백업(Logical Backup)이다. 두 방식은 목적이 일부 겹치지만 운영 모델은 크게 다르다. Atlas는 클라우드 데이터베이스 인프라와 통합된 관리형 백업 기능을 제공하고, mongodump는 데이터베이스 내용을 이식 가능한 BSON 기반 백업 파일로 내보낸다.

MongoDB Atlas 백업(MongoDB Atlas Backup)은 Atlas 관리형 플랫폼에서 운영되는 클러스터를 위해 설계되었다. 백업 구성은 클러스터와 연결되며 관리자가 모든 데이터베이스를 수동으로 내보내지 않아도 예약된 복구 시점(Recovery Points)을 제공할 수 있다. Atlas 배포 및 백업 구성에 따라 스냅샷(Snapshots)을 정의된 정책에 따라 보존하고 보호된 클러스터 데이터를 복원하는 데 사용할 수 있다. 이러한 방식은 관리형 MongoDB 환경에서 별도의 백업 메커니즘을 구축하고 유지하는 데 필요한 운영 작업을 줄여준다.

스냅샷 기반 백업(Snapshot-Based Backup)은 정의된 복구 시점의 데이터베이스 스토리지를 캡처하며, 반복적인 내보내기 비용이 큰 대규모 컬렉션(Collections)을 포함하는 데이터베이스에서 특히 유용하다. 매번 모든 문서를 논리적으로 읽어 새로운 BSON 내보내기를 생성하는 대신 관리형 인프라에서 스냅샷 기반 복구 데이터를 유지할 수 있다. 관리자는 특정 Atlas 클러스터 구성에서 제공되는 스냅샷 주기(Snapshot Frequency), 보존 기간(Retention Periods), 저장 위치(Storage Location), 복원 절차(Restore Procedures), 복구 기능(Recovery Capabilities)을 이해해야 한다.

연속 백업 기능(Continuous Backup Capabilities)은 예약된 스냅샷 사이의 데이터베이스 변경 사항을 보존하여 사용 가능한 복구 시점 사이의 간격을 더욱 줄일 수 있다. 이는 작업 이력(Task Histories), 임무 이벤트(Mission Events), 장치 상태(Device States), 구성 업데이트(Configuration Updates), 유지보수 기록(Maintenance Records), 운영자 작업(Operator Actions)이 지속적으로 변경되는 로봇 데이터베이스에서 유용하다. 백업 전략은 사고 발생 후 어느 정도의 최근 데이터 손실을 허용할 수 있는지 정의하고, 단순히 스냅샷이 존재한다는 이유만으로 충분한 보호가 이루어진다고 가정하기보다 해당 요구사항을 만족하는 복구 메커니즘을 선택해야 한다.

Atlas 백업은 인프라 관리를 단순화하지만 복구 준비 상태(Recovery Readiness)를 확보하려면 명확한 운영 계획이 필요하다. 관리자는 어떤 클러스터가 보호되고 있는지, 복구 시점이 얼마나 오래 보존되는지, 어떤 사용자가 복원을 시작할 수 있는지, 복원된 데이터가 어디에 생성되는지, 이후 애플리케이션이 어떻게 다시 연결되는지를 파악해야 한다. 클러스터 토폴로지(Cluster Topology), 데이터베이스 크기, 워크로드 특성(Workload Characteristics), 보안 요구사항 또는 로봇 플릿 운영 요구사항이 변경될 때마다 백업 정책을 검토해야 한다.

mongodump는 MongoDB 배포 환경에서 데이터를 읽고 데이터베이스 내용을 논리적인 형태로 생성하는 다른 백업 모델을 제공한다. 컬렉션 문서는 BSON 형식(BSON Format)으로 기록되며 관련 메타데이터도 이후 복원을 위해 함께 보존할 수 있다. 생성된 백업 디렉터리 또는 아카이브(Archive)는 NAS, 객체 스토리지(Object Storage), 이동식 매체(Removable Media), 원격 백업 인프라(Remote Backup Infrastructure)로 복사할 수 있다. 따라서 mongodump는 관리자가 직접 통제할 수 있는 이식 가능한 데이터베이스 수준 백업 아티팩트(Backup Artifacts)가 필요한 경우 유용하다.

mongodump 작업은 전체 MongoDB 배포 환경, 선택된 데이터베이스 또는 백업 요구사항에 따라 더욱 구체적인 데이터를 대상으로 할 수 있다. 이러한 유연성은 일부 데이터베이스에는 핵심 플릿 운영 정보가 포함되고 다른 데이터베이스에는 임시 분석 데이터 또는 재생성 가능한 중간 정보가 저장되는 로보틱스 시스템에서 유용하다. 선택적 논리 백업(Selective Logical Backup)은 백업 크기를 줄일 수 있지만, 관리자는 의도한 애플리케이션 상태를 재구성하는 데 필요한 모든 컬렉션과 메타데이터가 포함되었는지 확인해야 한다.

이에 대응하는 mongorestore 유틸리티(Utility)는 mongodump로 생성된 백업에서 MongoDB 데이터를 재구성한다. 복원은 재해 복구(Disaster Recovery), 마이그레이션(Migration), 테스트 또는 검증을 위해 적절한 MongoDB 환경으로 수행할 수 있다. 논리적 복원(Logical Restoration)은 단순히 물리적 스토리지 이미지를 마운트하는 것이 아니라 내보낸 BSON으로부터 데이터를 다시 생성하므로 복구 성능은 데이터베이스 크기, 문서 수(Document Count), 인덱스, 스토리지 처리량(Storage Throughput), CPU 자원, 대상 배포 구성에 영향을 받는다.

지속적으로 쓰기 작업이 발생하는 데이터베이스에서 mongodump를 실행할 경우 일관성(Consistency)이 중요해진다. 로봇 플릿 데이터베이스에서는 백업이 수행되는 동안에도 작업 할당, 텔레메트리 요약(Telemetry Summaries), 충전 이벤트, 임무 상태 전환(Mission Transitions), 운영자 명령(Operator Commands)을 처리할 수 있다. 적절한 일관성 전략 없이 관련 컬렉션이 서로 다른 논리적 시점에 캡처되면 생성된 백업이 복구 과정에서 기대하는 정확한 애플리케이션 상태를 나타내지 못할 수 있다. 따라서 백업 계획에서는 쓰기 작업과 배포 토폴로지를 함께 고려해야 한다.

복제 세트(Replica Sets)는 백업이 운영 환경에 미치는 영향을 제어할 수 있는 추가적인 방법을 제공한다. 애플리케이션에서 요구하는 일관성을 유지하면서 운영 워크로드의 중단을 최소화하도록 백업 작업을 설계할 수 있다. 그러나 다른 복제 세트 구성원(Replica-Set Member)을 백업 소스로 사용하는 것만으로 모든 백업이 자동으로 논리적 정합성을 확보하는 것은 아니다. 논리 백업을 어디에서 어떤 방식으로 생성할지 결정할 때 복제 지연(Replication Lag), 읽기 선호도(Read Preferences), 진행 중인 트랜잭션, 애플리케이션 수준의 데이터 관계를 고려해야 한다.

대규모 MongoDB 데이터베이스에서는 범용 백업 메커니즘으로서 mongodump의 중요한 한계가 나타난다. 수백 기가바이트 또는 수 테라바이트의 BSON 데이터를 내보내고 이후 다시 복원하려면 상당한 시간, CPU, 스토리지 대역폭, 임시 저장 용량이 필요할 수 있다. 지속적으로 운영되는 대규모 클러스터에서는 관리형 스냅샷(Managed Snapshots)이나 전문 백업 시스템이 더욱 실용적인 복구 특성을 제공할 수 있다. mongodump는 소규모 데이터베이스, 선택적 내보내기, 마이그레이션, 개발용 복사본, 추가적인 독립 백업 계층에서 여전히 유용하다.

압축(Compression)과 아카이브 형식(Archive Formats)을 사용하면 mongodump 출력 관리가 단순해질 수 있다. 개별 컬렉션 파일이 포함된 대규모 디렉터리 구조를 유지하는 대신 관리하기 쉬운 백업 아티팩트를 생성하여 보호된 스토리지로 전송할 수 있다. 데이터의 압축 효율이 높은 경우 압축을 통해 저장 공간과 네트워크 요구량을 줄일 수 있지만 추가적인 처리 부하가 발생한다. 따라서 백업 설계에서는 전송 대역폭(Transfer Bandwidth), 백업 윈도(Backup Window), 사용 가능한 CPU 용량, 복원 속도, 장기 저장 비용을 함께 평가해야 한다.

로봇 시스템에서는 문서 모델(Document Model)이 변화하는 스키마와 이질적인 레코드를 수용할 수 있기 때문에 반정형 운영 정보(Semi-Structured Operational Information)를 MongoDB에 저장하는 경우가 많다. 매니퓰레이터(Manipulator) 애플리케이션은 작업 정의(Task Definitions), 실행 이력(Execution Histories), 탐지 객체(Detected Objects), 파지 결과(Grasp Results), 오류 이벤트(Error Events), 상황 메타데이터(Contextual Metadata)를 관련 컬렉션에 저장할 수 있다. AMR 서비스는 임무, 로봇 상태, 충전 이력, 사용자 명령, 구성 문서를 저장할 수 있다. 백업 범위는 애플리케이션 맥락 없이 개별 컬렉션만 보호하기보다 이러한 관계를 재구성하는 데 필요한 정보를 보존해야 한다.

백업 스토리지(Backup Storage)는 MongoDB 운영 환경과 독립적으로 유지해야 한다. 활성 데이터베이스와 동일한 서버 및 파일시스템에 저장된 mongodump 아카이브는 스토리지 장애, 랜섬웨어 사고 또는 파괴적인 관리 작업이 발생할 경우 함께 손실될 수 있다. 따라서 논리적 백업은 전용 NAS, 객체 스토리지, 원격 인프라 또는 다른 보호 도메인(Protected Domain)으로 전송해야 한다. Atlas 백업 역시 유일한 데이터 보호 수단으로 간주하기보다 조직의 전체 재해 복구 전략(Disaster-Recovery Strategy)에 통합해야 한다.

3-2-1 원칙(3-2-1 Principle)은 여러 백업 메커니즘과 스토리지 도메인을 결합하여 MongoDB 보호를 강화할 수 있다. 운영 MongoDB 클러스터가 활성 복사본(Active Copy)을 구성하고, 독립적인 로컬 백업(Local Backup)은 빠른 복원을 지원하며, 또 다른 보호된 복사본은 원격 또는 격리 스토리지(Isolated Storage)에 저장할 수 있다. 서로 다른 복구 시나리오에서 두 방식이 모두 필요한 경우 Atlas 스냅샷과 mongodump 아카이브를 상호 보완적으로 사용하여 관리형 복구 시점과 독립적으로 보존할 수 있는 이식 가능한 논리적 복사본을 함께 확보할 수 있다.

논리적 덤프(Logical Dumps)는 운영 서버의 접근 제어 경계(Access-Control Boundary) 외부에서도 데이터베이스 내용을 노출할 수 있으므로 보안 제어(Security Controls)를 통해 MongoDB 백업 데이터를 보호해야 한다. 백업 자격 증명(Backup Credentials)에는 필요한 권한만 부여하고, 전송 채널(Transfer Channels)을 보호하며, 백업 저장소에는 제한된 접근 제어를 적용하고, 필요한 경우 저장된 백업 아티팩트를 암호화해야 한다. 암호화 키(Encryption Keys), 서비스 자격 증명(Service Credentials), 삭제 권한(Deletion Permissions), 감사 기록(Audit Records)을 적절하게 관리하여 백업 인프라가 민감한 로봇 운영 데이터에 접근하기 쉬운 경로가 되지 않도록 해야 한다.

보존 정책(Retention Policy)은 Atlas 복구 시점과 mongodump 아카이브가 얼마나 오랫동안 유용하게 유지되는지를 결정한다. 최신 백업 하나만 유지하면 손상이나 원하지 않는 변경 사항이 며칠 후 발견되는 상황에 충분히 대응하기 어렵다. 운영 요구사항과 스토리지 용량에 따라 일간(Daily), 주간(Weekly), 월간(Monthly) 또는 버전 기반(Version-Related) 복구 시점을 보존할 수 있다. 오래된 백업 삭제는 자동화할 수 있지만 새로운 백업이 검증되기 전에 필요한 재해 복구 복사본이 삭제되지 않도록 신중하게 관리해야 한다.

무결성 검증(Integrity Verification)은 백업 아티팩트가 완전하고 실제로 사용할 수 있는지 확인해야 한다. mongodump 아카이브는 전송 또는 장기 보존 전에 예상된 데이터베이스, 컬렉션, 메타데이터, 파일 크기, 체크섬(Checksums)을 검사할 수 있다. 관리형 백업 역시 성공적인 생성 여부와 사용 가능한 복구 시점을 모니터링해야 한다. 백업 상태가 성공으로 표시되는 것은 유용한 정보이지만 실제 복원을 수행하면 저장된 데이터를 정상적으로 동작하는 MongoDB 환경으로 재구성할 수 있는지 확인할 수 있으므로 더욱 강력한 검증 근거를 제공한다.

정기적인 복구 테스트(Recovery Tests)에서는 선택한 Atlas 백업 또는 mongodump 아카이브를 격리된 테스트 환경(Isolated Test Environment)에 복원해야 한다. 관리자는 문서 수, 인덱스, 컬렉션 구조(Collection Structures), 애플리케이션 쿼리(Application Queries), 로봇 작업 이력, 구성 데이터, 서비스 연결성(Service Connectivity)을 확인할 수 있다. 또한 복구 시간을 측정하면 데이터베이스 크기가 증가하더라도 현재 백업 방식이 운영 복구 목표(Operational Recovery Objectives)를 충족할 수 있는지 확인할 수 있다. 따라서 복구 테스트는 실제 장애가 발생한 후에만 수행하는 작업이 아니라 백업 수명주기(Backup Lifecycle)의 일부로 다루어야 한다.

견고한 MongoDB 전략에서는 Atlas Backup과 mongodump 중 하나를 모든 환경에서 우월한 방식으로 선택할 필요가 없다. Atlas 백업은 관리형 클러스터 보호와 운영상 편리한 복구에 적합하며, mongodump는 선택적 보호, 마이그레이션, 테스트, 독립적 보존에 유용한 이식 가능한 논리 백업을 제공한다. 로보틱스(Robotics) 및 피지컬 AI(Physical AI) 시스템에서는 적절한 MongoDB 백업 메커니즘을 격리된 스토리지, 보안 제어, 보존 관리, 모니터링, 검증된 복원 절차와 결합함으로써 데이터 손실에서 운영 복구까지 이어지는 더욱 신뢰성 높은 경로를 구축할 수 있다.

## 09.05 Object Storage Cross-Region Replication [w/Code]

![](images/image5.png){width="7.268055555555556in" height="7.268055555555556in"}

객체 스토리지 교차 리전 복제(Object Storage Cross-Region Replication)는 소스 스토리지 리전(Source Storage Region)의 객체를 지리적으로 분리된 대상 리전(Destination Region)으로 자동 복사하는 복원력 메커니즘(Resilience Mechanism)이다. 이는 리전 장애(Regional Outages), 인프라 장애(Infrastructure Failures), 우발적인 데이터 손실(Accidental Loss), 재해 상황(Disaster Scenarios)으로부터 대규모 비정형 데이터(Unstructured Data)를 보호하기 위해 설계된다. 로보틱스(Robotics) 및 피지컬 AI(Physical AI) 시스템에서 복제 대상 객체에는 센서 기록, ROS Bag, MCAP 파일, 이미지, LiDAR 포인트 클라우드(Point Clouds), 지도, AI 데이터셋, 모델 아티팩트(Model Artifacts), 로그, 시뮬레이션 결과 등이 포함될 수 있다.

객체 스토리지(Object Storage)는 기존 파일시스템 블록(Filesystem Blocks)이나 관계형 레코드(Relational Records)가 아니라 객체(Object) 단위로 정보를 구성한다. 각 객체는 일반적으로 데이터, 식별자 또는 키(Key), 관련 메타데이터(Metadata)를 포함한다. 이러한 아키텍처는 애플리케이션이 기존 디렉터리 기반 스토리지 인프라를 직접 관리하지 않고 개별 객체에 접근할 수 있기 때문에 매우 큰 규모의 로봇 생성 데이터(Robot-Generated Data)를 효과적으로 확장할 수 있다. 교차 리전 복제는 독립적인 지리적 스토리지 위치에 대응되는 객체 복사본을 유지함으로써 이 모델을 확장한다.

일반적인 복제 아키텍처(Replication Architecture)는 하나의 리전에 소스 버킷(Source Bucket)을 두고 다른 리전에 대상 버킷(Destination Bucket)을 구성한다. 복제 대상 객체가 소스에서 생성되거나 업데이트되면 스토리지 플랫폼이 변경을 감지하여 필요한 객체 데이터와 메타데이터를 대상 리전으로 비동기 전송한다. 애플리케이션은 정상 운영을 위해 계속 소스 리전을 사용하고, 원격 리전은 재해 복구(Disaster Recovery), 리전 마이그레이션(Regional Migration), 분석(Analytics), 장기 데이터 보호(Long-Term Data Protection)에 사용할 수 있는 보호된 복사본을 유지한다.

복제는 일반적으로 비동기 방식(Asynchronous Replication)으로 수행된다. 지리적으로 멀리 떨어진 리전 간에 동기식 전송(Synchronous Transfer)을 수행하면 정상적인 객체 쓰기에 네트워크 지연(Network Latency)이 추가될 수 있기 때문이다. 따라서 새롭게 생성된 로봇 센서 파일이 대상 리전에 복제되기 전에 소스 리전에 먼저 존재하는 시간이 발생할 수 있다. 이러한 간격을 복제 지연(Replication Lag)이라고 한다. 재해 복구 계획에서는 교차 리전 복제가 지리적 복원력을 크게 향상시키지만 즉각적인 리전 장애에서 반드시 데이터 손실 제로(Zero Data Loss)를 보장하는 것은 아니라는 점을 고려해야 한다.

복제 규칙(Replication Rules)은 어떤 객체를 복사하고 복제 프로세스가 어떻게 동작할지를 결정한다. 정책은 전체 버킷에 적용하거나 선택된 접두사(Prefixes), 태그(Tags) 또는 기타 지원되는 기준과 일치하는 객체에만 적용할 수 있다. 이를 통해 로보틱스 조직은 핵심 데이터와 임시 데이터를 구분할 수 있다. 검증된 지도, 임무 기록(Mission Records), 보정 파일(Calibration Files), 학습된 모델(Trained Models), 중요 데이터셋은 원격으로 복제하고, 임시 전처리 출력(Preprocessing Outputs), 캐시(Caches), 중간 시뮬레이션 파일 또는 재생성 가능한 아티팩트는 기본 리전에만 유지할 수 있다.

버전 관리(Versioning)는 객체를 덮어쓰거나 삭제할 때 원하지 않는 상태가 그대로 전파되는 문제를 줄일 수 있기 때문에 복제와 함께 사용하는 중요한 기능이다. 객체 버전 관리를 활성화하면 동일한 논리적 객체의 여러 버전을 유지할 수 있으므로 수정 이후에도 이전 콘텐츠를 복구할 수 있다. 어노테이션(Annotation), 전처리(Preprocessing), 검증(Validation), 모델 학습(Model Training) 단계를 거치며 변화하는 로봇 데이터셋에서는 버전 기반 스토리지를 통해 과거 데이터셋 상태와 해당 데이터를 사용한 실험 간의 관계도 보존할 수 있다.

삭제 동작(Deletion Behavior)은 의도적으로 구성해야 한다. 새롭게 생성된 객체를 복제하는 것은 분명한 이점이 있지만, 모든 삭제 작업을 자동으로 그대로 복제하면 운영자, 애플리케이션 또는 침해된 계정이 데이터를 잘못 삭제했을 때 재해 복구 가치가 감소할 수 있다. 객체 스토리지 플랫폼은 삭제 마커(Deletion Markers), 버전, 수명주기 만료(Lifecycle Expiration)를 처리하기 위한 다양한 메커니즘을 제공한다. 따라서 관리자는 삭제 이벤트를 대상 리전에 전파할 것인지 정의하고, 우발적 또는 악의적인 삭제가 발생하더라도 최소 하나의 보호된 복구 경로가 유지되도록 해야 한다.

교차 리전 복제는 백업(Backup)과 동일하지 않다. 복제는 주로 데이터의 다른 복사본을 유지하는 기능이며 구성된 규칙에 따라 원하지 않는 수정, 손상 또는 삭제도 함께 복제될 수 있다. 반면 백업은 과거 복구 시점(Historical Recovery Points), 보존 경계(Retention Boundaries), 현재 운영 상태와 독립된 불변 복사본(Immutable Copies)을 제공할 수 있다. 따라서 견고한 로봇 데이터 아키텍처에서는 지리적 가용성을 위한 복제와 과거 상태 복구를 위한 버전 관리, 스냅샷(Snapshots), 아카이브 복사본(Archival Copies), 불변 백업 스토리지를 함께 사용할 수 있다.

로봇 센서 아카이브(Robot Sensor Archives)의 복제 정책은 데이터 가치와 데이터 용량을 모두 고려해야 한다. 카메라, LiDAR, 레이더(Radar), 마이크 및 기타 센서로 구성된 플릿은 막대한 양의 정보를 생성할 수 있으므로 모든 데이터를 무조건 복제하면 높은 비용이 발생할 수 있다. 핵심 사고 기록(Critical Incident Recordings), 검증된 학습 샘플, 희귀 이벤트 데이터셋(Rare-Event Datasets), 규제 관련 증거 데이터는 즉각적인 원격 복제가 필요할 수 있지만, 반복적인 원시 데이터는 선택된 데이터만 복제하거나 오래된 객체를 저비용 스토리지 클래스(Storage Classes)로 이동하는 수명주기 정책을 적용할 수 있다.

피지컬 AI 데이터셋(Physical AI Datasets)은 하나의 데이터셋이 수백만 개의 객체와 어노테이션, 매니페스트(Manifests), 보정 데이터, 메타데이터, 데이터 출처 기록(Provenance Records)으로 구성될 수 있기 때문에 또 다른 과제를 제시한다. 원시 센서 파일만 복제하면 원격 복사본이 운영적으로 불완전할 수 있다. 복제 범위는 학습 또는 평가를 재현하는 데 필요한 완전한 논리적 데이터셋(Logical Dataset)을 보존해야 한다. 따라서 체크섬(Checksums), 매니페스트, 데이터셋 버전, 변환 기록(Transformation Records), 모델 연계 정보(Model Associations)도 보호 데이터 구조의 일부로 다루어야 한다.

지도(Maps)와 로봇 구성 아티팩트(Robot Configuration Artifacts)는 일반적으로 원시 센서 데이터셋보다 훨씬 작지만 즉각적인 운영 중요도는 더 높을 수 있다. 내비게이션 지도(Navigation Maps), 시맨틱 지도(Semantic Maps), 지오펜스(Geofences), 임무 정의(Mission Definitions), 보정 파일, 배포 패키지(Deployment Packages), AI 추론 모델(AI Inference Models)은 리전 서비스 장애 이후 로봇 운용을 복구하는 데 필요할 수 있다. 이러한 데이터는 상대적으로 크기가 작으므로 적극적인 교차 리전 복제를 적용하기에 적합하며 제한적인 대역폭 부담으로 재해 복구 환경에 최신 운영 자산을 유지할 수 있다.

대상 리전은 실질적으로 독립적인 장애 도메인(Failure Domain)을 구성해야 한다. 동일한 물리적 리전 안의 서로 다른 스토리지 시스템에 두 개의 복사본을 저장하면 일부 장치 장애에는 대응할 수 있지만 리전 전체의 네트워크, 전력, 제어 평면(Control Plane), 환경적 사고에 대해서는 동일한 수준의 복원력을 제공하지 못한다. 리전을 선택할 때는 지리적 분리(Geographic Separation), 서비스 가용성(Service Availability), 규제 요구사항(Regulatory Requirements), 네트워크 연결성, 데이터 주권(Data Sovereignty), 지연 시간, 조직의 재해 복구 아키텍처를 고려해야 한다.

보안 정책(Security Policies)은 하나의 침해된 관리 경로에 불필요하게 의존하지 않으면서 두 리전 전체에서 일관되게 관리되어야 한다. 복제를 수행하려면 대상 소스 객체를 읽고 대상 리전에 대응되는 객체를 생성할 수 있는 권한이 필요하다. 접근 권한은 최소 권한 원칙(Least-Privilege Principle)을 따라야 하며, 암호화(Encryption), 키 관리(Key Management), 감사 로깅(Audit Logging), 신원 제어(Identity Controls), 삭제 권한을 두 위치에 적절히 구성해야 한다. 대상 데이터가 복제본이라는 이유만으로 소스 데이터보다 낮은 수준으로 보호되어서는 안 된다.

암호화는 복제 아키텍처와 호환되도록 구성해야 한다. 인프라 요구사항에 따라 객체는 스토리지 플랫폼 관리형 키(Storage-Platform-Managed Keys) 또는 고객 관리형 키 관리 시스템(Customer-Controlled Key-Management Systems)을 사용하여 암호화할 수 있다. 고객 관리형 암호화를 사용하는 경우 복제 워크플로(Replication Workflow)에 소스 복호화(Source Decryption)와 대상 암호화(Destination Encryption)를 위한 명시적인 권한이 필요할 수 있다. 재해 복구 테스트에서는 기본 리전 또는 관련 서비스가 사용할 수 없는 상황에서도 대상 객체를 정상적으로 복호화할 수 있는지 확인해야 한다.

수명주기 관리(Lifecycle Management)는 지리적으로 복제된 객체 스토리지의 장기 비용을 제어하는 데 도움을 준다. 최근 로봇 데이터는 활성 분석을 위해 고성능 스토리지에 유지하고, 오래된 객체는 보존 정책에 따라 저비용 아카이브 클래스(Archival Classes)로 전환할 수 있다. 소스와 대상의 수명주기 규칙은 신중하게 조정해야 한다. 각 리전에서 객체를 독립적으로 만료시키면 복구 가능한 범위가 달라질 수 있기 때문이다. 따라서 스토리지 비용 절감과 과거 로봇 데이터를 복구 가능한 상태로 유지해야 하는 기간 사이의 균형을 고려해야 한다.

모니터링(Monitoring)은 대상 스토리지가 존재하는지만 확인하는 것이 아니라 복제 상태(Replication Health)에 대한 가시성을 제공해야 한다. 유용한 지표에는 대기 중인 객체(Pending Objects), 복제 실패, 복제 지연, 전송 데이터 용량, 권한 오류, 대상 저장 용량 또는 정책 문제가 포함된다. 복제되지 않은 센서 파일이 갑자기 증가한다면 네트워크 혼잡, 인증 실패(Authentication Failure), 구성 변경 또는 비정상적인 데이터 생성이 원인일 수 있다. 경고(Alert)를 통해 원격 복구 복사본이 크게 오래된 상태가 되기 전에 관리자가 대응할 수 있다.

수백만 개의 로봇 객체가 여러 리전에 분산되어 있는 환경에서는 무결성 검증(Integrity Verification)이 특히 중요하다. 객체 수(Object Counts), 체크섬, 메타데이터, 버전 식별자(Version Identifiers), 매니페스트, 복제 상태를 비교하여 누락되거나 일관되지 않은 데이터를 식별할 수 있다. 대규모 데이터셋은 수동 검사 대신 자동화된 인벤토리(Automated Inventory) 및 조정 프로세스(Reconciliation Processes)를 통해 검증해야 한다. 목표는 단순히 복제 작업이 실행되었다는 사실을 확인하는 것이 아니라 대상 리전에 논리적으로 완전하고 실제 사용 가능한 복구 복사본이 존재한다는 것을 입증하는 것이다.

재해 복구는 복제된 객체를 저장하는 것만으로 완성되지 않는다. 애플리케이션은 대상 버킷의 위치를 확인하고, 자격 증명을 획득하고, 암호화 키에 접근하며, 인덱스 또는 카탈로그(Catalogs)를 재구축하고, 로봇 서비스를 복구된 데이터에 다시 연결할 수 있어야 한다. 복구 절차에서는 기본 리전을 사용할 수 없을 때 보조 리전(Secondary Region)을 어떻게 운영 상태로 전환할지 정의해야 한다. 정기적인 복구 훈련을 통해 접근 경로, 권한, 데이터 완전성, 애플리케이션 구성, 핵심 로봇 서비스를 재개하는 데 실제로 필요한 시간을 검증할 수 있다.

원격 팀이나 컴퓨팅 환경에서 대규모 데이터셋에 접근해야 하는 경우 교차 리전 복제는 분산 분석(Distributed Analytics)과 AI 개발도 지원할 수 있다. GPU 학습 인프라(GPU Training Infrastructure)와 가까운 위치에 데이터셋 복사본을 두면 반복적인 장거리 데이터 전송을 줄이고 분석 워크로드를 운영 스토리지와 분리할 수 있다. 그러나 분석 편의성을 위해 재해 복구 제어를 약화시켜서는 안 된다. 복제된 운영 데이터, AI 작업용 복사본(Working AI Copies), 보호된 백업 버전은 목적과 보존 요구사항에 따라 명확하게 구분되어야 한다.

따라서 견고한 객체 스토리지 아키텍처(Object-Storage Architecture)는 교차 리전 복제를 버전 관리, 수명주기 관리, 보안 제어, 무결성 검증, 모니터링, 독립적인 백업 정책과 결합한다. 복제는 지리적 중복성(Geographic Redundancy)과 원격 복사본에 대한 빠른 접근을 제공하며, 과거 백업과 불변 보존(Immutable Retention)은 논리적 오류와 파괴적인 변경으로부터 데이터를 보호한다. 로보틱스 및 피지컬 AI에 이러한 계층형 접근 방식(Layered Approach)을 적용하면 대규모 센서 및 AI 데이터 자산을 보호하면서 인프라 장애와 데이터 수준 사고(Data-Level Incidents) 모두에 대한 복구 능력을 확보할 수 있다.

## 09.06 RTO / RPO Target Setting and Backup Policy Design

![](images/image6.png){width="7.268055555555556in" height="7.268055555555556in"}

복구 시간 목표(Recovery Time Objective, RTO)는 서비스 중단이 발생한 시점부터 필요한 비즈니스 또는 운영 기능이 복구될 때까지 허용할 수 있는 최대 시간을 정의한다. 이는 장애가 발생한 이후 시스템을 얼마나 빠르게 다시 사용할 수 있어야 하는가라는 실질적인 질문에 답한다. 로보틱스(Robotics) 인프라에서 RTO는 플릿 관리 데이터베이스(Fleet-Management Database), 임무 서버(Mission Server), 지도 저장소(Map Repository), 구성 서비스(Configuration Service), 객체 스토리지(Object Storage), AI 추론 서비스(AI Inference Service), 클라우드 및 엣지 시스템(Cloud and Edge Systems)에 적용될 수 있다.

복구 시점 목표(Recovery Point Objective, RPO)는 사고 발생 시점을 기준으로 과거 방향으로 측정했을 때 허용할 수 있는 최대 데이터 손실량을 정의한다. 데이터베이스의 RPO가 15분이라면 복구 아키텍처는 장애 시점보다 약 15분 이상 오래되지 않은 상태로 복구할 수 있어야 한다. 따라서 RPO는 백업 빈도(Backup Frequency), 트랜잭션 로그 보호(Transaction-Log Protection), 복제 설계(Replication Design), 스냅샷 주기(Snapshot Intervals), 연속 데이터 보호(Continuous Data Protection) 메커니즘에 영향을 준다.

RTO와 RPO는 복구의 서로 다른 차원을 설명하므로 동일한 개념으로 취급해서는 안 된다. RTO는 서비스 복원에 얼마나 많은 시간이 허용되는지를 의미하고, RPO는 최근 생성된 데이터 중 어느 정도까지 손실을 허용할 수 있는지를 의미한다. 최근 데이터를 자주 보호하지만 복원에 수 시간이 필요하다면 짧은 RPO와 긴 RTO를 가질 수 있다. 반대로 빠른 장애 조치(Failover)를 통해 짧은 RTO를 달성하더라도 과거 데이터 보호가 충분하지 않으면 허용할 수 없는 RPO가 발생할 수 있다.

목표 설정(Target Setting)은 사용 가능한 백업 기술보다 운영 영향(Operational Impact)에 대한 분석에서 시작해야 한다. 관리자는 각 서비스가 사용할 수 없게 되었을 때 어떤 일이 발생하고 최근 정보가 손실될 경우 어떤 결과가 발생하는지 파악해야 한다. 로봇 임무 데이터베이스(Robot Mission Database)는 재생성 가능한 시뮬레이션 출력(Simulation Output)을 보관하는 아카이브보다 엄격한 목표가 필요할 수 있다. 내비게이션 지도와 안전 관련 구성(Safety-Related Configuration)은 데이터 크기가 작더라도 해당 정보가 없으면 전체 로봇 플릿의 운영 재개가 불가능할 수 있다.

데이터 분류(Data Classification)는 복구 목표를 할당하는 실용적인 기준을 제공한다. 임무 핵심 운영 데이터(Mission-Critical Operational Data)를 중요한 개발 자산, 장기 아카이브(Long-Term Archives), 임시 또는 재생성 가능한 정보와 구분할 수 있다. 이후 각 분류에 적절한 RTO, RPO, 보존 기간(Retention Period), 백업 빈도, 복제 방식, 복구 절차를 지정할 수 있다. 이를 통해 동일한 복구 특성이 필요하지 않은 모든 데이터에 비용이 높은 고가용성 보호(High-Availability Protection)를 일률적으로 적용하는 것을 방지할 수 있다.

로봇 플릿 데이터베이스(Robot Fleet Database)는 데이터가 지속적으로 변경되고 실제 운영을 직접 지원하기 때문에 비교적 엄격한 복구 목표가 필요한 경우가 많다. 작업 할당(Task Assignments), 임무 상태(Mission States), 충전 이벤트(Charging Events), 로봇 상태, 유지보수 기록(Maintenance Records), 운영자 작업(Operator Actions)은 하루 동안 지속적으로 기록될 수 있다. 빈번한 백업, 트랜잭션 로그 아카이빙(Transaction-Log Archiving), 데이터베이스 복제 또는 시점 복구(Point-in-Time Recovery, PITR)를 통해 잠재적인 데이터 손실을 줄이고, 대기 인프라(Standby Infrastructure)를 통해 데이터베이스 서비스를 재개하는 시간을 줄일 수 있다.

지도(Maps)와 구성 저장소(Configuration Repositories)는 변경 빈도가 낮으면서도 운영 중요도가 높을 수 있기 때문에 다른 정책이 필요하다. 내비게이션 지도(Navigation Maps), 시맨틱 지도(Semantic Maps), 지오펜스(Geofences), 보정 파라미터(Calibration Parameters), 도킹 위치(Docking Positions), 네트워크 구성(Network Configuration), 안전 설정(Safety Settings)은 통제된 배포 과정에서만 변경될 수 있다. 따라서 승인된 변경이 발생할 때 버전 기반 백업(Versioned Backup)을 실행하고, 현재 버전을 복제하여 인프라 장애 이후 빠른 복원을 지원할 수 있다.

대규모 센서 아카이브(Large Sensor Archives)는 일반적으로 운영 데이터베이스와 다른 RTO 및 RPO 값을 허용할 수 있다. 카메라 스트림(Camera Streams), LiDAR 포인트 클라우드(Point Clouds), 레이더 기록(Radar Recordings), ROS Bag, MCAP 파일은 막대한 저장 용량을 소비하므로 지속적인 다중 복사본 보호는 높은 비용을 발생시킬 수 있다. 백업 정책에서는 검증된 데이터셋, 희귀 이벤트 기록(Rare-Event Recordings), 사고 증거(Incident Evidence), 대체할 수 없는 현장 데이터(Field Data)를 우선적으로 보호하고 재생성 가능하거나 가치가 낮은 원시 데이터에는 수명주기 관리(Lifecycle Management) 또는 완화된 복구 목표를 적용할 수 있다.

AI 및 피지컬 AI 데이터셋(Physical AI Datasets)은 단순한 파일 가용성뿐만 아니라 재현성(Reproducibility)을 보존하는 복구 정책이 필요하다. 어노테이션(Annotations), 매니페스트(Manifests), 전처리 파라미터(Preprocessing Parameters), 데이터셋 버전(Dataset Versions), 데이터 출처 정보(Provenance Information), 모델 연계 정보(Model Associations)가 손실된다면 원시 센서 객체만으로는 충분하지 않을 수 있다. 따라서 RPO는 완전한 논리적 데이터셋 상태(Logical Dataset State)에 대해 허용할 수 있는 손실을 표현해야 하며, RTO는 학습, 평가 또는 배포를 위한 사용 가능한 데이터셋 환경을 재구성하는 데 필요한 시간까지 고려해야 한다.

백업 빈도는 RPO와 밀접하게 관련되어 있지만 백업 주기 자체가 RPO 달성을 자동으로 보장하지는 않는다. 매시간 실행되는 백업은 1시간 단위의 복구 간격을 의미하는 것처럼 보이지만 실패한 작업, 지연된 전송, 손상된 아카이브(Corrupted Archives), 불완전한 트랜잭션 데이터로 인해 실제 복구 간격은 훨씬 길어질 수 있다. 따라서 백업 정책은 설정된 스케줄에만 의존하지 않고 가장 최근에 검증된 복구 시점(Verified Recovery Point)의 경과 시간을 모니터링해야 한다. 복구 목표는 측정 가능한 증거(Measurable Evidence)를 통해 뒷받침되어야 한다.

RTO는 단순히 백업 파일을 복사하는 데 필요한 시간보다 훨씬 많은 요소의 영향을 받는다. 복원 과정에는 서버 프로비저닝(Server Provisioning), 스토리지 볼륨 생성, 아카이브 객체 검색, 데이터 복호화, 데이터베이스 재구축, 인덱스 복원, 트랜잭션 로그 적용, 일관성 검증(Consistency Validation), 네트워크 구성, 자격 증명 업데이트, 애플리케이션 재연결 등이 필요할 수 있다. 로봇 서비스는 자율 운용을 안전하게 재개하기 전에 지도, 구성, 모델, 미들웨어 의존성(Middleware Dependencies)까지 필요할 수 있다.

따라서 정책 설계 과정에서 복구 의존성(Recovery Dependencies)을 식별해야 한다. 플릿 관리 데이터베이스가 성공적으로 복원되더라도 신원 서비스(Identity Services), 메시지 브로커(Message Brokers), 객체 스토리지, 지도 저장소 또는 네트워크 구성을 사용할 수 없다면 실제 서비스는 사용할 수 없다. 효과적인 RTO는 독립된 데이터베이스 프로세스의 복원이 아니라 필요한 운영 기능(Operational Capability)의 복원을 의미해야 한다. 의존성 매핑(Dependency Mapping)은 서로 연결된 로봇 서비스를 올바른 순서로 복구하는 데 도움을 준다.

복제(Replication)는 준비되었거나 거의 준비된 데이터 및 서비스 복사본을 유지하여 RTO를 줄일 수 있지만 복제만으로 완전한 백업 보호를 제공하지는 않는다. 우발적인 삭제, 애플리케이션 손상(Application Corruption), 악의적인 변경이 복제본으로 빠르게 전파될 수 있다. 이전의 정상 상태로 돌아가야 하는 경우에는 과거 백업(Historical Backups), 버전 관리(Versioning), 불변 스토리지(Immutable Storage), 시점 복구가 필요하다. 따라서 백업 정책은 가용성 메커니즘(Availability Mechanisms)과 과거 복구 메커니즘(Historical Recovery Mechanisms)을 함께 구성해야 한다.

3-2-1 백업 원칙(3-2-1 Backup Principle)은 독립적인 스토리지와 장애 도메인(Failure Domains)에 복구 복사본을 분산함으로써 RTO 및 RPO 목표 달성을 지원할 수 있다. 로컬 백업(Local Backup)은 빠른 복원을 제공하여 짧은 RTO를 지원할 수 있고, 오프사이트(Offsite) 또는 격리된 복사본(Isolated Copy)은 사이트 전체의 재해로부터 데이터를 보호한다. 그러나 여러 개의 복사본이 존재한다는 사실만으로 복구 목표가 자동으로 충족되지는 않는다. 생성 빈도, 전송 지연, 무결성, 접근 가능성, 복원 성능이 정의된 목표와 일치해야 한다.

보존 정책(Retention Policy)은 과거 어느 시점까지 복구할 수 있는지를 결정한다. 최근 장애에 대해 15분의 RPO를 만족하는 시스템이라도 오래된 복구 시점이 이미 삭제되었다면 몇 주 후 발견된 데이터 손상으로부터 복구하지 못할 수 있다. 일간(Daily), 주간(Weekly), 월간(Monthly), 이벤트 기반(Event-Based) 보존을 통해 서로 다른 과거 복구 범위를 유지할 수 있다. 따라서 백업 보존은 RPO, 사고 탐지 예상 시간(Incident-Detection Expectations), 스토리지 용량, 규제 요구사항(Regulatory Requirements), 데이터 가치와 함께 설계해야 한다.

복구 계층(Recovery Tiers)을 정의하면 서로 다른 특성을 가진 로봇 인프라 전체의 정책 관리를 단순화할 수 있다. 가장 높은 우선순위 계층은 복제, 빈번한 복구 시점, 로컬 대기 자원(Local Standby Resources), 원격 불변 백업(Remote Immutable Backups)을 사용할 수 있다. 두 번째 계층은 주기적인 스냅샷과 예약 복제를 사용할 수 있으며, 아카이브 또는 재생성 가능한 데이터는 더 느린 복원과 더 큰 복구 간격을 허용할 수 있다. 계층화(Tiering)의 목적은 단순한 비용 절감이 아니라 보호 수준과 운영 결과(Operational Consequence)를 일치시키는 것이다.

보안 요구사항(Security Requirements)은 달성 가능한 복구 목표에도 영향을 준다. 암호화(Encryption), 접근 제어(Access Controls), 격리된 자격 증명(Isolated Credentials), 불변 스토리지, 승인 메커니즘(Approval Mechanisms)은 추가적인 운영 단계를 필요로 할 수 있지만 이러한 제어를 제거하면 모든 복구 복사본이 동일한 침해 위험에 노출될 수 있다. 백업 정책은 정상적인 보안을 약화시키지 않으면서 재해 상황에서 승인된 복구 담당자가 필요한 자격 증명과 암호화 키를 확보할 수 있도록 해야 한다. 복구 문서에는 민감한 비밀 정보를 노출하지 않으면서 이러한 의존성을 포함해야 한다.

모니터링(Monitoring)은 실제 보호 상태와 정의된 복구 목표를 지속적으로 비교해야 한다. 유용한 측정 항목에는 가장 최근 검증된 복구 시점의 경과 시간, 백업 완료 시간, 복제 지연(Replication Lag), 아카이브 적체(Archive Backlog), 실패한 작업, 사용 가능한 보존 범위(Retention Depth), 복원 처리량(Restore Throughput), 스토리지 용량 등이 포함된다. 가장 최근의 유효한 복구 시점이 RPO 목표보다 오래된 상태가 되면 실제 사고가 발생하여 보호 공백이 드러날 때까지 기다리지 않고 운영 경고(Operational Alert)를 생성해야 한다.

복원 테스트(Restore Testing)는 RTO를 검증하는 데 필수적이다. 이론적인 전송 시간 계산만으로는 전체 복구 과정을 정확하게 표현하기 어렵기 때문이다. 정기적인 훈련에서는 데이터베이스, 객체 저장소, 지도, 구성, 선택된 데이터셋을 격리된 환경(Isolated Environment)에 복원하고 실제 사용 가능한 상태에 도달하는 데 필요한 시간을 측정해야 한다. 테스트를 통해 누락된 자격 증명, 사용할 수 없는 의존 서비스, 손상된 백업, 느린 아카이브 검색, 잘못된 절차, 애플리케이션 호환성 문제(Application Compatibility Problems)를 발견할 수 있으며 이러한 문제는 일반적인 백업 모니터링만으로 확인하기 어렵다.

복구 정책(Recovery Policy)은 역할(Role), 책임(Responsibility), 의사결정 지점(Decision Points)도 정의해야 한다. 사고가 발생하면 누가 복구를 선언하고, 누가 복구 시점을 선택하며, 누가 인프라를 복원하고, 누가 데이터를 검증하며, 누가 로봇 운영 재개를 승인하는지를 팀이 알고 있어야 한다. 기술적인 백업 기능이 존재하더라도 복구 작업이 문서화되지 않은 지식이나 불명확한 권한에 의존한다면 실질적인 가치는 크게 감소한다. 따라서 절차를 문서화하고 유지하며 반복적으로 훈련하고 인프라 변화에 맞추어 업데이트해야 한다.

RTO 및 RPO 목표는 로봇 운영, 데이터 용량, 애플리케이션 아키텍처 또는 인프라 의존성이 변경될 때마다 검토해야 한다. 소규모 로봇 배포에서 실용적이었던 복구 정책도 플릿 규모가 증가하고 센서 데이터가 증가하거나 클라우드 및 엣지 서비스가 더욱 긴밀하게 연결되면 충분하지 않을 수 있다. 측정된 백업 소요 시간, 복제 동작, 복원 성능, 복구 테스트 결과를 근거로 목표와 보호 메커니즘을 조정해야 한다.

성숙한 백업 정책(Mature Backup Policy)은 비즈니스 및 운영 요구사항을 기술적인 복구 메커니즘과 직접 연결한다. RPO는 복구 가능한 상태를 얼마나 자주 생성하고 보존해야 하는지를 결정하고, RTO는 이러한 상태를 얼마나 빠르게 다시 정상적으로 동작하는 서비스로 전환해야 하는지를 결정한다. 데이터 분류, 복제, 백업, 시점 복구(PITR), 교차 리전 보호(Cross-Region Protection), 보존, 모니터링, 보안, 복구 테스트를 결합함으로써 로보틱스 및 피지컬 AI 시스템은 단순히 백업 복사본을 축적하는 것이 아니라 측정 가능하고 검증 가능한 복구 능력(Measurable and Verifiable Recovery Capability)을 구축할 수 있다.

## 09.07 Backup Automation and Scheduling [w/Code]

![](images/image7.png){width="7.268055555555556in" height="7.268055555555556in"}

백업 자동화(Backup Automation)는 수동으로 수행하던 데이터 보호 작업을 정의된 정책에 따라 복구 복사본을 생성하는 반복 가능한 운영 프로세스로 전환한다. 로보틱스(Robotics) 인프라에서는 데이터베이스, 지도, 구성, 로그, 센서 아카이브, AI 데이터셋이 서로 다른 속도로 변경되기 때문에 자동화가 중요하다. 신뢰할 수 있는 설계에서는 무엇을 보호할지, 백업을 얼마나 자주 실행할지, 복사본을 어디에 저장할지, 얼마나 오래 유지할지, 장애를 어떻게 탐지할지를 결정해야 한다.

스케줄링(Scheduling)은 복구 요구사항을 실제 실행 시간과 실행 주기로 변환한다. 엄격한 복구 시점 목표(Recovery Point Objective, RPO)가 필요한 시스템에서는 빈번한 데이터베이스 스냅샷(Database Snapshots), 트랜잭션 로그 아카이빙(Transaction-Log Archiving), 연속 복제(Continuous Replication)가 필요할 수 있으며, 대규모 센서 아카이브는 매일 또는 중요한 데이터 수집 작업 이후에 보호할 수 있다. 구성 저장소와 지도는 승인된 변경이 발생할 때마다 이벤트 기반 백업(Event-Driven Backup)을 사용할 수 있다. 따라서 모든 자원에 동일한 주기를 적용하기보다 데이터 가치와 변경 빈도에 따라 스케줄을 결정해야 한다.

백업 스케줄러(Backup Scheduler)는 운영체제 서비스, 데이터베이스 자체 기능, 오케스트레이션 플랫폼(Orchestration Platform), 클라우드 관리형 서비스(Cloud-Managed Service)를 통해 구현할 수 있다. Linux 환경에서는 일반적으로 cron 또는 systemd timer를 이용하여 정의된 주기에 백업 스크립트를 실행하며, Kubernetes 환경에서는 컨테이너화된 워크로드(Containerized Workloads)에 CronJob을 사용할 수 있다. 클라우드 데이터베이스와 객체 스토리지 플랫폼은 관리형 스케줄링 기능을 제공할 수 있다. 선택한 스케줄러는 예측 가능한 실행, 로깅, 장애 보고, 관리 가능한 구성을 제공해야 한다.

백업 스크립트(Backup Scripts)는 가능한 경우 구성(Configuration)과 실행 로직(Execution Logic)을 분리해야 한다. 데이터베이스 주소, 대상 경로, 보존 기간, 스토리지 버킷(Storage Buckets), 기타 환경별 값은 스크립트 곳곳에 직접 삽입하기보다 구성 파일 또는 보호된 환경 설정에 유지할 수 있다. 이러한 구조는 유지보수성을 향상시키고 동일한 자동화를 개발, 연구실, 운영, 엣지(Edge), 재해 복구(Disaster Recovery) 환경에 배포할 때 발생할 수 있는 실수를 줄여준다.

데이터베이스 자동화(Database Automation)는 해당 데이터베이스 기술에 적합한 백업 메커니즘을 사용해야 한다. PostgreSQL 워크플로에서는 pg_basebackup, WAL 아카이빙(WAL Archiving), 논리적 덤프(Logical Dumps), 관리형 스냅샷을 조정하여 사용할 수 있으며, MongoDB 워크플로에서는 mongodump 또는 관리형 Atlas 백업 기능을 사용할 수 있다. 활성 데이터베이스 파일을 일반적인 파일시스템 명령으로 단순 복사하면 일관되지 않은 백업이 생성될 수 있다. 자동화는 데이터베이스 인식형 절차(Database-Aware Procedures)를 실행하고 작업 완료를 검증한 이후에만 성공으로 판단해야 한다.

객체 및 파일 백업 자동화(Object and File Backup Automation)는 지도, 로봇 구성, 모델, 로그, 데이터셋, 센서 기록을 처리하는 경우가 많다. 도구를 이용하여 선택된 디렉터리 또는 객체를 NAS, 객체 스토리지(Object Storage), 원격 서버 또는 아카이브 저장소(Archival Repository)로 동기화할 수 있다. 정책에서는 필요에 따라 임시 캐시와 재생성 가능한 중간 파일을 제외하는 동시에 완전한 복구에 필요한 매니페스트(Manifests), 메타데이터, 체크섬(Checksums), 보정 정보(Calibration Information), 구성 의존성이 보호 대상 데이터와 함께 저장되도록 해야 한다.

백업 워크플로(Backup Workflow)는 명령이 성공적으로 실행된 것을 전체 프로세스 중 하나의 단계로만 취급해야 한다. 백업을 생성한 이후 자동화 과정에서 예상 파일의 존재 여부를 확인하고, 아카이브 크기를 검사하고, 체크섬을 생성하거나 비교하며, 데이터베이스 백업 메타데이터를 검사하고, 대상 스토리지에 접근할 수 있는지 검증할 수 있다. 이러한 검사가 모두 성공한 이후에만 백업을 유효한 복구 시점(Valid Recovery Point)으로 기록해야 한다. 이를 통해 불완전한 아티팩트가 성공한 백업으로 잘못 기록되는 것을 방지할 수 있다.

명명 규칙(Naming Conventions)과 메타데이터(Metadata)는 자동화된 백업을 쉽게 식별하고 복원할 수 있도록 한다. 백업 아티팩트에는 시스템 식별자(System Identifiers), 데이터베이스 이름, 타임스탬프(Timestamps), 백업 유형, 버전, 환경 레이블(Environment Labels)을 메타데이터 또는 명명 구조에 포함할 수 있다. 별도의 매니페스트에는 생성 시간, 소스, 객체 수, 체크섬 정보, 소프트웨어 버전, 의존성을 기록할 수 있다. 수백 또는 수천 개의 예약 백업이 여러 로봇 시스템에 축적될수록 일관된 식별 체계의 중요성이 증가한다.

보존 자동화(Retention Automation)는 백업 스토리지가 제한 없이 증가하는 것을 방지한다. 정책에 따라 최근 일간 백업(Daily Backups)을 유지하고, 더 적은 수의 주간 백업(Weekly Backups), 선택된 월간 백업(Monthly Backups), 주요 릴리스 또는 로봇 배포와 연계된 이벤트별 복구 시점(Event-Specific Recovery Points)을 보존할 수 있다. 정리 프로세스(Cleanup Process)는 새로운 백업이 검증된 이후에만 실행되어야 하며 불변 또는 규제 기반 보존 요구사항을 준수해야 한다. 잘못된 정리 규칙이 중요한 복구 이력을 삭제할 수 있으므로 자동 삭제는 자동 생성만큼 신중하게 설계해야 한다.

전체 백업(Full Backup), 증분 백업(Incremental Backup), 차등 백업(Differential Backup)을 함께 스케줄링하여 복구 복잡성과 자원 사용량의 균형을 맞출 수 있다. 시스템은 주기적으로 전체 기준 백업(Full Baseline)을 생성하고 전체 백업 사이에서 크기가 작은 증분 백업을 생성할 수 있다. 차등 백업은 가장 최근 전체 백업 이후의 모든 변경 사항을 보존하는 또 다른 절충안을 제공한다. 자동화는 백업 의존성(Backup Dependencies)을 추적하여 종속된 복구 체인이 보존 기간 안에 존재하는 동안 필요한 상위 백업(Parent Backups)이 삭제되지 않도록 해야 한다.

대규모 데이터를 생성하는 로보틱스 시스템에서는 자원 스케줄링(Resource Scheduling)이 특히 중요하다. 백업 작업은 CPU, 스토리지 대역폭(Storage Bandwidth), 네트워크 용량, 경우에 따라 데이터베이스 입출력(Database I/O)을 사용하며, 이는 내비게이션, 인지(Perception), 플릿 조정(Fleet Coordination), AI 워크로드와 경쟁할 수 있다. 대용량 전송은 부하가 낮은 시간에 예약하고 대역폭을 제한하며 작업을 시간적으로 분산할 수 있다. 그러나 핵심 운영 데이터의 보호는 단순한 인프라 사용량 최적화를 위해 지나치게 지연해서는 안 된다.

동시성 제어(Concurrency Control)는 여러 백업 작업이 서로 간섭하는 것을 방지한다. 지연된 일간 백업이 아직 실행 중인 상태에서 다음 예약 작업이 시작되거나 여러 로봇이 동시에 대규모 아카이브를 업로드할 수 있다. 잠금 파일(Lock Files), 스케줄러 동시성 정책(Scheduler Concurrency Policies), 작업 큐(Job Queues), 분산 조정 메커니즘(Distributed Coordination Mechanisms)을 이용하여 원하지 않는 중복 실행을 방지할 수 있다. 자동화에서는 스토리지 및 복구 요구사항에 따라 중복 작업을 건너뛸지, 지연할지, 대기열에 넣을지 또는 동시에 실행할지를 정의해야 한다.

장애 처리(Failure Handling)는 일시적인 문제와 지속적인 문제를 구분해야 한다. 네트워크 중단, 객체 스토리지 일시적 사용 불가, 일시적인 데이터베이스 연결 오류는 점진적으로 대기 시간을 증가시키는 제어된 재시도(Retry)를 적용할 수 있다. 인증 실패(Authentication Failures), 스토리지 용량 부족, 손상된 원본 데이터, 반복적인 백업 오류는 일반적으로 운영자의 개입이 필요하다. 실패한 프로세스가 무한히 반복되면서 과도한 트래픽을 발생시키거나 해결되지 않은 인프라 문제를 숨기지 않도록 재시도 로직에는 명확한 제한을 설정해야 한다.

모니터링(Monitoring)은 예약된 백업 작업을 관찰 가능한 보호 시스템(Observable Protection System)으로 전환한다. 유용한 정보에는 시작 및 완료 시간, 백업 소요 시간, 전송 데이터 크기, 가장 최근의 유효한 복구 시점, 실패한 작업, 복제 지연(Replication Lag), 사용 가능한 스토리지 용량, 보존 상태 등이 포함된다. 대시보드(Dashboard)는 이러한 지표를 요약할 수 있으며, 경고(Alert)는 백업의 경과 시간이 정의된 RPO를 초과하는 상황을 탐지할 수 있다. 중요한 것은 스케줄러가 실행되었는지가 아니라 실제로 복구 가능한 데이터가 생성되었는지 여부이다.

로깅(Logging)은 비밀번호, 접근 토큰(Access Tokens), 암호화 키 또는 기타 비밀 정보를 노출하지 않으면서 장애를 진단하기에 충분한 정보를 제공해야 한다. 각 백업 실행에는 고유한 작업 식별자(Job Identifier)를 부여하고 소스, 대상, 시작 시간, 완료 상태, 검증 결과, 오류 정보를 기록할 수 있다. 백업이 로봇 엣지 컴퓨터, 온프레미스 서버(On-Premises Servers), 클라우드 서비스에 분산되어 있는 경우 중앙 집중식 로그(Centralized Logs)를 사용하면 공통 모니터링 환경에서 전체 보호 상태를 조사할 수 있다.

자격 증명 관리(Credential Management)는 무인 자동화(Unattended Automation)에 필수적이다. 스크립트에 하드코딩된 비밀번호나 클라우드 키는 불필요한 보안 위험을 발생시키며 자격 증명 교체(Credential Rotation)를 어렵게 만든다. 백업 프로세스는 보호된 비밀 관리 메커니즘(Secret-Management Mechanisms), 운영체제 자격 증명 저장소, 워크로드 신원(Workload Identities) 또는 이에 상응하는 서비스를 통해 자격 증명을 획득해야 한다. 권한은 최소 권한 원칙(Least-Privilege Principle)을 적용하여 침해된 백업 프로세스가 관련 없는 운영 자원을 수정하거나 보호된 과거 복사본을 삭제하지 못하도록 해야 한다.

암호화(Encryption)는 자동화된 전송 및 저장 워크플로에 포함되어야 한다. 로봇, 엣지 서버, NAS 시스템, 데이터센터, 클라우드 객체 스토리지 사이에서 이동하는 백업 데이터에는 보호된 통신 채널(Protected Communication Channels)을 사용해야 한다. 저장된 백업 아티팩트에는 저장 데이터 암호화(Encryption at Rest)가 필요할 수 있다. 재해 복구에 필요한 암호화 키를 보호된 복구 프로세스를 통해 사용할 수 있도록 자동화해야 한다. 필요한 키에 접근할 수 없는 암호화된 백업은 운영 관점에서 사용할 수 없는 데이터와 동일하기 때문이다.

3-2-1 원칙(3-2-1 Principle)은 수동 복사가 아니라 조정된 자동화(Coordinated Automation)를 통해 구현할 수 있다. 예약된 로컬 백업은 빠른 복구를 지원하고, 또 다른 복사본은 서로 다른 스토리지 기술로 전송하며, 오프사이트(Offsite) 또는 격리된 복사본은 시설 전체의 사고로부터 데이터를 보호할 수 있다. 각 단계는 독립적으로 모니터링해야 한다. 로컬 백업이 성공했더라도 원격 전송 또는 격리 보존 단계가 실패했다면 전체 3-2-1 워크플로가 성공한 것은 아니다.

이벤트 기반 자동화(Event-Driven Automation)는 시간 기반 스케줄링(Time-Based Scheduling)을 보완한다. 새로운 로봇 소프트웨어 릴리스, 지도 업데이트, 보정 변경, 모델 배포(Model Deployment), 데이터베이스 마이그레이션(Database Migration), 구성 변경이 발생할 때 변경 전후로 보호된 복구 시점을 자동 생성할 수 있다. 이를 통해 예약된 작업 사이에서 발생하는 운영상 중요한 이벤트도 백업할 수 있다. 이벤트 메타데이터를 이용하면 백업을 배포 버전, 실험, 유지보수 작업 또는 변경 관리 기록(Change-Management Record)과 연결할 수도 있다.

복구 테스트(Recovery Testing) 자체도 자동화할 수 있다. 선택된 백업 아티팩트를 주기적으로 격리된 환경에 복원한 후 데이터베이스 연결, 문서 또는 행 수(Document or Row Counts), 인덱스, 파일 체크섬, 지도, 구성 구조, 애플리케이션 쿼리를 스크립트로 검사할 수 있다. 자동 복원 테스트(Automated Restore Tests)는 보호된 데이터를 실제로 재구성할 수 있음을 입증하므로 백업 생성 로그보다 강력한 근거를 제공한다. 측정된 복원 시간은 정의된 복구 시간 목표(Recovery Time Objective, RTO)와 비교할 수도 있다.

로봇 플릿(Robot Fleets)은 간헐적인 네트워크 연결(Intermittent Connectivity)을 가진 다수의 엣지 컴퓨터에서 백업 활동이 이루어질 수 있기 때문에 분산 스케줄링(Distributed Scheduling) 문제를 발생시킨다. 각 로봇은 로그, 임무 기록 또는 센서 데이터를 로컬에 일시적으로 버퍼링하고 네트워크 상태가 허용될 때 동기화할 수 있다. 중앙 오케스트레이션(Central Orchestration)은 업로드 상태를 추적하고 누락된 장치를 식별하며, 로컬 자동화는 연결이 끊어진 동안 데이터를 보호한다. 정책에서는 일시적인 통신 장애가 성공적인 중앙 보호로 잘못 판단되지 않도록 해야 한다.

자동화 구성(Automation Configuration)은 비공식적인 관리 스크립트가 아니라 통제된 인프라(Controlled Infrastructure)로 관리해야 한다. 백업 스크립트, 스케줄러 정의, 보존 규칙, 인프라 템플릿(Infrastructure Templates), 복구 절차를 버전 관리(Version Control)하고 배포 전에 검토할 수 있다. 경로, 필터, 자격 증명 또는 삭제 규칙의 작은 변경 하나가 이후 모든 백업에 영향을 줄 수 있으므로 변경 사항을 테스트해야 한다. 구성 이력(Configuration History)은 특정 시점에 어떤 보호 정책이 적용되고 있었는지를 확인하는 근거도 제공한다.

성숙한 백업 스케줄링 시스템(Mature Backup Scheduling System)은 보호할 데이터를 식별하고, 백업 생성을 예약하거나 트리거하며, 복사본을 적절한 스토리지로 전송하고, 무결성을 검증하고, 보존 정책을 적용하며, 상태를 모니터링하고, 장애 발생 시 경고하며, 주기적으로 복원을 테스트하는 연속적인 운영 루프(Continuous Operational Loop)를 구성한다. 로보틱스 및 피지컬 AI(Physical AI) 시스템에서 이러한 자동화는 수동 작업에 대한 의존성을 줄이는 동시에 백업 빈도, 보안, 스토리지 분리, RPO, RTO, 검증된 복구(Verified Recovery)를 측정 가능하고 반복 가능한 데이터 보호 프로세스로 연결한다.

## 09.08 Recovery Test Automation: Periodic DR Drill [w/Code]

![](images/image8.png){width="7.268055555555556in" height="7.268055555555556in"}

복구 테스트(Recovery Testing)는 장애 발생 이후 백업 데이터를 실제로 사용 가능한 시스템으로 전환할 수 있는지를 검증한다. 백업 작업의 성공은 데이터가 복사되거나 아카이빙되었다는 사실만을 의미하며, 해당 백업이 완전하고 읽을 수 있으며 일관성을 유지하고 실제 운영 환경에서 복구 가능한지를 보장하지 않는다. 로보틱스(Robotics) 및 피지컬 AI(Physical AI) 시스템에서는 데이터베이스, 지도, 구성, 모델, 센서 아카이브, 자격 증명, 의존성, 로봇 운영 재개에 필요한 서비스를 복구 테스트를 통해 검증해야 한다.

재해 복구 훈련(Disaster Recovery Drill, DR Drill)은 일반적인 복원 테스트를 확장하여 더 광범위한 장애 시나리오를 검증한다. 하나의 파일이나 데이터베이스만 복원하는 대신 운영 서버, 스토리지 시스템, 데이터센터, 클라우드 리전 또는 핵심 플랫폼 서비스의 손실을 가정할 수 있다. 목표는 사람, 절차, 인프라, 백업 복사본, 애플리케이션 의존성이 함께 동작하여 정의된 복구 목표(Recovery Objectives) 안에서 필요한 운영 기능을 복원할 수 있는지를 확인하는 것이다.

시스템이 지속적으로 변화하기 때문에 주기적인 실행(Periodic Execution)이 중요하다. 6개월 전에 정상적으로 동작했던 복구 절차도 데이터베이스 증가, 소프트웨어 업그레이드, 자격 증명 교체(Credential Rotation), 네트워크 변경, 새로운 암호화 정책 또는 다른 스토리지로의 마이그레이션 이후에는 실패할 수 있다. 예약된 DR 훈련은 시스템을 처음 배포했을 당시 작성된 문서나 가정에 의존하지 않고 현재 환경에서도 복구 아키텍처가 정상적으로 동작한다는 반복적인 증거를 제공한다.

복구 테스트 자동화(Recovery Test Automation)는 이러한 훈련을 반복 가능하고 측정 가능한 프로세스로 만든다. 자동화된 워크플로(Automated Workflow)는 복구 시점을 선택하고, 격리된 테스트 환경을 프로비저닝하며, 백업 아티팩트를 가져오고, 데이터베이스와 파일을 복원하고, 트랜잭션 로그를 적용하고, 서비스를 구성하고, 검증 작업을 실행하고, 시간 정보를 수집한 뒤 임시 자원을 제거할 수 있다. 자동화는 테스트 간 편차를 줄이고 동일한 복구 절차를 시간에 따라 일관되게 평가할 수 있도록 한다.

테스트 환경(Test Environment)은 일반적으로 운영 시스템으로부터 격리되어야 한다. 복원된 데이터베이스에는 운영 환경의 식별자, 로봇 구성, 자격 증명 또는 과거 명령이 포함될 수 있으며, 이러한 정보가 활성 로봇이나 외부 서비스와 실수로 상호작용해서는 안 된다. 네트워크 분리(Network Segmentation), 별도의 네임스페이스(Namespaces), 테스트 자격 증명, 비활성화된 외부 인터페이스 또는 시뮬레이션 엔드포인트(Simulated Endpoints)를 이용하면 복원된 애플리케이션의 내부 동작을 검증하면서 실제 명령이 전송되는 것을 방지할 수 있다.

복구 시점 선택(Recovery-Point Selection)은 항상 최신 백업만 복원하기보다 현실적인 장애 시나리오를 반영해야 한다. 한 테스트에서는 가장 최근에 검증된 복사본을 복원하여 일반적인 재해 복구를 평가하고, 다른 테스트에서는 오래된 버전을 선택하여 데이터 손상이나 우발적인 삭제가 뒤늦게 발견된 상황을 모의할 수 있다. PostgreSQL 시점 복구(Point-in-Time Recovery, PITR)는 타임스탬프 또는 복원 지점(Restore Point)을 기준으로 검증할 수 있으며, 버전 관리된 객체 스토리지와 MongoDB 백업은 과거 스냅샷 또는 논리적 백업 아티팩트를 이용하여 평가할 수 있다.

데이터베이스 복구 테스트(Database Recovery Test)는 단순히 서버 프로세스가 시작되는지만 확인해서는 안 된다. PostgreSQL 테스트에서는 기본 백업(Base Backup)과 WAL 복구 이후 복원된 데이터베이스, 스키마(Schemas), 테이블, 인덱스, 트랜잭션 일관성(Transaction Consistency), 애플리케이션 쿼리를 확인할 수 있다. MongoDB 테스트에서는 Atlas 복원 또는 mongorestore 이후 컬렉션, 문서, 인덱스, 메타데이터, 대표적인 쿼리를 검증할 수 있다. 데이터베이스별 검증은 단순한 연결 성공 메시지보다 강력한 복구 증거를 제공한다.

파일 및 객체 복구 테스트(File and Object Recovery Tests)는 접근 가능성뿐만 아니라 완전성(Completeness)도 검증해야 한다. 로봇 지도, 구성 패키지, AI 모델, 보정 파일, ROS Bag, MCAP 기록, 이미지, 포인트 클라우드(Point Clouds), 데이터셋 매니페스트(Dataset Manifests)를 예상된 인벤토리(Expected Inventory)와 비교할 수 있다. 파일 크기, 체크섬(Checksums), 객체 수, 메타데이터, 버전 식별자, 디렉터리 또는 키 구조를 자동으로 검사하여 누락되거나 잘렸거나 손상되었거나 잘못 복원된 콘텐츠를 탐지할 수 있다.

애플리케이션 수준 검증(Application-Level Validation)은 복원된 데이터가 실제로 유용한지를 판단한다. 플릿 관리 서비스(Fleet-Management Service)는 최근 로봇 레코드를 불러오고, 임무 이력을 검색하고, 지도 참조를 확인하고, 대표적인 API 쿼리를 실행하여 테스트할 수 있다. AI 시스템은 모델 파일, 구성, 전처리 메타데이터, 필요한 데이터셋 구성 요소가 서로 호환되는지를 검증할 수 있다. 의도된 서비스가 복원된 정보를 정상적으로 사용할 수 있어야 복구가 완료된 것으로 판단할 수 있다.

현대의 로봇 서비스는 독립적으로 동작하는 경우가 드물기 때문에 의존성 검증(Dependency Validation)이 필수적이다. 메시지 브로커(Message Brokers), 신원 서비스(Identity Services), 객체 스토리지, 지도 서비스, DNS, 인증서(Certificates), 네트워크 경로, 구성 서버 또는 암호화 키를 사용할 수 없다면 데이터베이스를 복원하더라도 운영 기능은 복구되지 않을 수 있다. 따라서 DR 훈련에서는 개별 스토리지 구성 요소만 평가하는 것이 아니라 실제 로봇 서비스를 재구성하는 데 필요한 의존성 체인(Dependency Chain)과 복구 순서를 검증해야 한다.

RTO 측정(RTO Measurement)은 모의 사고가 선언되는 시점부터 필요한 서비스가 사전에 정의된 사용 가능 상태에 도달할 때까지 수행해야 한다. 여기에는 장애 탐지, 의사결정, 인프라 프로비저닝(Infrastructure Provisioning), 백업 검색, 복원, 검증, 구성, 애플리케이션 시작, 운영 재개 승인 과정이 포함된다. 파일 전송 시간이나 데이터베이스 복원 시간만 측정하면 전체 시스템의 실제 복구 시간 목표(Recovery Time Objective, RTO) 성능을 크게 과소평가할 수 있다.

RPO 검증(RPO Validation)은 복원된 상태가 허용된 데이터 손실 범위를 만족하는지를 확인한다. 테스트에서는 사고 발생 시각과 가장 최근에 사용할 수 있는 복구 시점의 타임스탬프를 비교하여 그 차이가 정의된 복구 시점 목표(Recovery Point Objective, RPO) 안에 있는지를 확인할 수 있다. 실패한 백업 작업, 복제 지연(Replication Lag), 지연된 아카이브 또는 손상된 복구 시점으로 인해 실제 복구 가능한 상태가 예상보다 오래될 수 있으므로 백업 스케줄만으로는 충분한 증거가 되지 않는다.

자동 검증(Automated Validation)은 명확한 성공 또는 실패(Pass or Fail) 결과를 생성해야 한다. 테스트에서는 예상 값과 복원된 객체 수, 데이터베이스 레코드, 체크섬, 스키마 버전, 구성 해시(Configuration Hashes), 애플리케이션 응답, 복구 타임스탬프를 비교할 수 있다. 또한 측정된 RTO와 RPO가 정책 범위 안에 있는지를 임계값(Thresholds)을 통해 판정할 수 있다. 자동화 스크립트가 운영체제 오류 없이 마지막 단계까지 실행되었다는 이유만으로 복구 테스트를 성공으로 판단해서는 안 된다.

장애 주입(Failure Injection)을 사용하면 DR 훈련을 더욱 현실적으로 만들 수 있다. 테스트 시나리오는 데이터베이스 손상, 삭제된 객체, 사용할 수 없는 기본 스토리지, 장애가 발생한 엣지 서버(Edge Server), 클라우드 연결 손실 또는 리전 서비스 중단 등을 가정할 수 있다. 목적은 통제되지 않은 운영 위험을 만드는 것이 아니라 안전한 조건에서 알려진 복구 경로를 검증하는 것이다. 통제된 장애 시나리오는 일반적인 복원 테스트에서 발견하기 어려운 가정을 드러내고 서로 다른 장애에 필요한 복구 메커니즘을 파악하는 데 도움을 준다.

로봇 플릿(Robot Fleets)은 운영 상태가 클라우드 서비스, 온프레미스(On-Premises) 인프라, 엣지 컴퓨터, 개별 로봇에 분산될 수 있기 때문에 추가적인 DR 요구사항이 발생한다. 훈련에서는 중앙 서비스가 사용할 수 없을 때 로봇이 어떻게 동작하는지, 로컬 데이터가 어떻게 버퍼링되는지, 동기화가 어떻게 재개되는지, 복구 이후 중복되거나 충돌하는 임무 기록이 발생하는지를 확인할 수 있다. 복구 절차는 중앙 데이터의 무결성과 서비스 전환 과정에서의 안전한 로봇 동작을 모두 보존해야 한다.

기술적 복원이 고도로 자동화되어 있더라도 주기적인 DR 훈련에는 사람의 의사결정(Human Decision Making)이 포함되어야 한다. 팀은 누가 재해를 선언하고, 누가 복구 시점을 선택하며, 누가 장애 조치(Failover)를 승인하고, 누가 복원된 데이터를 검증하며, 누가 로봇 운영 재개를 승인하는지 알고 있어야 한다. 자동화 시스템이 복구 단계를 빠르게 실행하더라도 불명확한 권한이나 의사소통은 다운타임을 증가시킬 수 있다. 따라서 훈련은 기술적 준비 상태뿐만 아니라 조직적 준비 상태(Organizational Readiness)도 검증한다.

훈련 빈도(Drill Frequency)는 시스템 중요도(System Criticality), 변경 속도, 복구 목표, 운영 위험을 기반으로 결정해야 한다. 핵심 플릿 데이터베이스와 서비스에는 더 빈번한 자동 복원 테스트가 필요할 수 있으며, 전체 리전을 대상으로 하는 포괄적인 DR 훈련은 더 많은 자원을 사용하므로 상대적으로 낮은 빈도로 수행할 수 있다. 중요한 아키텍처 변경, 데이터베이스 마이그레이션, 스토리지 재설계, 보안 변경 또는 대규모 로봇 배포가 발생하면 다음 예약 훈련까지 기다리지 않고 추가 테스트를 실행할 수도 있다.

모든 복구 훈련에는 모니터링 및 증거 수집(Monitoring and Evidence Collection)이 함께 수행되어야 한다. 로그에는 선택된 백업, 각 복구 단계의 타임스탬프, 전송 데이터 용량, 검증 결과, 오류, 재시도, 수동 개입(Manual Interventions), 최종 서비스 상태를 기록할 수 있다. 대시보드 또는 보고서를 이용하면 여러 훈련 결과를 비교하여 증가하는 복원 시간, 반복되는 장애 또는 의존성 병목(Dependency Bottlenecks)을 식별할 수 있다. 과거 측정 결과를 축적하면 복구 능력을 주관적인 판단이 아니라 관찰 가능한 상태로 관리할 수 있다.

복구 테스트 중에도 보안(Security)은 유지되어야 한다. 복원된 운영 정보를 포함하는 테스트 환경에는 접근 제어, 암호화, 감사 로깅(Audit Logging), 보호된 자격 증명, 테스트 종료 후 통제된 폐기 절차가 필요하다. 또한 복구 담당자는 승인된 비상 절차를 통해 필요한 암호화 키와 비밀 정보(Secrets)를 확보할 수 있어야 한다. 재해 상황에서 복호화할 수 없는 백업은 사용할 수 없으며, 반대로 제한 없는 비상 자격 증명은 별도의 보안 취약점을 만들 수 있다.

정리(Cleanup)는 자동화된 복구 테스트의 일부이다. 임시 테스트 환경은 상당한 컴퓨팅 및 스토리지 자원을 소비하거나 민감한 데이터를 노출된 상태로 남길 수 있기 때문이다. 검증 및 증거 수집 이후 자동화는 테스트 인스턴스를 종료하고, 임시 볼륨을 제거하고, 단기 자격 증명(Short-Lived Credentials)을 폐기하며, 정책에 따라 테스트 복사본을 삭제할 수 있다. 정리 과정 자체도 로그로 기록하여 복구 훈련으로 인해 보호된 로봇 데이터의 관리되지 않는 복사본이 남지 않았음을 확인해야 한다.

각 DR 훈련의 결과는 백업 및 복구 정책(Backup and Recovery Policy)에 다시 반영되어야 한다. 복원이 RTO를 초과하면 더 빠른 스토리지, 로컬 복구 복사본, 대기 인프라(Standby Infrastructure), 단순화된 절차 또는 개선된 자동화가 필요할 수 있다. RPO를 위반하면 백업 빈도, WAL 아카이빙, 복제 또는 스냅샷 정책을 조정해야 할 수 있다. 누락된 의존성과 검증 실패 역시 비공식적인 관찰로 남겨두지 않고 추적 가능한 시정 조치(Corrective Actions)로 연결해야 한다.

복구 자동화(Recovery Automation)는 인프라 및 애플리케이션 구성과 함께 버전 관리(Version Control)되어야 한다. 복원 스크립트, 오케스트레이션 정의(Orchestration Definitions), 검증 테스트, 복구 매니페스트(Recovery Manifests), DR 절차는 시스템 변화에 따라 지속적으로 변경된다. 이러한 아티팩트를 통제된 코드(Controlled Code)로 관리하면 검토, 테스트, 롤백(Rollback), 추적성(Traceability)을 확보할 수 있다. 이를 통해 복구 프로세스를 비상 상황에서만 사용하는 스크립트 모음이 아니라 운영 인프라와 동일한 규율로 개발하고 유지할 수 있다.

성숙한 DR 프로그램(Mature DR Program)은 백업을 생성하고, 검증하고, 주기적으로 복원하며, 애플리케이션 요구사항에 따라 테스트하고, RTO와 RPO를 기준으로 측정하고, 훈련 결과를 이용해 지속적으로 개선하는 검증 순환 구조(Continuous Verification Cycle)를 형성한다. 로보틱스 및 피지컬 AI 시스템에서 주기적으로 수행되는 자동화 DR 훈련은 백업을 수동적으로 저장된 데이터에서 실제로 입증된 복구 능력(Demonstrated Recovery Capability)으로 전환한다. 궁극적인 목표는 단순히 백업을 보유하는 것이 아니라 핵심 로봇 서비스를 정의된 운영 목표 안에서 안전하고 정확하게 복원할 수 있음을 반복적으로 입증하는 것이다.

## 09.09 Embedded System Backup: eMMC Image [w/Code]

![](images/image9.png){width="7.268055555555556in" height="7.268055555555556in"}

임베디드 시스템 백업(Embedded System Backup)은 운영체제, 부트로더(Bootloader), 장치 구성(Device Configuration), 애플리케이션 소프트웨어, 드라이버, 영구 데이터(Persistent Data)가 하나의 eMMC 장치에 함께 존재할 수 있기 때문에 일반적인 서버 백업과 다른 전략이 필요하다. 로봇 컨트롤러 또는 엣지 컴퓨터(Edge Computer)에서 이러한 스토리지에 장애가 발생하면 전체 장치가 부팅되지 않을 수 있다. 따라서 eMMC 이미지 백업(eMMC Image Backup)은 교체 장치를 알려진 정상 운영 상태로 복원할 수 있도록 임베디드 스토리지의 복원 가능한 표현을 보존한다.

eMMC 이미지(eMMC Image)는 일반적으로 임베디드 플래시 스토리지 전체 또는 선택된 영역에 대한 블록 수준 복사본(Block-Level Copy)이다. 표시되는 디렉터리와 파일만 복사하는 파일 백업과 달리 이미지는 파티션 테이블(Partition Tables), 파일시스템, 부트 파티션(Boot Partitions), 운영체제 파일, 설치된 패키지, 애플리케이션 바이너리(Application Binaries), 구성 데이터를 보존할 수 있다. 따라서 임베디드 컨트롤러를 수동으로 재구축하는 데 많은 설치 및 구성 단계가 필요한 경우 이미지 기반 복구(Image-Based Recovery)가 유용하다.

로봇 플랫폼은 일반적으로 소프트웨어와 하드웨어 구성이 긴밀하게 결합되어 있다. 임베디드 컴퓨터에는 Linux, ROS 또는 ROS 2, 장치 드라이버, 미들웨어(Middleware), 네트워크 설정, 센서 보정(Sensor Calibration), 하드웨어 인터페이스, AI 런타임 라이브러리(AI Runtime Libraries), 로봇 전용 애플리케이션이 포함될 수 있다. 애플리케이션 소스 코드만 복원해서는 이러한 전체 환경을 재구성할 수 없다. 검증된 eMMC 이미지는 배포된 로봇 컨트롤러를 재현하는 데 필요한 소프트웨어 기준 상태(Software Baseline)를 보존할 수 있다.

이미지 생성(Image Creation)은 일관된 시스템 상태에서 수행해야 한다. 원시 블록 이미지(Raw Block Image)를 캡처하는 동안 파일이 변경되면 파일시스템의 서로 다른 부분이 서로 다른 시점을 나타낼 수 있다. 가장 안전한 방법은 관련 서비스를 중지하고, 가능한 경우 파일시스템을 마운트 해제(Unmount)하거나, 외부 복구 미디어(External Recovery Media)로 부팅하거나, 공급업체가 지원하는 이미징 메커니즘(Vendor-Supported Imaging Mechanism)을 사용하는 것이다. 목적은 활성 쓰기 작업으로 인해 내부적으로 일관되지 않은 복구 이미지가 생성되는 것을 방지하는 것이다.

Linux 시스템에서는 dd와 같은 블록 수준 유틸리티(Block-Level Utility)를 사용하여 eMMC 블록 장치를 읽고 그 내용을 이미지 파일로 기록할 수 있다. dd는 스토리지 블록에 직접 접근하므로 실행 전에 소스와 대상 장치 이름을 매우 신중하게 확인해야 한다. 복원 과정에서 두 장치를 반대로 지정하면 잘못된 디스크를 덮어쓸 수 있다. 따라서 자동화된 절차에서는 장치를 안전하게 탐색하고, 식별자를 검증하고, 용량을 기록하며, 파괴적인 쓰기 작업 전에 명시적인 안전장치(Safeguards)를 적용해야 한다.

원시 eMMC 이미지(Raw eMMC Image)는 사용되지 않는 파일시스템 공간까지 포함하여 전체 장치 용량을 나타낼 수 있기 때문에 크기가 클 수 있다. 사용되지 않거나 반복적인 블록의 압축 효율이 높은 경우 압축(Compression)을 통해 저장 공간 요구량을 크게 줄일 수 있다. 워크플로에서는 이미지를 생성한 후 압축 아카이브(Compressed Archive) 형태로 저장할 수 있지만 압축에는 추가적인 CPU 시간과 복구 단계가 필요하다. 따라서 백업 설계에서는 이미지 크기, 전송 시간, 스토리지 비용, 복원 속도의 균형을 고려해야 한다.

파티션 인식 백업(Partition-Aware Backup)은 전체 eMMC 장치를 복사하는 방법의 대안이 될 수 있다. 플랫폼 아키텍처가 지원하는 경우 부트 파티션, 루트 파일시스템(Root Filesystem), 애플리케이션 파티션, 영구 구성 영역, 사용자 데이터 영역을 개별적으로 보호할 수 있다. 이 방법은 백업 크기를 줄이고 선택적 복원을 가능하게 하지만 파티션 구조와 부트 메타데이터(Boot Metadata)까지 함께 보존해야 하므로 복구 복잡성이 증가한다. 정확한 시스템 재구성이 주된 목적이라면 전체 장치 이미징(Full-Device Imaging)이 더 단순한 방법이 될 수 있다.

루트 파일시스템만 복원한다고 해서 부팅 가능한 임베디드 시스템이 만들어지는 것은 아니므로 부트 체인(Boot Chain)을 특별히 고려해야 한다. 플랫폼에 따라 복구에는 파티션 테이블, 부트로더 구성 요소, 부트 구성(Boot Configuration), 커널 이미지(Kernel Images), 디바이스 트리(Device Trees), 펌웨어(Firmware), 전용 부트 파티션이 필요할 수 있다. 백업 문서에서는 어떤 요소가 eMMC에 존재하고 어떤 요소가 다른 위치에 저장되는지를 식별하여 완전한 부팅 가능 상태(Bootable State)를 재구성할 수 있도록 해야 한다.

로봇 전용 영구 데이터(Robot-Specific Persistent Data)는 기본 시스템 이미지(Base System Image)와 개념적으로 분리해야 한다. 지도, 보정 파라미터, 로봇 식별 정보, 네트워크 구성, 임무 데이터, 로그, 학습된 파라미터, 로컬 데이터베이스는 운영체제 이미지보다 훨씬 자주 변경될 수 있다. 실용적인 전략은 안정적인 소프트웨어 기준 상태에 대해 주기적인 골든 이미지(Golden Image)를 생성하면서 자주 변경되는 운영 데이터는 별도의 더 빈번한 메커니즘으로 백업하는 것이다.

골든 이미지(Golden Image)는 호환 가능한 여러 장치를 프로비저닝하거나 복구하는 데 사용할 수 있는 검증된 기준 상태(Reference State)를 의미한다. 승인된 운영체제, 미들웨어, 드라이버, 런타임 라이브러리, 로봇 애플리케이션, 기본 구성을 포함할 수 있다. 배포 전에는 호스트 이름(Hostname), 인증서, 네트워크 신원(Network Identity), 보정 정보 또는 로봇 ID와 같은 장치별 정보를 별도로 주입할 수 있다. 이를 통해 모든 로봇에 대해 완전히 고유한 전체 이미지를 각각 유지해야 하는 부담을 줄일 수 있다.

여러 로봇 하드웨어 및 소프트웨어 리비전(Revision)이 존재하는 경우 버전 관리(Version Management)가 필수적이다. 특정 보드 리비전(Board Revision), 스토리지 용량, 커널, 드라이버 스택(Driver Stack), 주변장치 구성에 맞춰 생성된 이미지는 다른 플랫폼에서 정상적으로 동작하지 않을 수 있다. 따라서 이미지 메타데이터에는 하드웨어 모델, 보드 리비전, 소프트웨어 릴리스, 커널 버전, 아키텍처, 이미지 생성 날짜, 파티션 구조, 관련 애플리케이션 버전을 기록해야 한다. 복구 담당자는 추측 없이 올바른 이미지를 식별할 수 있어야 한다.

체크섬(Checksums)은 이미지 손상을 탐지하기 위한 간단한 메커니즘을 제공한다. 이미지 생성 이후 SHA-256과 같은 암호학적 해시(Cryptographic Hash)를 계산하여 백업 메타데이터와 함께 저장할 수 있다. 이미지를 NAS, 객체 스토리지(Object Storage), 이동식 미디어(Removable Media) 또는 다른 복구 저장소로 복사한 이후 체크섬을 다시 검증할 수 있다. 손상된 이미지를 eMMC에 기록하면 대상 시스템을 사용할 수 없는 상태로 만들고 추가적인 복구 작업이 필요할 수 있으므로 복원 전 검증이 특히 중요하다.

백업 스토리지(Backup Storage)는 보호 대상 임베디드 장치와 독립적으로 구성해야 한다. 유일한 eMMC 이미지를 동일한 로봇 컴퓨터에 저장하는 것은 장치 분실, 스토리지 장애, 전기적 손상 또는 물리적 파괴에 대해 거의 보호 기능을 제공하지 못한다. 이미지는 엔지니어링 워크스테이션, NAS, 중앙 서버, 객체 스토리지 또는 오프사이트 저장소(Offsite Repository)로 전송할 수 있다. 중요한 운영 이미지는 3-2-1 원칙(3-2-1 Principle)을 적용하여 독립적인 스토리지와 장애 도메인(Failure Domains)에 여러 복사본을 유지할 수도 있다.

완전한 eMMC 이미지에는 자격 증명, 인증서, 네트워크 설정, 애플리케이션 비밀 정보(Application Secrets), 독점적인 로봇 소프트웨어, 운영 정보가 포함될 수 있으므로 보안(Security)이 중요하다. 따라서 이미지에는 적절한 접근 제어(Access Control)와 암호화(Encryption)를 적용해야 한다. 민감한 자격 증명은 골든 이미지에서 제외하고 복원 이후 안전하게 프로비저닝할 수 있다. 또한 신뢰할 수 있는 복구 이미지가 몰래 교체되지 않도록 백업 저장소에서 삭제 및 수정 권한을 제한해야 한다.

보안 부팅(Secure Boot)과 하드웨어 기반 보안(Hardware-Backed Security)은 이미지 복원에 영향을 줄 수 있다. 복사된 이미지에는 올바르게 서명되었거나 특정 부트 키(Boot Keys), 장치 신원, 신뢰할 수 있는 하드웨어와 연결된 경우에만 유효한 소프트웨어가 포함될 수 있다. 따라서 복구 절차에서는 부트 검증(Boot Verification), 키 프로비저닝(Key Provisioning), 암호화된 파티션, 신뢰 플랫폼 모듈(Trusted Platform Module, TPM) 또는 보안 요소(Secure Element)의 의존성, 공급업체별 보안 메커니즘을 고려해야 한다. 바이트 단위로 동일한 이미지(Byte-for-Byte Image)만 복사한다고 해서 외부 보안 상태까지 재구성되는 것은 아니다.

이미지를 복원할 때는 스토리지 용량 차이(Storage Capacity Differences)를 고려해야 한다. 더 큰 eMMC 장치에서 생성된 원시 이미지는 실제 사용 데이터가 적더라도 명목상 더 작은 교체 장치에는 기록할 수 없을 수 있다. 반대로 작은 이미지를 더 큰 장치에 복원하면 파티션과 파일시스템을 확장하기 전까지 남은 공간을 사용하지 못할 수 있다. 교체용 스토리지 크기를 표준화하고 복원 이후 크기 조정(Post-Restoration Resizing) 절차를 문서화하면 현장 복구를 더욱 예측 가능하게 만들 수 있다.

플래시 메모리 상태(Flash Memory Health) 역시 백업 및 복구 정책에 영향을 준다. eMMC 장치는 프로그램 및 삭제 수명(Program and Erase Endurance)이 제한되어 있으며 스토리지 열화(Storage Degradation)가 진행되면 읽기 또는 쓰기 오류가 발생할 수 있다. 하드웨어가 관련 지표를 제공하는 경우 수명 상태 정보, 사용 가능 수명 지표(Available Lifetime Indicators), 파일시스템 오류, 입출력 오류를 모니터링할 수 있다. 최근 검증된 이미지와 예방적 교체(Preventive Replacement)를 결합하면 예상하지 못한 스토리지 장애를 통제된 유지보수 작업으로 전환할 수 있다.

자동화(Automation)를 통해 이미지 생성, 명명, 압축, 체크섬 생성, 전송, 보존, 검증 과정을 표준화할 수 있다. 백업 스크립트는 플랫폼을 식별하고, 이미지 식별자를 생성하고, 시스템 메타데이터를 수집하고, 이미지를 생성하거나 압축하고, 체크섬을 계산하고, 아티팩트를 보호된 스토리지로 전송하고, 결과를 매니페스트(Manifest)에 기록할 수 있다. 자동화된 안전장치는 장치 식별 정보, 사용 가능한 공간 또는 대상 검증 결과가 예상 조건과 일치하지 않을 경우 프로세스를 중단해야 한다.

보존 정책(Retention Policies)은 중요한 소프트웨어 릴리스와 배포 이정표(Deployment Milestones)에 대응하는 이미지를 유지해야 한다. 최근 배포된 소프트웨어 버전에서 이후 결함이 발견될 수 있으므로 최신 이미지 하나만 유지하는 것은 위험할 수 있다. 안정적인 운영 이미지, 이전의 검증된 릴리스, 공장 기준 이미지(Factory Baselines), 주요 업그레이드 체크포인트(Upgrade Checkpoints)를 운영 요구사항에 따라 보존할 수 있다. 지원되는 하드웨어와 더 이상 호환되지 않는 이미지는 통제된 수명주기 프로세스(Lifecycle Process)를 통해 아카이빙하거나 제거해야 한다.

복원(Restoration)은 이미지 생성이 성공했다는 이유만으로 정상 동작한다고 가정하지 말고 대표 하드웨어에서 실제로 테스트해야 한다. 복구 테스트에서는 이미지를 예비 eMMC 장치 또는 호환 가능한 대상에 기록하고, 시스템을 부팅하고, 파티션과 파일시스템을 검증하고, 필요한 서비스를 시작하고, 네트워크 연결을 테스트하고, 로봇 애플리케이션을 실행하고, 연결된 하드웨어를 확인할 수 있다. 이후 로봇별 데이터를 별도로 복원하여 전체 복구 워크플로가 컨트롤러를 실제 운영 가능한 상태로 되돌리는지 검증할 수 있다.

복구 시간(Recovery Time)은 단순히 이미지를 기록하는 시간만을 의미하지 않는다. 기술자는 하드웨어를 교체하고, 복구 모드(Recovery Mode)로 진입하고, 외부 미디어를 연결하고, 대용량 이미지를 전송하고, eMMC를 플래싱(Flashing)하고, 체크섬을 검증하고, 파일시스템을 확장하고, 장치별 구성을 복원하고, 자격 증명을 프로비저닝하고, 센서를 다시 연결하고, 기능 테스트를 수행해야 할 수 있다. 주기적인 훈련에서 전체 절차를 측정하면 임베디드 시스템 복구가 요구되는 복구 시간 목표(Recovery Time Objective, RTO)를 충족하는지 현실적으로 평가할 수 있다.

플릿 규모 복구(Fleet-Scale Recovery)에서는 표준화된 이미지와 자동화된 프로비저닝(Automated Provisioning)이 큰 이점을 제공한다. 수십 대 또는 수백 대의 로봇이 호환 가능한 컴퓨팅 플랫폼을 사용하는 경우 통제된 이미지 카탈로그(Image Catalog)를 유지하면 현장 서비스의 복잡성을 줄일 수 있다. 기술자는 하드웨어 및 소프트웨어 버전에 적합한 승인 이미지를 선택하고, 기준 시스템을 복원하고, 로봇별 신원과 구성을 주입하고, 최신 운영 데이터를 동기화하고, 검증 테스트를 실행한 후 로봇을 다시 서비스에 투입할 수 있다.

성숙한 임베디드 백업 전략(Mature Embedded Backup Strategy)은 블록 수준 eMMC 이미징(Block-Level eMMC Imaging)을 자주 변경되는 로봇 데이터에 대한 별도 보호, 이미지 메타데이터, 체크섬, 보안 스토리지, 버전 관리, 검증된 복원 절차와 결합한다. 골든 이미지는 재현 가능한 시스템 기준 상태(Reproducible System Baseline)를 제공하고, 정기적인 데이터 백업은 최신 운영 상태를 보존한다. 이러한 메커니즘을 함께 적용하면 장애가 발생한 임베디드 컨트롤러에서 개별 파일만 복구하는 것이 아니라 검증된 부팅 가능 시스템(Verified Bootable System) 전체를 재구성할 수 있다.

## 09.10 Robot Fleet Disaster Recovery Scenarios and Procedures

![](images/image10.png){width="7.268055555555556in" height="7.268055555555556in"}

로봇 플릿 재해 복구 전략(Robot Fleet Disaster Recovery Strategy)은 데이터베이스, 플릿 관리 서버(Fleet-Management Server), 클라우드 서비스, 엣지 인프라(Edge Infrastructure), 네트워크, 스토리지 또는 여러 로봇에 동시에 영향을 미치는 장애 이후 분산된 로봇 운영을 어떻게 복원할 것인지를 정의한다. 단일 컴퓨터의 복구와 달리 플릿 복구에서는 중앙 서비스와 자율 엣지 장치를 조정하면서 운영 안전성을 유지해야 한다. 목표는 정의된 RTO와 RPO 안에서 정보 무결성과 통제된 로봇 운영을 모두 복원하는 것이다.

재해 시나리오(Disaster Scenarios)는 모든 장애를 동일한 사건으로 취급하기보다 장애 도메인(Failure Domains)을 기준으로 정의해야 한다. 플릿에서는 하나의 로봇 컨트롤러, 온프레미스 서버(On-Premises Server), 데이터베이스 클러스터, 공유 스토리지, 네트워크 인프라, 클라우드 리전(Cloud Region) 또는 전체 운영 사이트의 장애가 발생할 수 있다. 각 시나리오는 서로 다른 구성 요소에 영향을 주며 다른 복구 경로가 필요하다. 시나리오 기반 계획은 어떤 서비스가 계속 사용 가능하고 어떤 의존성을 재구성해야 하는지 식별하는 데 도움을 준다.

개별 로봇 장애(Individual Robot Failure)는 가장 작은 복구 도메인 중 하나이지만 여전히 체계적인 절차가 필요하다. 임베디드 컨트롤러 또는 eMMC 장치에 장애가 발생하면 로봇은 운영 환경, 로컬 구성, 캐시된 임무 또는 최근 수집 데이터를 잃을 수 있다. 복구 과정에서는 검증된 골든 이미지(Golden Image)를 복원하고, 로봇별 신원과 보정 정보를 주입하고, 최근 운영 데이터를 복원하고, 로봇을 플릿 서비스에 다시 연결한 뒤 기능 검사를 수행하여 자율 운영으로 복귀시킬 수 있다.

중앙 플릿 관리 시스템 장애(Central Fleet-Management Failure)는 로봇의 온보드 컴퓨터가 정상인 경우에도 많은 로봇에 영향을 줄 수 있다. 임무 할당, 교통 조정, 충전 관리, 지도 배포, 텔레메트리 집계(Telemetry Aggregation), 운영자 인터페이스를 사용할 수 없게 될 수 있다. 로봇은 허용된 로컬 작업 완료, 안전 위치에서 정지 또는 제한된 기능으로 운영하는 것과 같은 사전에 정의된 안전 동작(Safe Behavior)으로 전환해야 한다. 복구 절차에서는 서버 복원뿐 아니라 중앙 서비스가 손실된 동안의 플릿 동작도 정의해야 한다.

데이터베이스 손상(Database Corruption)은 단순한 인프라 장애와 다른 대응이 필요하다. 손상되거나 실수로 변경된 데이터가 이미 복제되었다면 단순히 복제본(Replica)으로 장애 조치하는 것만으로 문제를 해결할 수 없다. 이전의 유효한 복구 시점을 선택하고, 백업을 복원하고, 트랜잭션 로그(Transaction Logs)를 통제된 시점까지 적용하고, 데이터베이스 일관성을 검증한 후 애플리케이션을 다시 연결해야 할 수 있다. PostgreSQL 시점 복구(Point-in-Time Recovery, PITR), MongoDB 백업 복원, 변경 불가능한 과거 복사본(Immutable Historical Copies)이 이러한 복구를 지원할 수 있다.

스토리지 장애(Storage Failure)는 지도, AI 모델, 로그, 센서 아카이브, 구성 패키지, 데이터베이스 백업 저장소에 영향을 줄 수 있다. 복구 절차에서는 복제본, 보조 스토리지, 객체 스토리지 버전(Object-Storage Versions), NAS 복사본 또는 오프사이트 아카이브(Offsite Archives)가 여전히 사용 가능한지를 확인해야 한다. 현재 지도와 로봇 구성처럼 운영에 중요한 아티팩트는 대규모 과거 센서 아카이브보다 먼저 복원하는 것이 일반적이다. 복구 우선순위는 전체 데이터 용량보다 서비스 의존성과 운영 가치를 기준으로 결정해야 한다.

네트워크 장애(Network Failure)는 저장된 데이터를 손상시키지 않으면서 로봇과 플릿 서버 사이의 연결을 분리할 수 있다. 연결이 끊어진 동안 엣지 시스템은 사전에 정의된 정책에 따라 제한적인 로컬 기능을 계속 수행하고 텔레메트리, 임무 이벤트 또는 센서 기록을 버퍼링(Buffering)할 수 있다. 연결이 복원된 이후 동기화 과정에서는 누락된 레코드, 중복 이벤트, 오래된 명령, 충돌하는 상태를 식별해야 한다. 따라서 복구는 단순한 네트워크 재연결이 아니라 분산 상태(Distributed State)의 조정까지 포함한다.

클라우드 리전 장애(Cloud-Region Outage)가 발생하면 서비스를 다른 장애 도메인에서 재구성하거나 활성화해야 한다. 리전 간 객체 복제(Cross-Region Object Replication), 데이터베이스 복제본, 인프라 템플릿, 컨테이너 이미지(Container Images), 구성 저장소, 보호된 비밀 정보(Protected Secrets)를 이용하면 복구 시간을 단축할 수 있다. DNS, 라우팅, 인증서, 신원 서비스, 외부 연동도 함께 고려해야 한다. 데이터만 존재하고 이러한 의존성이 없는 보조 리전은 실제로 사용 가능한 플릿 관리 환경을 제공하지 못할 수 있다.

사이트 수준 재해(Site-Level Disaster)는 로컬 서버, NAS 장치, 네트워크 장비, 엔지니어링 워크스테이션, 충전 인프라, 로봇이 함께 영향을 받을 수 있기 때문에 더 광범위한 복구 시나리오가 된다. 로컬 복구 복사본을 사용할 수 없는 경우 오프사이트 백업(Offsite Backup)이 필수적이다. 조직은 생존한 로봇의 상태를 별도로 평가하면서 원격 데이터센터 또는 클라우드 환경에서 플릿 서비스를 복원해야 할 수 있다. 3-2-1 백업 원칙(3-2-1 Backup Principle)은 하나의 물리적 사고로 모든 복구 복사본이 동시에 손실되는 것을 방지하는 데 도움을 준다.

사이버보안 사고(Cybersecurity Incident)에서는 복원 이전에 격리(Isolation)가 필요할 수 있다. 랜섬웨어, 자격 증명 유출, 악의적인 구성 변경 또는 비인가 삭제가 의심되는 상황에서 복원된 시스템을 손상된 환경에 즉시 다시 연결하면 동일한 사고가 반복될 수 있다. 복구 절차에서는 신뢰할 수 있는 백업을 식별하고, 영향을 받은 자격 증명을 교체하고, 소프트웨어와 구성의 무결성을 검증하고, 복구 네트워크를 격리한 후 구성 요소를 단계적으로 다시 연결해야 한다. 변경 불가능한 백업(Immutable Backups)은 과거 복구 시점의 변조를 방지하는 추가적인 보호 기능을 제공한다.

재해 선언(Disaster Declaration)은 즉흥적인 기술 작업이 아니라 정의된 복구 워크플로(Recovery Workflow)를 시작해야 한다. 먼저 사고를 분류하고, 영향을 받은 서비스를 식별하고, 운영 영향을 평가해야 한다. 이후 복구 팀은 적절한 복구 시나리오를 선택하고, 의사소통 및 권한 체계를 확립하고, 목표 복구 시점(Target Recovery Point)을 결정하고, 남아 있는 유효한 데이터를 보호한다. 명확한 의사결정 지점(Decision Points)은 성급한 조치로 유용한 증거를 파괴하거나 복구 가능한 정보를 덮어쓰는 위험을 줄인다.

복구 우선순위(Recovery Priorities)는 의존성을 고려한 서비스 계층 구조(Dependency-Aware Service Hierarchy)를 따라야 한다. 네트워크, 신원 관리, DNS, 시간 동기화(Time Synchronization), 자격 증명, 스토리지 접근과 같은 핵심 인프라는 데이터베이스와 플릿 애플리케이션보다 먼저 복원해야 할 수 있다. 이후 데이터베이스, 지도 저장소, 메시지 브로커(Message Brokers), 구성 서비스가 플릿 관리 기능을 지원할 수 있다. 대규모 센서 아카이브와 과거 분석 데이터는 즉각적인 로봇 운영에 필요하지 않으므로 일반적으로 나중에 복원할 수 있다.

인프라 재구성(Infrastructure Reconstruction)은 가능한 한 재현 가능해야 한다. 코드형 인프라(Infrastructure as Code, IaC) 템플릿, 컨테이너 정의, 구성 관리(Configuration Management), 문서화된 네트워크 설정, 통제된 소프트웨어 저장소를 이용하면 문서화되지 않은 수작업 지식에 의존하지 않고 서버를 재구축할 수 있다. 임베디드 시스템은 검증된 eMMC 골든 이미지를 사용할 수 있으며, 클라우드와 온프레미스 플랫폼은 표준화된 배포 아티팩트를 사용할 수 있다. 재현성(Reproducibility)은 구성 편차(Configuration Drift)를 줄이고 실제 재해 이전에 복구 절차를 테스트하기 쉽게 만든다.

데이터 복원(Data Restoration)은 단순히 가장 최신 파일을 선택하는 것이 아니라 검증된 복구 시점(Verified Recovery Point)을 사용해야 한다. 백업 매니페스트, 타임스탬프, 체크섬, 트랜잭션 로그, 복제 상태, 검증 기록을 통해 복구 시점의 신뢰성을 판단할 수 있다. 가능하면 선택된 데이터를 격리된 환경(Isolated Environment)에 복원한 후 구조적 및 논리적 일관성을 검사해야 한다. 이는 특히 데이터 손상, 우발적인 삭제 또는 악의적인 변경이 재해 원인인 경우 중요하다.

중앙 서비스를 사용할 수 없는 동안 로봇이 계속 동작하거나 정보를 버퍼링했다면 플릿 상태 조정(Fleet-State Reconciliation)이 필요하다. 복구된 서버에는 로봇 위치, 임무 상태, 충전 상태 또는 유지보수 이벤트에 대한 오래된 정보가 존재할 수 있으며, 동시에 로봇에는 더 최신의 로컬 정보가 존재할 수 있다. 복구 절차에는 권위 있는 상태(Authoritative State)를 결정하고, 오래된 명령을 거부하고, 중복 레코드를 해결하고, 안전하지 않거나 논리적으로 모순된 플릿 동작을 만들지 않으면서 버퍼링된 이벤트를 동기화하는 규칙이 필요하다.

로봇 서비스 복귀(Return-to-Service)는 전체 플릿을 동시에 다시 연결하는 방식이 아니라 통제된 방식으로 진행해야 한다. 먼저 소수의 대표 로봇을 복구된 환경에 연결하여 인증, 지도, 임무 교환, 텔레메트리, 충전 인터페이스, 교통 조정을 검증할 수 있다. 이러한 검사가 성공하면 추가 로봇을 단계적으로 연결할 수 있다. 점진적 복원(Progressive Restoration)은 인프라와 애플리케이션 검증 과정에서 발견되지 않은 구성 오류가 전체 운영에 미치는 영향을 제한한다.

안전 검증(Safety Validation)은 로봇 재해 복구에서 독립적인 요구사항이다. 데이터베이스와 서버가 성공적으로 복원되었다고 해서 로봇이 즉시 자율 주행을 시작할 준비가 되었다는 의미는 아니다. 내비게이션 지도, 지오펜스(Geofences), 보정 정보, 위치추정 기준(Localization References), 안전 파라미터, 소프트웨어 버전, 임무 규칙을 운영 재개 전에 검증해야 한다. 불확실성이 남아 있는 경우 운영자가 복원된 디지털 환경과 실제 물리적 운영 환경이 일치한다고 확인할 때까지 로봇은 안전 상태(Safe State)를 유지해야 한다.

복구 시간 목표(Recovery Time Objective, RTO)와 복구 시점 목표(Recovery Point Objective, RPO)는 각 복구 시나리오에 대해 측정 가능한 승인 기준을 제공한다. 핵심 플릿 서비스는 과거 센서 아카이브나 개발 데이터셋보다 더 짧은 복구 시간과 더 작은 데이터 손실 허용 범위를 요구할 수 있다. 측정에는 사고 선언, 의사결정, 인프라 재구성, 데이터 복원, 검증, 로봇 재연결, 운영 승인까지 포함해야 한다. 로보틱스 플랫폼 전체에 하나의 공통 복구 목표를 적용하는 것보다 시나리오별 측정이 더 유용한 증거를 제공한다.

플릿 재해 상황에서는 의사소통 절차(Communication Procedures)가 기술적 절차만큼 중요하다. 책임 체계에서는 누가 사고를 선언하고, 누가 복구 인프라를 통제하며, 누가 복구 데이터를 선택하고, 누가 사이버보안을 검증하고, 누가 로봇 안전을 확인하며, 누가 서비스 복원을 승인하는지를 정의해야 한다. 상태 정보는 공통 사고 타임라인(Incident Timeline)을 통해 기록해야 한다. 명확한 의사소통은 여러 팀이 데이터베이스, 스토리지, 네트워크 또는 로봇에서 서로 충돌하는 복구 작업을 수행하는 것을 방지한다.

복구 런북(Recovery Runbooks)은 아키텍처를 실행 가능한 절차로 변환한다. 런북에는 사전 조건, 필요한 도구, 백업 위치, 의존성, 검증 단계, 에스컬레이션 경로(Escalation Paths), 다음 복구 단계로 진행하기 위한 기준을 명시해야 한다. 원래 시스템을 설계하지 않은 숙련된 담당자도 실행할 수 있을 정도로 절차가 구체적이어야 한다. 자동화는 반복 가능한 기술 단계를 수행할 수 있으며, 런북은 이러한 자동화 작업을 둘러싼 운영 맥락과 의사결정 논리를 제공한다.

주기적인 재해 복구 훈련(Periodic DR Drills)은 실제 운영 환경에서 필요해지기 전에 플릿 복구 절차를 검증해야 한다. 테스트에서는 데이터베이스 손실, 스토리지 장애, 중앙 서비스 중단, 네트워크 분할(Network Partition), 엣지 컴퓨터 교체 또는 리전 장애를 모의할 수 있다. 각 훈련에서는 복구 시간을 측정하고, 복구 시점을 검증하고, 수동 개입을 기록하고, 누락된 의존성을 식별하며, 로봇 동작을 확인해야 한다. 훈련에서 발견된 사항은 백업 정책, 자동화, 인프라, 런북에 대한 추적 가능한 개선 사항으로 연결되어야 한다.

백업 아키텍처(Backup Architecture)는 여러 계층의 복구 방식을 지원해야 한다. 로컬 복제본과 대기 서비스(Standby Services)는 일반적인 인프라 장애에서 빠른 복구를 제공할 수 있고, 과거 백업은 데이터 손상이나 삭제로부터 복구할 수 있으며, 오프사이트 복사본은 사이트 수준 재해로부터 보호하고, 골든 임베디드 이미지(Golden Embedded Images)는 장애가 발생한 로봇 컨트롤러를 재구축할 수 있다. 하나의 메커니즘만으로 모든 재해 시나리오를 처리할 수 없다. 플릿 복원력(Fleet Resilience)은 가용성, 백업, 격리, 버전 관리, 복제, 검증된 재구성 절차를 결합함으로써 형성된다.

성숙한 로봇 플릿 재해 복구 프로세스(Mature Robot Fleet Disaster Recovery Process)는 사고 탐지부터 안전한 운영 복원까지 통제된 순서를 형성한다. 조직은 장애 도메인을 식별하고, 남아 있는 데이터를 보호하고, 검증된 복구 시점을 선택하고, 인프라를 재구성하고, 데이터와 의존성을 복원하고, 애플리케이션을 검증하고, 분산된 로봇 상태를 조정하고, 로봇을 단계적으로 다시 연결한 뒤 정상 운영을 재개하기 전에 안전성을 확인한다. 반복적인 DR 훈련은 플릿과 인프라가 변화하더라도 이러한 절차가 지속적으로 유효하다는 증거를 제공한다.
