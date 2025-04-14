# Conncted-Car-Controller
차량 내 편의 기능을 무선으로 제어할 수 있는 임베디드 시스템과 차량 편의 기능 조작 어플리케이션


## 기능 구성도
![기능구성도](https://github.com/user-attachments/assets/fdbc6eaa-85c4-44ea-ab1e-d80eba6d7139)

## 시연 영상
[![시연영상_최종본_커버](https://github.com/user-attachments/assets/7b01780c-6ee0-4be4-92dd-e2fe8999e99d)](https://youtu.be/Mb-joNRislY)


## 하드웨어 구성
### 시스템 아키텍처
![시스템아키텍쳐](https://github.com/user-attachments/assets/179bceb4-3356-4d23-ba89-1d335de0ba12)


## 소프트웨어 구성
### 기능 흐름도
![기능흐름도](https://github.com/user-attachments/assets/eea84c9f-b560-480b-b41d-ce3ff3ac5579)

### SW 단위 설계
#### iOS 앱 모듈 구조
![iOS모듈구조](https://github.com/user-attachments/assets/892b297e-f148-46b0-865e-373091186e11)
#### ESP32 모듈 구조
![EPS모듈구조](https://github.com/user-attachments/assets/9c588a73-865b-4874-8534-e89a0ee43569)
#### STM32 모듈 구조
![STM32모듈](https://github.com/user-attachments/assets/ea27663d-ac9d-4251-9ac8-33e244897b0f)

### SW 인터페이스 설계
<details>
<summary>01 iOS앱-BLE 통신 인터페이스</summary>
<div>

| 구분 | 항목 | 설명 |
| --- | --- | --- |
| 서비스 UUID | `12345678-1234-5678-1234-56789abcdef0` | ESP32가 Advertising하는 메인 서비스 UUID |
| 기기 이름 | `"ESP32-BLE-WoonKwan"` | 앱에서 필터링할 장치명 |
| 연결 방식 | Central-Peripheral (iOS가 Central) |  |
| 연결 절차 | iOS에서 Scan → 서비스 UUID 필터 → 연결 성공 시 특성 탐색 |  |
| 특성 UUID 목록 | FAN, MP3, TEM, NAV | 각 기능별로 BLECharacteristic 등록됨 |
| 연결 성공 후 | iOS는 BLE Write 또는 Notify 수신 가능 |  |
- **인터페이스 목록**(IF-BLE-000)
    
    
    | 인터페이스 번호 | 명령 목적 | 송신 시스템 | 송신 방식 | 수신 시스템 | 수신 처리 | 관련 UUID |
    | --- | --- | --- | --- | --- | --- | --- |
    | IF-BLE-001 | BLE 연결 요청 | iOS 앱 | BLE Scan + Connect | ESP32 | 서버 모드로 서비스 광고 시작 | `123456789-1234-5678-1234-56789abcdef0` |
    | IF-BLE-002 | FAN 제어 | iOS 앱 | BLE Write | ESP32 | STM32에 UART로 전달 | `11111111-2222-3333-4444-5555abcdef01` |
    | IF-BLE-003 | MP3 제어 | iOS 앱 | BLE Write | ESP32 | STM32에 UART로 전달 | `22222222-3333-4444-5555-6666abcdef01` |
    | IF-BLE-004 | 온도 조회 | iOS 앱 | BLE Write | ESP32 | STM32에 요청 후 BLE Notify | `33333333-4444-5555-6666-7777abcdef01` |
    | IF-BLE-005 | 주소 검색 | iOS 앱 | BLE Write | ESP32 | STM32에 요청 후 BLE Notify | `44444444-5555-6666-7777-8888abcdef01` |
- **인터페이스 명세**
    - **IF-BLE-001 – BLE 연결 요청**
        
        
        | 구분 | 항목 | 값 |
        | --- | --- | --- |
        | 송신 형식 | 명령어 | `BLE_CONNECT` |
        | 송신 UUID | 특성 UUID | `12345678-1234-5678-1234-56789abcdef0` |
        | 수신 처리 | ESP32가 BLE 연결 요청을 수신하고, 서비스 광고 시작 |  |
        | 응답 | 없음 |  |
        | 응답 UUID | 없음 |  |
    - **IF-BLE-002 – FAN 제어**
        
        
        | 구분 | 항목 | 값 |
        | --- | --- | --- |
        | 송신 형식 | 명령어 | `FAN:a`, `FAN:b`, `FAN:c`,`FAN:s` ,`FAN:0` |
        | 송신 UUID | 특성 UUID | `11111111-2222-3333-4444-5555abcdef01` |
        | 수신 처리 | ESP32가 UART로 STM32에 명령 전송 |  |
        | 응답 | `FAN:1` ~ `FAN:3` (BLE Notify) |  |
        | 응답 UUID | 동일 |  |
    - **IF-BLE-003 – MP3 제어**
        
        
        | 구분 | 항목 | 값 |
        | --- | --- | --- |
        | 송신 형식 | 명령어 | `MP3:1`, `MP3:0`, `MP3:r` ,`MP3:p`,`MP3:n`,`MP3:u`,`MP3:d`,`MP3:s` |
        | 송신 UUID | 특성 UUID | `22222222-3333-4444-5555-6666abcdef01` |
        | 수신 처리 | ESP32가 UART로 STM32에 명령 전송 |  |
        | 응답 | 예: `MP3:5`~`MP3:30`(BLE Notify) |  |
        | 응답 UUID | 동일 |  |
    - **IF-BLE-004 – 온도 조회**
        
        
        | 구분 | 항목 | 값 |
        | --- | --- | --- |
        | 송신 형식 | 명령어 | `TEM:s` |
        | 송신 UUID | 특성 UUID | `33333333-4444-5555-6666-7777abcdef01` |
        | 수신 처리 | STM32가 센서 측정 후 ESP32에 UART 응답 |  |
        | 응답 | 예: `TEM:22.5` (℃) |  |
        | 응답 UUID | 동일 |  |
    - **IF-BLE-005 – 주소 검색**
        
        
        | 구분 | 항목 | 값 |
        | --- | --- | --- |
        | 송신 형식 | 명령어 | `NAV:a` ~ `NAV:d`  |
        | 송신 UUID | 특성 UUID | `44444444-5555-6666-7777-8888abcdef01` |
        | 수신 처리 | STM32가 주소 검색 후 결과 전달 |  |
        | 응답 | `NAV:OK=city` / `NAV:Fail` |  |
        | 응답 UUID | 동일 |  |
- **통신 방식 요약**
    - **전송 방식**: BLE Write (iOS → ESP32), BLE Notify (ESP32 → iOS)
    - **명령 포맷**: `"기능:값"` 형식 (예: `FAN:2`, `MP3:a`)
    - **BLE UUID 기반 분기 처리**:
        - 송신 시 각 기능 별 UUID 사용
        - 수신 시 ESP32가 CentralCommandRouter를 통해 BLE → UART 매핑
- **예외 처리 정의**
    
    
    | 예외 상황 | 처리 방식 |
    | --- | --- |
    | UUID 없음 | 로그 출력 후 무시 |
    | 명령 포맷 오류 (`FAN=3`) | 무시 + `[CMD] 잘못된 명령 형식` 출력 |
    | STM 응답 없음 | BLE Notify 미수신, 로그 출력 |

</div>
</details>

<details>
<summary>02 ESP32 - STM32 UART 통신 인터페이스</summary>
<div>

| 항목 | 내용 |
| --- | --- |
| 통신 방식 | UART (양방향) |
| 보드 간 역할 | ESP32 → 송신, STM32 → 수신 및 응답 |
| 전송 속도 | 9600 bps |
| 명령어 포맷 | 고정 길이 5바이트 명령 (`기능:값`) |
| 수신 방식 | STM32에서 HAL_UART_Receive_IT 기반 인터럽트 수신 |
| 응답 방식 | STM32에서 HAL_UART_Transmit으로 응답 전송 (필요 시) |
- **인터페이스 목록(IF-UART-000)**
    
    
    | 인터페이스 번호 | 명령 목적 | 송신 시스템 | 송신 방식 | 수신 시스템 | 수신 처리 방식 | 응답 방식 |
    | --- | --- | --- | --- | --- | --- | --- |
    | IF-UART-001 | FAN 제어 | ESP32 | UART 송신 | STM32 | FAN 기능 수행 | UART 응답 (선택적) |
    | IF-UART-002 | MP3 제어 | ESP32 | UART 송신 | STM32 | MP3 모듈 제어 | UART 응답 (선택적) |
    | IF-UART-003 | 온도 요청 | ESP32 | UART 송신 | STM32 | 온도 센서 값 측정 | UART 응답 |
    | IF-UART-004 | 주소 검색 입력 | ESP32 | UART 송신 | STM32 | 주소 입력 및 검색 수행 | UART 응답 |
    
- **인터페이스 명세**
    - **IF-UART-001 – FAN 제어**
        
        
        | 구분 | 항목 | 값 |
        | --- | --- | --- |
        | 송신 형식 | 명령어 | `FAN:0` (OFF), `FAN:a` (약풍), `FAN:b` (중풍), `FAN:c` (강풍), `FAN:s` (상태 요청) |
        | 수신 처리 | STM32에서 팬 구동 제어 및 현재 풍속 반영 |  |
        | 응답 형식 | `FAN:a`~`FAN:c` (현재 풍속, 상태 요청 시) |  |
    - **IF-UART-002 – MP3 제어**
        
        
        | 구분 | 항목 | 값 |
        | --- | --- | --- |
        | 송신 형식 | 명령어 | `MP3:1` (재생), `MP3:0` (정지),`MP3:r` (랜덤), `MP3:n` (다음), `MP3:p` (이전), `MP3:u` (볼륨 업), `MP3:d` (볼륨 다운), `MP3:s` (상태 요청) |
        | 수신 처리 | STM32에서 MP3 모듈 제어 |  |
        | 응답 형식 | `MP3:볼륨&트랙` (예: `MP3:1&5`) |  |
    - **IF-UART-003 – 온도 조회**
        
        
        | 구분 | 항목 | 값 |
        | --- | --- | --- |
        | 송신 형식 | 명령어 | `TEM:s` (온도 상태 요청) |
        | 수신 처리 | STM32가 센서를 통해 현재 온도 측정 |  |
        | 응답 형식 | `TEM:22` (예: 현재 온도 22도) |  |
    - **IF-UART-004 – 주소 검색**
        
        
        | 구분 | 항목 | 값 |
        | --- | --- | --- |
        | 송신 형식 | 명령어 | `NAV:1` (입력 시작), `NAV:0` (입력 종료 및 검색), `NAV:a~z` (도시 입력 문자) |
        | 수신 처리 | STM32가 입력된 도시명으로 주소 검색 수행 |  |
        | 응답 형식 | `NAV:OK=city` / `NAV:Fail` |  |
    
- **예외 처리 정의**
    
    
    | 예외 상황 | 처리 방식 |
    | --- | --- |
    | 명령 포맷 오류 (`FAN=2`) | 무시 후 `[UART] 잘못된 명령 형식` 로그 |
    | 등록되지 않은 명령 키워드 | `parseCommand()` 내에서 무시 |
    | 응답 없음 | BLE Notify 미전송, 내부 로그 출력 처리 |

</div>
</details>

<details>
<summary>03 STM32 - LCD 출력 인터페이스</summary>
<div>

| 항목 | 내용 |
| --- | --- |
| 디바이스 | STM32 NUCLEO-L073RZ, LCD 1602 Keypad Shield |
| 통신 방식 | 4비트 병렬 모드 (GPIO 직접 제어) |
| 연결 핀 | RS, EN, D4~D7,VCC,GND(STM32 GPIO 핀에 연결) |
| 전원 | 5V 전원 공급, GND 공유 |
- **인터페이스 명세**
    
    
    | 인터페이스 ID | 함수명 | 입력 파라미터 | 설명 |
    | --- | --- | --- | --- |
    | IF-LCD-01 | `void lcdInit(void)` | 없음 | LCD 초기화 |
    | IF-LCD-02 | `void lcdClearDisplay()` | 없음 | 현재 LCD에 출력된 모든 문자를 지움 |
    | IF-LCD-03 | `void lcdSetCursor(int row, int col)` | row: 출력 행 번호 col:  출력 열 번호 | LCD 출력 위치(커서 위치)를 지정함 |
    | IF-LCD-04 | `void lcdSendString(char *str)` | str: 출력할 문자열 포이터 | LCD의 지정된 위치에 문자열을 출력함 |
- **출력 데이터 포맷**
    
    **[에어컨]**
    
    - **에어컨 On: 1단계, 2단계, 3단계**
        
        ```jsx
        #1 FAN ON
        #2 STEP 1
        ```
        
    - **에어컨 Off**
        
        ```jsx
        #1 FAN OFF
        #2 
        ```
        
    
    **[MP3]**
    
    - **재생 / 랜덤 재생 / 이전 트랙 / 다음 트랙**
        
        ```c
        #1 Music Track: 1
        #2 Volume: 15
        ```
        
    - **정지 후 다시 재생**
        
        ```c
        #1 Resume Tack: 1
        #2 Volume: 15
        ```
        
    - **정지**
        
        ```c
        #1 Music Stop
        #2 
        ```
        
    - **볼륨 업**
        
        ```c
        #1 Volume: 10
        #2
        ```
        
        ```jsx
        // 볼륨 최대인 경우
        #1 Volume: 30
        #2 Volume Max
        ```
        
    - **볼륨 다운**
        
        ```jsx
        #1 Volume: 5
        #2
        ```
        
        ```jsx
        // 볼륨 최소
        #1 Volume: 0
        #2 Volume Min
        ```
        
    
    **[네비게이션]**
    
    - **문자 입력(a~z)**
        
        ```jsx
        #1 Location...
        #2 s
        ```
        
        ```jsx
        #1 Location..
        #2 se
        ```
        
    - **주소를 찾은 경우**
        
        ```jsx
        To: busan
        35N, 129E
        ```
        
    - **주소를 못 찾은 경우**
        
        ```jsx
        #1 Not Found
        #2 
        ```

</div>
</details>

## 개발 환경
- **SW 개발 환경**
    
    
    | OS | Windows 10, macOS |
    | --- | --- |
    | 개발환경(IDE) | Xcode, Arduino IDE, KEIL MDK-ARM |
    | 개발도구 | Visual Studio Code, STM32CubeMX |
    | 개발언어 | Swift, C |
- **HW 개발 환경**
    
    
    | 디바이스 | iPhone, STM32 NUCLEO-L073RZ, ESP32 DevKit  |
    | --- | --- |
    | 센서 | LM35 |
    | 부품 | 세라믹 커패시터, DC 5V 팬, 릴레이 모듈, DFPlayer(DFR0299), Micro SD Card 32GB, 스피커, Micro SD Card Adapter, LCD 1602 Keypad Shield(WK-ADB-M007), 브레드보드, 점프선, 소켓 점퍼 케이블 |

## 프로젝트 구조
```bash
Connected-Car-Controller
├── ESP32/    # ESP32 코드 
├── iOS/      # iOS앱 코드
└── STM32/    # STM32 코드
```

## 프로젝트 관리
| 구분 | 방법 | 상세 내용 |
| --- | --- | --- |
| 형상관리 | GitHub| 버전 관리 및 협업 코드 공유 <br> * Git Flow 전략 적용 <br> - main, develop, feature 브랜치 구분 <br> - 안정적인 코드 병합 및 배포 관리 |
| 의사소통관리 | Notion| 문서 및 회의 기록 공유 <br> - 일정 관리 및 역할 분담 <br> - 회의록, To-Do 리스트 실시간 관리) |
- 매일 **오전 9시 팀 회의** 진행
    - 전날 작업 내용 공유 및 당일 할 일 정리
    - 진행 상황 확인 및 이슈 논의
 
## 개발 일정
| 날짜 | 주요 내용 | 세부 작업 내용 |
| --- | --- | --- |
| 03/18(화) | 사전 기획 | - 프로젝트 기획 및 주제 선정 <br> - 팀원 역할 분배 <br>- 일정 분기점 설정 |
| 03/19(수) | 사용 부품 테스트 | - 각 센서 및 모듈별 작동 확인 <br> - 통신 테스트 및 초기 점검 |
| 03/20(목) | 시스템 설계 및 상세 설계 | - 기능 모듈화 및 코드 구조 설계 <br> - 파일 분리 및 설계 명세 작성 |
| 03/21(금) | 단위 테스트 | - 개별 기능 단위 테스트 수행 (LCD, MP3, 통신 등) |
| 03/22(토) | CubeMX 및 인터페이스 설계 | - STM32CubeMX 설정 <br> - 각 부품 간 인터페이스 통신 설계 |
| 03/24(월) | SW/HW 통합 | - 소프트웨어 기능 통합 <br> - 하드웨어 부품 연동 통합 |
| 03/25(화) | HW 프레임 제작 및 통합 테스트 | - 브레드보드 또는 구조물 조립 <br> - 전체 시스템 통합 테스트 |
| 03/26(수) | 산출물 정리 및 인수 테스트 | - 결과 보고서 및 발표자료 정리 <br> - 최종 인수 테스트 수행 |

## 팀 구성원
| 이름 | Github | 담당 부품 | 담당 업무 |
| --- | --- | --- | --- |
| 김윤아 |  [@Kyalivia](https://github.com/Kyalivia) | LCD 1602 Keypad Shield, Micro SD Card Adapter | - 기능별 상태 LCD 출력 구현 <br> - SD카드 파일 읽기 및 SPI 통신 처리 |
| 송종찬 | [@sjch1127](https://github.com/sjch1127)  | DFPlayer, 스피커 | - DFPlayer 제어 <br> - DFPlayer ↔ STM32 간 UART 통신 처리 |
| 안수현 | [@justansu](https://github.com/justansu)  | LM35, DC 5V팬 | - LM35 온도 센싱 연동 <br> - DC 팬 제어 <br> - STM32 ↔ ESP32 양방향 통신 구성 |
| 정운관 | [@UnGwan](https://github.com/UnGwan)  | ESP32 | - iOS 앱 ↔ ESP32간 BLE 통신 구성 <br> - ESP32 ↔ STM32 간 UART 통신 처리 |
