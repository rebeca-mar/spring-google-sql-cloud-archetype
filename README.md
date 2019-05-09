# Este es un fork al proyecto original por adamo-fig

Este es un arquetipo en [Maven](https://maven.apache.org/download.cgi) para funcionar en [Google App Engine](https://cloud.google.com/appengine/?hl=es), con la versión 8 de [Java](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html). Los comandos mostrados son para la plataforma Windows.
<br>
Ya que se trata de un proyecto en Maven, es requisito tenerlo instalado en la máquina, así como el [SDK de Google](https://cloud.google.com/sdk/). Recuerde **colocar en la variable de entorno PATH** la ruta de Maven después de instalarlo.

## Ejecución en local
El comando para ejecutar en local es:
```sh
mvn install
mvn clean
mvn appengine:run
```
Si desea evitar ejecutar las clases test en local use el comando
```sh
mvn appengine:run -DskipTests
```
Si usa gradle utilice
```sh
gradle clean appengineRun -x test
```
Se trabaja en la descripción para un proyecto con gradle...

En la ruta `src/main/resources` se encuentra `keys.json`, el es una cuenta de servicio con `rol` de `SQL Client` para ejecutar el componente o backend en local y realizar despliegues en la nube en un ambiente de desarrollo o QA. Se obtiene desde el apartado de `IAM` de Google App Engine. Este archivo es `vital` para la ejecución del backend, de lo contrario no se podrá conectar a la Base de Datos en Cloud SQL.

## Despliegue en la nube
Se recomienda realizar el despliegue después de hacer pruebas en local.<br>
El despliegue en la nube no obedece los ambientes destinados, algo diferente a como sucede en el ambiente de desarrollo y por eso el cambio de propiedades debe hacerse de forma manual cada vez que se requiera. Se puede evitar esto ejecutando tareas con automic, por ejemplo.<br>
Para realizar un despliegue siga estos pasos:

* Comente la propiedad `spring.cloud.gcp.credentials.location` dentro del archivo application.properties<br>
Autentíquese con el usuario autorizado para realizar el despliegue. Se puede hacer desde la consola:
```sh
gcloud auth login
```
* Coloque el ID del proyecto de Google al cual se va a realizar el despliegue. **PROJECT_ID** es el id del proyecto:
```sh
gcloud config set project PROJECT_ID
```
* Verifique que el nombre del servicio de App Engine que contendrá el backend sea el correcto<br>
* Ejecute el comando para realizar el despliegue en la nube:
```sh
mvn clean appengine:deploy
```
El `app.yaml` de la aplicación lo construye el plugin por default al realizar el despliegue. Puede consultarlo en la parte de Servicios del componente dentro de la Console de Google.
<br>

## Archivos a considerar

* El archivo `pom.xml` contiene las dependencias del proyecto.<br>
* `application.properties`, que está en `src/main/resources` contiene la configuración del proyecto. Su contenido se expone más adelante.
* `appengine-web.xml` que está en `src/main/webapp/WEB-INF`
* ServletInitializer.java en la ruta `src/main/java/com/example/demo` es la clase que inicia la aplicación.
* la configuración de CORS se hace en el archivo `CORSConfig`, está en `src/main/java/com/example/demo/config`

En el `pom.xml` los valores que contienen las etiquetas `groupId` y `artifactId` se pueden renombrar para describir el propósito del proyecto.<br>

## Archivo aplication.properties
Este archivo es muy, muy importante porque contiene la configuración para conectarse a la BD, el dialecto que usará el motor de BD, entre otras.

- **spring.cloud.gcp.sql.database-name** contiene el nombre de la BD en Cloud SQL
- **spring.cloud.gcp.sql.instance-connection-name** es el nombre de la instancia de conexión a la BD en Cloud SQL
- **spring.cloud.gcp.project-id** es el ID del proyecto en Google
- **spring.datasource.username** es el usuario que tiene acceso a la BD
- **spring.datasource.password** es la contraseña que da acceso a la BD
- **spring.jpa.show-sql** muestra los queries que se ejecutan en las peticiones
- **maven.test.skip** evita que se "brinquen" los tests
- **spring.cloud.gcp.credentials.location** especifica la ruta donde se encuentra la cuenta de servicio con rol de SQL Client para conectarse a la BD, ejemplo: `classpath:keys-gcp.json`
- **spring.jpa.hibernate.ddl-auto** la instruccion que se ejecuta al inicializar el backend, puede ser *none*, *validate*, *update*, *create*, *create-drop*. Paraconocer su uso véase la [documentación](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html#howto-database-initialization). Esto ayuda a mapear la BD
- **spring.datasource.initialization-mode** y **spring.datasource.continue-on-error**: estas propiedades se explican con detalle en la [documentación](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html#howto-initialize-a-database-using-spring-jdbc)
- **spring.jpa.database-platform** es el dialecto del motor de BD
- **spring.jpa.hibernate.naming.physical-strategy** es la notación con la que se nombrarán las tablas y campos de la BD, es muy util en caso de ocupar `nativeQueries`. Véase la documentación [acá](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-data-access.html) y [aquí](https://docs.jboss.org/hibernate/orm/5.3/userguide/html_single/Hibernate_User_Guide.html#naming) para conocer cuál hace referencia a PascalCase, lowerCamelCase, snake_case; incluso puede personalizar su notación.
- **spring.datasource.sqlScriptEncoding** ayuda a que los acentos se muestren correctamente al exportar el archivo data.sql

# Swagger 2
Swagger 2 es una herramienta para documentar los servicios de cada API. Muestra una lista detallada con las
operaciones permitidas, sus direcciones, parámetros que espera recibir y la respuesta de cada servicio. También permite realizar pruebas a cada operación antes de hacer el despliegue.<br>Se recomienda su uso para describir los métodos y especificar sus rutas de acceso.

## Documente el backend con Swagger 2
Aunque es una herramienta que nos ayuda para conocer los métodos, inputs y outputs del backend, al realizar el despliegue en la nube incluyendo esta herramienta puede significar un tema de alto riesgo para la seguridad de nuestra aplicación.<br>

## Incluyendo Swagger al proyecto
Debe agregar estas dependencias a su proyecto:
* springfox - swagger2 vr. 2.9.2
* springfox - swagger-ui vr. 2.7.0

Colóquelas en el pom.xml:
```xml
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.9.2</version>
</dependency>

<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.7.0</version>
</dependency>
```
Después en la carpeta `config`, ubicada en `src/main/java/com/example/demo`, se creó una nueva clase de nombre SwaggerConfig:
```java
package com.example.demo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
    }
}
```
En la clase `DemoApplication.java`, se muestran propiedades del proyecto como el nombre de la conexión
de la instancia de BD, ID del proyecto, la versión, el ambiente actual y la URL que redirige a la documentación en Swagger del backend. Esta pantalla default será visible si la ejecución del backend es correcta.<br>
Es necesaria la anotación `@RestController` en esta clase para que se muestre el mensaje default al abrir el backend y no un 404, not found.
```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

	@Value("${spring.cloud.gcp.sql.instance-connection-name}")
	private String instanciaSql;

	@Value("${spring.cloud.gcp.project-id}")
	private String projectId;

	@Value("${app.version}")
	private String version;

	@Value("${app.environment}")
	private String environment;

	private String swaggerLink = "<a href='/swagger-ui.html'> /swagger-ui.html <a>";

	@RequestMapping("/")
	private String getInfo(){

		return "Ambiente: " + environment + "<br>" + "Versión: " + version
				+ "<br>Conexión Instancia SQL:  " + instanciaSql + "<br>Project ID: " + projectId + "<br>"
				+ " <b>Documentación backend: </b>" + swaggerLink;
	}
}
```
Bastará navegar a `localhost:8080` (puede cambiar el puerto) ó hacia la URL del despliegue en la nube para ver la página de bienvenida al backend.
Haga click en el link a swagger para ir a la lista de servicios.

## Describiendo los métodos del backend
Para describir un API, agregue la notación @Api en su RestController para que se muestre una descripción breve de la función del método.