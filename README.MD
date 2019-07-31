# Giriş
Bu yazıda ubuntu üzerinde apache server kurmak isteyenlere bir yardımcı kaynak olması amacıyla bazı şeyler paylaşmayı planlıyorum. Aslında burada yapacağım asıl olay bir bilgisayar üzerinde birden fazla port açıp apache'yi o iki portu dinleyerek aynı anda http servislerini kullanmalarını sağlamak olacak. Aşağıdaki adımları takip ederek bunu rahatlıkla yapabilirsiniz.
## Notlar
1. İşlemlerimi ubuntu 18.04 üzerinde gerçekleştirdim.
2. İşlemleri gerçekleştirirken root kullanıcısında başka bir kullanıcı tercih ediniz.
3. Adımları sıralı bir şekilde uygulayınız. 
4. Eğer komutlar sonucunda beklenen sonucu alamazsanız internetten araştırmaya çekinmeyiniz.
5. Daha önceden sisteminizde apache2 yüklüyse ve başarısız olduysanız lütfen kaldırınız.

# Adım 1: Apache Kurulumu
Yerel paket dizininizi güncelleyerek işe başlayınız. `sudo apt update` Daha sonrasında *apache2*'yi kurunuzu `sudo apt install apache2`
# Adım 2: Güvenlik Duvarını Ayarlama
Mevcut olan `ufw` profillerini kontrol ediniz. Bunun için `sudo ufw app list` komutunu çalıştırmanız gerekecektir ve output aşağıdakine benzer olacaktır.
```sh
sbi@ubuntu:~$ sudo ufw app list
Available applications:
  Apache
  Apache Full
  Apache Secure
  CUPS
```
Apache'yi etkinleştiriniz. `sudo ufw allow 'Apache'` ve ardından `sudo ufw status` komutunu uygulayınız. Çıktınız aşağıdaki gibi olmalıdır.
```sh
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Apache                     ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Apache (v6)                ALLOW       Anywhere (v6)
```
Eğer problem yaşıyorsanız. `Status: inactive` gibi bir cevap alıyorsanız. Adım 2.1'i uygulayınız. Eğer problem yaşamıyorsanız 3'ten devam ediniz.

## Adım 2.1: Problemi Gider
```
sbi@ubuntu:~$ sudo ufw enable
Firewall is active and enabled on system startup
sbi@ubuntu:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
Apache                     ALLOW       Anywhere                  
Apache (v6)                ALLOW       Anywhere (v6)             

```
# Adım 3: Web Sunucunuzu kontrol edin
`sudo systemctl status apache2` komutunu yazın ve aşağıdaki gibi bir geri dönüş alın. Eğer active çıktısı alırsanız işler yolunda demektir.
```
● apache2.service - The Apache HTTP Server
   Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
  Drop-In: /lib/systemd/system/apache2.service.d
           └─apache2-systemd.conf
   Active: active (running) since Tue 2018-04-24 20:14:39 UTC; 9min ago
 Main PID: 2583 (apache2)
    Tasks: 55 (limit: 1153)
   CGroup: /system.slice/apache2.service
           ├─2583 /usr/sbin/apache2 -k start
           ├─2585 /usr/sbin/apache2 -k start
           └─2586 /usr/sbin/apache2 -k start
```
Tarayıcınızda http://localhost ya da http://127.0.0.1 adresine gidebilir ve aşağıdaki çıktıyı alabilirsiniz.
<img src="/dosyalar/apache.png" style="width:100%;">
# Adım 4: Sanal Sunucuları Ayarlama
Apache sayesinde tek bir sunucuda birden fazla alan barındırabilirsiniz. Aşağıdaki adımları takip ediniz. Böylece hem 80. hem de 81. portta çalışan internet siteleri ayağa kaldıracağız.
```sh
sudo mkdir -p /var/www/example.com/html
sudo chown -R $USER:$USER /var/www/example.com/html
sudo chmod -R 755 /var/www/example.com
```
Daha sonrasında basit bir html kodu yazalım. `sudo nano /var/www/example.com/html/index.html`
```sh
<html>
    <head>
        <title>Ben 80 de çalışırım.</title>
    </head>
    <body>
        <h1>Çalışıyor...</h1>
    </body>
</html>
```
`sudo nano /etc/apache2/sites-available/example.com.conf` şimdi bir conf dosyası yazalım ve 80. portta hangi dizin altında çalışacağını belirtelim.
```sh
<VirtualHost *:80>
    ServerAdmin admin@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Dosyayı etkinleştirin. `sudo a2ensite example.com.conf` ve varsayılan ayarları devre dışı bırakın `sudo a2dissite 000-default.conf`. Şimdi bir test gerçekleştirin. `sudo apache2ctl configtest` aşağıdaki gibi bir çıktı görmelisiniz.
```sh
Syntax OK
```

Şimdi diğer internet sayfamız için ayarlamalarımızı yapalım. Aslında 4. adımdakilerin aynısını tekrardan yapacağız. Bundan dolayı tek bir kod bloğunda göstereceğim.
```
sudo mkdir -p /var/www/example2.com/html
sudo chown -R $USER:$USER /var/www/example2.com/html
sudo chmod -R 755 /var/www/example2.com
sudo nano /var/www/example2.com/html/index.html

<html>
    <head>
        <title>Ben 81 de çalışırım.</title>
    </head>
    <body>
        <h1>Çalışıyor...(81)</h1>
    </body>
</html>

sudo nano /etc/apache2/sites-available/example2.com.conf

<VirtualHost *:81>
    ServerAdmin admin@example.com
    ServerName example2.com
    ServerAlias www.example2.com
    DocumentRoot /var/www/example2.com/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

sudo a2ensite example2.com.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
```

Artık portlarımızı açmalıyız. Ve dinlemeye başlamalıyız. Bunun için aşağıdkai adımları takip ediniz. `sudo vi /etc/apache2/ports.conf` Daha sonra `Listen 81` kodunu ekleyip dosyayı kaydediniz. Şimdi `sudo systemctl restart apache2` işleminden sonra tarayıcınızda 127.0.0.1:80 için ve 127.0.0.1:81 için farklı cevaplar göreceksiniz.

# PHP Kurulumu
Php kurulumu için lütfen aşağıdaki adımları takip ediniz. Ben kurulum yaparken php5'i kurmayı deneyeceğim ancak php güvenlik nedenlerinden dolayı php7'yi kurabilir. Bundan dolayı daha sonrasında versiyon değiştirme işlemi yapacağım.  
```sh
sudo su
apt-get purge `dpkg -l | grep php| awk '{print $2}' |tr "\n" " "`
add-apt-repository ppa:ondrej/php
apt-get update
apt-get install build-essential libaio1 php5.6-dev php-pear php5.6-soap php5.6-sybase php5.6-gd php5.6-xdebug php5.6-xml
apt-get install libapache2-mod-php5.6
a2dismod php7.0 ; a2enmod php5.6 ; service apache2 restart
```
Ve php5 sisteme yüklendi.
