## AMI ExtensionStateList Usage

### 1. Connect to 5038

	[root@localhost ~]# telnet 127.0.0.1 5038

>
	Trying 127.0.0.1...
	Connected to 127.0.0.1.
	Escape character is '^]'.
	Asterisk Call Manager/5.0.1

### 2. Login

	Action: Login
	Username: admin
	Secret: amp111

>
	Response: Success
	Message: Authentication accepted

### 3. ExtensionStateList

	Action: ExtensionStateList

>
	Response: Success
	EventList: start
	Message: Extension Statuses will follow
>
	...
>
	Event: ExtensionStateListComplete
	EventList: Complete
	ListItems: 18

### 4. Logoff

	Action: Logoff

>
	Response: Goodbye
	Message: Thanks for all the fish.
>
	Connection closed by foreign host.





