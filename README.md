# Sprint 3 - Evidencias de Explotación

Repositorio de evidencias del **Sprint 3** para el curso
**"Anti Hacking y Nuevas Tendencias de Seguridad" (UPC)**

---

## 1. Objetivo del Sprint

Documentar la fase de **explotación** sobre los entornos configurados:

* Kali Linux (atacante)
* Metasploitable 2 (víctima)
* Aplicaciones vulnerables: DVWA, WordPress Triphasik (cliente real)

Se registran:

* Comandos ejecutados
* Capturas de pantalla que evidencian:

  * Configuración de red
  * Escaneo de servicios/puertos
  * Explotación de vulnerabilidades
  * Acceso obtenido

---

## 2. Estructura de Carpetas

```text
evidencias/
├── FASE A — EXPLOTACIÓN EN METASPLOITABLE2 (PoC CRÍTICA).txt
├── kali_ifconfig.txt
├── kali_ip addr.txt
├── metasploitable_ip addr.txt
├── metasploitable_ipconfig.txt
├── dvwa_login
├── triphasik_app_login
├── triphasik_wp_login
└── Screenshot/
    ├── Captura de pantalla 2025-12-05 014610.png
    ├── Captura de pantalla 2025-12-05 015147.png
    ├── ...
```

---

## 3. Preparación General

En Kali:

```bash
mkdir -p ~/evidencias_sprint3
```

---

## FASE C – TRIPHASIK (PRUEBAS CONTROLADAS)

### Endpoints evaluados:

* `https://app.triphasikperformance.com/login`
* `https://triphasikperformance.com/wp-login.php`

> Triphasik Performance es una plataforma real. Las pruebas se ejecutaron con autorización académica y bajo alcance controlado.

Objetivos:

* Comprender el flujo de login.
* Evaluar comportamiento ante credenciales incorrectas.
* Detectar mecanismos de defensa (rate limiting, CAPTCHA).
* Generar evidencia técnica para el informe.

---

## C1. Explotación de `app.triphasikperformance.com/login`

### C1.1. Reconocimiento

Acceso vía navegador (con proxy Burp activo):

```text
https://app.triphasikperformance.com/login
```

Se identificó el formulario con campos `username` y `password`.

 `S3-C-App-01_login_page.png`

---

### C1.2. Captura del Request

Con Burp Intercept + Repeater se analizó el `POST` hacia `/login`.

Parámetros observados:

```text
username=<usuario>&password=<clave>
```

`triphasik_app_login.txt`
 `S3-C-App-02_request_intercept.png`
 `S3-C-App-03_repeater_request_response.png`

---

### C1.3. PoC de Autenticación Controlada

Se simularon varios intentos con credenciales inválidas:

* Manualmente en Repeater.
* Observando códigos HTTP y mensajes.

También se aplicó:

* Pequeño diccionario controlado.
* Observación del comportamiento del servidor:

  * ¿Hay bloqueo?
  * ¿Captchas?
  * ¿Rate limiting?

📸 `S3-C-App-04_intentos_login_invalidos.png`
📝 `S3-C-App-05_resumen_explotacion_app.txt`

---

## C2. Explotación de `triphasikperformance.com/wp-login.php` (WordPress)

### C2.1. Reconocimiento

Ingreso al panel de login WordPress:

```text
https://triphasikperformance.com/wp-login.php
```

Campos observados:

* `log` (usuario)
* `pwd` (contraseña)
* `wp-submit`, `redirect_to`, `testcookie`

📸 `S3-C-WP-01_wp_login_page.png`

---

### C2.2. Captura del Request POST

Con Burp se interceptó y reenvió la siguiente estructura:

```http
log=Triphasik&pwd=contrasena&wp-submit=Acceder&testcookie=1
```

📄 `triphasik_wp_login.txt`
📸 `S3-C-WP-02_wp_request_intercept.png`
📸 `S3-C-WP-03_wp_repeater_request_response.png`

---

### C2.3. PoC de Explotación Controlada

Pruebas con Hydra:

```bash
hydra -l Triphasik -P Triphasik.txt triphasikperformance.com https-post-form \
"/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Acceder&testcookie=1:S=Location" \
-f -t 1 -V -I
```

Comportamiento:

* Inicialmente se devolvió `200` o `302`.
* Luego apareció **CAPTCHA** matemático.
* Finalmente se mostró un **mensaje de bloqueo** por actividad sospechosa.

Evidencia en Burp:

* Request/response visible con mensaje de restricción.
* Al intentar login manual → desconexión del sitio (posible cuarentena temporal).

📸 `S3-C-WP-04_wp_multiples_intentos.png`
📝 `S3-C-WP-05_resumen_explotacion_wp.txt`

---

## C3. Conclusión Fase Triphasik

* No se comprometieron cuentas.
* El sitio aplicó defensas progresivas:

  * CAPTCHA
  * Bloqueo temporal
  * Probable cuarentena de IP

---

## C4. Recomendaciones

* Activar 2FA para usuarios admin.
* Limitar intentos por IP y cuenta.
* Registrar eventos de login fallido.
* Monitorear accesos sospechosos en logs.

---

## C5. Capturas Finales

### Captura App Login

![Captura App Login](evidencias/Screenshot/Captura%20de%20pantalla%202025-12-05%20033511.png)

---

### Captura WordPress Login

![Captura WP Login](evidencias/Screenshot/Captura%20de%20pantalla%202025-12-05%20034627.png)

---

## Enlaces de Referencia

* [Metasploitable 2 Docs][1]
* [SQLMap avanzado en Medium][2]
* [Burp Proxy Configuración][3]
* [Triphasik Training][4]

[1]: https://docs.rapid7.com/metasploit/metasploitable-2/?utm_source=chatgpt.com
[2]: https://medium.com/%401200km/sqlmap-a-deep-dive-into-automated-sql-injection-testing-part-2-advanced-custom-setup-0136ac6ffe53?utm_source=chatgpt.com
[3]: https://portswigger.net/burp/documentation/desktop/settings/tools/proxy?utm_source=chatgpt.com
[4]: https://triphasictraining.com/?utm_source=chatgpt.com

---
