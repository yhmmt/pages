% SQUID

大阪大学D3センター大規模計算機システムSQUIDの使用方法を説明する。

## 初期設定

二段階認証でSQUIDにログインできるようにする。

### 参考
- [ログイン方法(SQUID)](https://www.hpc.cmc.osaka-u.ac.jp/system/manual/squid-use/login/)


### Google Authenticatorのインストール
iPhoneでApp Storeを開いてGoogle Authenticatorをインストールする

### 二段階認証の設定

`ssh`で初めてSQUIDにログインするとき、パスワードを入力すると二段階認証の設定が始まる。

~~~{.console}
$ ssh ユーザ名@squidhpc.hpc.cmc.osaka-u.ac.jp
...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'squidhpc.hpc.cmc.osaka-u.ac.jp' (ED25519) to the list of known hosts.
(ユーザ名@squidhpc.hpc.cmc.osaka-u.ac.jp) Password: 事前に設定したパスワードを入力
 Initialize google-authenticator
...
QRコードが表示されるのでGoogle Authenticatorでスキャンする
...
Enter code from app (-1 to skip): -1
Code confirmation skipped
Your emergency scratch codes are:
 Completed to initialized google-authenticator
 After logout, and please relogin. It will be authenticated 'OTP' by Google Authenticator.
Hit [Enter] key
Connection to squidhpc.hpc.cmc.osaka-u.ac.jp closed.
~~~

### SQUIDにログイン

次回以降`ssh`でSQUIDにログインするときは、パスワードに加えて認証コードを入力する。

~~~{.console}
$ ssh ユーザ名@squidhpc.hpc.cmc.osaka-u.ac.jp
(ユーザ名@squidhpc.hpc.cmc.osaka-u.ac.jp) Password: 事前に設定したパスワードを入力
(ユーザ名@squidhpc.hpc.cmc.osaka-u.ac.jp) Verification code: Google Authenticatorに表示された6桁の数字を入力
[ユーザ名@squidhpc3 ~]$
~~~

## 利用者の追加

ウェブサイトからグループ利用者を追加する。

### 参考

- [利用者管理WEBシステム](https://manage.hpc.cmc.osaka-u.ac.jp/saibed/)
- [グループ利用者管理手順（登録、削除、情報変更）](https://www.hpc.cmc.osaka-u.ac.jp/wp-content/uploads/2014/10/118.pdf)
- ウェブサイトから必要事項を記入して追加申請

## 利用状況の確認

ポイントとストレージの利用状況は`usage_view`で確認できる。

~~~{.console}
$ usage_view

---------------+ Group: G16193 +---------------

[Group summary]

          SQUID points   HDD(GiB)   SSD(GiB)
------- -------------- ---------- ----------
usage              0.0        0.0        0.0
limit           5250.0     5120.0        0.0
remain          5250.0     5120.0        0.0
rate(%)            0.0        0.0        0.0


[Detail]

          SQUID points     Home(GiB)   HDD(GiB)   SSD(GiB)
------- -------------- ------------- ---------- ----------
v60916             0.0   0.0 /  10.0        0.0        0.0


[Node-hours]

                            Usage     Available*
------------------ -------------- --------------
CPU           node           0.00       20601.96
GPU           node           0.00        3366.29
Vector        node           0.00        5460.10

 * Available Node-hour is estimated based on remains of SQUID, Bonus point.

~~~

## ジョブの投入のコツ

elapstim_reqが長いと待ち時間が長くなる可能性があるので、
予め計算時間を見積もっておいて、elapstim_reqはそれより少し長く設定する。


### 参考
- [SQUIDポータル](https://squidportal.hpc.cmc.osaka-u.ac.jp/portal/login)
- [計算の実行待ち時間を短縮するために](https://www.hpc.cmc.osaka-u.ac.jp/system/manual/reducing-wait-time/)
