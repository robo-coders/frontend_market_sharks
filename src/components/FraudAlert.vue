<script setup lang="ts">
import { ref } from "vue";

const visible = ref(true);

function dismiss() {
  visible.value = false;
}
</script>

<template>
  <Teleport to="body">
    <Transition name="fade">
      <div
        v-if="visible"
        class="fixed inset-0 z-50 flex items-center justify-center p-4"
        style="background: rgba(0,0,0,0.6);"
      >
        <div class="relative bg-background border rounded-2xl shadow-xl max-w-lg w-full p-8">

          <!-- Close button -->
          <button
            @click="dismiss"
            class="absolute top-4 right-4 text-muted-foreground hover:text-foreground transition-colors"
            aria-label="Close"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24"
              fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/>
            </svg>
          </button>

          <!-- Header -->
          <div class="mb-5 text-center">
            <p class="text-sm font-semibold uppercase tracking-widest text-orange-500 mb-1">Important Notice</p>
            <h2 class="text-xl font-bold">⚠️ Fraud & Impersonation Warning</h2>
          </div>

          <!-- Body -->
          <p class="text-sm text-muted-foreground mb-5 leading-relaxed">
            We've been made aware of fake accounts and individuals impersonating Market Sharks
            across WhatsApp, Telegram and other platforms asking people for money or personal
            information. Please be aware:
          </p>

          <ul class="space-y-3 mb-6">
            <li class="flex items-start gap-3 text-sm">
              <span class="mt-0.5 text-orange-500 font-bold">›</span>
              <span>Market Sharks will <strong>never</strong> ask for deposits, payments, or personal credentials via private messages.</span>
            </li>
            <li class="flex items-start gap-3 text-sm">
              <span class="mt-0.5 text-orange-500 font-bold">›</span>
              <span>All legitimate transactions happen exclusively through our official website: <strong>marketsharks.live</strong></span>
            </li>
            <li class="flex items-start gap-3 text-sm">
              <span class="mt-0.5 text-orange-500 font-bold">›</span>
              <span>Always verify you are speaking with official Market Sharks accounts before sending anything.</span>
            </li>
          </ul>

          <p class="text-xs text-muted-foreground mb-6">
            If you've been contacted by someone claiming to be Market Sharks, please report it to
            <a href="mailto:support@marketsharks.live" class="text-primary underline underline-offset-2">
              support@marketsharks.live
            </a>
          </p>

          <button
            @click="dismiss"
            class="w-full bg-primary text-primary-foreground hover:bg-primary/90 transition-colors rounded-lg py-2.5 text-sm font-semibold"
          >
            I understand, continue to site
          </button>

        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>