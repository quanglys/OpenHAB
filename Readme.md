chạy cụm OpenHAB của tôi:
	clone các file cấu hình của OpenHAB và dockerfile để build image cho sensor giả lập lại https://github.com/quanglys/OpenHAB, lưu lại tại /OpenHAB
	Sau đó vào file /OpenHAB/configurations/openhab.cfg tìm dòng 
		mqtt:mqttIn.url=tcp://192.168.1.40:1883

		mqtt:mqttOut.url=tcp://192.168.1.40:1883
	sửa ip cho thích hợp, mqttIn là ip của mqtt mà openHAB lắng nghe, mqttOut là ip của mqtt mà nhận các thông tin mà OpenHAB gửi lên

chạy platform:
	sudo docker run -p 8080:8080 -p 8443:8443 -v /OpenHAB/addons:/openhab/addons -v /OpenHAB/configurations:/openhab/configurations --name openhab ngocluanbka/openhab
	
build image cho sensor giả lập, tôi gọi image đó là sensor 
tạo file cấu hình cho Sensor tên là config.cfg, đặt tại thư mục /SimulateSensor/config với cấu trúc
	dòng đầu: ip mqttIn (như trong file 
	dòng 2: số lượng sensor cần giả lập
	VD 192.168.0.100
	   10
chạy sensor 
	sudo docker run -v /SimulateSensor/config:/SimulateSensor/config -v /OpenHAB/configurations:/openhab --name sensor sensor

chạy mqtt: ông có thể dùng image mqtt có sẵn (ở máy tôi thì tôi dùng cái ncarlier/mqtt)
