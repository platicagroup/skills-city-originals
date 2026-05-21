---
name: sk-modals-v1
description: >-
  Guidelines, templates, and patterns for creating dark-themed, glassmorphic success, error, and failure modals in React/Next.js.
---

# Skill de Creación de Modales de Estado (Éxito, Error y Fallo) - V1
Guidelines, templates, and patterns for creating dark-themed, glassmorphic success, error, and failure modals in React/Next.js.

## Resumen
Esta habilidad detalla el flujo de trabajo, los patrones de diseño y las plantillas de código para crear e integrar modales de estado (éxito, error y fallos de autenticación o procesos) con una estética de terminal retro-moderna premium utilizando React/Next.js y CSS Modules.

## Dependencias
* React 18+ / Next.js (App Router o Pages Router)
* CSS Modules (Vanilla CSS)

## Guía de Ejecución Interactiva (Instrucciones para el Agente)
Cuando un desarrollador te solicite implementar o modificar un modal de éxito, error o fallo, realiza las siguientes preguntas de alineación antes de proceder:
1. **Flujo de Origen**: ¿Qué acción desencadenará el modal? (ej. Registro de usuario, Inicio de sesión, Guardado de datos, Pago exitoso/fallido).
2. **Mensajería Requerida**: ¿Los textos del título y descripción provienen de respuestas dinámicas del servidor/API o se definirán estáticamente en el frontend?
3. **Persistencia del Modal**: ¿El modal debe cerrarse automáticamente tras un intervalo de tiempo (e.g. 3 segundos) o requerir una interacción obligatoria del usuario haciendo clic en un botón?

---

## Flujo de Trabajo y Patrones de Diseño

### 1. Estructura de Componentes
Los modales de estado deben ser independientes y recibir propiedades claras para maximizar su reutilización. La firma básica del componente debe ser:

```typescript
interface StatusModalProps {
  isOpen: boolean;
  type: "success" | "error";
  title: string;
  message: string;
  onClose: () => void;
  actionText?: string;
  onAction?: () => void;
}
```

### 2. Estética Visual Premium
Para encajar con la temática de desarrollador/terminal de SkillsCity:
* **Fondo de Cristal (Glassmorphism)**: El overlay debe usar opacidad y desenfoque (`backdrop-filter: blur(8px)`) para sumergir al usuario.
* **Marcos y Contenedores**: El contenedor principal del modal debe usar un fondo oscuro sólido (ej. `#1e1e1e`) con bordes vibrantes y sombras neón que coincidan con el tipo de estado:
  * **Success**: Bordes verdes (`#2b9348` o `#4ade80`), sombra difusa verde y un tag terminal que indique `$ STATUS: SUCCESS`.
  * **Error/Fallo**: Bordes rojos (`#ef4444` o `#f87171`), sombra difusa roja y un tag terminal que indique `$ STATUS: ERROR`.
* **Cabecera de Ventana**: Incluir botones simulados de control de ventana estilo terminal (`_`, `□`, `X`) para reforzar el aspecto de comando del sistema.

### 3. Gestión de Scroll del Body
Es crítico bloquear el scroll del fondo cuando el modal esté abierto para evitar que la página se desplace detrás del mismo. Esto se maneja usando un efecto de React:

```typescript
useEffect(() => {
  if (isOpen) {
    document.body.style.overflow = "hidden";
  } else {
    document.body.style.overflow = "unset";
  }
  return () => {
    document.body.style.overflow = "unset";
  };
}, [isOpen]);
```

---

## Plantillas de Código de Referencia

### Componente React (`StatusModal.tsx`)
```tsx
"use client";

import React, { useEffect } from "react";
import styles from "./StatusModal.module.css";

interface StatusModalProps {
  isOpen: boolean;
  type: "success" | "error";
  title: string;
  message: string;
  onClose: () => void;
  actionText?: string;
  onAction?: () => void;
}

export default function StatusModal({
  isOpen,
  type,
  title,
  message,
  onClose,
  actionText = "Aceptar",
  onAction,
}: StatusModalProps) {
  useEffect(() => {
    if (isOpen) {
      document.body.style.overflow = "hidden";
    } else {
      document.body.style.overflow = "unset";
    }
    return () => {
      document.body.style.overflow = "unset";
    };
  }, [isOpen]);

  if (!isOpen) return null;

  const handleAction = () => {
    if (onAction) {
      onAction();
    } else {
      onClose();
    }
  };

  const isSuccess = type === "success";

  return (
    <div className={styles.overlay} onClick={onClose}>
      <div
        className={`${styles.container} ${isSuccess ? styles.successBorder : styles.errorBorder}`}
        onClick={(e) => e.stopPropagation()}
      >
        <div className={styles.terminalHeader}>
          <div className={styles.terminalTitle}>C:\skillscity\system_response.exe</div>
          <div className={styles.winButtons}>
            <button className={styles.winButtonClose} onClick={onClose}>X</button>
          </div>
        </div>

        <div className={styles.content}>
          <div className={styles.iconContainer}>
            {isSuccess ? (
              <div className={styles.successIconWrapper}>
                <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2">
                  <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14" />
                  <polyline points="22 4 12 14.01 9 11.01" />
                </svg>
              </div>
            ) : (
              <div className={styles.errorIconWrapper}>
                <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2">
                  <circle cx="12" cy="12" r="10" />
                  <line x1="15" y1="9" x2="9" y2="15" />
                  <line x1="9" y1="9" x2="15" y2="15" />
                </svg>
              </div>
            )}
          </div>

          <div className={styles.textSection}>
            <div className={styles.terminalTag}>
              $ STATUS: <span className={isSuccess ? styles.greenText : styles.redText}>{type.toUpperCase()}</span>
            </div>
            <h3 className={styles.title}>{title}</h3>
            <p className={styles.message}>{message}</p>
          </div>

          <div className={styles.buttonContainer}>
            <button
              onClick={handleAction}
              className={`${styles.actionButton} ${isSuccess ? styles.successButton : styles.errorButton}`}
            >
              {actionText}
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}
```

