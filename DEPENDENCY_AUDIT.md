# Auditoría inicial de dependencias — `school-club-management` (BJXLS)

Este documento recoge un análisis inicial de las dependencias usadas por el proyecto `BJXLS/school-club-management` (extraído de `pom.xml`) y recomendaciones de seguridad/actualización.

## 1) Resumen de dependencias encontradas (extracto)
- org.springframework.boot:spring-boot-devtools (runtime)
- org.springframework.boot:spring-boot-starter-test (test)
- org.springframework.boot:spring-boot-starter-web
- org.springframework.boot:spring-boot-starter-aop
- mysql:mysql-connector-java (runtime)
- com.alibaba:druid-spring-boot-starter:1.1.9
- com.baomidou:mybatis-plus-boot-starter:3.3.1
- com.alibaba:fastjson:1.2.47
- cn.hutool:hutool-core:5.1.2
- cn.hutool:hutool-poi:5.1.2
- org.apache.poi:poi-ooxml:4.1.1
- eu.bitwalker:UserAgentUtils:1.21
- org.apache.tika:tika-core:1.20
- com.artofsolving:jodconverter:2.2.1
- com.github.livesense:jodconverter-core:1.0.5
- org.springframework.boot:spring-boot-starter-mail
- org.apache.shiro:shiro-spring:1.3.0
- org.apache.shiro:shiro-ehcache:1.4.0
- com.ibeetl:beetl:3.0.16.RELEASE
- com.github.whvcse:easy-captcha:1.6.2

> Nota: también se utiliza `spring-boot-maven-plugin` en la sección `build`.

## 2) Observaciones de seguridad y compatibilidad
- fastjson (com.alibaba:fastjson:1.2.47):  
  - fastjson es una librería que históricamente ha tenido vulnerabilidades graves (ej. RCE) en versiones antiguas. Se recomienda encarecidamente revisar su uso y considerar dos opciones:
    1. Actualizar a una versión parcheada que no tenga vulnerabilidades conocidas (consultar la última versión estable en el repositorio oficial y revisar changelogs/security advisories).  
    2. Sustituir su uso por una librería más mantenida y con mejor historial de seguridad (por ejemplo Jackson) si la migración es factible.
  - Acción inmediata recomendada: ejecutar un escaneo de dependencias (OWASP Dependency-Check, Snyk, GitHub Dependabot) para identificar CVEs concretos y versiones afectadas.

- Spring Boot / Java: el `pom.xml` usa Spring Boot 2.2.2.RELEASE y Java 1.8.  
  - Estas versiones son funcionales pero antiguas; planear una actualización a Java 11+ y Spring Boot 2.6+ (o 3.x con JDK 17) mejorará soporte y parches de seguridad.  
  - Esta actualización puede requerir revisar compatibilidades de MyBatis-Plus, Druid, Shiro y otras dependencias.

- fastjson vs fastjson2: si se opta por seguir con la familia de fastjson, validar si fastjson2 (o versiones recientes) cubren las necesidades y mitigaciones; muchas organizaciones prefieren Jackson para interoperabilidad y seguridad.

- Otras librerías a revisar:
  - druid-spring-boot-starter (monitorización/conexión de DB): revisar configuración para evitar exposición de consola de administración en producción.
  - shiro (shiro-spring + shiro-ehcache): confirmar configuración de sesión, tiempo de expiración y uso de cache para evitar fugas.
  - jodconverter + LibreOffice: requiere dependencia del sistema (LibreOffice) — revisar permisos y sandboxing.

## 3) Plan de acción (pasos concretos)
1. Ejecutar un escaneo automatizado:
   - Añadir y ejecutar OWASP Dependency-Check o usar GitHub Dependabot alerts / Snyk para obtener lista de CVEs y versiones vulnerables.
2. Mitigación rápida:
   - Para cualquier dependencia con CVE crítico (p.ej. RCE), actualizar a la versión parcheada o aplicar un WAF / reglas temporales.
3. Sustitución estratégica:
   - Evaluar migración de `fastjson` a `Jackson` (o actualizar a versión segura) y probar serialización/deserialización de payloads usados en la app.
4. Pruebas:
   - Ejecutar pruebas integradas después de actualizar dependencias (especialmente JSON, mybatis, shiro).
5. Hardening de configuración:
   - Deshabilitar consolas no necesarias (e.g., consola Druid), revisar `application.properties` para valores inseguros.

## 4) Checklist inmediato (qué entregaré en el PR)
- [x] Fichero `DEPENDENCY_AUDIT.md` con este análisis inicial (este archivo).  
- [ ] Resultado del escaneo OWASP Dependency-Check / Dependabot (añadir informe como artefacto en PR).  
- [ ] Propuesta de versiones objetivo para actualizar (cuando el escaneo devuelva CVEs concretos).

## 5) Notas y limitaciones
- Este análisis es inicial y se basa en el `pom.xml` publicado; un escaneo automatizado proporcionará evidencia de CVEs específicas y versiones afectadas.  
- No hemos modificado código aún: este PR añade solo el informe inicial y la propuesta de seguimiento.

---

Si te parece bien, en la siguiente iteración puedo:
- Ejecutar OWASP Dependency-Check en el repositorio y adjuntar el reporte.  
- Proponer versiones seguras concretas para `fastjson` y otras dependencias críticas.  
- Implementar la actualización de `fastjson` a una versión parcheada (o sustituir por Jackson) en una rama aparte y ejecutar pruebas básicas.

(Autor: auditoría inicial generada por asistente; revisa y ajusta según tus políticas.)
