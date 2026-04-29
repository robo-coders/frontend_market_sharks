<script lang="ts" setup>
import { ref } from "vue";

import { useColorMode } from "@vueuse/core";
const mode = useColorMode();
mode.value = "dark";

import {
  NavigationMenu,
  NavigationMenuItem,
  NavigationMenuLink,
  NavigationMenuList,
} from "@/components/ui/navigation-menu";
import {
  Sheet,
  SheetContent,
  SheetFooter,
  SheetHeader,
  SheetTitle,
  SheetTrigger,
} from "@/components/ui/sheet";

import { Button } from "@/components/ui/button";
import { Separator } from "@/components/ui/separator";

import { Menu, LogIn, LayoutDashboard } from "lucide-vue-next";
import ToggleTheme from "./ToggleTheme.vue";

interface RouteProps {
  href: string;
  label: string;
}

const routeList: RouteProps[] = [
  { href: "#community", label: "Community" },
  { href: "#pricing", label: "Pricing" },
  { href: "#contact", label: "Contact" },
  { href: "#faq", label: "FAQ" },
];

const isOpen = ref(false);

const isLoggedIn = !!localStorage.getItem("auth_token");
</script>

<template>
  <header
    :class="{
      'shadow-light': mode === 'light',
      'shadow-dark': mode === 'dark',
      'w-[90%] md:w-[70%] lg:w-[75%] lg:max-w-screen-xl top-5 mx-auto sticky border z-40 rounded-2xl flex justify-between items-center p-2 bg-card shadow-md':
        true,
    }"
  >
    <!-- Logo -->
    <a href="/" class="font-bold text-lg flex items-center">
      <img src="/final.png" alt="Market Sharks" class="w-10 h-10" />
      
      &nbsp; Market Sharks
    </a>

    <!-- Mobile -->
    <div class="flex items-center lg:hidden">
      <Sheet v-model:open="isOpen">
        <SheetTrigger as-child>
          <Menu class="cursor-pointer" />
        </SheetTrigger>

        <SheetContent
          side="left"
          class="flex flex-col justify-between rounded-tr-2xl rounded-br-2xl bg-card"
        >
          <div>
            <SheetHeader class="mb-4 ml-4">
              <SheetTitle class="flex items-center">
                <a href="/" class="flex items-center">
                  <img
                    src="/final.png"
                    alt="Market Sharks"
                    class="w-9 h-9 mr-2 rounded-lg object-contain"
                  />
                  Market Sharks
                </a>
              </SheetTitle>
            </SheetHeader>

            <div class="flex flex-col gap-2">
              <Button
                v-for="{ href, label } in routeList"
                :key="label"
                as-child
                variant="ghost"
                class="justify-start text-base"
              >
                <a :href="href" @click="isOpen = false">
                  {{ label }}
                </a>
              </Button>
            </div>
          </div>

          <SheetFooter class="flex-col items-start">
            <Separator class="mb-2" />

            <div class="flex w-full items-center justify-between gap-2">
              <ToggleTheme />

              <Button as-child size="sm" aria-label="Login">
                <a
                  :href="isLoggedIn ? '/app/dashboard' : '/login'"
                  @click="isOpen = false"
                  class="inline-flex items-center gap-2"
                >
                  <LayoutDashboard v-if="isLoggedIn" class="size-4" />
                  <LogIn v-else class="size-4" />
                  {{ isLoggedIn ? "Dashboard" : "Login" }}
                </a>
              </Button>
            </div>
          </SheetFooter>
        </SheetContent>
      </Sheet>
    </div>

    <!-- Desktop Navigation -->
    <NavigationMenu class="hidden lg:block">
      <NavigationMenuList>
        <NavigationMenuItem>
          <NavigationMenuLink asChild>
            <Button
              v-for="{ href, label } in routeList"
              :key="label"
              as-child
              variant="ghost"
              class="justify-start text-base"
            >
              <a :href="href">{{ label }}</a>
            </Button>
          </NavigationMenuLink>
        </NavigationMenuItem>
      </NavigationMenuList>
    </NavigationMenu>

    <!-- Desktop Right Actions -->
    <div class="hidden lg:flex items-center gap-2">
      <ToggleTheme />

      <Button
        as-child
        size="sm"
        class="rounded-full px-4 shadow-sm"
        :aria-label="isLoggedIn ? 'Dashboard' : 'Login'"
      >
        <a
          :href="isLoggedIn ? '/app/dashboard' : '/login'"
          class="inline-flex items-center gap-2"
        >
          <LayoutDashboard v-if="isLoggedIn" class="size-4" />
          <LogIn v-else class="size-4" />
          {{ isLoggedIn ? "Dashboard" : "Login" }}
        </a>
      </Button>
    </div>
  </header>
</template>

<style scoped>
.shadow-light {
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.085);
}

.shadow-dark {
  box-shadow: inset 0 0 5px rgba(255, 255, 255, 0.141);
}
</style>