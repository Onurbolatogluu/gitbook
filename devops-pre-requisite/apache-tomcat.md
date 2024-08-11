# ⬛ Apache Tomcat

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Apache Software Foundation tarafından geliştirilen Tomcat, dinamik Java tabanlı web uygulamalarını çalıştırmak için yaygın olarak kullanılır.

* Tomcat, Java Servlet API'sini destekler ve Java Servlet'lerini çalıştırır.
* JavaServer Pages (JSP) teknolojisini destekler ve JSP sayfalarını çalıştırır.
* Tomcat, Java WebSocket API'sini destekler ve gerçek zamanlı, çift yönlü iletişim sağlar.

Ubuntu 22.04 için kurulum rehberi : [https://www.webhi.com/how-to/how-to-install-tomcat-on-ubuntu/](https://www.webhi.com/how-to/how-to-install-tomcat-on-ubuntu/)



#### **Tomcat  Temel Yapılandırma Dosyaları**

1. server.xml: `server.xml`, Tomcat'in ana yapılandırma dosyasıdır ve sunucu yapılandırmasını içerir. Bu dosya `conf/` dizininde bulunur.

```xml
<Server port="8005" shutdown="SHUTDOWN">
    <Service name="Catalina">
        <Connector port="8080" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" />
        <Engine name="Catalina" defaultHost="localhost">
            <Host name="localhost"  appBase="webapps"
                  unpackWARs="true" autoDeploy="true">
            </Host>
        </Engine>
    </Service>
</Server>
```



2. web.xml: `web.xml` dosyası, uygulama tabanlı yapılandırma ayarlarını içerir. Bu dosya, her web uygulamasının `WEB-INF/` dizininde bulunur.

```xml
<web-app>
    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>com.example.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
</web-app>
```



#### Diğer Dosyalar:

```bash
# 3. context.xml
# Bireysel web uygulamaları için konteks yapılandırmalarını içerir.
# Her web uygulaması için özel yapılandırmalar bu dosyada belirtilir.
# Dosya yolu: conf/context.xml (global) ve her web uygulaması için META-INF/context.xml

<Context>
    <Resource name="jdbc/MyDB" auth="Container"
              type="javax.sql.DataSource" maxActive="100" maxIdle="30" maxWait="10000"
              username="dbuser" password="dbpassword" driverClassName="com.mysql.jdbc.Driver"
              url="jdbc:mysql://localhost:3306/mydb"/>
</Context>

# 4. tomcat-users.xml
# Tomcat'in kullanıcıları ve rollerini tanımlayan yapılandırma dosyasıdır.
# Yönetici ve diğer kullanıcı hesapları bu dosyada tanımlanır.
# Dosya yolu: conf/tomcat-users.xml

<tomcat-users>
    <role rolename="manager-gui"/>
    <user username="admin" password="admin" roles="manager-gui"/>
</tomcat-users>

# 5. logging.properties
# Tomcat'in loglama yapılandırmalarını belirler.
# Hangi logların, nereye ve nasıl yazılacağını tanımlar.
# Dosya yolu: conf/logging.properties

handlers = java.util.logging.ConsoleHandler
.level = INFO
java.util.logging.ConsoleHandler.level = FINE
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter

# 6. catalina.properties
# Tomcat'in iç ayarlarını ve çeşitli sistem özelliklerini belirler.
# Dosya yolu: conf/catalina.properties

common.loader="${catalina.base}/lib,${catalina.base}/lib/*.jar"
server.loader=
shared.loader=

# 7. catalina.policy
# Java güvenlik politikalarını belirler.
# Hangi kodun hangi kaynaklara erişebileceğini kontrol eder.
# Dosya yolu: conf/catalina.policy

grant {
    permission java.security.AllPermission;
};

# 8. host-manager.xml
# Tomcat Host Manager web uygulamasının yapılandırma dosyasıdır.
# Dosya yolu: conf/Catalina/localhost/host-manager.xml

<Context privileged="true" antiResourceLocking="false" docBase="${catalina.home}/webapps/host-manager">
    <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.\d+\.\d+\.\d+|::1"/>
</Context>

# 9. manager.xml
# Tomcat Manager web uygulamasının yapılandırma dosyasıdır.
# Dosya yolu: conf/Catalina/localhost/manager.xml

<Context privileged="true" antiResourceLocking="false" docBase="${catalina.home}/webapps/manager">
    <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.\d+\.\d+\.\d+|::1"/>
</Context>

```
