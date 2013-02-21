---
layout: post
title: "Ubuntu 12.04上使用Postfix+Dovecot+MySQL安装支持多域名的邮件服务器"
date: 2012-10-21 16:14
comments: true
categories: Linux Ubuntu Postfix Dovecot MySQL
---

基本可按照Linode上的[这篇教学](https://library.linode.com/email/postfix/dovecot-mysql-ubuntu-10.04-lucid)进行，但此文是基于Ubuntu 10.04的，在Ubuntu 12.04上安装时需要做出以下变动。

本文内容已经在[Hostigation](https://hostigation.com/billing/aff.php?aff=405)的KVM-VPS上测试通过。

<!--more-->

## 安装软件包

需增加`dovecot-mysql`包。

{% codeblock lang:bash %}
apt-get install dovecot-mysql
{% endcodeblock %}

## 配置Postfix

* 最后一步编辑Postfix配置文件时，执行以下代码替换原文代码，删除了新版本Postfix不支持的一些配置项了，注意将主机名替换为你自己的主机名。

{% codeblock lang:bash %}
postconf -e 'myhostname = server.example.com'
postconf -e 'mydestination = server.example.com, localhost, localhost.localdomain'
postconf -e 'mynetworks = 127.0.0.0/8'
postconf -e 'message_size_limit = 30720000'
postconf -e 'virtual_alias_domains ='
postconf -e 'virtual_alias_maps = proxy:mysql:/etc/postfix/mysql-virtual_forwardings.cf, mysql:/etc/postfix/mysql-virtual_email2email.cf'
postconf -e 'virtual_mailbox_domains = proxy:mysql:/etc/postfix/mysql-virtual_domains.cf'
postconf -e 'virtual_mailbox_maps = proxy:mysql:/etc/postfix/mysql-virtual_mailboxes.cf'
postconf -e 'virtual_mailbox_base = /home/vmail'
postconf -e 'virtual_uid_maps = static:5000'
postconf -e 'virtual_gid_maps = static:5000'
postconf -e 'smtpd_sasl_auth_enable = yes'
postconf -e 'broken_sasl_auth_clients = yes'
postconf -e 'smtpd_sasl_authenticated_header = yes'
postconf -e 'smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination'
postconf -e 'smtpd_use_tls = yes'
postconf -e 'smtpd_tls_cert_file = /etc/postfix/smtpd.cert'
postconf -e 'smtpd_tls_key_file = /etc/postfix/smtpd.key'
postconf -e 'proxy_read_maps = $local_recipient_maps $mydestination $virtual_alias_maps $virtual_alias_domains $virtual_mailbox_maps $virtual_mailbox_domains $relay_recipient_maps $relay_domains $canonical_maps $sender_canonical_maps $recipient_canonical_maps $relocated_maps $transport_maps $mynetworks $virtual_mailbox_limit_maps'
postconf -e 'virtual_transport = dovecot'
{% endcodeblock %}

* 编辑/etc/postfix/master.cf文件，取消submission配置的注释，此举可打开STARTTLS默认的587端口。

{% codeblock lang:bash %}
submission inet n - n – – smtpd
{% endcodeblock %}

## 配置SASL认证服务

修改文件`/etc/postfix/sasl/smtpd.conf`为如下内容。

{% codeblock lang:bash %}
pwcheck_method: saslauthd
mech_list: plain login
allow_plaintext: true
auxprop_plugin: sql
sql_engine: mysql
sql_hostnames: 127.0.0.1
sql_user: mail_admin
sql_passwd: mail_admin_password
sql_database: mail
sql_select: select password from users where email = '%u@%r'
{% endcodeblock %}

## 配置Dovecot

这是符合新版本Dovecot要求的`/etc/dovecot/dovecot.conf`配置文件。

{% codeblock lang:bash %}
log_timestamp = "%Y-%m-%d %H:%M:%S "
mail_location = maildir:/home/vmail/%d/%n/Maildir
namespace {
  inbox = yes
  location =
  prefix = INBOX.
  separator = .
  type = private
}
passdb {
  args = /etc/dovecot/dovecot-sql.conf
  driver = sql
}
protocols = imap pop3
service auth {
  unix_listener /var/spool/postfix/private/auth {
    group = postfix
    mode = 0660
    user = postfix
  }
  unix_listener auth-master {
    mode = 0600
    user = vmail
  }
  user = root
}
ssl = required
ssl_cert = </etc/ssl/certs/dovecot.pem
ssl_key = </etc/ssl/private/dovecot.pem
userdb {
  args = uid=5000 gid=5000 home=/home/vmail/%d/%n allow_all_users=yes
  driver = static
}
protocol lda {
  auth_socket_path = /var/run/dovecot/auth-master
  log_path = /home/vmail/dovecot-deliver.log
  postmaster_address = postmaster@example.com
}
protocol pop3 {
  pop3_uidl_format = %08Xu%08Xv
}
{% endcodeblock %}