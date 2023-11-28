FROM python:3.10-slim-buster
# Install python packages
COPY requirements.txt /tmp
RUN pip install --upgrade pip
RUN pip install -r /tmp/requirements.txt
