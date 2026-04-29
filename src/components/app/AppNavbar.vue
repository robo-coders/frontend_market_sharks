<script setup lang="ts">
import { computed, ref } from "vue";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";

import {
  Sheet,
  SheetContent,
  SheetHeader,
  SheetTitle,
  SheetTrigger,
} from "@/components/ui/sheet";

import {
  LayoutDashboard,
  CreditCard,
  User as UserIcon,
  TrendingUp,
  Video,
  Menu,
  LogOut,
} from "lucide-vue-next";

type UserStatus = "pending" | "active" | "blocked" | "rejected";

interface AppUser {
  name: string;
  email?: string;
  status: UserStatus;
}

const props = defineProps<{ user: AppUser }>();

const isOpen = ref(false);

const nav = [
  { href: "/app/dashboard", label: "Dashboard", icon: LayoutDashboard },
  // { href: "/app/Live", label: "Live", icon: TrendingUp },
  // { href: "/app/Lectures", label: "Lectures", icon: Video },
  { href: "/app/billing", label: "Billing", icon: CreditCard },
  { href: "/app/profile", label: "Profile", icon: UserIcon },
];

const currentPath = computed(() => window.location.pathname);
const isActive = (href: string) => currentPath.value === href;

const go = (href: string) => {
  window.history.pushState({}, "", href);
  window.dispatchEvent(new PopStateEvent("popstate"));
  isOpen.value = false;
};

const statusLabel = computed(() => (props.user.status ?? "pending").toUpperCase());
const statusVariant = computed(() => {
  if (props.user.status === "active") return "default";
  if (props.user.status === "pending") return "secondary";
  return "destructive";
});

const logout = () => {
  localStorage.removeItem("auth_token");
  window.dispatchEvent(new Event("storage"));
  window.history.pushState({}, "", "/login");
  window.dispatchEvent(new PopStateEvent("popstate"));
};
</script>

<template>
  <header
    class="sticky top-0 z-50 border-b bg-card/70 backdrop-blur supports-[backdrop-filter]:bg-card/60"
  >
    <div class="mx-auto w-[90%] md:w-[70%] lg:w-[75%] lg:max-w-screen-xl">
      <div class="h-14 flex items-center justify-between gap-4">
        <!-- Logo -->
        <a href="/" class="flex items-center gap-2 font-semibold tracking-tight">
          <img src="/final.png" alt="Market Sharks" class="w-7 h-7" />
          <span class="text-sm md:text-base">Market Sharks</span>
        </a>

        <!-- Desktop Nav -->
        <nav class="hidden md:flex items-center gap-1">
          <button
            v-for="item in nav"
            :key="item.href"
            type="button"
            @click="go(item.href)"
            class="relative px-3 py-2 rounded-lg text-sm transition"
            :class="
              isActive(item.href)
                ? 'text-foreground'
                : 'text-muted-foreground hover:text-foreground hover:bg-muted/40'
            "
          >
            <span class="inline-flex items-center gap-2">
              <component :is="item.icon" class="size-4 opacity-80" />
              {{ item.label }}
            </span>

            <!-- Keep underline (but make it premium) -->
            <span
              v-if="isActive(item.href)"
              class="absolute left-3 right-3 -bottom-[6px] h-[2px] rounded-full bg-primary/90"
            />
          </button>
        </nav>

        <!-- Desktop Right -->
        <div class="hidden md:flex items-center gap-3">
          <div class="text-right leading-tight">
            <div class="text-sm font-medium">{{ props.user.name }}</div>
            <div v-if="props.user.email" class="text-xs text-muted-foreground">
              {{ props.user.email }}
            </div>
          </div>

          <Badge :variant="statusVariant" class="px-3 text-xs">
            {{ statusLabel }}
          </Badge>

          <!-- Sleeker logout (not button-y) -->
          <button
            type="button"
            @click="logout"
            class="inline-flex items-center gap-2 text-sm text-muted-foreground hover:text-foreground transition"
          >
            <LogOut class="size-4" />
            Logout
          </button>
        </div>

        <!-- Mobile -->
        <div class="md:hidden">
          <Sheet v-model:open="isOpen">
            <SheetTrigger as-child>
              <Button variant="secondary" size="sm" class="gap-2">
                <Menu class="size-4" />
                Menu
              </Button>
            </SheetTrigger>

            <SheetContent side="left" class="bg-card">
              <SheetHeader class="mb-4">
                <SheetTitle class="flex items-center gap-2">
                  <img src="/final.png" alt="Market Sharks" class="w-7 h-7" />
                  Market Sharks
                </SheetTitle>
              </SheetHeader>

              <div class="mb-4">
                <div class="text-sm font-semibold">{{ props.user.name }}</div>
                <div v-if="props.user.email" class="text-xs text-muted-foreground">
                  {{ props.user.email }}
                </div>

                <div class="mt-2">
                  <Badge :variant="statusVariant">{{ statusLabel }}</Badge>
                </div>
              </div>

              <div class="flex flex-col gap-1">
                <button
                  v-for="item in nav"
                  :key="item.href"
                  type="button"
                  @click="go(item.href)"
                  class="w-full flex items-center gap-2 px-3 py-2 rounded-lg text-sm transition text-left"
                  :class="
                    isActive(item.href)
                      ? 'bg-muted text-foreground'
                      : 'text-muted-foreground hover:text-foreground hover:bg-muted/40'
                  "
                >
                  <component :is="item.icon" class="size-4 opacity-80" />
                  {{ item.label }}
                </button>
              </div>

              <div class="mt-4">
                <button
                  type="button"
                  @click="logout"
                  class="w-full flex items-center gap-2 px-3 py-2 rounded-lg text-sm text-muted-foreground hover:text-foreground hover:bg-muted/40 transition"
                >
                  <LogOut class="size-4" />
                  Logout
                </button>
              </div>
            </SheetContent>
          </Sheet>
        </div>
      </div>
    </div>
  </header>
</template>
