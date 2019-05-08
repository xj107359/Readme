##Activity四种launchMode的区别##

Activity一共有以下四种launchMode：

- standard
- singleTop
- singleTask
- singleInstance

###1.standard###
每次跳转系统都会在task中生成一个新的FirstActivity实例，并且放于栈结构的顶部，当我们按下后退键时，才能看到原来的FirstActivity实例。
这就是standard启动模式，不管有没有已存在的实例，都生成新的实例。
![](https://raw.githubusercontent.com/xj107359/Readme/master/Picutres/Activity_launchModes/standard.gif)

###2.singleTop###
跳转时系统会先在栈结构中寻找是否有一个FirstActivity实例正位于栈顶，如果有则不再生成新的，而是直接使用。
当从SecondActivity跳转到FirstActivity时，系统发现存在有FirstActivity实例,但不是位于栈顶，于是重新生成一个实例。
这就是singleTop启动模式，如果发现有对应的Activity实例正位于栈顶，则重复利用，不再生成新的实例。

###3.singleTask###
在图中的下半部分是SecondActivity跳转到FirstActivity后的栈结构变化的结果，我们注意到，SecondActivity消失了，没错，在这个跳转过程中系统发现有存在的FirstActivity实例，于是不再生成新的实例，而是将FirstActivity之上的Activity实例统统出栈，将FirstActivity变为栈顶对象，显示到幕前。也许朋友们有疑问，如果将SecondActivity也设置为singleTask模式，那么SecondActivity实例是不是可以唯一呢？在我们这个示例中是不可能的，因为每次从SecondActivity跳转到FirstActivity时，SecondActivity实例都被迫出栈，下次等FirstActivity跳转到SecondActivity时，找不到存在的SecondActivity实例，于是必须生成新的实例。但是如果我们有ThirdActivity，让SecondActivity和ThirdActivity互相跳转，那么SecondActivity实例就可以保证唯一。

这就是singleTask模式，如果发现有对应的Activity实例，则使此Activity实例之上的其他Activity实例统统出栈，使此Activity实例成为栈顶对象，显示到幕前。

###4.singleInstance###
这种启动模式比较特殊，因为它会启用一个新的栈结构，将Acitvity放置于这个新的栈结构中，并保证不再有其他Activity实例进入。
我们看到从FirstActivity跳转到SecondActivity时，重新启用了一个新的栈结构，来放置SecondActivity实例，然后按下后退键，再次回到原始栈结构；图中下半部分显示的在SecondActivity中再次跳转到FirstActivity，这个时候系统会在原始栈结构中生成一个FirstActivity实例，然后回退两次，注意，并没有退出，而是回到了SecondActivity，为什么呢？是因为从SecondActivity跳转到FirstActivity的时候，我们的起点变成了SecondActivity实例所在的栈结构，这样一来，我们需要“回归”到这个栈结构。


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
