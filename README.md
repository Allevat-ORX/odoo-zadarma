# Odoo SMS Modules - Zadarma & LabsMobile

Módulos de Odoo 18 para envío de SMS a través de proveedores alternativos Zadarma y LabsMobile.

[![Odoo Version](https://img.shields.io/badge/Odoo-18.0-blue)](https://www.odoo.com/)
[![License](https://img.shields.io/badge/License-LGPL--3.0-green)](https://www.gnu.org/licenses/lgpl-3.0.html)
[![Python](https://img.shields.io/badge/Python-3.12-blue)](https://www.python.org/)

## 📋 Descripción

Este repositorio contiene dos módulos de Odoo 18 que extienden la funcionalidad de SMS mediante proveedores alternativos:

- **sms_zadarma_v2**: Integración con Zadarma VoIP para envío de SMS
- **sms_labsmobile**: Integración con LabsMobile para envío de SMS

Ambos módulos funcionan en conjunto con `iap_alternative_provider` (OCA) para permitir el uso de múltiples proveedores SMS de manera flexible y económica.

## ✨ Características

- ✅ Compatible con Odoo 18.0
- ✅ Soporte multi-proveedor (Zadarma + LabsMobile)
- ✅ Integración con framework IAP de OCA
- ✅ Configuración de credenciales por cuenta
- ✅ Selección automática de proveedor por ID más bajo
- ✅ Autenticación HMAC para Zadarma
- ✅ HTTP Basic Auth para LabsMobile
- ✅ Logs detallados de envío
- ✅ Manejo de errores robusto

## 📦 Dependencias

### Módulo OCA Requerido
```bash
# iap_alternative_provider (OCA)
https://github.com/OCA/server-tools
```

### Módulos de Este Repositorio
- `sms_zadarma_v2`: Proveedor Zadarma
- `sms_labsmobile`: Proveedor LabsMobile

## 🚀 Instalación

### 1. Instalar iap_alternative_provider (OCA)

```bash
cd /path/to/odoo/addons
git clone https://github.com/OCA/server-tools.git oca_server_tools
```

Habilitar el módulo `iap_alternative_provider` desde la interfaz de Odoo:
- Apps → Update Apps List
- Buscar "IAP Alternative Provider"
- Instalar

### 2. Instalar Módulos SMS

```bash
cd /path/to/odoo/addons
git clone https://github.com/aleixrvr/odoo-zadarma.git
```

Copiar los módulos a la carpeta de addons:
```bash
cp -r odoo-zadarma/sms_zadarma_v2 /path/to/odoo/addons/
cp -r odoo-zadarma/sms_labsmobile /path/to/odoo/addons/
```

Ajustar permisos:
```bash
sudo chown -R odoo:odoo /path/to/odoo/addons/sms_zadarma_v2
sudo chown -R odoo:odoo /path/to/odoo/addons/sms_labsmobile
```

Reiniciar Odoo:
```bash
sudo systemctl restart odoo
# o para Bitnami:
sudo /opt/bitnami/ctlscript.sh restart odoo
```

### 3. Activar Módulos

Desde la interfaz de Odoo:
1. Apps → Update Apps List
2. Buscar "Zadarma SMS" e instalar
3. Buscar "LabsMobile SMS" e instalar

## ⚙️ Configuración

### Configurar Credenciales Zadarma

1. Ir a **Settings → Technical → IAP → Accounts**
2. Crear nueva cuenta IAP:
   - Provider: `sms_api_zadarma`
   - Zadarma User Key: `tu_user_key`
   - Zadarma Secret Key: `tu_secret_key`
   - Zadarma Base URL: `https://api.zadarma.com`

### Configurar Credenciales LabsMobile

1. Ir a **Settings → Technical → IAP → Accounts**
2. Crear nueva cuenta IAP:
   - Provider: `sms_api_labsmobile`
   - LabsMobile Username: `tu_username`
   - LabsMobile Token: `tu_token`
   - LabsMobile Base URL: `https://api.labsmobile.com`

### Ejemplo de Credenciales

Ver archivo `CREDENCIALES-EJEMPLO.md` para un ejemplo de configuración.

## 📱 Uso

### Envío de SMS desde Odoo

Una vez configurado, puedes enviar SMS de dos formas:

#### 1. Desde el modelo de contacto
```python
partner = self.env['res.partner'].browse(partner_id)
partner.mobile = '+524424751707'
partner._message_sms('Tu mensaje aquí')
```

#### 2. Directamente con sms.sms
```python
sms = self.env['sms.sms'].create({
    'number': '+524424751707',
    'body': 'Tu mensaje aquí'
})
sms.send()
```

### Selección de Proveedor

El sistema selecciona automáticamente el proveedor con **ID más bajo** en la tabla `iap.account`:
- Si LabsMobile tiene ID=4 y Zadarma ID=5 → Usa LabsMobile
- Si Zadarma tiene ID=4 y LabsMobile ID=5 → Usa Zadarma

Para cambiar el proveedor predeterminado, ajusta los IDs en la base de datos o elimina/desactiva la cuenta que no deseas usar.

## 🔧 Desarrollo

### Estructura del Proyecto

```
odoo-zadarma/
├── sms_zadarma_v2/
│   ├── __init__.py
│   ├── __manifest__.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── iap_account.py
│   │   └── sms_sms.py
│   └── views/
│       └── iap_account.xml
├── sms_labsmobile/
│   ├── __init__.py
│   ├── __manifest__.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── iap_account.py
│   │   └── sms_sms.py
│   └── views/
│       └── iap_account.xml
├── README.md
└── CREDENCIALES-EJEMPLO.md
```

### Cambios de Odoo 17 a Odoo 18

**Sintaxis XML deprecated:**
```xml
<!-- ❌ Odoo 17 (Ya no funciona) -->
<field name="zadarma_user_key" attrs="{'invisible': [('provider', '!=', 'sms_api_zadarma')]}"/>

<!-- ✅ Odoo 18 (Correcto) -->
<field name="zadarma_user_key" invisible="provider != 'sms_api_zadarma'"/>
```

## 🧪 Testing

### Prueba Manual via XML-RPC

```python
import xmlrpc.client

url = "http://tu-servidor:8069"
db = "tu_base_datos"
username = "admin@ejemplo.com"
password = "tu_password"

common = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/common')
uid = common.authenticate(db, username, password, {})
models = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/object')

# Crear SMS
sms_id = models.execute_kw(db, uid, password,
    'sms.sms', 'create',
    [{
        'number': '+524424751707',
        'body': 'Mensaje de prueba'
    }])

# Enviar SMS
models.execute_kw(db, uid, password,
    'sms.sms', 'send',
    [[sms_id]])

# Verificar estado
sms_state = models.execute_kw(db, uid, password,
    'sms.sms', 'read',
    [[sms_id]], {'fields': ['state', 'number']})

print(f"Estado: {sms_state[0]['state']}")
```

## 🐛 Troubleshooting

### Error 401 Unauthorized (Zadarma)
**Causa:** Credenciales incorrectas o firma HMAC inválida
**Solución:** Verificar User Key y Secret Key en la configuración IAP

### Error 402 Payment Required (LabsMobile)
**Causa:** Saldo insuficiente en cuenta LabsMobile
**Solución:** Recargar saldo en https://www.labsmobile.com

### Error "Invalid field 'iap_account_id'"
**Causa:** Versión incorrecta del módulo o dependencias faltantes
**Solución:** Actualizar a última versión y verificar iap_alternative_provider instalado

### SMS no se envía (queda en estado 'outgoing')
**Causa:** Problema con credenciales o cron de envío de SMS no ejecutándose
**Solución:**
1. Verificar credenciales
2. Ejecutar manualmente: Settings → Technical → Scheduled Actions → "Send SMS"

## 📄 Licencia

Este proyecto está licenciado bajo LGPL-3.0 - ver archivo LICENSE para detalles.

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor:

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📞 Soporte

Para soporte y consultas:
- Issues: https://github.com/aleixrvr/odoo-zadarma/issues
- Email: aleix@onrentx.com

## 🔗 Enlaces Útiles

- [Documentación Zadarma API](https://zadarma.com/en/support/api/)
- [Documentación LabsMobile API](https://apidocs.labsmobile.com/)
- [OCA Server Tools](https://github.com/OCA/server-tools)
- [Odoo Documentation](https://www.odoo.com/documentation/18.0/)

## 📝 Changelog

### Version 18.0.1.0.0 (2025-11-06)
- ✅ Migración completa a Odoo 18
- ✅ Actualización sintaxis XML (attrs → invisible)
- ✅ Testing completo en producción OnRentX
- ✅ Documentación actualizada
- ✅ Soporte para LabsMobile y Zadarma verificado

---

**Made with ❤️ by OnRentX Team**
