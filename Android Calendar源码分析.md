## Android calendar源码分析 ##

**MarkdownPad** is a full-featured Markdown editor for Windows.

### 日历的代码进行这样归类 ###

处理各种视图的模块
月视图	/month/MonthByWeekFragment.java
周视图	DayFragment.java
日视图	DayFragment.java
日程视图	/agenda/AgendaFragment.java

设置模块	CalendarSettingsActivity.java

日程编辑和新建部分		/event/EditEventFragment.java

单条日程信息的显示部分	/EventInfoActivity.java

Alerts日程消息通知部分	/alert/AlertActivity.java

Utils.java
工具函数，具体日期的数字显示值的获取就是通过它，各个账户颜色的取值也是通过它，公历的一些转换也是它。

CalendarController.java
CalendarViewAdapter是专门为了显示视图切换而定义的一个适配器类
CalendarController从字面上来理解应该叫做控制器，我们知道日历的很多事件响应都是在AllInOneActivity中发生的，但是我们的处理却不是在这里，而是在响应的视图对应的activity或者fragment之中，如何让捕获到事件的AllInOneActivity去通知相应的fragment去处理这些事件呢，就是通过CalendarController 。CalendarController相当于一个中转站。


### AllInOneActivity中有这些事件 ###
- 切换视图的spinner事件
- actionbar上的跳转到今天的按钮事件。
- menu菜单中的新建，刷新，搜索，设置等菜单事件。


### CalendarController事件分发 ###

当在某个Fragment中想要发出一个事件的时候，该Fragment会用到自己实例化的CalendarController对象（mController），例如下面的样子：

	mController.sendEvent(mContext, EventType.GO_TO, day, day, -1,
		mIsMiniMonth ? ViewType.CURRENT : ViewType.DETAIL,
		CalendarController.EXTRA_GOTO_DATE
			| CalendarController.EXTRA_GOTO_BACK_TO_PREVIOUS, null, null);

这里的sendEvent函数有很多参数，但是他们最终都会调用CalendarController中重载了的public void sendEvent(Object sender, final EventInfo event)，而之前的那些零散的参数都被封装在了这两个参数里。

sendEvent函数最终是要把EventInfo（事件信息）传递给事件处理对象（eventHandler），一个事件的处理对象可能还不止一个，因此需要遍历一次所有存在的事件处理对象，并调用这些对象的handleEvent(event)函数。

	for (Iterator<Entry<Integer, EventHandler>> handlers =
			eventHandlers.entrySet().iterator(); handlers.hasNext();) {
		Entry<Integer, EventHandler> entry = handlers.next();
		int key = entry.getKey();
		if (mFirstEventHandler != null && key == mFirstEventHandler.first) {
			// If this was the 'first' handler it was already handled
			continue;
		}
		EventHandler eventHandler = entry.getValue();
		if (eventHandler != null
			&& (eventHandler.getSupportedEventTypes() & event.eventType) != 0) {
			if (mToBeRemovedEventHandlers.contains(key)) {
				continue;
			}
			eventHandler.handleEvent(event);
			handled = true;
		}
	}

那么事件处理对象，究竟是什么东西呢，其实他们非常平常，基本上，那些能通过自己的CalendarController成员变量mController发送事件的类本身也是事件处理对象，也就是我们熟悉的AllInOneActivity、MonthByWeekFragment、AgendaFragment、DayFragment。

这些类都继承了EventHandler这个接口（也就是事件处理对象的原型），并实现了其中的void handleEvent(EventInfo event);方法。EventHandler这个接口是在CalendarController中定义的，如下：

	public interface EventHandler {
		long getSupportedEventTypes();
		void handleEvent(EventInfo event);

		/**
		* This notifies the handler that the database has changed and it should
		* update its view.
		*/
		void eventsChanged();
	}

