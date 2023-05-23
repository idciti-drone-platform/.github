# IDCITI ADS (Anti Drone System)
IDCITI ADS (비공식) Documentation 입니다.

## Index
1) Update Summary
2) Repositories
3) Getting Started
4) Important Notes

***

## Update Summary

### Old
- DJI Aeroscope G8의 웹 클라이언트를 주기적으로 크롤링하여 G8이 스캔한 드론을 지도에 시각화
- Onvif 프로토콜 사용해 PTZ 캠 RTSP 스트리밍 지원

### New
- UI 업데이트: DR Platform의 UI 스타일 채용
- ADSB: OpenSkyNetwork API와 컴퓨터에 연결된 SDR 장비 (RTL-SDR 또는 HackRF-One) 사용하여 항공기 식별 가능
- 웹 크롤링 대신 ssh 포트 포워딩을 통해 G8의 Dockerized MongoDB 에 접속하여 G8 이 스캔한 드론 정보 실시간으로 가져오기
- Onvif 프로토콜 사용해 PTZ 캠 RTSP 스트리밍 지원
- Pelco-D 프로토콜 사용해 PTZ 캠을 물리/소프트웨어 컨트롤러로 조종 시 방향각과 줌값 시각화
- 날짜별 드론 탐지 기록 가능: PTZ 캠 스트리밍 화면을 24시간 단위로 연속 녹화하며 녹화본 영상에서 드론이 탐지된 시각을 찾아서 재생 가능
    * 아직 G8 Activation이 되지 않아 해당 기능은 테스트를 하지 않았습니다. 추후 Activation 이 완료되고 드론 탐지 기록이 G8 DB에서 실시간으로 저장되는것이 확인되면 테스트가 가능합니다.

## Repositories

