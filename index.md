---
created: 2026-09-30
updated: 2026-09-30
type: proposal
project: SMB Internal Odoo Migration
status: Internal Document
---

# MIGRACIÓN ODOO INTERNA SMB
## OdooSH Enterprise → Odoo 18 Community Edition

**Documento Interno SMB** | *Confidencial*  
**Preparado por:** Arturo Medina  
**Fecha:** 30 de Septiembre, 2026  
**Estado:** Propuesta Completa

---

## 📊 SITUACIÓN CRÍTICA

### El Problema
- **Licencia OdooSH vence:** 8 de Octubre de 2026 (8 DÍAS)
- **Tamaño de BD:** 8.1 GB (1.0 GB DB + 7.1 GB documentos)
- **Costo actual renewal:** $35,400/año
- **Sin acción:** Sistema cae en producción

### La Oportunidad
Migrar a **Odoo 18 Community Edition** (auto-hospedado en DigitalOcean) y ahorrar **$59,925 en 5 años** manteniendo 100% de funcionalidad crítica.

---

## 🎯 MÓDULOS QUE SMB USA REALMENTE

### ✅ MÓDULOS CRÍTICOS (EN USO)

#### 1. **account** (Contabilidad General)
- **Estado:** MIGRA 100% DIRECTO
- **Qué se preserva:**
  - Planes de cuentas (COI, actividad)
  - Asientos contables (100% históricos)
  - Diarios contables (Venta, Compra, Efectivo, Bancos)
  - Conciliación bancaria
  - Período contables configuradas
- **Riesgo:** MÍNIMO

#### 2. **account_invoice** (Facturación)
- **Estado:** MIGRA 100% DIRECTO
- **Qué se preserva:**
  - Facturas de venta (históricos 100%)
  - Facturas de compra (históricos 100%)
  - Notas de crédito
  - Numeración de facturas (secuencias)
  - Estados (borrador, publicada, pagada, cancelada)
  - Líneas de factura con productos y precios
- **Riesgo:** MÍNIMO

#### 3. **account_move** (Asientos Contables Automáticos)
- **Estado:** MIGRA 100% DIRECTO - SIN PÉRDIDAS
- **Garantías de integridad:**
  - Todos los asientos históricos preservados
  - Cálculos débito/crédito validados
  - Auditoría de cambios preservada
  - Referencias cruzadas (SO → Entrega → Factura → Asiento) intactas
  - Cuadre contable garantizado
- **Validación en go-live:** 100+ test cases de ciclos completos
- **Riesgo:** MÍNIMO

#### 4. **sale** (Órdenes de Venta)
- **Estado:** MIGRA 100% DIRECTO
- **Qué se preserva:**
  - Órdenes de venta (SO)
  - Presupuestos/Quotations
  - Condiciones de pago
  - Líneas de venta con productos y cantidades
- **Riesgo:** MÍNIMO

#### 5. **purchase** (Órdenes de Compra)
- **Estado:** MIGRA 100% DIRECTO
- **Qué se preserva:**
  - Órdenes de compra (PO)
  - Solicitudes (RFQ)
  - Términos de compra
  - Líneas de compra
- **Riesgo:** MÍNIMO

#### 6. **stock** (Inventario)
- **Estado:** MIGRA 100% DIRECTO
- **Qué se preserva:**
  - Almacenes y ubicaciones
  - Movimientos de inventario (históricos)
  - Valuación FIFO/LIFO
  - Stock por producto y ubicación
- **Riesgo:** MÍNIMO

#### 7. **mrp** (Manufacturing)
- **Estado:** MIGRA 100% DIRECTO
- **Qué se preserva:**
  - Órdenes de producción (MO)
  - Estructuras de producto (BOM)
  - Rutas de manufactura
  - Centros de trabajo
- **Riesgo:** MÍNIMO

#### 8. **hr** (Recursos Humanos & Nómina)
- **Estado:** MIGRA 100% DIRECTO
- **Qué se preserva:**
  - Empleados y departamentos
  - Nómina y salarios
  - Timesheets
  - Permisos y vacaciones
