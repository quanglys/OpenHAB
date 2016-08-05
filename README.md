OpenHAB
Yêu cầu: Máy tính chạy Java phiên bản từ 1.7
1.Cài đặt:
	- Môi trường chạy:
		+ Giải nén distribution-<version>-runtime.zip (nên giải nén vào c:\openhab trên windows hoặc /opt/openhab trên mac/linux)
		+ Trong thư mục configurations, copy openhab_default.cfg và đặt tên là openhab.cfg (lưu cấu hình của openHAB)
	- Binding:
		+ Giải nén distribution-<version>-addons.zip vào trong thư mục openhab/addons		
		+ Cấu hình trong file openhab.cfg theo hướng dẫn (1) 
		+ Kết nối thiết bị và các luật tới các binding
	- Cài đặt chương trình demo:
		+ Giải nén distribution-<version>-demo-configuration.zip vào thư mục cài đặt môi trường chạy
		+ Copy file openhab.cfg vào thư mục configurations
	
	- Designer: như một editor của OpenHAB để cấu hình cho các file của OpenHAB
		+ Giải nén distribution-<version>-designer-<platform>.zip
		+ Chạy openHAB-Designer.exe (lưu ý đây là chương trình 32b nên phải cài đặt jre 32b và đặt biến môi trường PATH tới thư mục bin của jre)
		+ Chọn "open folder" và trỏ tới thư mục  cấu hình "configurations", thực hiện chọn các file trong thư mục này và cấu hình
		
2.Cấu hình server
	- openhab.cfg: là file cấu hình cho openHAB với định dạng <bundle name>:<parameter name>=<parameter value>, các tham số cấu hình được hướng dẫn trong file openhab_default.cfg
	- file .item: quản lý các thiết bị hiện có (trong thư mục configurations\items) (2)
	
		Trong file .item, ta sẽ định nghĩa các groups và items.
		Cú pháp:
			itemtype itemname ["labeltext"] [<iconname>] [(group1, group2, ...)] [{bindingconfig}]
			
			VD Group:Number:AVG Lighting "Average lighting [Lux](%.2f)" <switch> (Status)
			Trong đó Group Lighting chứa giá trị thực là trung bình của các thành phần, có icon là switch và thuộc về Group Status
			
			VD:item liên kết tới các thiết bị vật lý
			Number Lighting_Room_Sensor "Lighting in the Room [Lux](%.2f)" <switch> (Room,Lighting) { knx = "<0/1/1" }: sensor thuộc loại Number, có tên Lighting_Room_Sensor ở địa chỉ 0/1/1 và sẽ được openHAB định kỳ query để lấy thông tin
	- file .sitemap:Cho biết các items sẽ được hiển thị trên giao diện (3)
	
			Tạo file .sitemap trong configurations/sitemap
				+Dòng đầu có cú pháp: sitemap [sitemapname] [label="<title of the main screen>"]
					VD: sitemap demo label="Main Menu" trong đó demo là tên sitemap và "Main Menu" là tiêu đề trên giao diện
				+Thân: là nội dung hiển thị trên giao diện
					VD				
					Frame label="Demo" {
						Text label="Group Demo" icon="1stfloor" {
								  Switch item=Lights mappings=[OFF="All Off"]
								  Group item=Heating
								  Group item=Windows
								  Text item=Temperature
						}
						Text label="Multimedia" icon="video" {
								  Selection item=Radio_Station mappings=[0=off, 1=HR3, 2=SWR3, 3=FFH, 4=Charivari]
								  Slider item=Volume
						}
					}
					Trong đó: Frame có tên Demo có 2 thành phần: mỗi thành phần lại có các thành phần con ( lần lượt là 4 và 2)
3.Start OpenHAB:
	- Start server: chạy file start.bat trên windows hoặc start.sh trên linux 
	- Client: vào trình duyệt chạy http://<openHAB address or hostname>:8080/openhab.app?sitemap=<sitemap_name> để truy cập vào giao diện quản lý thiết bị của OpenHAB
			VD: http://127.0.0.1:8080/openhab.app?sitemap=demo
		
4.REST API cho OpenHAB
	- Hỗ trợ Subscribe với nhiều định dạng respond (xml, json, jsonp)
	- Có thể định nghĩa định dạng của respond trong header của HTTP như sau: Accept:application/xxx
		hoặc "<URI>?type=xxx" 
		hoặc "<URI>?Accept:application/xxx" 
		trong đó xxx có thể là xml, json, jsonp
	- Các API:
		+ Log in vào hệ thống: 
			GET http://username:password@host:port/<REST_API>(nếu username và password sử dụng các ký tự đặc biệt thì phải đổi ra các ký hiệu urlencode)		
			VD http://ql:123456789@localhost:8080/rest
			
		+ Lấy thông tin các tài nguyên trong hệ thống:
			GET http://localhost:8080/rest
		+ Lấy thông tin về các items
			GET http://localhost:8080/rest/items
		+ Lấy trạng thái item
			GET http://localhost:8080/rest/items/<ITEM_NAME>/state
		+ Đặt trạng thái
			GET http://localhost:8080/CMD?<ITEM_NAME>=<VALUE>
			hoặc PUT http://localhost:8080/rest/items/<ITEM_NAME> với nội dung ở trong body
		+ Gửi lệnh tới 1 item
			POST http://localhost:8080/rest/items/<ITEM_NAME> với nội dung ở trong body
		
