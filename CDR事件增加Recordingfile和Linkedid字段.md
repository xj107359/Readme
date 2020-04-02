CDR事件增加Recordingfile和Linkedid字段

/etc/asterisk/cdr_manager.conf

	[general]
	enabled = yes
	[mappings]
	recordingfile => Recordingfile
	linkedid => Linkedid

AMI收到的事件会有Recordingfile和Linkedid字段

	Event: Cdr
	Privilege: cdr,all
	AccountCode: 
	Source: 8306
	Destination: 3333
	DestinationContext: from-internal
	CallerID: "寰愰獜" <8306>
	Channel: SIP/8306-00000107
	DestinationChannel: SIP/3333-00000108
	LastApplication: Dial
	LastData: SIP/3333,,HhTtrIb(func-apply-sipheaders^s^1)
	StartTime: 2019-10-22 15:00:55
	AnswerTime: 2019-10-22 15:00:55
	EndTime: 2019-10-22 15:00:56
	Duration: 1
	BillableSeconds: 1
	Disposition: ANSWERED
	AMAFlags: DOCUMENTATION
	UniqueID: 1571727655.266
	UserField: 
	Recordingfile: internal-3333-8306-20191022-150055-1571727655.266.wav
	Linkedid: 1571727655.266

