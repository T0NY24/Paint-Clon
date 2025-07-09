# 🎨 Paint App - Aplicación de Dibujo en Canvas

<div align="center">

![Paint App](https://img.shields.io/badge/Paint-App-ff6b6b?style=for-the-badge&logo=paint-brush&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![Canvas](https://img.shields.io/badge/Canvas-API-4285F4?style=for-the-badge&logo=google-chrome&logoColor=white)

**Una aplicación de dibujo completa y moderna construida con JavaScript vanilla y Canvas API**

</div>

---

## 📋 Tabla de Contenidos

- [✨ Características](#-características)
- [🚀 Instalación](#-instalación)
- [🎮 Modos de Funcionamiento](#-modos-de-funcionamiento)
- [🔧 Arquitectura del Código](#-arquitectura-del-código)
- [🎯 Funcionalidades Principales](#-funcionalidades-principales)
- [⚙️ Configuración](#️-configuración)
- [🤝 Contribución](#-contribución)

---

## ✨ Características

<table>
<tr>
<td width="50%">

### 🖊️ Herramientas de Dibujo
- **Lápiz** - Dibujo libre con trazo suave
- **Borrador** - Eliminación precisa de contenido
- **Rectángulo** - Formas geométricas perfectas
- **Elipse** - Círculos y óvalos (próximamente)

</td>
<td width="50%">

### 🎨 Funciones Avanzadas
- **Selector de Color** - Paleta completa de colores
- **Cuentagotas** - Extrae colores de la pantalla
- **Guardado PNG** - Exporta tu arte
- **Limpiar Canvas** - Reinicia rápidamente

</td>
</tr>
</table>

---

## 🚀 Instalación

```bash
# Clona el repositorio
git clone https://github.com/tu-usuario/paint-app.git

# Navega al directorio
cd paint-app

# Abre index.html en tu navegador favorito
open index.html
```

> **📝 Nota:** No requiere instalación adicional - funciona directamente en el navegador

---

## 🎮 Modos de Funcionamiento

<div align="center">

| Modo | Icono | Descripción | Atajo |
|------|-------|-------------|-------|
| **Dibujo** | 🖊️ | Dibuja líneas suaves y continuas | `D` |
| **Borrador** | 🧹 | Elimina contenido del canvas | `E` |
| **Rectángulo** | ⬜ | Crea rectángulos y cuadrados | `R` |
| **Selector** | 🎨 | Cambia el color de dibujo | `C` |
| **Cuentagotas** | 💧 | Extrae colores de la pantalla | `P` |
| **Guardar** | 💾 | Exporta como imagen PNG | `S` |

</div>

---

## 🔧 Arquitectura del Código

### 📁 Estructura Principal

```javascript
// 🎯 Constantes de Modos
const MODES = {
  DRAW: 'draw',
  ERASE: 'erase', 
  RECTANGLE: 'rectangle',
  ELLIPSE: 'ellipse',
  PICKER: 'picker',
  SAVE: 'save'
}

// 🔧 Utilidades DOM
const $ = selector => document.querySelector(selector)
const $$ = selector => document.querySelectorAll(selector)
```

### 🎨 Estado de la Aplicación

<table>
<tr>
<td width="50%">

**Variables de Control**
```javascript
let isDrawing = false      // 🖱️ Estado del mouse
let isShiftPressed = false // ⌨️ Tecla Shift
let mode = MODES.DRAW     // 🎯 Modo actual
```

</td>
<td width="50%">

**Coordenadas**
```javascript
let startX, startY        // 📍 Inicio del dibujo
let lastX = 0, lastY = 0  // 📍 Última posición
let imageData             // 💾 Backup del canvas
```

</td>
</tr>
</table>

---

## 🎯 Funcionalidades Principales

### 🖊️ Sistema de Dibujo

<details>
<summary><strong>📝 Función startDrawing()</strong></summary>

```javascript
function startDrawing(event) {
  isDrawing = true
  const { offsetX, offsetY } = event;
  
  // 📍 Guarda coordenadas iniciales
  [startX, startY] = [offsetX, offsetY];
  [lastX, lastY] = [offsetX, offsetY];

  // 💾 Backup del estado actual
  imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
}
```

**Funcionalidad:** Inicializa el proceso de dibujo y guarda el estado del canvas para poder restaurarlo.

</details>

<details>
<summary><strong>🎨 Función draw()</strong></summary>

```javascript
function draw(event) {
  if (!isDrawing) return

  const { offsetX, offsetY } = event

  if (mode === MODES.DRAW || mode === MODES.ERASE) {
    // ✏️ Dibuja línea continua
    ctx.beginPath()
    ctx.moveTo(lastX, lastY)
    ctx.lineTo(offsetX, offsetY)
    ctx.stroke();
    [lastX, lastY] = [offsetX, offsetY]
    return
  }

  if (mode === MODES.RECTANGLE) {
    // 🔄 Restaura imagen y dibuja rectángulo temporal
    ctx.putImageData(imageData, 0, 0);
    
    let width = offsetX - startX
    let height = offsetY - startY

    // ⌨️ Shift para cuadrado perfecto
    if (isShiftPressed) {
      const sideLength = Math.min(Math.abs(width), Math.abs(height))
      width = width > 0 ? sideLength : -sideLength
      height = height > 0 ? sideLength : -sideLength
    }

    ctx.beginPath()
    ctx.rect(startX, startY, width, height)
    ctx.stroke()
  }
}
```

**Funcionalidad:** Maneja el dibujo en tiempo real según el modo seleccionado.

</details>

### 🎨 Sistema de Modos

<details>
<summary><strong>⚙️ Función setMode()</strong></summary>

```javascript
async function setMode(newMode) {
  let previousMode = mode
  mode = newMode
  $('button.active')?.classList.remove('active')

  if (mode === MODES.DRAW) {
    // 🖊️ Configuración para dibujo
    $drawBtn.classList.add('active')
    canvas.style.cursor = 'crosshair'
    ctx.globalCompositeOperation = 'source-over'
    ctx.lineWidth = 2
    return
  }

  if (mode === MODES.PICKER) {
    // 💧 Cuentagotas con EyeDropper API
    $pickerBtn.classList.add('active')
    const eyeDropper = new window.EyeDropper()
    try {
      const result = await eyeDropper.open()
      ctx.strokeStyle = result.sRGBHex
      $colorPicker.value = result.sRGBHex
      setMode(previousMode)
    } catch (e) {
      // 🚨 Manejo de errores
    }
    return
  }

  if (mode === MODES.SAVE) {
    // 💾 Exportar como PNG
    ctx.globalCompositeOperation='destination-over'
    ctx.fillStyle = 'white'
    ctx.fillRect(0, 0, canvas.width, canvas.height)

    const link = document.createElement('a')
    link.href = canvas.toDataURL()
    link.download = 'my-paint.png'
    link.click()
    setMode(previousMode)
    return
  }
}
```

**Funcionalidad:** Cambia entre diferentes modos de operación y configura la interfaz correspondiente.

</details>

---

## ⚙️ Configuración

### 🎨 Personalización del Canvas

```javascript
// 🎯 Configuración inicial
ctx.lineJoin = 'round'    // Uniones suaves
ctx.lineCap = 'round'     // Extremos redondeados
ctx.lineWidth = 2         // Grosor de línea
```

### 🖱️ Eventos del Sistema

<table>
<tr>
<td width="50%">

**Eventos del Mouse**
- `mousedown` → `startDrawing()`
- `mousemove` → `draw()`
- `mouseup` → `stopDrawing()`

</td>
<td width="50%">

**Eventos del Teclado**
- `keydown` → Detecta Shift
- `keyup` → Libera Shift

</td>
</tr>
</table>

---


---

## 🔮 Características Avanzadas

### 💧 EyeDropper API

> **🚀 Tecnología Moderna:** Utiliza la API nativa del navegador para seleccionar colores desde cualquier parte de la pantalla.

```javascript
const eyeDropper = new window.EyeDropper()
const result = await eyeDropper.open()
ctx.strokeStyle = result.sRGBHex
```

### 🎨 Composición Global

```javascript
// Para dibujar
ctx.globalCompositeOperation = 'source-over'

// Para borrar
ctx.globalCompositeOperation = 'destination-out'
```

---

## 🤝 Contribución

¿Quieres mejorar esta aplicación? ¡Genial! 

1. 🍴 Haz un fork del proyecto
2. 🌱 Crea una rama para tu feature (`git checkout -b feature/nueva-herramienta`)
3. 💾 Commit tus cambios (`git commit -m 'Añade nueva herramienta'`)
4. 🚀 Push a la rama (`git push origin feature/nueva-herramienta`)
5. 📥 Abre un Pull Request

---

<div align="center">

## 🎉 ¡Disfruta Dibujando!

**Creado con ❤️ y mucho ☕**

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/tu-usuario/paint-app)
[![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)](https://developer.mozilla.org/es/docs/Web/JavaScript)
[![Canvas](https://img.shields.io/badge/Canvas-API-4285F4?style=for-the-badge&logo=google-chrome&logoColor=white)](https://developer.mozilla.org/es/docs/Web/API/Canvas_API)

</div>
