# Asterisk 1.4 实现强拆

## 功能介绍

用户A正在通话过程中，此时有另一个重要电话找A，具有强拆权限的话务员可强行拆除A目前正在进行的通话，保证重要电话的随时接通。

## 实现思路
强行插入可使用密语功能实现，属现有功能
强行拆除可参考密语功能，找出用户A的通道，挂断该通道后再呼叫用户A

## 涉及代码
asterisk-1.4.31/channels/chan_sip.c

## 修改思路

### 增加App描述

	static const char *app_xuHangup = "XuHangup";
	static const char *desc_xuHangup = 
	"  XuHangup(exten): This application is used to Hangup an Asterisk channel. \n"
	"                    So we can hangup any phone call.\n"
	;

### 在load_module和unload_module中增加注册和注销代码

	static int unload_module(void)
	{
		...
		res |= ast_unregister_application(app_xuHangup);
		...
	}
	
	static int load_module(void)
	{
		...
		res |= ast_register_application(app_xuHangup, xuHangup_exec, tdesc, desc_xuHangup);
		...
	}

### 实现xuHangup_exec

	static int xuHangup_exec(struct ast_channel *chan, void *data)
	{
		struct ast_module_user *u;
		char *spec = NULL;
		char *argv[1];
		int argc = 0;
		int res;

		data = ast_strdupa(data);
		u = ast_module_user_add(chan);
	
		if ((argc = ast_app_separate_args(data, '|', argv, sizeof(argv) / sizeof(argv[0])))) {
			spec = argv[0];
			ast_log(LOG_NOTICE, "spec is [%s] \n", spec);
		
			if (ast_strlen_zero(spec) || !strcmp(spec, "all"))
				spec = NULL;
		}
	
		res = common2_exec(chan, spec, NULL, NULL);
		ast_log(LOG_NOTICE, "Exiting xuHangup_exec \n");
		ast_module_user_remove(u);
		return res;
	}

### 拨号方案

	[app-chanspy]
	...
	
	exten => _*558X.,1,Macro(user-callerid,)
	exten => _*558X.,n,Answer
	exten => _*558X.,n,Wait(1)
	exten => _*558X.,n,StartMusicOnHold
	exten => _*558X.,n,XuHangup(SIP/${EXTEN:4})
	exten => _*558X.,n,VERBOSE(XuHangup(SIP/${EXTEN:4}))
	exten => _*558X.,n,wait(2)
	exten => _*558X.,n,StopMusicOnHold
	exten => _*558X.,n,Dial(SIP/${EXTEN:4})

### 设置分机权限

可在话务等级的拨号规则中，设置分机是否具有强拆权限

