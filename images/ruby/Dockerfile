FROM ruby:2.5-stretch

ARG USER_ID=1000

RUN useradd -u $USER_ID -m -U -d /home/app app

WORKDIR /opt/src