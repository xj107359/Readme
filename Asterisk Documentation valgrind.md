# Asterisk Documentation valgrind

NOTE: These pages are automatically updated once per day from the Asterisk subversion repository when the repository changes revisions. Any changes made to this page will be automatically overwritten with the latest version from http://svn.digium.com/view/asterisk/branches/.

If you're having certain types of crashes, such as those associated with memory corruption, a bug marshal may ask you to run Asterisk under valgrind. You should follow these steps, to give the bug marshal the maximum amount of information about the crash.

## 1. Run 'make menuselect' and in the Compiler Options, enable MALLOC_DEBUG and DONT_OPTIMIZE. 
A bug marshal may also ask you to enable additional compiler flags, such as DEBUG_THREADS, depending upon the nature of the issue.

## 2. Rebuild and install Asterisk.

## 3. Run Asterisk as follows:

>
	valgrind --suppressions=/usr/src/asterisk-16.3.0/contrib/valgrind.supp --leak-check=full --show-leak-kinds=all --log-fd=9 asterisk -vvvvcg 9>valgrind.txt
	
Where /usr/src/asterisk/ is location of asterisk source code.

>
	localhost*CLI> memory atexit list on
	localhost*CLI> memory atexit summary byline

## 4. Reproduce the issue. 
Following the manifestation of the issue (or when the process crashes), upload the valgrind.txt to the issue tracker.

Please note that even if valgrind prevents Asterisk from crashing, the information logged may STILL be of use to developers, so please upload the resulting log file whether Asterisk crashes or not.