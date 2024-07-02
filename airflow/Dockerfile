FROM registry.access.redhat.com/ubi8/python-312 as builder
USER root
RUN yum install -y go
RUN go install github.com/prometheus/statsd_exporter@v0.26.1


FROM registry.access.redhat.com/ubi8/python-312

RUN pip install "apache-airflow[statsd]==2.9.2" \
    --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.9.2/constraints-3.12.txt" \
    --disable-pip-version-check --no-cache

RUN mkdir -p /opt/app-root/src/airflow/
COPY airflow.cfg /opt/app-root/src/airflow/airflow.cfg
RUN airflow db init
RUN airflow users create --role Admin --username admin --email admin --firstname admin --lastname admin --password admin

COPY --from=builder /opt/app-root/src/go/bin /opt/app-root/src/bin

EXPOSE 8080
EXPOSE 9102

ENTRYPOINT ["sh", "-c", "airflow scheduler & airflow webserver -p 8080 & statsd_exporter --statsd.listen-udp localhost:8125 --log.level info & wait -n"]
