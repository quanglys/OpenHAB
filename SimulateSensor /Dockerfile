FROM hardbyte/python3
MAINTAINER quanglys@gmail.com 
RUN pip install paho-mqtt
WORKDIR "/"
RUN git clone https://github.com/quanglys/SimulateSensor && mkdir /openhab
VOLUME ["/SimulateSensor/config", "/openhab"]
CMD [ "python", "/SimulateSensor/Sensor.py" ]
