# slackware小记
------
## 常用的镜像及网址
**官网**  http://mirrors.slackware.com/
一些常用的源及网站:
> * https://slackbuilds.org/
> * ftp://ftp.slackware.com/pub/slackware/slackware64-14.2
> - http://ftp.slackware.com/pub/slackware/
## 配置代理及使用
* export http_proxy=http://proxy-server.mycorp.com:3128/
* export http_proxy=http://USERNAME:PASSOWRD@proxy-server.mycorp.com:3128/
* wget --proxy-user=USERNAME --proxy-password=PASSWORD http://path.to.domain.com/some.html
* lynx -pauth=USER:PASSWORD http://domain.com/path/html.file
* curl --proxy-user user:password http://url.com/
