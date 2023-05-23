# IDCITI ADS (Anti Drone System)

## 구버전
- DJI Aeroscope G8의 웹 클라이언트를 주기적으로 크롤링하여 G8이 스캔한 드론을 지도에 시각화
- Onvif 프로토콜 사용해 PTZ 캠 RTSP 스트리밍 지원

## 신버전
- UI 업데이트: DR Platform의 UI 스타일 채용
- ADSB: OpenSkyNetwork API와 컴퓨터에 연결된 SDR 장비 (RTL-SDR 또는 HackRF-One) 사용하여 항공기 식별 가능
- 웹 크롤링 대신 ssh 포트 포워딩을 통해 G8의 Dockerized MongoDB 에 접속하여 G8 이 스캔한 드론 정보 실시간으로 가져오기
- Onvif 프로토콜 사용해 PTZ 캠 RTSP 스트리밍 지원
- Pelco-D 프로토콜 사용해 PTZ 캠을 물리/소프트웨어 컨트롤러로 조종 시 방향각과 줌값 시각화
- 날짜별 드론 탐지 기록 가능: PTZ 캠 스트리밍 화면을 24시간 단위로 연속 녹화하며 녹화본 영상에서 드론이 탐지된 시각을 찾아서 재생 가능
    * 아직 G8 Activation이 되지 않아 해당 기능은 테스트를 하지 않았습니다. 추후 Activation 이 완료되고 드론 탐지 기록이 G8 DB에서 실시간으로 저장되는것이 확인되면 테스트가 가능합니다.

## Repos

### UI 관련
- **dp-client-new:** 기존 DR Platform UI를 기반하여 만들어진 React 클라이언트 입니다.
- **dp-client:** 여러 다른 ADS들의 UI를 참고하여 개발중인 새로운 React 클라이언트 입니다. 현재 개발 보류중 입니다.
  * Figma 스케치: https://www.figma.com/file/PUxoNvb3KMiDiqu4Rx4rh1/drplatform?type=design&node-id=0-1&t=BwK7nRwugxNCOYHt-0

### G8 관련
- **dp-server:** G8 DB에 접속해서 drone flight records를 받아오는 Restful API가 마련된 서버
- **dp-crawler (deprecated):** G8 웹 클라이언트를 크롤링하는 서버
- **dp-log-server:** 날짜별 드론 탐지 내역을 영상 기록으로 확인 가능한 Restful 서버

### ADSB
- **dp-adsb-server:** 연결된 SDR 장비로부터 수신되는 SBS-1 패킷을 decode 하는 서버

### PTZ 캠 스트리밍 및 컨트롤 관련
- **dp-ptz-controller:** Onvif (캠 팬/틸트/스트리밍), YoloV4 (Dataset training) 사용해 캠 화면상 드론 식별 및 추적 기능 지원
- **dp-ptz-streaming-server:** Onvif 사용하여 PTZ 캠을 스트리밍 해주는 서버
- **dp-pelcod-server:** Pelco-D 사용하여 PTZ 캠 상태 (팬/틸트/줌) 실시간 추적하는 서버
