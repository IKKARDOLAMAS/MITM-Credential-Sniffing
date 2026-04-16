# Ataque MITM y Captura de Credenciales (Sinkhole)

Este repositorio documenta el laboratorio del Día 7, enfocado en la interceptación de tráfico en una red local y la demostración de las vulnerabilidades del protocolo **HTTP** frente a ataques de **Man-in-the-Middle (MITM)**.

## 🎯 Objetivo
Ejecutar un ataque de interceptación para capturar credenciales de inicio de sesión enviadas desde un dispositivo físico externo (portátil) hacia una máquina de auditoría (Kali Linux), utilizando técnicas de envenenamiento ARP y suplantación de servidor.

## 🛠️ Herramientas Utilizadas
* **Kali Linux:** Sistema operativo atacante.
* **Ettercap:** Ejecución de *ARP Poisoning* para desviar el tráfico de la red.
* **Wireshark:** Análisis profundo de paquetes y reconstrucción de flujos TCP/HTTP.
* **Python 3 (`http.server`):** Despliegue de un servidor web falso (*Sinkhole*) para alojar el formulario de captura.
* **Nano:** Editor de texto en terminal para la creación del archivo malicioso.

## 🛡️ Metodología Aplicada: Suplantación de Servidor
Para maximizar la efectividad del ataque y evitar el cifrado HTTPS de sitios externos, se optó por una estrategia de **Server Impersonation**:
1. Se creó un archivo `login.html` legítimo en apariencia pero bajo control del atacante.
2. Se configuró un servidor local en el puerto 80 de Kali Linux.
3. Mediante el envenenamiento de tablas ARP, se redirigió a la víctima hacia la IP del atacante.

## 🚀 Paso a Paso

### 1. Preparación del Formulario (Phishing Local)
Se creó un formulario HTML básico en Kali Linux para recibir los datos por el método `POST`:
```html```
```<form method="POST">```
    Usuario: ```<input type="text" name="usuario"><br>```
    Password: ```<input type="password" name="password"><br>```
    ```<input type="submit" value="Entrar al Sistema">```
```</form>```
### 2. Despliegue del Servidor
Se activó el servidor de Python en la carpeta del archivo para escuchar las peticiones de la víctima:

Bash
```sudo python3 -m http.server 80```
### 3. Ejecución del Ataque MITM
Con Ettercap, se realizó el envenenamiento ARP entre el Router y la IP de la portátil (192.168.1.7), logrando que todo el tráfico de la víctima pasara por la interfaz de red de Kali.

### 4. Captura y Análisis en Wireshark
Al momento de la interacción de la víctima con el formulario, se aplicó el siguiente filtro en Wireshark:
http.request.method == "POST"

Posteriormente, se utilizó la función Follow > HTTP Stream para reconstruir la conversación y extraer las credenciales.

## 📈 Resultados
Se capturaron con éxito las credenciales en texto plano:

Usuario capturado: XXX

Contraseña capturada: xxx

Este resultado valida que el tráfico HTTP carece de integridad y confidencialidad, permitiendo a un atacante con acceso a la red local leer información sensible sin necesidad de romper cifrados complejos.

## ⚠️ Conclusiones y Mitigación
Uso de HTTPS: La implementación de TLS/SSL es obligatoria para proteger la comunicación cliente-servidor.

HSTS: Implementar HTTP Strict Transport Security para forzar conexiones seguras.

Seguridad de Red: Configurar el filtrado de DHCP y la inspección de ARP dinámico (DAI) en switches para prevenir el envenenamiento de tablas.