刚才提到了遍历事件处理对象，也给出了遍历的那段代码，可以看到其实实在遍历这个集合eventHandlers，eventHandlers中的成员是通过public void registerEventHandler(int key, EventHandler eventHandler)方法得到的。

	public void registerEventHandler(int key, EventHandler eventHandler) {
		synchronized (this) {
			if (mDispatchInProgressCounter > 0) {
				mToBeAddedEventHandlers.put(key, eventHandler);
			} else {
				eventHandlers.put(key, eventHandler);
			}
		}
	}

在mDispatchInProgressCounter > 0)（正在处理中的事件个数）的条件下，这时候只能将注册的新EventHandler放在mToBeAddedEventHandlers里，表示当前还不能立即处理，因为上次的还没处理完，如果mDispatchInProgressCounter<0则放在普通的eventHandlers里面，当sendEvent发生的时候，马上调用这些eventHandlers的handleevent方法。

mDispatchInProgressCounter正在处理的事件个数，是在sendEvent方法中被synchronized包括起来的程序段中赋值的。

AllInOneActivity中对四个视图的fragment进行了注册，当然并不是同时，假如当前是月视图，注册的当然是MonthByWeekFragment，要想了解如何注册的请看AllInOneActivity的setMainPane方法。或者参考我讲解AllInOneActivity的那篇文章。

其实，我们应该注意到，registerEventHandler只是对Fragment进行了注册（还有一些非AllInOneActivity的activity这里不讲解），但是事件处理对象中还有重要的AllInOneActivity，AllInOneActivity也有handleEvent的能力，AllInOneActivity为什么没有自己给自己注册一下呢，既然没有注册那么AllInOneActivity中的handleEvent方法不是永远不会被调用么？

当然不是AllInOneActivity也给自己注册了的，只不过是调用CalendarController的registerFirstEventHandler方法,方法定义如下：

	public void registerFirstEventHandler(int key, EventHandler eventHandler) {
		synchronized (this) {
			registerEventHandler(key, eventHandler);
			if (mDispatchInProgressCounter > 0) {
				mToBeAddedFirstEventHandler = new Pair<Integer, EventHandler>(key, eventHandler);
			} else {
				mFirstEventHandler = new Pair<Integer, EventHandler>(key, eventHandler);
			}
		}
	}

之所以要区分于其他，是因为整个日历的架构中要求优先调用AllInOneActivity的handleEvent。假如mFirstEventHandler不为空，则最先处理他，registerFirstEventHandler也在方法之类调用了registerEventHandler，这样AllInOneActivity就有优先和普通两种身份。

- 不管是registerFirstEventHandler还是registerEventHandler都有一个key参数，key是标识是不是同一个handler的键。


通过sendEvent发来的事件请求不一定都需要相应的handler来处理，如以下情况：

handler != null && (handler.getSupportedEventTypes() & event.eventType) != 0

Handler不存在的情况

事件类型不在支持的类型中

第一种情况好像还没遇到过，第二种情况的意思是，当我告知某个handler需要处理一个事件时，先判断这个handler自身是否支持该事件的处理，通过调用handler的getSupportedEventTypes方法就可以得到他所支持的事件类型。这里我们以AllInOneActivity为例：

getSupportedEventTypes本来是CalendarController的接口EventHandler的一个抽象方法，因为AllInOneActivity继承了该接口，因此AllInOneActivity重写了该方法，AllInOneActivity的实现如下：

	public long getSupportedEventTypes() {
		return EventType.GO_TO | EventType.VIEW_EVENT | EventType.UPDATE_TITLE;
	}

可以看到AllInOneActivity能处理的事件也只有三种，跳转、查看日程、更新日历title上的日期。对于其他的handler，能处理的可能更少。

我们来看看其他的handler支持哪些事件。可能还没有找出所有的handler。

MonthByWeekFragment
	public long getSupportedEventTypes() {
		return EventType.GO_TO | EventType.EVENTS_CHANGED;
	}

DayFragment                       
	public long getSupportedEventTypes() {
		return EventType.GO_TO | EventType.EVENTS_CHANGED;
	}

AgendaFragment
	public long getSupportedEventTypes() {
		return EventType.GO_TO | EventType.EVENTS_CHANGED | ((mUsedForSearch)?EventType.SEARCH:0);
	}

