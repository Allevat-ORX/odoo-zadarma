# Ejemplo de Configuración de Credenciales

⚠️ **IMPORTANTE**: Este archivo contiene **EJEMPLOS** únicamente. **NO** uses estas credenciales en producción.

## 🔑 Zadarma

### Obtener Credenciales
1. Accede a tu cuenta Zadarma: https://my.zadarma.com/
2. Ve a Settings → API
3. Genera tu User Key y Secret Key

### Configuración en Odoo
```
Provider: sms_api_zadarma
User Key: 1a2b3c4d5e6f7g8h9i0j
Secret Key: k1l2m3n4o5p6q7r8s9t0
Base URL: https://api.zadarma.com
```

### Ejemplo Python para verificar conexión
```python
import requests
import hmac
import hashlib
import time

user_key = "TU_USER_KEY_AQUI"
secret_key = "TU_SECRET_KEY_AQUI"
method = "/v1/info/balance/"

# Generar firma HMAC
signature = hmac.new(
    secret_key.encode('utf-8'),
    method.encode('utf-8'),
    hashlib.sha1
).hexdigest()

url = f"https://api.zadarma.com{method}"
headers = {'Authorization': f'{user_key}:{signature}'}

response = requests.get(url, headers=headers)
print(response.json())
```

## 🔑 LabsMobile

### Obtener Credenciales
1. Accede a tu cuenta LabsMobile: https://www.labsmobile.com/
2. Ve a My Account → API Settings
3. Obtén tu Username y Token

### Configuración en Odoo
```
Provider: sms_api_labsmobile
Username: tu-usuario@ejemplo.com
Token: ABCDEFGHIJKLMNOPQRSTUVWXYZ123456
Base URL: https://api.labsmobile.com
```

### Ejemplo Python para verificar conexión
```python
import requests
import base64

username = "TU_USERNAME_AQUI"
token = "TU_TOKEN_AQUI"

# Crear header de autenticación Basic
credentials = f"{username}:{token}"
auth_header = base64.b64encode(credentials.encode()).decode()

url = "https://api.labsmobile.com/json/balance"
headers = {'Authorization': f'Basic {auth_header}'}

response = requests.get(url, headers=headers)
print(response.json())
```

## 📋 Verificación de Credenciales en Odoo

### Método 1: Interfaz Web
1. Settings → Technical → IAP → Accounts
2. Verificar que las cuentas estén creadas correctamente
3. Campos no deben estar vacíos

### Método 2: XML-RPC (Testing)
```python
import xmlrpc.client

url = "http://tu-servidor:8069"
db = "tu_base_datos"
username = "admin@ejemplo.com"
password = "tu_password"

common = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/common')
uid = common.authenticate(db, username, password, {})
models = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/object')

# Ver cuentas IAP SMS
accounts = models.execute_kw(db, uid, password,
    'iap.account', 'search_read',
    [[['provider', 'like', 'sms_api']]],
    {'fields': ['id', 'provider']})

print(accounts)
```

## 🔐 Seguridad

### Buenas Prácticas
- ✅ Nunca commits credenciales reales a Git
- ✅ Usa variables de entorno en scripts
- ✅ Rota credenciales cada 6 meses
- ✅ Restringe acceso a cuentas IAP en Odoo
- ✅ Monitorea uso y logs de envío SMS

### Almacenamiento Seguro
```bash
# Usar variables de entorno
export ZADARMA_USER_KEY="tu_user_key"
export ZADARMA_SECRET_KEY="tu_secret_key"
export LABS_USERNAME="tu_username"
export LABS_TOKEN="tu_token"

# Acceder desde script
python script.py
```

### .env File (NO COMMITEAR)
```env
# .env - NO SUBIR A GITHUB
ZADARMA_USER_KEY=tu_user_key_real
ZADARMA_SECRET_KEY=tu_secret_key_real
LABSMOBILE_USERNAME=tu_username_real
LABSMOBILE_TOKEN=tu_token_real
```

## 📞 Costos Estimados

### Zadarma
- SMS nacional (México): ~$0.05 USD/SMS
- SMS internacional: Variable según país
- Sin cargos mensuales fijos

### LabsMobile
- SMS nacional (México): ~$0.03 USD/SMS
- SMS internacional: Variable según país
- Paquetes disponibles con descuento

**Recomendación**: LabsMobile suele ser más económico para México.

## 🧪 Testing

### Prueba de Envío
```python
# Reemplaza con tus credenciales de PRUEBA
phone = '+525512345678'  # Usa TU número para testing
message = 'SMS de prueba - Odoo Module Testing'

sms = env['sms.sms'].create({
    'number': phone,
    'body': message
})
sms.send()
```

### Verificación de Saldo

**Zadarma:**
```bash
curl -H "Authorization: USER_KEY:SIGNATURE" \
  https://api.zadarma.com/v1/info/balance/
```

**LabsMobile:**
```bash
curl -u "USERNAME:TOKEN" \
  https://api.labsmobile.com/json/balance
```

---

## ⚠️ Recordatorio

**Este archivo es SOLO para referencia. Las credenciales reales deben:**
- Estar en archivos locales protegidos (`.env`, configuración Odoo)
- Nunca commitear a repositorios públicos
- Guardarse en gestores de contraseñas seguros
- Rotarse periódicamente

---

Para más información:
- [Documentación Zadarma](https://zadarma.com/en/support/api/)
- [Documentación LabsMobile](https://apidocs.labsmobile.com/)