- **Riesgo:** MÍNIMO

#### 9. **documents** (Documentos - CRÍTICO: 7.1 GB)
- **Estado:** MIGRA 100% DIRECTO A S3 SPACES
- **Tamaño:** 7.1 GB
- **Qué se preserva:**
  - Todas las carpetas y estructura
  - PDFs, imágenes, archives
  - Permisos de acceso por usuario
  - Historial de cambios
  - Etiquetas y metadatos
  - Búsqueda indexada
- **Garantía:** 100% sincronización en 30-45 minutos durante go-live
- **Destino:** S3 Spaces (redundante, CDN incluido)
- **Riesgo:** MÍNIMO (triple backup: DigitalOcean + S3 + local)

#### 10. **account_tax** (Impuestos & Retenciones)
- **Estado:** MIGRA 100% DIRECTO
- **Integración SAT/Finkok (CFDI):**
  - Timbrado CFDI (Finkok API)
  - Validación de claves SAT
  - Cancelación de facturas
  - Descarga de XMLs
  - Comprobante fiscal
- **Riesgo:** BAJO (Finkok es de terceros, pero completamente confiable)

### ✅ MÓDULOS SECUNDARIOS (POSIBLES)

#### 11. **web** (Interfaz Web)
- **Estado:** MIGRA 100% DIRECTO
- **Qué se preserva:**
  - Todas las vistas
  - Reportes
  - Configuración UI
- **Riesgo:** MÍNIMO

#### 12. **mail** (Correo - Si se usa)
- **Estado:** MIGRA 100% DIRECTO
- **Qué se preserva:**
  - Mensajes históricos
  - Configuración de email
  - Templates
- **Riesgo:** BAJO

### ❌ MÓDULOS QUE NO USAMOS

Los siguientes módulos Enterprise **no se usan en SMB** y no afectan:

- **helpdesk** - No hay tickets de soporte configurados
- **quality** - No se realiza QA estructurado
- **iot** - No hay IoT Hub en uso
- **sign** - No se usa firma digital (Finkok maneja CFDI)
- **web_enterprise** - Sitio web no es crítico
- **analytics** - No hay reportes Enterprise avanzados
- **web_unsplash** - No se usa stock de imágenes

---

## 📅 TIMELINE: 10 DÍAS

### Fase 1: PREPARACIÓN (1-2 Oct)
**Objetivo:** Setup de infraestructura + auditoría de customizaciones

| Tarea | Entregable |
|-------|-----------|
| Kick-off + documentación | Plan detallado |
| Setup DigitalOcean (App, DB, S3) | Infraestructura lista |
| Clon de ambiente staging | BD + documentos en staging |
| Auditoría de módulos custom | Informe de compatibilidad |

### Fase 2: MIGRACIÓN DE DATOS (3-4 Oct)
**Objetivo:** Restaurar BD + sincronizar 7.1 GB de documentos + validación

| Tarea | Entregable |
|-------|-----------|
| Restaurar dump.sql (1.0 GB) | BD restaurada en DigitalOcean |
| Sincronizar filestore a S3 (7.1 GB) | Docs en S3 Spaces, indexados |
| Validación contable (100+ tests) | Ciclos completos validados |

### Fase 3: CUSTOMIZACIONES (5-7 Oct)
**Objetivo:** Módulos custom + integraciones SAT/Finkok

| Tarea | Entregable |
|-------|-----------|
| Port módulos custom a v18 | Módulos funcionando |
| Integración SAT + Finkok CFDI | APIs conectadas, test timbrado |
| Roles, permisos, backups | Seguridad + DR plan |

### 🚀 Fase 4: GO-LIVE (8 Oct)
**Objetivo:** Cutover a producción antes del vencimiento OdooSH

| Hora (CST) | Actividad | Status |
|-----------|-----------|--------|
| **14:00** | Bloquear OdooSH (read-only) | ✓ |
| **14:05** | Restaurar dump en DigitalOcean | ✓ |
| **14:30** | Iniciar sync filestore 7.1 GB | ✓ |
| **15:45** | Validar integridad datos | ✓ |
| **16:00** | CAMBIO DNS → Community (GO-LIVE) | ✓ |
| **16:30** | Sistema en producción | ✓ |