### 测试记录
8302呼叫8098，8098应答。8305拨打*5588098，8302与8098分别挂断，然后8305再呼叫8098

	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [8098@from-internal-spec3-pstn:1] Macro("SIP/8302-0000001a", "exten-vm|8098|8098") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-exten-vm:1] Macro("SIP/8302-0000001a", "user-callerid|") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:1] Set("SIP/8302-0000001a", "AMPUSER=8302") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:2] setlanguage("SIP/8302-0000001a", "8302|8098") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c: ooxx dst(8098) language is cn
	[May 29 14:28:00] NOTICE[32511] /home/sda3/svn/ip6000disksvn/code/cpp/asteriskmodule/app_sopho_ext/interface.cpp: Queue members successfully reloaded from database.
	[May 29 14:28:00] NOTICE[32511] /home/sda3/svn/ip6000disksvn/code/cpp/asteriskmodule/app_sopho_ext/main.c: 8302 8098 set setlanguage cn
	[May 29 14:28:00] VERBOSE[32511] logger.c: 8302 8098 set setlanguage cn
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: setlanguage
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:3] Verbose("SIP/8302-0000001a", "cn") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c: cn
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: VERBOSE
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:4] ExecIf("SIP/8302-0000001a", "1|Set|CHANNEL(language)=cn") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: ExecIf
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Last app: Set|CHANNEL(language)=cn
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:5] GotoIf("SIP/8302-0000001a", "0?report") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:6] ExecIf("SIP/8302-0000001a", "1|Set|REALCALLERIDNUM=8302") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: ExecIf
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:7] Set("SIP/8302-0000001a", "AMPUSER=8302") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:8] Set("SIP/8302-0000001a", "AMPUSERCIDNAME=8302") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:9] GotoIf("SIP/8302-0000001a", "0?report") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:10] Set("SIP/8302-0000001a", "AMPUSERCID=8302") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:11] Set("SIP/8302-0000001a", "CALLERID(all)="8302" <8302>") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:12] ExecIf("SIP/8302-0000001a", "1|Set|CHANNEL(language)=cn") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: ExecIf
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Last app: Set|CHANNEL(language)=cn
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:13] GotoIf("SIP/8302-0000001a", "0?continue") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:14] Set("SIP/8302-0000001a", "__TTL=64") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:15] GotoIf("SIP/8302-0000001a", "1?continue") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Goto (macro-user-callerid,s,22)
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-user-callerid:22] NoOp("SIP/8302-0000001a", "Using CallerID "8302" <8302>") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Noop
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Macro
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-exten-vm:2] Set("SIP/8302-0000001a", "RingGroupMethod=none") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-exten-vm:3] Set("SIP/8302-0000001a", "VMBOX=8098") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-exten-vm:4] Set("SIP/8302-0000001a", "EXTTOCALL=8098") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] DEBUG[32511] func_db.c: DB: CFU/8098 not found in database.
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-exten-vm:5] Set("SIP/8302-0000001a", "CFUEXT=") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] DEBUG[32511] func_db.c: DB: CFB/8098 not found in database.
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-exten-vm:6] Set("SIP/8302-0000001a", "CFBEXT=") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-exten-vm:7] Set("SIP/8302-0000001a", "RT=15") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Set
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-exten-vm:8] Macro("SIP/8302-0000001a", "record-enable|8098|IN") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-record-enable:1] GotoIf("SIP/8302-0000001a", "1?check") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Goto (macro-record-enable,s,4)
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-record-enable:4] ExecIf("SIP/8302-0000001a", "0|MacroExit|") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: ExecIf
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-record-enable:5] GotoIf("SIP/8302-0000001a", "0?Group:OUT") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Goto (macro-record-enable,s,15)
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-record-enable:15] GotoIf("SIP/8302-0000001a", "1?IN") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Goto (macro-record-enable,s,20)
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-record-enable:20] ExecIf("SIP/8302-0000001a", "1|MacroExit|") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: Macro
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-exten-vm:9] Macro("SIP/8302-0000001a", "dial|15|trWw|8098") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-dial:1] GotoIf("SIP/8302-0000001a", "1?dial") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Goto (macro-dial,s,3)
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-dial:3] AGI("SIP/8302-0000001a", "dialparties.agi") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Launched AGI Script /var/lib/asterisk/agi-bin/dialparties.agi
	[May 29 14:28:00] VERBOSE[32511] logger.c:   dialparties.agi: Starting New Dialparties.agi
	[May 29 14:28:00] VERBOSE[32511] logger.c:   dialparties.agi: Caller ID name is '8302' number is '8302'
	[May 29 14:28:00] VERBOSE[32511] logger.c:   dialparties.agi: Methodology of ring is  'none'
	[May 29 14:28:00] VERBOSE[32511] logger.c:     --  dialparties.agi: Added extension 8098 to extension map
	[May 29 14:28:00] VERBOSE[32511] logger.c:     --  dialparties.agi: Extension 8098 cf is disabled
	[May 29 14:28:00] VERBOSE[32511] logger.c:     --  dialparties.agi: Extension 8098 do not disturb is disabled
	[May 29 14:28:00] VERBOSE[32511] logger.c:   dialparties.agi: ExtensionState: 0
	[May 29 14:28:00] VERBOSE[32511] logger.c:   dialparties.agi: Extension 8098 has ExtensionState: 0
	[May 29 14:28:00] VERBOSE[32511] logger.c:     --  dialparties.agi: Checking CW and CFB status for extension 8098
	[May 29 14:28:00] VERBOSE[32511] logger.c:     --  dialparties.agi: dbset CALLTRACE/8098 to 8302
	[May 29 14:28:00] VERBOSE[32511] logger.c:     --  dialparties.agi: Filtered ARG3: 8098
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- AGI Script dialparties.agi completed, returning 0
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: AGI
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-dial:7] NoOp("SIP/8302-0000001a", "") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: noop
	[May 29 14:28:00] ERROR[32511] pbx.c: Function EXTENSION_STATE not registered
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-dial:8] NoOp("SIP/8302-0000001a", "") in new stack
	[May 29 14:28:00] DEBUG[32511] app_macro.c: Executed application: noop
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Executing [s@macro-dial:9] Dial("SIP/8302-0000001a", "SIP/8098|15|trWw") in new stack
	[May 29 14:28:00] VERBOSE[32511] logger.c:     -- Called 8098
	[May 29 14:28:01] VERBOSE[32511] logger.c:     -- SIP/8098-0000001b is ringing
	[May 29 14:28:03] VERBOSE[32511] logger.c:     -- SIP/8098-0000001b answered SIP/8302-0000001a
	[May 29 14:28:03] VERBOSE[32511] logger.c:     -- fixed jitterbuffer created on channel SIP/8098-0000001b
	[May 29 14:28:03] VERBOSE[32511] logger.c:     -- fixed jitterbuffer created on channel SIP/8302-0000001a
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [*5588098@from-internal:1] Macro("SIP/8305-0000001c", "user-callerid|") in new stack
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:1] Set("SIP/8305-0000001c", "AMPUSER=8305") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: Set
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:2] setlanguage("SIP/8305-0000001c", "8305|*5588098") in new stack
	[May 29 14:28:15] VERBOSE[32547] logger.c: ooxx src(8305) language is cn
	[May 29 14:28:15] NOTICE[32547] /home/sda3/svn/ip6000disksvn/code/cpp/asteriskmodule/app_sopho_ext/interface.cpp: Queue members successfully reloaded from database.
	[May 29 14:28:15] NOTICE[32547] /home/sda3/svn/ip6000disksvn/code/cpp/asteriskmodule/app_sopho_ext/main.c: 8305 *5588098 set setlanguage cn
	[May 29 14:28:15] VERBOSE[32547] logger.c: 8305 *5588098 set setlanguage cn
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: setlanguage
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:3] Verbose("SIP/8305-0000001c", "cn") in new stack
	[May 29 14:28:15] VERBOSE[32547] logger.c: cn
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: VERBOSE
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:4] ExecIf("SIP/8305-0000001c", "1|Set|CHANNEL(language)=cn") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: ExecIf
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Last app: Set|CHANNEL(language)=cn
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:5] GotoIf("SIP/8305-0000001c", "0?report") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: GotoIf
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:6] ExecIf("SIP/8305-0000001c", "1|Set|REALCALLERIDNUM=8305") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: ExecIf
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:7] Set("SIP/8305-0000001c", "AMPUSER=8305") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: Set
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:8] Set("SIP/8305-0000001c", "AMPUSERCIDNAME=8305") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: Set
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:9] GotoIf("SIP/8305-0000001c", "0?report") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: GotoIf
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:10] Set("SIP/8305-0000001c", "AMPUSERCID=8305") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: Set
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:11] Set("SIP/8305-0000001c", "CALLERID(all)="8305" <8305>") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: Set
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:12] ExecIf("SIP/8305-0000001c", "1|Set|CHANNEL(language)=cn") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: ExecIf
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Last app: Set|CHANNEL(language)=cn
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:13] GotoIf("SIP/8305-0000001c", "0?continue") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: GotoIf
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:14] Set("SIP/8305-0000001c", "__TTL=64") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: Set
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:15] GotoIf("SIP/8305-0000001c", "1?continue") in new stack
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Goto (macro-user-callerid,s,22)
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: GotoIf
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [s@macro-user-callerid:22] NoOp("SIP/8305-0000001c", "Using CallerID "8305" <8305>") in new stack
	[May 29 14:28:15] DEBUG[32547] app_macro.c: Executed application: Noop
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [*5588098@from-internal:2] Answer("SIP/8305-0000001c", "") in new stack
	[May 29 14:28:15] VERBOSE[32547] logger.c:     -- Executing [*5588098@from-internal:3] Wait("SIP/8305-0000001c", "1") in new stack
	[May 29 14:28:16] VERBOSE[32547] logger.c:     -- Executing [*5588098@from-internal:4] StartMusicOnHold("SIP/8305-0000001c", "") in new stack
	[May 29 14:28:16] VERBOSE[32547] logger.c:     -- Started music on hold, class 'default', on SIP/8305-0000001c
	[May 29 14:28:16] VERBOSE[32547] logger.c:     -- Executing [*5588098@from-internal:5] XuHangup("SIP/8305-0000001c", "SIP/8098") in new stack
	[May 29 14:28:16] NOTICE[32547] app_chanspy.c: spec is [SIP/8098] 
	[May 29 14:28:16] NOTICE[32547] app_chanspy.c: checking peer [SIP/8098-0000001b] 
	[May 29 14:28:16] NOTICE[32547] app_chanspy.c: hangup [SIP/8098-0000001b] 
	[May 29 14:28:16] NOTICE[32547] app_chanspy.c: Exiting xuHangup_exec 
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Executing [h@macro-dial:1] GotoIf("SIP/8302-0000001a", "1?setmanmon:hangup") in new stack
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Goto (macro-dial,h,2)
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Executing [h@macro-dial:2] Set("SIP/8302-0000001a", "dbkey=passval/manmon/SIP/8302-0000001a") in new stack
	[May 29 14:28:16] DEBUG[32511] func_db.c: DB: passval/manmon/SIP/8302-0000001a not found in database.
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Executing [h@macro-dial:3] Set("SIP/8302-0000001a", "dbval=") in new stack
	[May 29 14:28:16] VERBOSE[32547] logger.c:     -- Executing [*5588098@from-internal:6] Verbose("SIP/8305-0000001c", "XuHangup(SIP/8098)") in new stack
	[May 29 14:28:16] VERBOSE[32547] logger.c: XuHangup(SIP/8098)
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Executing [h@macro-dial:4] GotoIf("SIP/8302-0000001a", "1?hangup") in new stack
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Goto (macro-dial,h,6)
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Executing [h@macro-dial:6] DBdel("SIP/8302-0000001a", "passval/manmon/SIP/8302-0000001a") in new stack
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- DBdel: family=passval, key=manmon/SIP/8302-0000001a
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- DBdel: Error deleting key from database.
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Executing [h@macro-dial:7] Macro("SIP/8302-0000001a", "hangupcall") in new stack
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Executing [s@macro-hangupcall:1] GotoIf("SIP/8302-0000001a", "1?skiprg") in new stack
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Goto (macro-hangupcall,s,4)
	[May 29 14:28:16] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Executing [s@macro-hangupcall:4] GotoIf("SIP/8302-0000001a", "1?skipblkvm") in new stack
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Goto (macro-hangupcall,s,7)
	[May 29 14:28:16] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Executing [s@macro-hangupcall:7] GotoIf("SIP/8302-0000001a", "1?theend") in new stack
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Goto (macro-hangupcall,s,9)
	[May 29 14:28:16] DEBUG[32511] app_macro.c: Executed application: GotoIf
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- Executing [s@macro-hangupcall:9] Hangup("SIP/8302-0000001a", "") in new stack
	[May 29 14:28:16] VERBOSE[32511] logger.c:   == Spawn extension (macro-hangupcall, s, 9) exited non-zero on 'SIP/8302-0000001a' in macro 'hangupcall'
	[May 29 14:28:16] VERBOSE[32511] logger.c:   == Spawn h extension (macro-dial, h, 7) exited non-zero on 'SIP/8302-0000001a'
	[May 29 14:28:16] VERBOSE[32547] logger.c:     -- Executing [*5588098@from-internal:7] Wait("SIP/8305-0000001c", "2") in new stack
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- fixed jitterbuffer destroyed on channel SIP/8098-0000001b
	[May 29 14:28:16] VERBOSE[32511] logger.c:   == Spawn extension (macro-dial, s, 9) exited non-zero on 'SIP/8302-0000001a' in macro 'dial'
	[May 29 14:28:16] VERBOSE[32511] logger.c:   == Spawn extension (macro-exten-vm, s, 9) exited non-zero on 'SIP/8302-0000001a' in macro 'exten-vm'
	[May 29 14:28:16] VERBOSE[32511] logger.c:   == Spawn extension (from-internal-spec3-pstn, 8098, 1) exited non-zero on 'SIP/8302-0000001a'
	[May 29 14:28:16] VERBOSE[32511] logger.c:     -- fixed jitterbuffer destroyed on channel SIP/8302-0000001a
	[May 29 14:28:18] VERBOSE[32547] logger.c:     -- Executing [*5588098@from-internal:8] StopMusicOnHold("SIP/8305-0000001c", "") in new stack
	[May 29 14:28:18] VERBOSE[32547] logger.c:     -- Stopped music on hold on SIP/8305-0000001c
	[May 29 14:28:18] VERBOSE[32547] logger.c:     -- Executing [*5588098@from-internal:9] Dial("SIP/8305-0000001c", "SIP/8098") in new stack
	[May 29 14:28:18] VERBOSE[32547] logger.c:     -- Called 8098
	[May 29 14:28:18] VERBOSE[32547] logger.c:     -- SIP/8098-0000001d is ringing
	[May 29 14:28:24] VERBOSE[32547] logger.c:     -- SIP/8098-0000001d answered SIP/8305-0000001c
	[May 29 14:28:24] VERBOSE[32547] logger.c:     -- fixed jitterbuffer created on channel SIP/8098-0000001d
	[May 29 14:28:24] VERBOSE[32547] logger.c:     -- fixed jitterbuffer created on channel SIP/8305-0000001c
	[May 29 14:28:40] VERBOSE[32547] logger.c:     -- Executing [h@from-internal:1] Macro("SIP/8305-0000001c", "hangupcall") in new stack
	[May 29 14:28:40] VERBOSE[32547] logger.c:     -- Executing [s@macro-hangupcall:1] GotoIf("SIP/8305-0000001c", "1?skiprg") in new stack
	[May 29 14:28:40] VERBOSE[32547] logger.c:     -- Goto (macro-hangupcall,s,4)
	[May 29 14:28:40] DEBUG[32547] app_macro.c: Executed application: GotoIf
	[May 29 14:28:40] VERBOSE[32547] logger.c:     -- Executing [s@macro-hangupcall:4] GotoIf("SIP/8305-0000001c", "1?skipblkvm") in new stack
	[May 29 14:28:40] VERBOSE[32547] logger.c:     -- Goto (macro-hangupcall,s,7)
	[May 29 14:28:40] DEBUG[32547] app_macro.c: Executed application: GotoIf
	[May 29 14:28:40] VERBOSE[32547] logger.c:     -- Executing [s@macro-hangupcall:7] GotoIf("SIP/8305-0000001c", "1?theend") in new stack
	[May 29 14:28:40] VERBOSE[32547] logger.c:     -- Goto (macro-hangupcall,s,9)
	[May 29 14:28:40] DEBUG[32547] app_macro.c: Executed application: GotoIf
	[May 29 14:28:40] VERBOSE[32547] logger.c:     -- Executing [s@macro-hangupcall:9] Hangup("SIP/8305-0000001c", "") in new stack
	[May 29 14:28:40] VERBOSE[32547] logger.c:   == Spawn extension (macro-hangupcall, s, 9) exited non-zero on 'SIP/8305-0000001c' in macro 'hangupcall'
	[May 29 14:28:40] VERBOSE[32547] logger.c:   == Spawn h extension (from-internal, h, 1) exited non-zero on 'SIP/8305-0000001c'
	[May 29 14:28:40] VERBOSE[32547] logger.c:     -- fixed jitterbuffer destroyed on channel SIP/8098-0000001d
	[May 29 14:28:40] VERBOSE[32547] logger.c:   == Spawn extension (from-internal, *5588098, 9) exited non-zero on 'SIP/8305-0000001c'
	[May 29 14:28:40] VERBOSE[32547] logger.c:     -- fixed jitterbuffer destroyed on channel SIP/8305-0000001c
	
