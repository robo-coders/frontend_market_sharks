<script setup lang="ts">
import { computed } from "vue";
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

const props = defineProps<{
  title?: string;
  user?: Partial<AppUser>;
}>();

/**
 * Professional: layout merges defaults, pages can pass partial dummy user.
 */
const currentUser = computed<AppUser>(() => ({
  name: props.user?.name ?? "Ahmed",
  email: props.user?.email ?? "ahmed@example.com",
  status: (props.user?.status as UserStatus) ?? "active",
  plan: props.user?.plan ?? "premium",
  expires_at: props.user?.expires_at ?? "2026-03-25",
}));
</script>

<template>
  <div class="min-h-screen bg-background">
    <!-- ONE navbar for all /app/* pages -->
    <AppNavbar :user="currentUser" />

    <!-- Page Container -->
    <main class="mx-auto w-[90%] md:w-[70%] lg:w-[75%] lg:max-w-screen-xl py-8">
      <div class="mb-6">
        <h1 class="text-2xl md:text-3xl font-bold">
          {{ props.title ?? "App" }}
        </h1>
        <p class="text-muted-foreground mt-1">
          Your Market Sharks member area.
        </p>
      </div>

      <slot :user="currentUser" />
    </main>
  </div>
</template>
