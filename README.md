
		SCSM Agent Restart
        LogRhythm System Montior Agent Remote Maintenance SmartResponse
		Greg Foss | @heinzarelli | greg.foss@logrhythm.com
		v0.1 -- December, 2016

## [About]

Blog Post => TBD

From time to time, LogRhythm’s System Monitor agents may become unresponsive, get disabled, or become overloaded and stop checking in with the Mediator. Every time this happens, we end up getting notified via alarm escalations to the SOC. Then, we need to log in to the affected server(s), attempt to restart the agent, and investigate the initial cause of the agent going down in the first place. So, to resolve the issue, we utilize PowerShell Remoting to check the current agent status and restart the agent if it shows a down or hung state. Then, the script gathers general diagnostics from the system to determine if restarting the agent was successful. Normally, restarting the agent alone will fix most issues, however if there are other problems, we now have a place to start looking.

![SCSM=Restart](/images/scsm-restart.png)

## [How To]

#####Run SCSM-Restart on local host:

 		PS C:\> .\scsm-restart.ps1
        This gathers default data and stores the results in the directory that the script was executed from.
				or download and execute the script (you should stage this on your own server / within your own repo)
		PS C:\> IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/LogRhythm-Labs/System-Monitor-Agent-Maintenance/master/scsm-restart.ps1');

#####Run SCSM-Restart on remote host:

		PS C:\> .\scsm-restart.ps1 -target 10.10.10.10
			or
		PS C:\> .\scsm-restart.ps1 -target 10.10.10.10 -username <name> -password <pass>

    Caveats:
        You will need to ensure that psremoting and remote signed execution is enabled on the remote host.

#####What if PSRemoting and Unrestricted Execution are disabled?

    Remotely enable PSRemoting and Unrestricted PowerShell Execution using PsExec and PSSession, then run PSRecon

        Option 1 -- WMI:
            PS C:\> wmic /node:"10.10.10.10" process call create "powershell -noprofile -command Enable-PsRemoting -Force" -Credential Get-Credential

        Option 2 - PsExec:
            PS C:\> PsExec.exe \\10.10.10.10 -u [admin account name] -p [admin account password] -h -d powershell.exe "Enable-PSRemoting -Force"

        Next...

            PS C:\> Test-WSMan 10.10.10.10
            PS C:\> Enter-PSSession 10.10.10.10
            [10.10.10.10]: PS C:\> Set-ExecutionPolicy RemoteSigned -Force

        Then run the scsm-restart.ps1 script as described above. Be careful to lock down who has the ability to execute PowerShell on the host remotely.

## [Parameter Breakdown]

		-target 	:	Define the remote host to restart the SCSM agent on.
		-username 	:	Administrative Username - can be supplied on the command-line or hard-coded into the script.
		-password 	: 	Administrative Password - can be supplied on the command-line or hard-coded into the script. <== Bad idea!!

        If neither username / password parameter is supplied, you will be prompted for credentials -- the safest option aside from local execution.

## [License]

Copyright (c) 2016 LogRhythm

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
