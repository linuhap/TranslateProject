如何在Debian中配置Tripewire IDS
================================================================================
本文是一篇关于Debian中安装和配置Tripewire的文章。它是Linux环境下基于主机的入侵检测系统（IDS）。tripwire的高级功能有检测并报告任何Linux中未授权的更改（文件和目录）。tripewire安装之后，会先创建一个基本的数据库，tripewire监控并检测新文件的创建修改和谁修改了它等等。如果修改过是合法的，你可以接受修改并更新tripwire的数据库。

### 安装和配置 ###

tripwire在Debian VM中的安装如下。

    # apt-get install tripwire

![installation](http://blog.linoxide.com/wp-content/uploads/2015/11/installation.png)

安装中，tripwire会有下面的配置提示。

#### 站点密钥创建 ####

tripwire需要一个站点口令来加密tripwire的配置文件tw.cfg和策略文件tw.pol。tripewire使用指定的密码加密两个文件。一个tripewire实例必须指定站点口令。


![site key1](http://blog.linoxide.com/wp-content/uploads/2015/11/site-key1.png)

#### 本地密钥口令 ####

本地口令用来保护tripwire数据库和报告文件。本地密钥用于阻止非授权的tripewire数据库修改。

![local key1](http://blog.linoxide.com/wp-content/uploads/2015/11/local-key1.png)

#### Tripwire配置路径 ####

tripewire配置存储在/etc/tripwire/twcfg.txt。它用于生成加密的配置文件tw.cfg。

![configuration file](http://blog.linoxide.com/wp-content/uploads/2015/11/configuration-file.png)

**Tripwire策略路径**

tripwire在/etc/tripwire/twpol.txt中保存策略文件。它用于生成加密的策略文件tw.pol。

![tripwire policy](http://blog.linoxide.com/wp-content/uploads/2015/11/tripwire-policy.png)

安装完成后如下图所示。

![installed tripewire1](http://blog.linoxide.com/wp-content/uploads/2015/11/installed-tripewire1.png)

#### Tripwire配置文件 (twcfg.txt) ####

tripewire配置文件（twcfg.txt）细节如下图所示。加密策略文件（tw.pol）,站点密钥（site.key）和本地密钥（hostname-local.key）如下所示。

    ROOT         =/usr/sbin
    
    POLFILE       =/etc/tripwire/tw.pol
    
    DBFILE       =/var/lib/tripwire/$(HOSTNAME).twd
    
    REPORTFILE   =/var/lib/tripwire/report/$(HOSTNAME)-$(DATE).twr
    
    SITEKEYFILE   =/etc/tripwire/site.key
    
    LOCALKEYFILE =/etc/tripwire/$(HOSTNAME)-local.key
    
    EDITOR       =/usr/bin/editor
    
    LATEPROMPTING =false
    
    LOOSEDIRECTORYCHECKING =false
    
    MAILNOVIOLATIONS =true
    
    EMAILREPORTLEVEL =3
    
    REPORTLEVEL   =3
    
    SYSLOGREPORTING =true
    
    MAILMETHOD   =SMTP
    
    SMTPHOST     =localhost
    
    SMTPPORT     =25
    
    TEMPDIRECTORY =/tmp

#### Tripwire策略配置 ####

在生成基础数据库之前先配置tripwire配置。有必要经用一些策略如/dev、 /proc 、/root/mail等。详细的twpol.txt策略文件如下所示。

    @@section GLOBAL
    TWBIN = /usr/sbin;
    TWETC = /etc/tripwire;
    TWVAR = /var/lib/tripwire;
    
    #
    # File System Definitions
    #
    @@section FS
    
    #
    # First, some variables to make configuration easier
    #
    SEC_CRIT      = $(IgnoreNone)-SHa ; # Critical files that cannot change
    
    SEC_BIN       = $(ReadOnly) ;        # Binaries that should not change
    
    SEC_CONFIG    = $(Dynamic) ;         # Config files that are changed
    # infrequently but accessed
    # often
    
    SEC_LOG       = $(Growing) ;         # Files that grow, but that
    # should never change ownership
    
    SEC_INVARIANT = +tpug ;              # Directories that should never
    # change permission or ownership
    
    SIG_LOW       = 33 ;                 # Non-critical files that are of
    # minimal security impact
    
    SIG_MED       = 66 ;                 # Non-critical files that are of
    # significant security impact
    
    SIG_HI        = 100 ;                # Critical files that are
    # significant points of
    # vulnerability
    
    #
    # Tripwire Binaries
    #
    (
    rulename = "Tripwire Binaries",
    severity = $(SIG_HI)
    )
    {
    $(TWBIN)/siggen            -> $(SEC_BIN) ;
    $(TWBIN)/tripwire        -> $(SEC_BIN) ;
    $(TWBIN)/twadmin        -> $(SEC_BIN) ;
    $(TWBIN)/twprint        -> $(SEC_BIN) ;
    }
    {
    /boot            -> $(SEC_CRIT) ;
    /lib/modules        -> $(SEC_CRIT) ;
    }
    
    (
    rulename = "Boot Scripts",
    severity = $(SIG_HI)
    )
    {
    /etc/init.d        -> $(SEC_BIN) ;
    #/etc/rc.boot        -> $(SEC_BIN) ;
    /etc/rcS.d        -> $(SEC_BIN) ;
    /etc/rc0.d        -> $(SEC_BIN) ;
    /etc/rc1.d        -> $(SEC_BIN) ;
    /etc/rc2.d        -> $(SEC_BIN) ;
    /etc/rc3.d        -> $(SEC_BIN) ;
    /etc/rc4.d        -> $(SEC_BIN) ;
    /etc/rc5.d        -> $(SEC_BIN) ;
    /etc/rc6.d        -> $(SEC_BIN) ;
    }
    
    (
    rulename = "Root file-system executables",
    severity = $(SIG_HI)
    )
    {
    /bin            -> $(SEC_BIN) ;
    /sbin            -> $(SEC_BIN) ;
    }
    
    #
    # Critical Libraries
    #
    (
    rulename = "Root file-system libraries",
    severity = $(SIG_HI)
    )
    {
    /lib            -> $(SEC_BIN) ;
    }
    
    #
    # Login and Privilege Raising Programs
    #
    (
    rulename = "Security Control",
    severity = $(SIG_MED)
    )
    {
    /etc/passwd        -> $(SEC_CONFIG) ;
    /etc/shadow        -> $(SEC_CONFIG) ;
    }
    {
    #/var/lock        -> $(SEC_CONFIG) ;
    #/var/run        -> $(SEC_CONFIG) ; # daemon PIDs
    /var/log        -> $(SEC_CONFIG) ;
    }
    
    # These files change the behavior of the root account
    (
    rulename = "Root config files",
    severity = 100
    )
    {
    /root                -> $(SEC_CRIT) ; # Catch all additions to /root
    #/root/mail            -> $(SEC_CONFIG) ;
    #/root/Mail            -> $(SEC_CONFIG) ;
    /root/.xsession-errors        -> $(SEC_CONFIG) ;
    #/root/.xauth            -> $(SEC_CONFIG) ;
    #/root/.tcshrc            -> $(SEC_CONFIG) ;
    #/root/.sawfish            -> $(SEC_CONFIG) ;
    #/root/.pinerc            -> $(SEC_CONFIG) ;
    #/root/.mc            -> $(SEC_CONFIG) ;
    #/root/.gnome_private        -> $(SEC_CONFIG) ;
    #/root/.gnome-desktop        -> $(SEC_CONFIG) ;
    #/root/.gnome            -> $(SEC_CONFIG) ;
    #/root/.esd_auth            -> $(SEC_CONFIG) ;
    #    /root/.elm            -> $(SEC_CONFIG) ;
    #/root/.cshrc                -> $(SEC_CONFIG) ;
    #/root/.bashrc            -> $(SEC_CONFIG) ;
    #/root/.bash_profile        -> $(SEC_CONFIG) ;
    #    /root/.bash_logout        -> $(SEC_CONFIG) ;
    #/root/.bash_history        -> $(SEC_CONFIG) ;
    #/root/.amandahosts        -> $(SEC_CONFIG) ;
    #/root/.addressbook.lu        -> $(SEC_CONFIG) ;
    #/root/.addressbook        -> $(SEC_CONFIG) ;
    #/root/.Xresources        -> $(SEC_CONFIG) ;
    #/root/.Xauthority        -> $(SEC_CONFIG) -i ; # Changes Inode number on login
    /root/.ICEauthority            -> $(SEC_CONFIG) ;
    }
    
    #
    # Critical devices
    #
    (
    rulename = "Devices & Kernel information",
    severity = $(SIG_HI),
    )
    {
    #/dev        -> $(Device) ;
    #/proc        -> $(Device) ;
    }

#### Tripwire 报告 ####

**tripwire –check** 命令检查twpol.txt文件并基于此文件生成tripwire报告如下。如果twpol.txt中有任何错误，tripwire不会生成报告。

![tripwire report](http://blog.linoxide.com/wp-content/uploads/2015/11/tripwire-report.png)

**文本形式报告**

    root@VMdebian:/home/labadmin# tripwire --check
    
    Parsing policy file: /etc/tripwire/tw.pol
    
    *** Processing Unix File System ***
    
    Performing integrity check...
    
    Wrote report file: /var/lib/tripwire/report/VMdebian-20151024-122322.twr
    
    Open Source Tripwire(R) 2.4.2.2 Integrity Check Report
    
    Report generated by:         root
    
    Report created on:           Sat Oct 24 12:23:22 2015
    
    Database last updated on:     Never
    
    Report Summary:
    
    =========================================================
    
    Host name:                   VMdebian
    
    Host IP address:             127.0.1.1
    
    Host ID:                     None
    
    Policy file used:             /etc/tripwire/tw.pol
    
    Configuration file used:     /etc/tripwire/tw.cfg
    
    Database file used:           /var/lib/tripwire/VMdebian.twd
    
    Command line used:           tripwire --check
    
    =========================================================
    
    Rule Summary:
    
    =========================================================
    
    -------------------------------------------------------------------------------
    
    Section: Unix File System
    
    -------------------------------------------------------------------------------
    
    Rule Name                       Severity Level   Added   Removed Modified
    
    ---------                       --------------   -----   ------- --------
    
    Other binaries                 66               0       0       0      
    
    Tripwire Binaries               100               0       0       0      
    
    Other libraries                 66               0       0       0      
    
    Root file-system executables   100               0       0       0      
    
    Tripwire Data Files             100               0       0       0      
    
    System boot changes             100               0       0       0      
    
    (/var/log)
    
    Root file-system libraries     100               0       0       0      
    
    (/lib)
    
    Critical system boot files     100               0       0       0      
    
    Other configuration files       66               0       0       0      
    
    (/etc)
    
    Boot Scripts                   100               0       0       0      
    
    Security Control               66               0       0       0      
    
    Root config files               100               0       0       0      
    
    Invariant Directories           66               0       0       0      
    
    Total objects scanned: 25943
    
    Total violations found: 0
    
    =========================Object Summary:================================
    
    -------------------------------------------------------------------------------
    
    # Section: Unix File System
    
    -------------------------------------------------------------------------------
    
    No violations.
    
    ===========================Error Report:=====================================
    
    No Errors
    
    -------------------------------------------------------------------------------
    
    *** End of report ***
    
    Open Source Tripwire 2.4 Portions copyright 2000 Tripwire, Inc. Tripwire is a registered
    
    trademark of Tripwire, Inc. This software comes with ABSOLUTELY NO WARRANTY;
    
    for details use --version. This is free software which may be redistributed
    
    or modified only under certain conditions; see COPYING for details.
    
    All rights reserved.
    
    Integrity check complete.

### 总结 ###

本篇中，我们学习安装配置开源入侵检测软件tripwire。首先生成基础数据库并通过比较检测出任何改动（文件/文件夹）。然而，tripwire并不是实时监测的IDS。

--------------------------------------------------------------------------------

via: http://linoxide.com/security/configure-tripwire-ids-debian/

作者：[nido][a]
译者：[geekpi](https://github.com/geekpi)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]:http://linoxide.com/author/naveeda/


