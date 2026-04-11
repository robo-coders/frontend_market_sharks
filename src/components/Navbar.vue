<script lang="ts" setup>
import { ref } from "vue";

import { useColorMode } from "@vueuse/core";
const mode = useColorMode();
mode.value = "dark";

import {
  NavigationMenu,
  NavigationMenuContent,
  NavigationMenuItem,
  NavigationMenuLink,
  NavigationMenuList,
  NavigationMenuTrigger,
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

import { ChevronsDown, Menu, LogIn, LayoutDashboard } from "lucide-vue-next";
import ToggleTheme from "./ToggleTheme.vue";

interface RouteProps {
  href: string;
  label: string;
}

interface FeatureProps {
  title: string;
  description: string;
}

const routeList: RouteProps[] = [
  { href: "#testimonials", label: "Testimonials" },
  { href: "#pricing", label: "Pricing" },
  { href: "#contact", label: "Contact" },
  { href: "#faq", label: "FAQ" },
];

const featureList: FeatureProps[] = [
  {
    title: "Showcase Your Value",
    description: "Highlight how your product solves user problems.",
  },
  {
    title: "Build Trust",
    description: "Leverages social proof elements to establish trust and credibility.",
  },
  {
    title: "Capture Leads",
    description: "Make your lead capture form visually appealing and strategically.",
  },
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
      <img src="/test.svg" alt="Market Sharks" class="w-8 h-8" />
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
                  <ChevronsDown
                    class="bg-gradient-to-tr from-primary/70 via-primary to-primary/70 rounded-lg size-9 mr-2 border text-white"
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

              <!-- Mobile Button -->
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
          <NavigationMenuTrigger class="bg-card text-base">
            Features
          </NavigationMenuTrigger>
          <NavigationMenuContent>
            <div class="grid w-[600px] grid-cols-2 gap-5 p-4">
              <img
                src="https://www.radix-vue.com/logo.svg"
                alt="Preview"
                class="h-full w-full rounded-md object-cover"
              />
              <ul class="flex flex-col gap-2">
                <li
                  v-for="{ title, description } in featureList"
                  :key="title"
                  class="rounded-md p-3 text-sm hover:bg-muted"
                >
                  <p class="mb-1 font-semibold leading-none">
                    {{ title }}
                  </p>
                  <p class="text-muted-foreground line-clamp-2">
                    {{ description }}
                  </p>
                </li>
              </ul>
            </div>
          </NavigationMenuContent>
        </NavigationMenuItem>

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

      <!-- Desktop Button -->
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