### 1. UI 관련
- **[dp-client-new](https://github.com/idciti-drone-platform/dp-client-new):** 기존 DR Platform UI를 기반하여 만들어진 React 클라이언트 입니다.
- **[dp-client](https://github.com/idciti-drone-platform/dp-client):** 여러 다른 ADS들의 UI를 참고하여 개발중인 새로운 React 클라이언트 입니다. 현재 개발 보류중 입니다.
  * [Figma UI 스케치](https://www.figma.com/file/PUxoNvb3KMiDiqu4Rx4rh1/drplatform?type=design&node-id=0-1&t=BwK7nRwugxNCOYHt-0)

### 2. G8 관련
- **[dp-server](https://github.com/idciti-drone-platform/dp-server):** G8 DB에 접속해서 drone flight records를 받아오는 Restful API가 마련된 서버
- **[dp-crawler](https://github.com/idciti-drone-platform/dp-crawler):** G8 웹 클라이언트를 크롤링하는 서버
- **[dp-log-server](https://github.com/idciti-drone-platform/dp-log-server):** 날짜별 드론 탐지 내역을 영상 기록으로 확인 가능한 Restful 서버

### 3. ADSB
- **[dp-adsb-server](https://github.com/idciti-drone-platform/dp-adsb-server):** 연결된 SDR 장비로부터 수신되는 SBS-1 패킷을 decode 하는 서버

### 4. PTZ 캠 스트리밍 및 컨트롤 관련
- **[dp-ptz-controller](https://github.com/idciti-drone-platform/dp-ptz-controller):** Onvif (캠 팬/틸트/스트리밍), YoloV4 (Dataset training) 사용해 캠 화면상 드론 식별 및 추적 기능 지원
- **[dp-ptz-streaming-server](https://github.com/idciti-drone-platform/dp-ptz-streaming-server):** Onvif 사용하여 PTZ 캠을 스트리밍 해주는 서버
- **[dp-pelcod-server](https://github.com/idciti-drone-platform/dp-pelcod-server):** Pelco-D 패킷을 decode하여 PTZ 캠 상태 (팬/틸트/줌) 실시간 추적하는 서버

## Getting Started
이 섹션은 ADS 서버와 클라이언트를 실행하는 순서와 방법을 개괄적으로 다루고 있습니다.

> 현재 환경에서는 GPS 신호 생성을 위한 SDR 장비가 연결되어있지 않아 Spoofing 관련 기능 관련 설명을 생략하였습니다.

### Hardware Requirements

#### Computers
a. React 클라이언트가 실행되고 있는 NUC (우분투 20.04)

b. PTZ 캠이 연결된 NUC (윈도우 10)

c. Aeroscope G8과 연결된 컴퓨터 (우분투 16.04)

#### Peripherals
d. SDR 장비: ADSB 기능 구현을 위한 장비이므로 둘 중 하나만 있어도 됩니다.
   * HackRF-One
   * RTL-SDR
  
e. PTZ 캠
   * XNZ-L6320 (카메라 본체)
   * SPC-2010 (물리 컨트롤러)

### How To Run

#### 1. Connect Peripheral Devices
* (d) ADSB용 SDR 장비 (HackRF-One or RTL-SDR)를 (a) NUC 에 마운트합니다.
* PTZ 본체와 컨트롤러를 (b) NUC에 연결 *(자세한 설명 필요)*
* 신호 생성 SDR 장비 연결 *(자세한 설명 필요)*

#### 2. G8에서 쉘 스크립트 실행
* `/home/aeroscope` 디렉터리에 `start.sh`, `status.sh`, `stop.sh` 등의 docker 관련 쉘 스크립트가 있습니다. `sudo ./start.sh $UID` 로 docker를 활성화합니다.
* `sudo ./status.sh` 로 각 컨테이너의 상태를 테이블 포맷으로 확인 가능하며, `sudo ./stop.sh` 로 모든 컨테이너를 종료할 수 있습니다.

#### 3. Portforward Dockerized MongoDB
로컬 네트워크에서 G8의 MongoDB 를 접속하는 방법은 여러가지가 있겠지만, 간편한 방법 중 하나는 ssh의 reverse-tunneling 옵션을 사용하는 것입니다.

`ssh -N -R localhost:1234:192.168.77.3:27017 id-fense@172.16.8.32` 에서 `192.168.77.3:27017` 이 MongoDB 의 주소, `id-fense@172.16.8.32` 가 Remote host의 hostname과 주소 (= dp-server 가 실행될 컴퓨터의 주소), `localhost:1234` 가 바인딩 주소 (= `dp-server` 가 실행될 컴퓨터에서 MongoDB를 접속할 수 있는 주소) 입니다.

#### 4. Run `dp-server`
* 해당 repo를 (a) React 클라이언트가 실행될 NUC에 클론합니다.
* `. <venv name>/bin/activate` 로 Python 가상환경을 활성화합니다. 지금 repo에 `venv`라는 이름으로 가상환경이 업로드가 되어있기 때문에, `. venv/bin/activate`로 활성화하면 됩니다.
* `python -m venv <venv name> --python=3.5` 만일 `venv` 디렉토리가 삭제된 경우 이 커맨드를 통해 가상환경을 다시 생성할 수 있습니다. Python 버전은 3.5로 세팅합니다.
* `python3 manage.py run` 로 서버를 실행합니다.
* 실행 포트: `8888`

#### 5. Portforward PTZ Streaming Address
* Onvif에서 RTSP가 통상적으로 사용하는 포트는 554번입니다. (b) PTZ가 연결된 NUC에서 `ssh -N -R localhost:10554:localhost:554 id-fense@172.16.8.32` 를 터미널에 입력합니다. 바인딩 포트는 6554 가 아닌 다른 어떤 빈 포트라도 가능합니다.
* 또한 Onvif에서 PTZ 팬/틸트 조종을 위해 사용하는 포트는 80번입니다. 위 커맨드와 같은 방식으로 `ssh -N -R localhost:10080:localhost:80 id-fense@172.16.8.32` 를 입력합니다. 마찬가지로, 바인딩 포트는 무엇이든 상관 없습니다.
* 추천 바인딩 포트: RTSP: `10554`, Control: `10080`

#### 6. Run `dp-pelcod-server`
* 해당 repo를 (b) NUC에 클론합니다.
* `nodemon app` 이나 `node app.js` 를 입력합니다.
* 실행 포트: `19999`

#### 7. Run `dp-log-server`
* 해당 repo를 (a) React 클라이언트가 실행될 NUC에 클론합니다.
* `nodemon app` 이나 `node app.js` 를 입력합니다.
* 실행 포트: `3002`

#### 8. Run `dp-client-new`
* 해당 repo를 (a) React 클라이언트가 실행될 NUC에 클론합니다.
* `npm start` 를 입력합니다.
* 실행 포트: `3000`

## Important Notes

### 포트 포워딩시 호스트 IP 알아내기
와이파이와 같은 유동 네트워크에 연결되어있는 경우, 우분투에서 상단 바 와이파이 아이콘 클릭 -> Manage Network 클릭해서 현재 아이피 주소를 확인할 수 있습니다.

### Docker의 MongoDB Container IP 및 포트 알아내기
* `docker inspect <containerID>` 를 입력하면 해당 컨테이너의 low-level 정보가 json 형식으로 출력됩니다. NetworkSettings field에서 IP 및 포트 확인 가능합니다.
* `containerID` 는 `sudo ./status.sh` 실행 후 `aeroscopedockeren_mongodb_1` 에 해당하는 컨테이너의 ID 필드값입니다.

### 클라이언트 또는 서버 작동이 안될때
* 실행 포트 또는 포트포워딩 포트가 코드 상에서 매칭이 되는지 확인해보시고 (작업 중 실수로 포트가 잘못 바뀌었을 수 있습니다),
* 때론 방화벽에서 inbound/outbound 패킷을 차단하는 경우가 있습니다. 이땐 해당 포트나 프로토콜에 대한 차단을 해제해주시고 핑 테스트를 해보시면 됩니다.

### 13000번 소켓 포트의 용도?

    useEffect(() => {
      setSocket(connectSocketAPIMethod("http://localhost:13000"));
    }, []);
    
React 클라이언트의 App.js 를 살펴보면 위 코드가 있습니다.

본래 Onvif 프로토콜을 사용해 UI상에서 캠 조종이 가능하게끔 PTZ 조종 서버를 만들었으나 기술적 한계 (카메라 모델이 Onvif의 RelativeMove, AbsoluteMove, GetStatus를 지원하지 않음)로 인해 사장된 상태입니다 (깃헙에는 업로드되지 않았습니다).

관련 코드는 지우셔도 기능에 문제는 없습니다.
