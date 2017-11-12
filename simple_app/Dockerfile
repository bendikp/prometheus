FROM python:2.7.14-alpine3.6

RUN pip install prometheus_client

WORKDIR /usr/src/app

COPY  python.py ./

EXPOSE 8000

CMD [ "python", "./python.py" ]