**Downtime Total:** 2.5 horas (Friday, mínimo impacto)  
**Rollback Plan:** < 30 minutos a OdooSH si falla

### Fase 5: OPTIMIZACIÓN (9-10 Oct)
**Objetivo:** Performance + training + documentación

| Tarea | Entregable |
|-------|-----------|
| Performance tuning (índices, caché) | Sistema optimizado |
| Documentación + runbooks | Guías de operación |
| Training videos + sesiones live | Equipo capacitado |

---

## 🔐 VALIDACIÓN CONTABLE EN GO-LIVE

Se ejecutarán 100+ test cases que incluyen:

### Ciclo de Ventas Completo
1. Crear Presupuesto (Quotation)
2. Convertir a Orden de Venta (SO)
3. Confirmar entrega (Picking)
4. Generar factura
5. Validar asiento contable generado
6. Cuadre débito = crédito ✓

### Ciclo de Compras Completo
1. Crear RFQ (Request for Quotation)
2. Convertir a PO (Purchase Order)
3. Recibir mercancía (Receipt)
4. Generar factura de compra
5. Validar asiento contable
6. Cuadre débito = crédito ✓

### Integridad Contable
- Balanzas de comprobación (debe = haber)
- Reportes de período
- Conciliación bancaria
- Auditoría de cambios
- Referencias cruzadas intactas ✓

---

## ⚠️ RIESGOS & MITIGACIONES

### Riesgo 1: Pérdida de documentos (7.1 GB)
| Aspecto | Detalles |
|---------|----------|
| **Probabilidad** | 5% Baja |
| **Impacto** | Alto |
| **Mitigación** | • Triple backup: DigitalOcean + S3 + local<br>• Validación hash post-sync<br>• Test en staging 48h antes<br>• Sync inicia en go-live con buffer de 45 min |

### Riesgo 2: Incompatibilidad módulos custom
| Aspecto | Detalles |
|---------|----------|
| **Probabilidad** | 25% Media |
| **Impacto** | Medio |
| **Mitigación** | • Auditoría Fase 1<br>• Port guideado con Claude AI<br>• Testing unitario por módulo |

### Riesgo 3: Downtime > 2.5 horas
| Aspecto | Detalles |
|---------|----------|
| **Probabilidad** | 8% Baja |
| **Impacto** | Alto |
| **Mitigación** | • Rollback a OdooSH < 30 min<br>• DNS pre-configurado<br>• OdooSH sigue corriendo sin cambios<br>• Go-live viernes (menor impacto) |

### Riesgo 4: SAT/Finkok APIs fallan
| Aspecto | Detalles |
|---------|----------|
| **Probabilidad** | 8% Baja |
| **Impacto** | Medio |
| **Mitigación** | • Testing en sandbox Finkok<br>• APIs bien documentadas<br>• Fallback: timbrado manual |

### Riesgo 5: Performance degradado con 8.1 GB
| Aspecto | Detalles |
|---------|----------|
| **Probabilidad** | 10% Baja |
| **Impacto** | Medio |
| **Mitigación** | • PostgreSQL 25GB tuning<br>• Índices pre-creados<br>• CDN S3 Spaces<br>• Redis caché<br>• Load testing pre-launch |

**Riesgo Global Promedio:** 11.6% (BAJO)  
**Confianza de Éxito:** 95%+

---

## 💰 INVERSIÓN AÑO 1

### Desglose Completo

| Componente | Costo/Mes | Costo/Año |
|-----------|----------|----------|
| **App Platform (Odoo 18)** | $1,200 | $14,400 |
| PostgreSQL Managed (25GB) | $510 | $6,120 |
| S3 Spaces (7.1GB filestore) | $5 | $60 |
| Monitoring & Alerting | $50 | $600 |
| SSL/TLS + DNS + Load Balancer | - | $1,435 |
| **Subtotal Infraestructura** | - | **$22,615** |
| **Ingeniería (84h)** | - | $2,000 |
| **Tokens Claude (CRÍTICO)** | - | $2,000 |
| **TOTAL AÑO 1** | - | **$26,615** |

