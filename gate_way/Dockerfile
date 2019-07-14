FROM python:2.7
MAINTAINER ted
ENV REFERSHED_AT 2019-07-14

COPY ./requirements.txt  /requirements.txt

WORKDIR /

RUN pip  install -r requirements.txt

ADD  ./app /app

