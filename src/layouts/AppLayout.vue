<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useColorMode } from "@vueuse/core";
import AppNavbar from "@/components/app/AppNavbar.vue";

const mode = useColorMode();
mode.value = "dark";

type UserStatus = "pending" | "active" | "blocked" | "rejected";

interface AppUser {
  name: string;
  email: string;
  status: UserStatus;
  plan: string;
  expires_at?: string | null;
}

const props = defineProps<{ title?: string }>();

const currentUser = ref<AppUser | null>(null);

onMounted(async () => {
  const token = localStorage.getItem("auth_token");
  const res = await fetch(`${import.meta.env.VITE_API_URL}/api/me`, {
    headers: { Authorization: `Bearer ${token}` },
  });

  if (res.ok) {
    const data = await res.json();
    currentUser.value = {
      name: data.user.name,
      email: data.user.email,
      status: data.user.status,
      plan: data.plan ?? "none",
      expires_at: data.expires_at ?? null,
    };
  } else {
    localStorage.removeItem("auth_token");
    window.location.href = "/login";
  }
});
</script>

<template>
  <div class="min-h-screen bg-background">
    <AppNavbar v-if="currentUser" :user="currentUser" />

    <main class="mx-auto w-[90%] md:w-[70%] lg:w-[75%] lg:max-w-screen-xl py-8">
      <div v-if="!currentUser" class="text-muted-foreground text-sm">Loading...</div>

      <template v-else>
        <div class="mb-6">
          <h1 class="text-2xl md:text-3xl font-bold">{{ props.title ?? "App" }}</h1>
          <p class="text-muted-foreground mt-1">Your Market Sharks member area.</p>
        </div>
        <slot :user="currentUser" />
      </template>
    </main>
  </div>
</template>