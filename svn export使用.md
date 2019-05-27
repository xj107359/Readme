	#!/bin/sh
	echo "
	
	SVN exporting..."
	rm -f ./svn_up.log
	
	echo "svn export html" >> ./svn_up.log
	rm -Rf /var/www/html
	svn export svn://X.X.X.X/html /var/www/html > svn_html.log 2>&1
	tail -n 1 svn_html.log >> ./svn_up.log
	
	echo "svn export html-app" >> ./svn_up.log
	rm -Rf /var/www/html-app
	svn export svn://X.X.X.X/html-app /var/www/html-app > svn_app.log 2>&1
	tail -n 1 svn_app.log >> ./svn_up.log

	...

	echo "

	SVN Log..."
	cat ./svn_up.log


>
	svn export html
	已导出版本 800。
	svn export html-app
	已导出版本 243。
