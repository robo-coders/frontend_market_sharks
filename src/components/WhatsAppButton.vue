<script setup lang="ts">
const whatsappUrl = "https://chat.whatsapp.com/GIuCPbJ7V9W3wtYsQQz1x4";
</script>

<template>
  <a
    :href="whatsappUrl"
    target="_blank"
    rel="noopener noreferrer"
    aria-label="Join our WhatsApp community"
    class="whatsapp-fab"
  >
    <!-- Slide-in label -->
    <span class="fab-label">Join our community</span>

    <!-- Icon button -->
    <span class="fab-icon">
      <!-- Notification dot -->
      <span class="fab-dot" />

      <!-- WhatsApp SVG -->
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="28"
        height="28"
        viewBox="0 0 24 24"
        fill="currentColor"
      >
        <path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.884-9.885 9.884m8.413-18.297A11.815 11.815 0 0012.05 0C5.495 0 .16 5.335.157 11.892c0 2.096.547 4.142 1.588 5.945L.057 24l6.305-1.654a11.882 11.882 0 005.683 1.448h.005c6.554 0 11.89-5.335 11.893-11.893a11.821 11.821 0 00-3.48-8.413z"/>
      </svg>
    </span>
  </a>
</template>

<style scoped>
/* Wrapper — flex row so label + icon sit side by side */
.whatsapp-fab {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  z-index: 50;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  text-decoration: none;
  animation: slideUp 0.5s cubic-bezier(0.16, 1, 0.3, 1) both;
  animation-delay: 1.2s;
}

@keyframes slideUp {
  from { transform: translateY(24px); opacity: 0; }
  to   { transform: translateY(0);    opacity: 1; }
}

/* Circular icon button */
.fab-icon {
  position: relative;
  width: 60px;
  height: 60px;
  border-radius: 9999px;
  background: #25D366;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  box-shadow:
    0 4px 16px rgba(37, 211, 102, 0.45),
    0 2px 6px rgba(0, 0, 0, 0.12);
  transition:
    transform 0.2s cubic-bezier(0.16, 1, 0.3, 1),
    box-shadow 0.2s ease;
}

.whatsapp-fab:hover .fab-icon {
  transform: scale(1.08);
  box-shadow:
    0 8px 28px rgba(37, 211, 102, 0.55),
    0 4px 12px rgba(0, 0, 0, 0.18);
}

.whatsapp-fab:active .fab-icon {
  transform: scale(0.94);
}

/* Red notification dot */
.fab-dot {
  position: absolute;
  top: 4px;
  right: 4px;
  width: 11px;
  height: 11px;
  border-radius: 9999px;
  background: #ef4444;
  border: 2px solid #fff;
  animation: dotPulse 2.5s ease-in-out infinite;
}

@keyframes dotPulse {
  0%, 100% { transform: scale(1);    opacity: 1; }
  50%       { transform: scale(1.25); opacity: 0.7; }
}

/* Slide-in label pill */
.fab-label {
  background: #111;
  color: #fff;
  font-size: 0.78rem;
  font-weight: 600;
  letter-spacing: 0.01em;
  padding: 0.45rem 1rem;
  border-radius: 9999px;
  white-space: nowrap;
  margin-right: 0.65rem;
  opacity: 0;
  transform: translateX(10px);
  transition:
    opacity 0.22s ease,
    transform 0.22s cubic-bezier(0.16, 1, 0.3, 1);
  pointer-events: none;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

/* Dark mode — flip label to light surface */
@media (prefers-color-scheme: dark) {
  .fab-label {
    background: #f5f5f5;
    color: #111;
  }
}

.whatsapp-fab:hover .fab-label {
  opacity: 1;
  transform: translateX(0);
}

/* Mobile — hide label, slightly smaller icon */
@media (max-width: 640px) {
  .whatsapp-fab {
    bottom: 1.5rem;
    right: 1.5rem;
  }

  .fab-icon {
    width: 54px;
    height: 54px;
  }

  .fab-label {
    display: none;
  }
}
</style>