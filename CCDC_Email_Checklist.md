# CCDC 2024 - Email

        Prepared by Gabe Ron on 2024-01-23
        Edited by Gabe Ron on 2024-01-30


__**TAKE SCREENSHOTS**__

## Phase 1 - Survive (Don't Die)
* Checkpoints:
    - Start:    00:00
    - End:      00:30

---

1. Baseline

        find / -ctime -7 -or -mtime -0.5 | grep -Ev “^(/sys|/dev|/proc|/run|/boot)” > changed.log
        who > who.log
        last > last.log
        netstat -tulnpa > netstat.log

2. Rotate credentials & create a backdoor

        passwd root
        useradd -m -o -u 0 BACKUP_USER
        passwd BACKUP_USER

3. Disable sshd & cron

        systemctl disable --now sshd
        systemctl disable --now CRON_SERVICE
        systemctl disable --now atd
        killall sshd
        killall ssh
        killall CRON_SERVICE

        atq
        killall atd
    
4. Create backups in `/usr/local/share`

        mkdir -p /usr/local/share/librechutney
        cp -r /etc /usr/local/share/librechutney
        cp -r /usr /usr/local/share/librechutney
        cp -r /var /usr/local/share/librechutney
        cp -r /home /usr/local/share/librechutney
        cp -r /bin /usr/local/share/librechutney
        cp -r /sbin /usr/local/share/librechutney
        
4. Downloads

        wget http://tinyurl.com/4sh2m3m3
        git clone yabbadabbadoo
        sources.list

4. Purge unnecessary packages and (re)install necessary packages

        apt -y update --fix-missing
        apt -y remove iptables ufw firewalld cron --purge
        apt -y install iptables auditd

5. Deploy firewall script
    * Allow
        - Loopback
        - Any necessary competition connections (i.e. your SSH)
        - Necessary host services

6. Rotate credentials

        passwd root
        passwd BACKUP_USER

7. Curl other scored webpages

        curl WEB_PAGE_ADDRESS > /usr/local/share/librechutney/something
        chattr +i /usr/local/share/librechutney
        rm -rf $(which chattr)

## Phase 2 - Adapt (Stay Alive)
* Checkpoints:
    - Start:    00:30
    - End:      01:00

---

1. More baselining

        uname -a
        ip a
        ip ro

2. Check running processes

        ps -e | less
        ps aux | less


3. Check running services

        systemctl [status]
        service --status-all

4. Check service configuration
    * Usually in `/etc`
    * Sometimes in `/usr`
        - `/usr/lib`
        - `/usr/share`
        - `/usr/local`
    * Check service file
        - systemd
            * `systemd status SERVICE`
            * `/etc/systemd/system`
            * `/usr/lib/systemd/system`
        - SysV
            * `/etc/init`
            * `/etc/init.d`

3. Disable unnecessary services

        systemctl disable --now SERVICE
        echo 'manual' | sudo tee /etc/init/SERVICE.override
        update-rc.d SERVICE disable && service SERVICE stop

4. Check and disable any webshells or risky functionality
    * Disable php execution in php.ini
    * Find which php file is loaded

        php_conf=$(php -i | grep "php.ini")
        vim $php_conf
            disable_functions=php_uname,listen,shell_exec,dl,set_time_limit,exec,system,source,virtual,posix_getpwnam,posix_getpwuid,posix_kill,phpinfo,popen,passthru,eval,get_shell

5. Copy backups to other systems
    * `python -m http.server`
    * `rsync`
    * `scp` won't work as ssh will most likely be disabled

6. Inspect `/etc/sudoers`, `/etc/passwd`, `/etc/group`, `/etc/shadow`

        visudo
        vipw
        vigr

    * If these commands aren't installed, use whatever editor your prefer to view these files, but be aware this can break the permissions on them!
    * Disable users, don't delete

        usermod -L bad_user

7. Check & set file permissions and attributes (TEST THIS)
    * web root --- +i
    * `/etc/passwd` 0644 +i
    * `/etc/shadow` 0600 +i
    * `/etc/groups` 0644 +i
    * `/etc/sudoers` 0644 +i
    * config files 755 +i

8. Check for SUID binaries

        find / -type f -perm -4000 -ls -exec chmod -s {} \; 2>/dev/null
        chmod u+s `which su` `which sudo

9. Check `/tmp`, `/run`, `/root` and any other admin home directories, and the home directory of suspicious users

10. Secure scored services

11. Check for cron or at jobs

## Phase 3 - Overcome (Bunker Down)
1. Check passwd binary

### Fail2ban
A strict `fail2ban` config makes it far easier to defend your box as repeat failures to access services will ban the IP it's coming from.
Here's a basic `jail.local` to set up incremental ban times and enable the SSH jail.

        [DEFAULT]
        bantime.increment = true
        bantime.rndtime = 60s
        bantime.multipliers = 1 5 30 60 300 720 1440 2880
        bantime.overalljails = true
        ignoreself = true
        ignoreip = ::1 127.0.0.1/8 [ANY EXTRA IPs TO IGNORE]
        maxretry = 3

        [sshd]
        enable = true
        port = ssh
        logpath = %(sshd_log)s
        backend = %(sshd_backend)s

If you notice a malicious IP, or you need to block something from your box, you can manually add IPs with

        fail2ban-client <JAIL> banip <IP> ... <IP>

And you can unban all IPs with

        fail2ban-client unban --all

## Phase 4 - Recover (Get Back Up)
1. Collect screenshots of the `.bash_history` of compromised acounts, edited or misconfigured files, system logs of the attack, and other relevant files or documents

2. Investigate how the attack happened and determine what security measures failed

3. Write a report or explain your findings to whoever is writing the report. Be thorough and concise, and write the report soon after the attack.
