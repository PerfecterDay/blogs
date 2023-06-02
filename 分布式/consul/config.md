<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-config</artifactId>
    <version>3.1.2</version>
</dependency>


spring:
  config:
    import: consul
  application:
    name: user-center-service
  cloud:
    consul:
      host: localhost
      port: 8500
      config:
        enabled: true
        format: yaml
        prefix: user-center
        default-context: default