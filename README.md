# tomcat-bluegreen

Portainer(GitOps) + Podman 기반 Tomcat 블루-그린 배포

## 구조
- proxy (nginx, :8090) → tomcat-blue / tomcat-green
- war는 git에 넣지 않음. 서버 볼륨에 배치
  - blue : /container-volume/tomcat-blue/webapps/ROOT.war
  - green: /container-volume/tomcat-green/webapps/ROOT.war
- 원본 war 보관: /container-volume/releases/<앱>-<버전>.war
- 전환 스위치: proxy/default.conf 의 proxy_pass

## 사전 준비 (최초 1회)
- podman network create bluegreen-net
- /container-volume/tomcat-{blue,green}/webapps 디렉터리 생성

## 배포 절차
1. 현재 운영 색 확인: curl -si http://<서버>:8090/ | grep X-Upstream
2. 대기 쪽 webapps에 releases의 새 war를 ROOT.war로 복사
3. 대기 쪽 기동 확인 (컨테이너 IP:8080 직접 호출)
4. default.conf 의 proxy_pass를 대기 쪽으로 변경 → push
5. Portainer Stack → Pull and redeploy
6. 8090 응답 및 X-Upstream 확인

## 롤백
- default.conf 를 이전 색으로 되돌려 push → Pull and redeploy
- 안정화 전까지 이전 색 컨테이너와 war는 삭제하지 않는다

## 주의
- 전환 시 톰캣 메모리 세션은 유지되지 않음 (재로그인 발생)
- DB 스키마 변경은 양쪽 버전이 모두 동작하도록 (추가 먼저, 삭제는 다음 배포)
- 현재 전환 방식은 proxy 컨테이너 재생성 → 수 초 끊김 있음 (reload 방식 개선 예정)
