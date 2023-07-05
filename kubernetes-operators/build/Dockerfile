FROM python:3.7
COPY templates ./templates
COPY mysql-operator.py ./mysql-operator.py
RUN pip3 install kopf kubernetes pyyaml jinja2
CMD kopf run /mysql_operator.py
