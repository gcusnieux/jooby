This module exists for strict environments where the ONLY option is to deploy into a Servlet Container.
It's strongly recommended to avoid this and rather opt for using one of {{netty}}, {{jetty}} or {{undertow}}.

## usage

In order to deploy into a Servlet Container, you need to generate a ```*.war``` file. Here's how:

* Create a ```war.activator``` file in the ```src/etc``` directory.

* Open a console and type: ```mvn clean package```

* Find the ```*.war``` file in the ```target``` directory.

## limitations

* web-sockets are not supported
* some properties has no effect when deploying into a Servlet Container:
  - application.path / contextPath
  - appplication.port
  - max upload file sizes
  - any other server specific property: server.*, jetty.*, netty.*, undertow.*


### special note on contextPath

To avoid potential headaches, make sure to use the ```contextPath``` variable while loading static/dynamic
resources (.css, .js, etc..).

For example:

```html
<html>
<head>
  <link rel="stylesheet" text="text/css" href="{{contextPath}}/css/styles.css">
  <script src="{{contextPath}}/js/app.js"></script>
</head>
</html>
```

The expression: ```{{contextPath}}``` corresponds to the template engine (handlebars in that case) or ```${contextPath}``` for Freemarker.

## how does it work?

The presence of the ```src/etc/war.activator``` file triggers a Maven profile. The contents of the file doesn't matter, it just needs to be present.

The Maven profile builds the ```*.war``` file using the ```maven-assembly-plugin```. The assembly descriptor can be found
[here](https://github.com/jooby-project/jooby/blob/master/jooby-dist/src/main/resources/assemblies/jooby.war.xml)

### web.xml

A default ```web.xml``` file is generated by the assembly plugin. This file looks like:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
  version="3.1">
  <context-param>
    <param-name>application.class</param-name>
    <param-value>${application.class}</param-value>
  </context-param>

  <listener>
    <listener-class>org.jooby.servlet.ServerInitializer</listener-class>
  </listener>

  <servlet>
    <servlet-name>jooby</servlet-name>
    <servlet-class>org.jooby.servlet.ServletHandler</servlet-class>
    <load-on-startup>0</load-on-startup>
    <!-- MultiPart setup -->
    <multipart-config>
      <file-size-threshold>0</file-size-threshold>
      <!-- Default 200k -->
      <max-request-size>${war.maxRequestSize}</max-request-size>
    </multipart-config>
  </servlet>

  <servlet-mapping>
    <servlet-name>jooby</servlet-name>
    <url-pattern>/*</url-pattern>
  </servlet-mapping>
</web-app>
```

#### max upload size

The default max upload size is set to ```204800b``` (200kb). If you need to increase this, add
the ```war.maxRequestSize``` property to ```pom.xml```:

```xml
<properties>
  <war.maxRequestSize>1048576</war.maxRequestSize> <!-- 1mb -->
</properties>
```

#### custom web.xml

When the generated file isn't enough, follow these steps:

1. create a dir: ```src/etc/war/WEB-INF```
2. save a ```web.xml``` file inside that dir
3. run: ```mvn clean package```

{{appendix}}