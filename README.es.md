<p align="left">
  🇬🇧 <a href="README.md">English</a> | 🇪🇸 <b>Español</b>
</p>

# Jigsaw Decryptor

<h3 align="center">🖱️ Un clic para ejecutar. Presiona un botón y... 

❌️¡Ransomware eliminado! y 📄¡Archivos recuperados!</h3>

<p align="center">
  <img width="635" height="558" alt="Jigsaw Decryptor" src="https://github.com/user-attachments/assets/91215231-0695-4ef9-9269-e774110e2429" />
</p>

## Descripción

**Jigsaw Decryptor** es una aplicación *Windows Forms* desarrollada en C# (.NET Framework 4.7.2) que desinfecta una máquina afectada por el ransomware **Jigsaw**: descifra los archivos secuestrados y elimina todos los artefactos del malware, dejando el equipo como si el ataque no hubiese ocurrido.

Qué hace la aplicación:

1. **Descifra** los archivos cifrados por Jigsaw (extensión `.fun`), restaurando el original y borrando la copia cifrada.
2. **Termina** los procesos del malware.
3. **Elimina** los ficheros que el malware dejó en el sistema.
4. **Elimina la persistencia** que el malware creó en el registro de Windows.

Probado con la muestra de SHA256: `3ae96f73d805e1d3995253db4d910300d8442ea603737a1428b613061e7f61e7`.

> **Limitación:** el descifrado usa la clave y el IV estáticos que incorpora Jigsaw. Las variantes que usen otra clave no se descifrarán con esta herramienta.

> [!NOTE] Nota
> Proyecto realizado durante el TFM **"Análisis de malware del ransomware Jigsaw"**.

## Arquitectura

Aplicación de dos capas, sin dependencias externas (solo la librería base de .NET Framework):

```
Jigsaw-Decryptor/
├── Program.cs          Punto de entrada: abre el formulario principal
├── Form1.cs            UI: confirmación, progreso y orquestación de la limpieza
├── Form1.Designer.cs   Diseño del formulario (generado por Visual Studio)
├── Resources/          Logo de la aplicación
└── Utils/
    ├── Config.cs       Constantes y rutas de los artefactos del malware
    ├── Decryptor.cs    Descifrado AES de los archivos
    ├── Cleaner.cs      Eliminación de procesos, ficheros y persistencia
    └── Logger.cs       Registro de actividad en un archivo de log
```

Flujo de ejecución:

1. El usuario pulsa **Decrypt My Files!** y confirma el aviso.
2. `Decryptor` lee la lista de archivos cifrados que dejó el malware (`%APPDATA%\System32Work\EncryptedFileList.txt`).
3. Un `BackgroundWorker` descifra cada `<archivo>.fun` con AES-128 a su ruta original, sin bloquear la interfaz, y actualiza la barra de progreso.
4. Al terminar, `Cleaner`:
   - mata los procesos `drpbx.exe` y `firefox.exe` del malware (comparando la **ruta completa**, para no afectar al Firefox legítimo);
   - borra `%APPDATA%\System32Work`, `%LOCALAPPDATA%\Drpbx` y `%APPDATA%\Frfx`;
   - elimina el valor `firefox.exe` de `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`.
5. Se informa si la eliminación fue total o parcial.

Todas las operaciones quedan registradas en `log_<fecha>.txt`, en la carpeta del ejecutable con su timestamp. En este fichero se puede ver si algún elemento no se pudo descifrar. 

## Compilación

**Requisitos:** Windows, Visual Studio 2022 (carga de trabajo *Desarrollo de escritorio con .NET*) y .NET Framework 4.7.2 Developer Pack.

### Con Visual Studio

1. Abre `Jigsaw-Decryptor.sln`.
2. Selecciona la configuración **Release** (Any CPU).
3. Menú **Compilar → Compilar solución** (`Ctrl+Mayús+B`).
4. El ejecutable se genera en `Jigsaw-Decryptor\bin\Release\Jigsaw-Decryptor.exe`.

### Desde la línea de comandos

En una *Developer Command Prompt for VS 2022*:

```
msbuild Jigsaw-Decryptor.sln /p:Configuration=Release
```

El resultado queda en la misma ruta: `Jigsaw-Decryptor\bin\Release\Jigsaw-Decryptor.exe`. El `.exe` es autónomo y no necesita instalación.

## Uso

1. Copia `Jigsaw-Decryptor.exe` a la máquina infectada.
2. Ejecútalo con **doble clic**, con la sesión del usuario afectado (el malware trabaja sobre su perfil y su registro).
3. Pulsa **Decrypt My Files!** y acepta el aviso.
4. Espera a que la barra de progreso llegue al final. Verás el archivo que se está descifrando en cada momento.
5. Al finalizar, la aplicación muestra si el malware se eliminó **totalmente** o **parcialmente** y se cierra sola.

Notas:

- No cierres la aplicación mientras descifra.
- Si falta `EncryptedFileList.txt`, la herramienta no puede saber qué archivos descifrar y se cerrará.
- Si la eliminación fue parcial, consulta el archivo `log_<fecha>.txt` junto al ejecutable para ver qué falló.
