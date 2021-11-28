---
layout: post
title: 含泪记录postfix+javaMail实现邮件发送
date: 2018-12-25 11:04:18
tags: Linux
---
昨天刚刚实现了echo "content" |mail -s "title" xxxx@qq.com这种方式发送邮件. 但我需要在Java环境中发送邮件,所以又踏上了Java发送邮件的征程.

如同曾经在知识盲区的探索, 死磕一个小问题,找不到出口的感觉很痛苦. 直到找到一篇[博客](https://blog.csdn.net/qq_34292044/article/details/80468722), 不但解决了我的问题,还与我有相同的感想, 真的是一模一样的感受啊! 原话叫"中间经历了千辛万苦" , 还有一句同感的是"如果不出意料的话，你会得到以下的报错，而且翻遍全网基本都找不到一个有用的解释". 

# 安装sasldb saslauthd
```
#提供smtp的虚拟账户和密码服务 sasldb2包含在saslauthd当中
yum -y install cyrus-sasl cyrus-sasl-lib cyrus-sasl-plain  cyrus-sasl-devel
#当前mta查看
alternatives --display mta
#设置mta
alternatives --set mta /usr/sbin/sendmail.postfix
#再次查看mta
alternatives --display mta
#输出结果最后一行会有类似如下的提示：mta即设置完毕
#Current `best' version is /usr/sbin/sendmail.postfix.
```

# 在postfix配置文件中新增内容
```

下面的配置默认是没有的，需要在配置文件最后加上
#SMTP Config
#指定postfix兼容MUA使用不规则的smtp协议--主要针对老版本的outlook 此项对于本次配置无意义
broken_sasl_auth_clients = yes
#指定可以向postfix发起SMTP连接的客户端的主机名或ip地址
smtpd_client_restrictions = permit_sasl_authenticated

#此处permit_sasl_authenticated意思是允许通过sasl认证（也就是smtp链接时通过了账号、密码效验的用户）的所有用户
smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, 

#发件人在执行RCPT TO命令时提供的地址进行限制规则 此处照搬复制即可reject_unauth_destination

#指定postfix使用sasl验证 通俗的将就是启用smtp并要求进行账号、密码效验
smtpd_sasl_auth_enable = yes

#指定SMTP认证的本地域名 本次配置可以使用 smtpd_sasl_local_domain = '' 或干脆注释掉 默认为空
smtpd_sasl_local_domain = $mydomain

#取消smtp的匿名登录 此项默认值为noanonymous smtp若能匿名登录危害非常大 此项请务必指定为noanonymous
smtpd_sasl_security_options = noanonymous

#指定通过postfix发送邮件的体积大小 此处表示5M
message_size_limit = 5242880

######
```


# sasldb2建立smtp用户和密码
某种意义上来讲：smtp的账户密码建立也就是建立邮箱账户（类似jjonline@jjonline.cn的邮箱地址）。

```
#配置postfix启用sasldb2作为smtp的账号秘密效验方式
#编辑通过sasl启用smtp账号密码效验的配置
vi /etc/sasl2/smtpd.conf #vi写入或编辑内容如下：
#####
pwcheck_method: auxprop
auxprop_plugin: sasldb
mech_list: plain login CRAM-MD5 DIGEST-MD5
#####
#这里需要注意的是：这个配置文件的位置是64位机器上的，32位机器应该在：/usr/lib/sasl2/smtpd.conf

#创建smtp账号
saslpasswd2 -c -u `postconf -h mydomain` test #回车会要求输入密码，连续两次
#表示创建test@$mydomain的邮箱账号（也是smtp的账号）和密码
#本例就是创建test@jjonline.com.cn账号和密码
#此处注意的是smtp登录用的账号并不是单纯的用户名 而是整个邮箱地址字符串
#假设此处设置的smtp账号test@jjonline.com.cn密码为test123 下方测试时要用到

#查看sasldb2的用户和密码
sasldblistusers2
#此命令进用户查看sasldb的用户情况
#此命令回车后会输出诸如这样的内容：test@jjonline.com.cn: userPassword

#每次添加smtp用户完毕之后需重启postfix或reload
```

# 测试postfix配置文件并启动

```

#没有问题的话会返回着色[ok]字样
#启动postfix
systemctl restart postfix 

#更改sasldb2数据的权限，让postfix可以读取
chmod 755 /etc/sasldb2
```

# 测试smtp
```
telnet 192.168.2.182 25
#输入auth login 进行登录操作
auth login
334 VXN1cm5hbWU6 #会显示出这个
#输入账号为base64的加密字符串
dGVzdEBqdW1wc2VydmVyLmNvbQ==
#输入密码为base64的加密字符串
0dGVzdDEyMw==
235 2.7.0 Authentication seccessful  #认证成功后出来的东西

```

# 使用JavaMail发送邮件
```
package mail;
 
import javax.mail.*;
import javax.mail.internet.AddressException;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import java.io.UnsupportedEncodingException;
import java.util.Properties;
 
public class MailTest {
 
    private String sendMailUserName;
 
    private String sendMailPassword;
 
    private String toMail;
 
    private String title;
 
    private String content;
 
    public MailTest(String sendMailUserName, String sendMailPassword, String toMail, String title,
                    String content){
        this.sendMailPassword = sendMailPassword;
        this.sendMailUserName= sendMailUserName;
        this.toMail=toMail;
        this.title =title;
        this.content=content;
    }
 
 
    public void send() throws MessagingException, UnsupportedEncodingException {
        Properties props = new Properties();
        props.put("mail.smtp.host", "mail.xxx.com.cn");
        props.put("mail.smtp.auth", "true");
        Session session = Session.getInstance(props, new Authenticator() {
            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(sendMailUserName,sendMailPassword);
            }
        });
        Transport transport = session.getTransport("smtp");
        Message message = new MimeMessage(session);
        message.setFrom(new InternetAddress("root@xxx.com.cn","社会人")); //此处root可以随意更改
        message.setSubject(title);
        message.setText(content);
        message.setRecipient(Message.RecipientType.TO,new InternetAddress(toMail));
        transport.send(message);
    }
}
```
调用方法如下:
```
public static void main(String[] args) throws Exception {
        MailTest test = new MailTest("root@xxx.com.cn","your_password","xxx@163.com","test","test");
        test.send();
    }
```