SearchActivity
	public long getSupportedEventTypes() {
		return EventType.VIEW_EVENT | EventType.DELETE_EVENT;
	}

……………………此处略去若干字…………….

（getSupportedEventTypes（）方法之后紧跟着handleEvent方法，这里就不一一列出handleEvent的具体实现了）

当没有hanlder来处理这些不支持的事件的时候，sendEvent会继续执行下面的代码，这些特殊的事件往往都和视图没有什么区别，在任何视图打开都是相同的结果，比如打开设置界面，打开搜索界面，打开选择要显示的日历界面等。代码如下：

	if (event.eventType == EventType.LAUNCH_SETTINGS) {
		launchSettings();
		return;
	}
	// Launch Calendar Visible Selector
	if (event.eventType == EventType.LAUNCH_SELECT_VISIBLE_CALENDARS) {
		launchSelectVisibleCalendars();
		return;
	}

他会直接调用相应的函数，而不是交给handler处理。

Handler的这种处理机制运用了java中典型的回调机制，和观察者模式。

### 日视图 ###

相比其他的视图，日视图会显得很“简单”，说简单是因为在日视图下，除了actionbar之外没有任何其他可见的控件，唯一的控件是ViewSwitcher，但是是不可见的。ViewSwitcher里面装着两个显示日信息的自定义View—DayView。这两个View交替着切换，这使得我们能在日视图中一天一天的横向滚动。

ViewSwitcher不说了，这里要说的是这个DayView。

DayView是一个完全自定义的View，直接继承至View类。一般我们都是继承一些现成的控件，比如TextView，但是这里直接是将需要的东西一点一点的画出来，所以要弄懂，可能需要对android canvas绘图方面有比较多的了解。DayView的重点和难点有如下几个：

如何画出每天需要的信息，比如在恰当的位置画出每个小时的数字和横线，以及当天存在的事件。

如何实现在手指水平左右拖动时，能够看到天与天之间的过度效果；如何在垂直滚动时，响应用户的意图，并保证基本不和水平滚动冲突。

第一个问题纯粹是canvas方面的问题，很琐碎，但是逻辑不是那么强。

第二个问题涉及到触摸设备中的手势处理，要做到日历这么人性化的效果，还是比较难的。

DayView中的手势处理结合了比较原始的onTouchEvent()方法和GestureDetector（封装好了常用的几种手势）。onTouchEvent首先捕获触摸事件，将这个事件，传递给GestureDetector，然后交给GestureDetector分析。根据GestureDetector中的数据，我们得到一些有用的数值（比如在scroll滚动事件发生的时候，我们不断获取滚动点的水平偏移量，然后响应的View也移动相同的偏移量，从而实现水平滑动），然后根据这些数据来重绘View-即调用view的onDraw函数。

给出一小段代码让你找到感觉，不必细究每句话的含义，只需明白大致的处理方式，在ondraw开始几行有如下代码：

	float yTranslate = -mViewStartY + DAY_HEADER_HEIGHT + mAlldayHeight; 
	// offset canvas by the current drag and header position 
	canvas.translate(-mViewStartX, yTranslate); 
	// clip to everything below the allDay area 
	Rect dest = mDestRect; 
	dest.top = (int) (mFirstCell - yTranslate); 
	dest.bottom = (int) (mViewHeight - yTranslate); 
	dest.left = 0; 
	dest.right = mViewWidth; 
	canvas.save(); 
	canvas.clipRect(dest); 
	// Draw the movable part of the view 
	doDraw(canvas);
 
其中canvas.translate(-mViewStartX, yTranslate)可以实现view的平移。

手势处理我们先进入捕获触摸事件的onTouchEvent函数：

onScroll（）
onScroll函数不多，我就直接将整个函数贴出来了：

	@Override 
	public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) { 
		if (DEBUG) Log.e(TAG, "GestureDetector.onScroll"); 
		if (mTouchStartedInAlldayArea) { 
			if (Math.abs(distanceX) < Math.abs(distanceY)) { 
				return false; 
			} 
			// don't scroll vertically if this started in the allday area 
			distanceY = 0; 
		} 
		Log.i("scroll", "DayView::onScroll:distanceX="+distanceX+"and distanceY="+distanceY); 
		DayView.this.doScroll(e1, e2, distanceX, distanceY); 
		return true; 
	}

onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY)有四个参数，这里其实只用了后面两个distanceX，distanceY，字面意思上分别指水平和垂直滚动的距离。

根据我的log信息来看，当我手指向左的时候distanceX为正，手指向右的时候distanceX为负。手指向上的时候distanceY为正，手指向下的时候distanceY为负。

### 周视图 ###
周视图和日视图都是由DayFragment管理的，因为两者有很相似的地方：

- 都是直接用自定义的view绘出来的
- 都是两个在viewswitcher中的dayview交替显示
- 整个视图基本上都只有一个dayview，没有多余的控件（actionbar除外，周视图上面的周几提示除外）

在相同的Fragment下能显示出不同view的原因是allinoneactivity他们各自的参数不一样.

case ViewType.DAY:  //如果当前为日视图
…
frag = new DayFragment(timeMillis, 1);

case ViewType. WEEK:  //如果当前为周视图
…
frag = new DayFragment(timeMillis, 7);

一周有7天，所以第二个参数为7 ，一日只有一天，所以第二个参数为1.
DayFragment中在通过将第二个参数传递给DayView，DayView根据不同的参数绘出不同的界面。

因此要分析这两个视图，重点还是集中在DayFragment和DayView两个文件中。
除此之外，你还应该了解ViewSwitcher的用法。

我们从DayFragment开始吧。

DayFragment

首先是DayFragment的构造函数：

	public DayFragment(long timeMillis, int numOfDays) {
		mNumDays = numOfDays;
		if (timeMillis == 0) {
			mSelectedDay.setToNow();
		} else {
			mSelectedDay.set(timeMillis);
		}
	}

mNumDays保存了刚刚提到参数。