### Estilos CSS (`StatusModal.module.css`)
```css
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(8px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 4000;
  animation: overlayFadeIn 0.3s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

.container {
  width: 100%;
  max-width: 440px;
  background-color: #1e1e1e;
  border-radius: 6px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  font-family: 'Consolas', 'Courier New', monospace;
  animation: modalScaleUp 0.35s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

.successBorder {
  border: 1px solid #2b9348;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4), 0 0 15px rgba(43, 147, 72, 0.2);
}

.errorBorder {
  border: 1px solid #ef4444;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4), 0 0 15px rgba(239, 68, 68, 0.2);
}

.terminalHeader {
  background-color: #2a2a2a;
  padding: 0.5rem 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #333;
}

.terminalTitle {
  color: #a0a0a0;
  font-size: 0.8rem;
}

.winButtonClose {
  background: none;
  border: none;
  color: #888;
  font-size: 0.8rem;
  cursor: pointer;
  padding: 0.2rem 0.4rem;
}

.winButtonClose:hover {
  background-color: #e81123;
  color: #fff;
}

.content {
  padding: 2.5rem 2rem 2.2rem 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  gap: 1.5rem;
}

.successIconWrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background-color: rgba(43, 147, 72, 0.1);
  color: #4ade80;
  box-shadow: 0 0 20px rgba(43, 147, 72, 0.15);
  animation: pulseSuccess 2s infinite ease-in-out;
}

.errorIconWrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background-color: rgba(239, 68, 68, 0.1);
  color: #f87171;
  box-shadow: 0 0 20px rgba(239, 68, 68, 0.15);
  animation: pulseError 2s infinite ease-in-out;
}

.terminalTag {
  font-size: 0.75rem;
  color: #888;
}

.greenText {
  color: #4ade80;
}

.redText {
  color: #f87171;
}

.title {
  font-size: 1.25rem;
  font-weight: bold;
  color: #ffffff;
  margin: 0;
}

.message {
  font-size: 0.88rem;
  color: #a0a0a0;
  line-height: 1.5;
  margin: 0;
}

.actionButton {
  width: 100%;
  max-width: 220px;
  padding: 0.7rem;
  border-radius: 4px;
  font-weight: bold;
  cursor: pointer;
  border: none;
  transition: all 0.2s ease;
}

.successButton {
  background-color: #2b9348;
  color: #ffffff;
}

.successButton:hover {
  background-color: #38b056;
  transform: translateY(-1px);
}

.errorButton {
  background-color: #ef4444;
  color: #ffffff;
}

.errorButton:hover {
  background-color: #f87171;
  transform: translateY(-1px);
}

@keyframes overlayFadeIn {
  from { background-color: rgba(0, 0, 0, 0); backdrop-filter: blur(0px); }
  to { background-color: rgba(0, 0, 0, 0.6); backdrop-filter: blur(8px); }
}

@keyframes modalScaleUp {
  from { transform: scale(0.92); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

@keyframes pulseSuccess {
  0% { box-shadow: 0 0 0 0 rgba(74, 222, 128, 0.4); }
  70% { box-shadow: 0 0 0 12px rgba(74, 222, 128, 0); }
  100% { box-shadow: 0 0 0 0 rgba(74, 222, 128, 0); }
}

@keyframes pulseError {
  0% { box-shadow: 0 0 0 0 rgba(248, 113, 113, 0.4); }
  70% { box-shadow: 0 0 0 12px rgba(248, 113, 113, 0); }
  100% { box-shadow: 0 0 0 0 rgba(248, 113, 113, 0); }
}
```

---

## Errores Comunes a Evitar
1. **No Limpiar Eventos o Estados**: No restablecer el estado del modal a `isOpen: false` o no limpiar los campos del formulario tras hacer clic en el botón de acción, lo que puede causar fallos lógicos o que el modal se muestre repetidamente.
2. **Duplicar Overlays de Body**: No desacoplar o verificar los z-indexes de otros modales. Si el modal de estado se muestra detrás de otro modal de inicio de sesión debido a un z-index menor a 3000, el usuario no podrá interactuar con él.
3. **No Habilitar Accesibilidad**: No proveer un botón de cierre fácil de identificar o no vincular el evento del teclado `Escape` para cerrar el modal.