5.Mô hình kêt nối
	- Các sensor sẽ kết nối tới Arduino, arduino lấy dữ liệu từ sensor gửi lên server(OpenHAB). 
	- Các clien truy cập vào server để lấy dữ liệu của các sensor, điều khiển sensor
	
6. Security
	- Sử dụng HTTPS: có thể sử dụng HTTPS để truy cập vào server theo địa chỉ sau
		https://<openHAB address or hostname>:8443/openhab.app?sitemap=<sitemap_name>
		VD: https://127.0.0.1:8443/openhab.app?sitemap=demo
	- Xác thực:
		+ Kích hoạt việc xác thực: 
			trong file start thêm vào: -Djava.security.auth.login.config=./etc/login.conf (mặc định trong file này đã có)
		
			trong file configuration/users.cfg, thêm 
				user=password,user,role
				<username_1>=<password_1>
				.
				.
				.
				<username_n>=<password_n>
				
			trong file configuration/openhab.cfg, thêm tùy chọn
				security:option=
								ON: bật security
								OFF: tắt security
								EXTERNAL: Tắt security nội mạng, bật security ngoại mạng, nếu để chế độ này cần đặt security:netmask=<NET_MASK> để openHAB có thể xác định được một địa chỉ là trong hay ngoài mạng
		+ Phân quyền: Hiện tại, OpenHAB chưa hỗ trợ phân quyền
		
7. Kết nối tới MQTT Broker
	- Cài đặt MQTT Broker
	- Cấu hình trong openhab.cfg (4) 
		+ Cấu hình transport:
			mqtt:<broker>.url=<url>
			mqtt:<broker>.clientId=<clientId>
			mqtt:<broker>.user=<user>
			mqtt:<broker>.pwd=<password>
			mqtt:<broker>.qos=<qos>
			mqtt:<broker>.retain=<retain>
			mqtt:<broker>.async=<async>
			mqtt:<broker>.keepAlive=<keepAlive>
		VD:
			mqtt:mosquitto.url=tcp://localhost:1883
			mqtt:mosquitto.user=administrator
			mqtt:mosquitto.pwd=mysecret
			mqtt:mosquitto.qos=1
			mqtt:mosquitto.retain=true
			mqtt:mosquitto.async=false
		+ Cấu hình item binding trong file .items
			
			Cấu hình inbound để nhận dữ liệu:
				Itemtype ItemName {mqtt="<direction>[<broker>:<topic>:<type>:<transformer>], <direction>[<broker>:<topic>:<type>:<transformation>], ..."}
			VD:
			Number temperature "temp [%.1f]" {mqtt="<[publicweatherservice:/london-city/temperature:state:default]"}
			Number waterConsumption "consum [%d]" {mqtt="<[mybroker:/myHome/watermeter:state:XSLT(parse_water_message.xslt)]"} 
			Switch doorbell "bell [%s]" {mqtt="<[mybroker:/myHome/doorbell:command:ON]"}
			Number mfase1 "mfase1 [%.3f]" {mqtt="<[flukso:/sensor/9cf3d75543fa82a4662fe70df5bf4fde/gauge:state:REGEX(.*,(.*),.*)]"}
			
			Cấu hình outbound để gửi dữ liệu:
				ItemType itemName {mqtt="<direction>[<broker>:<topic>:<type>:<trigger>:<transformation>]" }
			VD:
			Switch mySwitch {mqtt=">[mybroker:/myhouse/office/light:command:ON:1],>[mybroker:/myhouse/office/light:command:OFF:0]"}
			Switch mySwitch {mqtt=">[mybroker:/myhouse/office/light:command:ON:1],>[mybroker:/myhouse/office/light:command:*:Switch ${itemName} was turned ${command}]"}
		
		
		+ Cấu hình event bus binding trong openhab.cfg: Ngoài việc cấu hính từng item, ta có thể cấu hình cho tất cả item trong file openhab.cfg như sau:
			mqtt-eventbus:broker=<broker>
			mqtt-eventbus:statePublishTopic=<statePublishTopic>
			mqtt-eventbus:commandPublishTopic=<commandPublishTopic>
			mqtt-eventbus:stateSubscribeTopic=<stateSubscribeTopic>
			mqtt-eventbus:commandSubscribeTopic=<commandSubscribeTopic>
			
		VD:
			mqtt-eventbus:broker=m2m-eclipse
			mqtt-eventbus:commandPublishTopic=/openHAB/out/${item}/command
			mqtt-eventbus:stateSubscribeTopic=/openHAB/in/${item}/state
8. Tài liệu tham khảo
	(1). https://github.com/openhab/openhab/wiki
	(2). https://github.com/openhab/openhab/wiki/Explanation-of-items
	(3). https://github.com/openhab/openhab/wiki/Explanation-of-Sitemaps
	(4). https://github.com/openhab/openhab/wiki/MQTT-Binding
