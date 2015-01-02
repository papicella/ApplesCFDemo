<h1> Apples Cloud Foundry Spring Boot / Spring Data JPA / Thymeleaf demo </h1>

The following demo shows how to bind to a MYSQL service from a data bound Spring Boot [Spring Data JPA]. 
This ensures you don't have to write any code to retrieve the bound data source connection details as 
spring auto configures this for you using spring cloud service connector library.

![alt tag](https://dl.dropboxusercontent.com/u/15829935/albums-view.png)

<h2> Steps to run locally </h2>

- Clone code as follows

```
> git clone https://github.com/papicella/ApplesCFDemo.git
```

- package as shown below

```
> mvn package
```

- cd target
- Run demo prior to deploying to Cloud Foundry as follows

```
> java -jar ApplesCFDemo-0.0.1-SNAPSHOT.jar
```

- Access as follows in a browser

```
http://localhost:8080/
```

<h2> Deploy to cloud foundry <h2>

When deploying to cloud foundry you must bind the app to a MYSQL service as shown in the manifest.yml below.

```
applications:
- name: pas-albums
  memory: 512M
  instances: 1
  host: pas-albums
  domain: apj.fe.pivotal.io
  path: ./ApplesCFDemo-0.0.1-SNAPSHOT.jar
  services:
    - dev-mysql
```

This time when pushed to cloud foundry it will use the MYSQL bound service, using a class in the deployed JAR file as follows

```
package pas.cloud.webapp;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.cloud.config.java.AbstractCloudConfig;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

public class DataSourceConfiguration
{
    private static Log logger = LogFactory.getLog(DataSourceConfiguration.class);
    @Configuration()
    @Profile({ "cloud" })
    public static class CloudConfiguration extends AbstractCloudConfig {
        @Bean(destroyMethod = "close")
        public javax.sql.DataSource dataSource() {
            logger.info(" CloudConfiguration : using connectionFactory to set data source");
            return connectionFactory().dataSource();
        }
    }
}
```

- Example of deployment

```
[Fri Jan 02 17:14:12 papicella@:~/cf/APJ-vcloud/spring-data-jpa-thymeleaf ] $ cf push -f manifest-apj.yml
Using manifest file manifest-apj.yml

Creating app pas-albums in org ANZ / space development as pas...
OK

Using route pas-albums.apj.fe.pivotal.io
Binding pas-albums.apj.fe.pivotal.io to pas-albums...
OK

Uploading pas-albums...
Uploading app files from: ApplesCFDemo-0.0.1-SNAPSHOT.jar
applications:
Uploading 837.3K, 135 files
Done uploading
OK
Binding service dev-mysql to app pas-albums in org ANZ / space development as pas...
OK

Starting app pas-albums in org ANZ / space development as pas...
-----> Downloaded app package (26M)
-----> Java Buildpack Version: v2.4 (offline) | https://github.com/cloudfoundry/java-buildpack.git#7cdcf1a
-----> Downloading Open Jdk JRE 1.7.0_60 from http://download.run.pivotal.io/openjdk/lucid/x86_64/openjdk-1.7.0_60.tar.gz (found in cache)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (0.9s)
-----> Downloading Spring Auto Reconfiguration 1.4.0_RELEASE from http://download.run.pivotal.io/auto-reconfiguration/auto-reconfiguration-1.4.0_RELEASE.jar (found in cache)

-----> Uploading droplet (57M)

0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App pas-albums was started using this command ``

Showing health and status for app pas-albums in org ANZ / space development as pas...
OK

requested state: started
instances: 1/1
usage: 512M x 1 instances
urls: pas-albums.apj.fe.pivotal.io
last uploaded: Fri Jan 2 06:14:58 +0000 2015

     state     since                    cpu    memory           disk
#0   running   2015-01-02 05:15:33 PM   0.0%   351.3M of 512M   118.3M of 1G
```