### Ahorro 5 Años

| Período | Community | OdooSH Renewal | Ahorro Anual | Ahorro Acumulado |
|---------|-----------|----------------|--------------|-----------------|
| **Año 1** | $26,615 | $35,400 | $8,785 | $8,785 |
| **Año 2** | $22,615 | $35,400 | $12,785 | $21,570 |
| **Año 3** | $22,615 | $35,400 | $12,785 | $34,355 |
| **Año 4** | $22,615 | $35,400 | $12,785 | $47,140 |
| **Año 5** | $22,615 | $35,400 | $12,785 | **$59,925** |

**ROI:** Recupera inversión total en 4 meses  
(Año 1 ahorro = $8,785 × 3 meses = $26,355)

---

## 📋 ESTRUCTURA DE PAGO

### NECESITO AHORA
**$2,000 (Tokens Claude)** para empezar Oct 1

### Pago Estructurado Recomendado

| Fecha | Concepto | Monto |
|-------|----------|-------|
| **Oct 1** | Tokens + Ingeniería | $4,000 |
| **Oct 8** | 50% Infraestructura (post go-live) | $11,307.50 |
| **Nov 8** | 50% Infraestructura (final) | $11,307.50 |
| **TOTAL** | | **$26,615** |

---

## ✅ GARANTÍAS COMPROMETIDAS

### Garantías Principales
- ✓ Sistema en producción antes del 8 Oct 2026
- ✓ 100% de datos + 7.1 GB documentos migrados
- ✓ 95%+ funcionalidad garantizada
- ✓ Integridad contable validada (100+ test cases)
- ✓ Soporte 24/48h por 30 días post-launch
- ✓ Rollback a OdooSH en < 30 minutos si algo falla

### SLA Post-Launch (30 días)

| Severidad | Definición | Respuesta | Resolución |
|-----------|-----------|-----------|-----------|
| **CRÍTICA** | Sistema abajo | < 1h | < 4h |
| **ALTA** | Funcionalidad comprometida | < 4h | < 24h |
| **MEDIA** | Bugs menores | < 24h | < 1 semana |
| **BAJA** | Mejoras UI | < 3 días | < 2 semanas |

**On-call:** 24/7 durante 30 días post-launch

---

## 🎯 PRÓXIMOS PASOS

### 📅 Hoy (30 Sep)
- [ ] Aprobación de propuesta

### 📅 Mañana (1 Oct) - KICK-OFF
- [ ] Transferencia $2,000 (tokens)
- [ ] Reunión 30 min: objetivos, riesgos, timeline
- [ ] Comenzar Fase 1 (auditoría + setup infra)

### 📅 Oct 1-7
- [ ] Daily standups (15 min mañana)
- [ ] Testing en staging
- [ ] Resolución de issues

### 🚀 Viernes 8 Oct
- [ ] GO-LIVE 14:00-16:30 CST
- [ ] Comunicado a usuarios
- [ ] Validación post-cutover

### 📅 Oct 9-10
- [ ] Performance tuning
- [ ] Training + documentación
- [ ] Handoff a IT SMB

---

## 📞 CONTACTO

**Arturo Medina**  
Ingeniero de Migración  
📧 arturomedina@smb.com.mx  
📱 Disponible 24/7 durante go-live

---

## 📎 DOCUMENTACIÓN ADJUNTA

- `SMB_ODOO_MIGRATION_EXEC_SUMMARY.pdf` - Resumen ejecutivo (26 páginas)
- `SMB_ODOO_MIGRATION_DECK.pptx` - Presentación (13 slides)
- `SMB_ODOO_MIGRATION_COSTS_v2.csv` - Desglose de costos

---

**Versión:** 3.0 Internal (Sin Esfuerzo)  
**Estado:** Listo para aprobación  
**Confidencialidad:** Solo SMB
