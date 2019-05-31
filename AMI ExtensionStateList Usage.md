# AMI ExtensionStateList Usage

	[root@localhost ~]# telnet 127.0.0.1 5038

>
	Trying 127.0.0.1...
	Connected to 127.0.0.1.
	Escape character is '^]'.
	Asterisk Call Manager/5.0.1

	Action: Login
	Username: admin
	Secret: amp111

>
	Response: Success
	Message: Authentication accepted

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

	Action: Logoff

>
	Response: Goodbye
	Message: Thanks for all the fish.
>
	Connection closed by foreign host.





