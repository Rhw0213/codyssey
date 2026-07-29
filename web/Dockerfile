# 1. 기존 베이스 이미지 선택 (공식 NGINX)
FROM nginx:latest

# 2. 커스텀 포인트: 기본 웹페이지를 내 파일로 교체
COPY src/index.html /usr/share/nginx/html/index.html

# 3. 문서화용: 80번 포트 사용을 명시
EXPOSE 80
