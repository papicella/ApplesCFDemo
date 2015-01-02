<h1> Apples Cloud Foundry Spring Boot / Spring Data JPA / Thymeleaf demo </h1>

The following demo shows how to bind to a MYSQL service from a data bound Spring Boot [Spring Data JPA]. 
This ensures you don't have to write any code to retrieve the bound data source connection details as 
spring auto configures this for you using spring cloud service connector library.

<h2> Steps to run locally <h2>

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
