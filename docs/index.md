## Introduction

tinyTerm2 is a rewrite of [tinyTerm](https://yongchaofan.github.io/tinyTerm) using C++ and [FLTK](http://fltk.org), [libssh2](http://libssh2.org) used for ssh2 functions same as tinyTerm. The result is a cross platform terminal emulator that supports all the features of tinyTerm that is still simple and small. At release 1.2.0, the win32 exectuable is 645KB, win64 executable 823KB, macOS executable 1MB, and Linux x86_64 executable 1MB. 
	
<table>
	<tr>
	    <td width="568">
		<video width="560" height="412" controls>
			<source src="tinyTerm2.mp4" type="video/mp4">
			Your browser does not support the video tag.
		</video>
	    </td>
	    <td>
		<h4>Stable release: <a href="https://github.com/yongchaofan/tinyterm2">1.2.0</a></h4>
		<h4>Appx package:<br/><a href="https://www.microsoft.com/store/apps/9NXGN9LJTL05">Microsoft Store</a></h4>
		<h4>Portable version:<br/>
	32-bit: <a href="https://github.com/yongchaofan/tinyTerm2/releases/download/1.2.0/tinyTerm2.exe">tinyTerm.exe</a><br/>
	64-bit: <a href="https://github.com/yongchaofan/tinyTerm2/releases/download/1.2.0/tinyTerm2_x64.exe">tinyTerm64.exe</a><br/>
	WinXP: <a href="https://github.com/yongchaofan/tinyTerm2/releases/download/1.2.0/tinyTerm2_XP.exe">tinyTerm64.exe</a></h4>
		<h4>License: <a href="https://github.com/yongchaofan/tinyTerm2/blob/master/LICENSE">GPL 3.0</a></h4>
	    </td>
	</tr>
</table>

Windows 10 user should install from Microsoft Store
Windows 7 or older, or prefer portable apps download exe files
Linux/macOS user please build from source

---

## Usage notes

> ### Terminal Emulation
> Each time a connection is made using the connect dialog, an entry will be added to the Term menu, simply select the menu entry to make the same connection again. 
> 
> When local edit is enabled, user is presented with a "tinyTerm >" prompt, simply type commands like "telnet 192.168.1.1" or "ssh admin@172.16.1.1" to make connection, or type commands like "ipconfig", "ping 192.168.1.1", "tracert google.com" to execute
>
> For serial connections, available serial ports will be auto detected and added to the ports dropdown list in connection dialog on Windows.
> 
> for netconf connections, typing netconf messages is possible but not really practical, it's better to use a text or xml editor to compose the messages and then drag&drop to the terminal window. 
> 
> Press left mouse button and drag to select text, double click to select the whole word under mouse pointer, middle click to paste selected text without copying to clipboard. Right click bring up context menu for copy, paste, select all and paste selection
>
> Scroll back buffer holds 8000 lines of text by default, use the options menu to change buffer size if desired. Use pageup key or mouse wheel to scroll back, scrollbar will appear when scrolled back, and will hide when scrolled all the way down.
>
> ### Command Autocompletion
> When local edit mode is enabled, key presses are not sent to remote host until "Enter" or "Tab" key is pressed, and the input is auto completed using command history, every command typed in local edit mode is added to command history to complete future inputs. Command history is saved to tinyTerm.hist at exit, then loaded into memory at the next start of tinyTerm. 
>  
> Command history file is saved as %USERPROFILE%\documents\tinyTerm\tinyTerm.hist on Windows, ~/tinyTerm/tinyTerm.hist on macOS/Unix/Linux by default, copy tinyTerm.hist to the same folder as tinyTerm.exe for portable use. Since the command history file is just a plain text file, user can edit the file outside of tinyTerm to put additional commands in the list for command auto-completion. For example put all TL1 commands in the history list to use as a dictionary.
> 
> Release 1.1 added one more option "Send to all", when enabled locally edited command will be sent to all open tabs on "Enter"
> 
> ### Batch Automation
> To automate the execution of commands, simply drag and drop a list of commands from text editor, or select "Run..." from Script menu and select a text file with all the commands to be executed, tinyTerm send one command at a time, wait for prompt string before sending the next command, to avoid overflowing the receive buffer of the remote host or network device. 
> 
> Most command line interface system uses a prompt string to tell user it’s ready for the next command, for example "> " or "$ " used by Cisco routers. tinyTerm will auto detect the prompt string used by remote host when user is typing commands interactively, and use the detected prompt string during scripting. Additionally, prompt string can be set in the script using special command "!Prompt {str}", refer to appendix A for details and other special commands supported for scripting. 
> 
> ### SCP integration
> When a SSH or SFTP session is established, simply drag and drop files to the terminal window will cause those files to be transfered to remote host using SCP or SFTP put, remote files will be created in the current directory. 
> 
> To copy file from server to a local folder, select the filename in the terminal window, then chose "scp_to_folder.js" from script menu.
> 
> ### xmodem support
> When a SERIAL sission is established, simply drag and drop a file to the terminal window will cause the file been sent using xmodem protocol, only send function of the original xmodem protocol is supported, CRC optional, xmodem-1K is not supported. This is added to 1.1 release for to support bootstraping of MCUs on embeded system like Ardiuno
> 
> ### FTPd/TFTPd/HTTPd
> FTPd/TFTPd has been removed to reduce amount of platform specific code, user require FTPd/TFTPd can continue to use tinyTerm
> 
> A built in HTTP server is started as soon as tinyTerm is started, for the first instance of tinyTerm running, HTTPd listens at 127.0.0.1:8080, the second instance listens at 127.0.0.1:8081, the third instance listens at 127.0.0.1:8082, so on and so forth. Since it's listening on 127.0.0.1 only, the HTTPd will only accept connections from local machine, for the purpose of scripting. 
> 

___

## Scripting interface

The Built in HTTP server will accept GET request from local machine, which means any program running on the same machine, be it a browser or a javascript or any other script, can connect to tinyTerm and request either a file or the result of a command, 

for example:

	- http://127.0.0.1:8080/tinyTerm.html	return tinyTerm.html from current working folder
	- http://127.0.0.1:8080/?ls%20-al	return the result of "ls -al" from remote host
	- http://127.0.0.1:8080/?!Selection 	return current selected text from scroll back buffer
	
Notice the "!" just before "Selection" in the last example, when a command is started with "!", it's being executed by tinyTerm instead of sent to remote host, There are about 30 tinyTerm commands supported for the purpose of making connections, setting options, sending commands, scp files, turning up ssh2 tunnels, see appendix for the list.

The script scp_to_folder.js referenced in the trailer, is a perfect example of scripting tinyTerm

```js
// Javascript to download a highlighted file via scp.
var xml = new ActiveXObject("Microsoft.XMLHTTP");
var port = "8080/?";
if ( WScript.Arguments.length>0 ) port = WScript.Arguments(0)+"/?";
var filename = term("!Selection");
var objShell = new ActiveXObject("Shell.Application")
var objFolder = objShell.BrowseForFolder(0, "Destination", 0x11, "")
if ( objFolder ) term("!scp :"+filename+" "+objFolder.Self.path);

function term( cmd )
{
   xml.Open ("GET", "http://127.0.0.1:"+port+cmd, false);
   xml.Send();
   return xml.responseText;
}
```



## Appendix. list of supported tinyTerm commands:
These commands can be used programatically for scripting or interactively in local edit mode.

### Connection
    !com3:9600,n,8,1    connect to serial port com3 with settings 9600,n,8,1
    !telnet 192.168.1.1 telnet to 192.168.1.1
    !ssh pi@piZero:2222 ssh to host piZero port 2222 with username pi
    !sftp -P 2222 jun01 sftp to host jun01 port 2222
    !netconf rtr1       netconf to port 830(default) of host rtr1
    !disconn            disconnect from current connection
    !{DOS command}      execute command and display result, e.g. ping 192.168.1.1
    !Find {string}      search for {string} in scroll back buffer

### Automation
    !Clear              set clear scroll back buffer
    !Prompt $%20        set command prompt to “$ “, for CLI script
    !Timeout 30	        set time out to 30 seconds for CLI script
    !Wait 10            wait 10 seconds during execution of CLI script
    !Waitfor 100%       wait for “100%” from host during execution of CLI script
    !Log test.log       start/stop logging with log file test.log

### Scripting
    !Disp test case #1  display “test case #1” in terminal window
    !Send exit          send “exit” to host
    !Recv               get all text received since last Send/Recv
    !Echo               toggle local echo on/off
    !Selection          get current selected text

### Options
    ~TermSize 100x40    set terminal size to 100 cols x 40 rows
    ~Transparency 192   set window transparency level to 192/255
    ~LocalEdit		Enable local edit
    ~FontFace Consolas  set font face to “Consolas”
    ~FontSize 18        set font size to 18

### Extras
    !scp tt.txt :t1.txt secure copy local file tt.txt to remote host as t1.txt
    !scp :*.txt d:/     secure copy remote files *.txt to local d:/

    !tun 127.0.0.1:2222 127.0.0.1:22 
                        start ssh2 tunnel from localhost port 2222 to remote host port 22
    !tun                list all ssh2 tunnels 
    !tun 3256           close ssh2 tunnel number 3256
