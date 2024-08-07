# 베이스 이미지로 Ubuntu 22.04 사용
FROM ubuntu:22.04

# 필요한 패키지 업데이트 및 설치
RUN apt-get update && apt-get install -y \
    openssh-server \
    sudo

# SSH 설정 디렉토리 생성
RUN mkdir /var/run/sshd

# root 계정의 비밀번호를 'root'로 설정
RUN echo 'root:root' | chpasswd

# SSH 설정: 루트 로그인 허용 및 비밀번호 인증 허용
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

# 포트 22 노출
EXPOSE 22

# SSH 데몬 실행
CMD ["/usr/sbin/sshd", "-D"]
