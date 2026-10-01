# STM32CubeMX <br> – Embedded Software Package Manager <br>서드파티 팩 정리

STM32CubeMX의 Embedded Software Package Manager에 나오는 이 항목들은 ST가 아닌 <br>
**서드파티(파트너) 업체가 제공하는 소프트웨어 팩(CMSIS-Pack 형식)** 입니다. <br>
RTOS, 통신 스택, 보안, 클라우드 연동 라이브러리를 CubeMX 안에서 바로 설치하고 프로젝트에 넣을 수 있게 해줍니다.

> 팩 구성은 CubeMX 버전마다 조금씩 달라서, 아래 설명 중 일부는 설치 후 팩 설명(Details)으로 확인하는 것을 권장합니다.

---

## 1. 항목별 요약

| 업체 | 내용 | 쓰는 경우 |
|---|---|---|
| **ITTIA_DB** | 임베디드용 SQL/시계열 DB (ITTIA DB Lite 등) | 센서 데이터를 MCU 내부에 <br>구조적으로 저장·조회하고 싶을 때 |
| **Infineon** | OPTIGA Trust M 같은 보안 칩, <br>AIROC Wi-Fi/BT 등 Infineon 디바이스용 드라이버·미들웨어 | STM32에 Infineon 보안 칩이나 무선 모듈을 붙일 때 |
| **RealThread** | RT-Thread RTOS (Nano 등) | 중국권에서 많이 쓰는 RTOS를 쓰고 싶을 때. FreeRTOS 대안 |
| **SEGGER** | embOS(RTOS), emWin(GUI), <br>emFile, emUSB, RTT, SystemView 등 | 상용급 RTOS/GUI가 필요하거나, <br>J-Link 기반 RTT 로그와 SystemView 분석을 쓰고 싶을 때 |
| **WES** | Weston Embedded Solutions의 <br>µC/OS-II·III 및 µC/TCP-IP, µC/FS, µC/USB 등 (옛 Micrium) | µC/OS 기반 레거시 자산이 있거나 <br>안전 인증용 RTOS가 필요할 때 |
| **emotas** | CANopen, CANopen FD, <br>J1939 등 CAN 상위 프로토콜 스택 | 산업용 CAN 기기(드라이브, I/O 노드)를 만들 때 |
| **quantropi** | 양자내성암호(PQC) 및 양자 보안 라이브러리 | 장기 보안이 필요한 장비에 PQC 대응을 미리 넣고 싶을 때 |
| **wolfSSL** | TLS/DTLS, 암호 라이브러리(wolfCrypt), <br>wolfMQTT, wolfSSH, wolfBoot 등 | MQTT/HTTPS 암호화 통신, 보안 부트, <br>서명 검증이 필요할 때. <br>STM32 하드웨어 암호 가속 지원 |
| **Avnet-IOTCONNECT** | Avnet IOTCONNECT 클라우드(AWS/Azure 기반)에 <br>디바이스를 붙이는 클라이언트 | 디바이스 등록, 텔레메트리, 원격 명령, <br>OTA를 클라우드 플랫폼으로 빠르게 구성할 때 |
| **Cesanta** | Mongoose 임베디드 웹서버/네트워크 라이브러리 <br>(HTTP, WebSocket, MQTT, TCP/IP) | 장비에 웹 UI나 REST API를 올릴 때. <br>자체 TCP/IP 스택과 STM32 이더넷 드라이버 포함 |
| **Embedded Office** | CANopen 스택 등 산업 통신 미들웨어 <br>(µC/CANopen 계열) | CANopen 기기를 만들 때. emotas와 용도가 겹침 |

---

## 2. 초기 설정 순서 (공통)

1. CubeMX에서 **Help → Manage embedded software packages** 를 엽니다 (단축키 `Alt+U`).
2. 업체 항목을 펼쳐 원하는 팩과 버전에 체크하고 **Install Now** 를 누릅니다.
   - 인터넷이 필요하고, 라이선스 동의나 업체 계정 로그인을 요구하는 팩도 있습니다.
   - 설치 위치는 보통 `~/STM32Cube/Repository/Packs` 입니다.
   - 인터넷이 안 되는 환경이면 **From Local** 로 `.pack` 파일을 직접 설치합니다.
3. 프로젝트를 만들 때 MCU/보드를 선택한 뒤, 상단 메뉴 **Software Packs → Select Components** 를 엽니다.
4. 사용할 컴포넌트에 체크합니다 (예: wolfSSL Core, SEGGER RTT).
5. **Pinout & Configuration → Software Packs** 아래에 나타난 항목에서 옵션을 설정합니다.
6. 프로젝트를 생성하고 CubeIDE, Keil, IAR 등에서 빌드합니다.

### 주의사항

- RTOS 계열(RT-Thread, embOS, µC/OS, FreeRTOS)은 **서로 동시에 쓰면 안 됩니다.** 하나만 선택하세요.
- 상용 라이브러리(SEGGER embOS/emWin, emotas, quantropi, WES 등)는 평가판이나 제한 라이선스인 경우가 많으니 약관을 꼭 확인하세요.
- 네트워크 스택(Mongoose, µC/TCP-IP)은 LwIP와 겹치지 않게 선택해야 합니다.

---

## 3. 간단한 예제

### ① SEGGER RTT 로그 (UART 없이 디버그 출력)

```c
#include "SEGGER_RTT.h"

SEGGER_RTT_Init();
SEGGER_RTT_printf(0, "Hello STM32, cnt=%d\n", cnt);
```

J-Link가 연결되어 있으면 RTT Viewer로 로그를 볼 수 있습니다.

### ② wolfSSL SHA-256 해시 (wolfCrypt만 사용)

```c
#include "wolfssl/wolfcrypt/sha256.h"

wc_Sha256 sha;
byte hash[WC_SHA256_DIGEST_SIZE];

wc_InitSha256(&sha);
wc_Sha256Update(&sha, (byte*)"hello", 5);
wc_Sha256Final(&sha, hash);
```

TLS까지 쓰려면 소켓 계층(LwIP 등)과 `wolfSSL_Init()`, `wolfSSL_CTX_new()` 흐름이 추가로 필요합니다.

### ③ Mongoose 간단 HTTP 서버

```c
static void fn(struct mg_connection *c, int ev, void *ev_data) {
  if (ev == MG_EV_HTTP_MSG)
    mg_http_reply(c, 200, "", "Hello from STM32\n");
}

struct mg_mgr mgr;
mg_mgr_init(&mgr);
mg_http_listen(&mgr, "http://0.0.0.0:80", fn, NULL);
for (;;) mg_mgr_poll(&mgr, 10);
```

이더넷 지원 보드(Nucleo-F767ZI 등)에서 IP를 설정하면 브라우저로 접속해 볼 수 있습니다.

---

## 4. 용도별 추천

| 용도 | 추천 팩 |
|---|---|
| 로그/디버깅 | SEGGER (RTT, SystemView) |
| 보안 통신, 보안 부트 | wolfSSL, Infineon OPTIGA |
| 웹 UI, 간단한 네트워크 | Cesanta Mongoose |
| 산업용 CAN | emotas 또는 Embedded Office |
| 클라우드 연동 | Avnet IOTCONNECT |
| 로컬 데이터 저장 | ITTIA DB |
| 다른 RTOS 실험 | RealThread (RT-Thread), WES (µC/OS) |
| 미래 보안 대응 | quantropi |

> 교육용 실습이라면 **SEGGER RTT, wolfSSL, Mongoose** 세 가지가 구성하기 가장 쉽습니다.
