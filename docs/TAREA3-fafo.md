# Tarea 3 — Despliegue automatizado de LAMP y Samba

## 1. Objetivo

Automatizar la instalación y configuración de servicios LAMP y Samba mediante Ansible en máquinas virtuales openSUSE Leap 16.0.

## 2. Arquitectura

| Máquina | Dirección IP | Función |
|---|---|---|
| MASTER | 192.168.6.20 | Controlador Ansible y repositorio Git |
| DESTINO | 192.168.6.21 | Servidor administrado con Ansible |
| Windows 11 | 192.168.6.1 | Cliente de pruebas HTTP y Samba |

El controlador administra el servidor Destino mediante SSH.

## 3. Estructura del proyecto

- `site.yml`: playbook principal.
- `hosts`: inventario del laboratorio.
- `roles/common`: paquetes básicos y firewall.
- `roles/lamp`: Apache, PHP, phpMyAdmin y MariaDB.
- `roles/samba`: usuario, directorio compartido y servidor Samba.
- `group_vars/all/vars.yml`: variables de configuración.
- `group_vars/all/vault.yml`: contraseñas cifradas, excluidas de Git.

## 4. Requisitos

- openSUSE Leap 16.0 en las máquinas Linux.
- Ansible instalado en el MASTER.
- Conectividad SSH entre MASTER y DESTINO.
- Clave SSH pública autorizada en el DESTINO.
- Colecciones de Ansible requeridas por los roles.
- Archivo Vault local con las contraseñas necesarias.

El inventario y las rutas de claves SSH deben adaptarse a cada entorno.

## 5. Configuración de contraseñas

Las credenciales se administran mediante Ansible Vault.

Variables secretas utilizadas:

- `vault_become_password`
- `vault_samba_user_password`
- `vault_mariadb_root_password`

Crear el archivo cifrado local:

`ansible-vault create --ask-vault-pass group_vars/all/vault.yml`

El archivo debe contener las variables secretas correspondientes. No publicar contraseñas, claves privadas ni archivos descifrados.

## 6. Ejecución

Validar la sintaxis:

`ansible-playbook -i hosts site.yml --syntax-check --ask-vault-pass`

Desplegar todo:

`ansible-playbook -i hosts site.yml --ask-vault-pass`

Ejecutar únicamente LAMP:

`ansible-playbook -i hosts site.yml --tags lamp --ask-vault-pass`

Ejecutar únicamente Samba:

`ansible-playbook -i hosts site.yml --tags samba --ask-vault-pass`

## 7. Servicios configurados

**LAMP**

- Apache como servidor HTTP.
- PHP 8.
- phpMyAdmin.
- MariaDB con autenticación de root.
- Página de comprobación `/info.php`.

**Samba**

- Usuario: `sambauser`.
- Recurso: `compartida`.
- Directorio: `/srv/compartida`.
- Autenticación mediante credenciales Samba.
- Conservación del recurso Samba existente `ASO`.

## 8. Seguridad

- Autenticación SSH mediante clave privada en Ansible.
- Contraseñas protegidas con Ansible Vault.
- Archivos sensibles excluidos del seguimiento actual de Git.
- MariaDB escuchando únicamente en `127.0.0.1:3306`.
- Reglas de firewall para HTTP desde `192.168.6.0/24`.
- Reglas de Samba para `192.168.6.0/24` y `192.168.1.0/24`.
- Conservación de SSH y las reglas anteriores de FTP.

El laboratorio utiliza HTTP sin cifrado; para un entorno de producción sería necesario configurar HTTPS y restringir adicionalmente el acceso administrativo.

## 9. Pruebas realizadas

- Acceso desde Windows al servidor Apache.
- Ejecución correcta de la página PHP.
- Acceso a la interfaz de phpMyAdmin.
- Autenticación de MariaDB 11.8.8.
- Conexión SMB desde Windows.
- Transferencia de un archivo de Windows a `/srv/compartida`.
- Conservación del recurso Samba anterior.
- Comprobación de las reglas permanentes del firewall.
- Validación de sintaxis de Ansible.
- Ejecuciones repetidas para verificar idempotencia.

## 10. Resultado final

La última ejecución completa obtuvo:

| Métrica | Resultado |
|---|---|
| Tareas correctas | 18 |
| Cambios | 0 |
| Errores | 0 |
| Hosts inaccesibles | 0 |
| Tareas omitidas | 5 |
| Tiempo real | 46,03 segundos |

El resultado `changed=0` confirma la idempotencia del despliegue en el servidor ya configurado.

## 11. Control de versiones

Repositorio personal: `https://github.com/fafoquipe/Ansible-scripts`

Rama de desarrollo: `fafo/tarea3-opensuse16`

El trabajo se mantiene separado de la rama principal.