然后在makeView（）方法中这个值传递给了DayView

	public View makeView() {
		mTZUpdater.run();
		DayView view = new DayView(getActivity(), CalendarController
			.getInstance(getActivity()), mViewSwitcher, mEventLoader, mNumDays);
		view.setId(VIEW_ID);
		view.setLayoutParams(new ViewSwitcher.LayoutParams(
			LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
		view.setSelected(mSelectedDay, false, false);
		return view;
	}

makeView会在ViewSwitcher加载的时候自动调用，因为我们在DayFragment里面将ViewSwitcher的“工厂”设置成了当前DayFragment对象。这是在onCreateView（）中完成的：

	public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState) {
		View v = inflater.inflate(R.layout.day_activity, null);
		mViewSwitcher = (ViewSwitcher) v.findViewById(R.id.switcher);
		mViewSwitcher.setFactory(this);
		mViewSwitcher.getCurrentView().requestFocus();
		((DayView) mViewSwitcher.getCurrentView()).updateTitle();
		return v;
	}

onCreateView获取到了Fragment的相对应的xml文件day_activity，里面有个ViewSwitcher控件，mViewSwitcher.setFactory(this)将给自己装入子元素的工厂设置成当前DayFragment对象，以便调用上面提到的makeView，然后下面两行

	mViewSwitcher.getCurrentView().requestFocus();
	((DayView) mViewSwitcher.getCurrentView()).updateTitle();

是对mViewSwitcher做一些初始化。

onCreate：
 
	public void onCreate(Bundle icicle) {
		super.onCreate(icicle);
		Context context = getActivity();

        mInAnimationForward = AnimationUtils.loadAnimation(context, R.anim.slide_left_in);
 
        mOutAnimationForward = AnimationUtils.loadAnimation(context, R.anim.slide_left_out);
 
        mInAnimationBackward = AnimationUtils.loadAnimation(context, R.anim.slide_right_in);
 
        mOutAnimationBackward = AnimationUtils.loadAnimation(context, R.anim.slide_right_out);

        mEventLoader = new EventLoader(context);
	}

### 月视图 ###

1.日期跳转问题：

这个问题是在处理点击today按钮之后，新建日程的日期不是当天，而是当月的1号这个故障时发现的。

对于总是所在月的1号，这个很好处理，因为google原生就是这样设计的。只需在
SimpleDayPickerFragment.java文件中将goto（）函数的

	mFirstDayOfMonth.set(mTempTime);  
	mFirstDayOfMonth.monthDay = 1;  
	millis = mFirstDayOfMonth.normalize(true);  
	if(isFirst==true){  
		isFirst=false;  
		setMonthDisplayed(mSelectedDay, true);  
	}else{  
		setMonthDisplayed(mFirstDayOfMonth, true);  
 	}
 

大概是在450行左右
改成：

	mFirstDayOfMonth.set(mSelectedDay);  
	mFirstDayOfMonth.monthDay = mSelectedDay.monthDay;  
	millis = mFirstDayOfMonth.normalize(true);  
	setMonthDisplayed(mSelectedDay, true);

这样改完之后，不会出现跳转之后总是跳转日期所在月的1号这样的问题了。

但是，会出现有时候跳转后的日期与我们期望的不一致（比如我点击的是today按钮，那么跳转后的日期应该是今天，如果我是在菜单里选着跳转的某一日期，那么跳转后的日期应该是我选择的这个日期）。开始以为是日期的问题，即我给出的跳转日期是错误的，然而在跟踪代码之后发现，只有当跳转过程中出现动画效果的时候才会出现这个情况。

于是我将      goto函数的      

	if (animate) {  
		mListView.smoothScrollToPositionFromTop(  
			position, LIST_TOP_OFFSET, GOTO_SCROLL_DURATION);  
 		onScrollStateChanged(mListView, OnScrollListener.SCROLL_STATE_IDLE);  
		return true;  
	}

这段代码去掉，发现正常了。

看来是在需要动画的情况下所执行的代码导致的。

至于为什么需要动画，好像是根据日期来判断的，判断的语句很奇怪，根本看不出需要动画的理由啊（经过思考，当离所要跳转的日期较远的时候，则没有动画效果，因为这样用户会等很久，而比较近，则慢慢滑动到目标日期，反正比较近，也花不了多长时间）。

在MonthByWeekFragment中

	boolean animate = true;  
	if (mDaysPerWeek * mNumWeeks * 2 < Math.abs(  
			Time.getJulianDay(event.selectedTime.toMillis(true), event.selectedTime.gmtoff)  
			- Time.getJulianDay(mFirstVisibleDay.toMillis(true), mFirstVisibleDay.gmtoff)  
			- mDaysPerWeek * mNumWeeks / 2)) {  
		animate = false;  
   }
 

 

而且为什么执行了动画为真条件下的代码之后跳转日期就会出现问题，也还没去研究


















Enjoy first-class Markdown support with easy access to  Markdown syntax and convenient keyboard shortcuts.

Give them a try:

- **Bold** (`Ctrl+B`) and *Italic* (`Ctrl+I`)
- Quotes (`Ctrl+Q`)
- Code blocks (`Ctrl+K`)
- Headings 1, 2, 3 (`Ctrl+1`, `Ctrl+2`, `Ctrl+3`)
- Lists (`Ctrl+U` and `Ctrl+Shift+O`)

### See your changes instantly with LivePreview ###

Don't guess if your [hyperlink syntax](http://markdownpad.com) is correct; LivePreview will show you exactly what your document looks like every time you press a key.

### Make it your own ###

Fonts, color schemes, layouts and stylesheets are all 100% customizable so you can turn MarkdownPad into your perfect editor.

### A robust editor for advanced Markdown users ###

MarkdownPad supports multiple Markdown processing engines, including standard Markdown, Markdown Extra (with Table support) and GitHub Flavored Markdown.

With a tabbed document interface, PDF export, a built-in image uploader, session management, spell check, auto-save, syntax highlighting and a built-in CSS management interface, there's no limit to what you can do with MarkdownPad.
