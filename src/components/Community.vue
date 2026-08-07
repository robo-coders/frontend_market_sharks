<script setup lang="ts">
import {
  Card,
  CardHeader,
  CardTitle,
  CardFooter,
  CardContent,
} from "@/components/ui/card";

import { Button } from "@/components/ui/button";

import WhatsAppIcon from "@/icons/WhatsAppIcon.vue";
</script>

<template>
  <section id="community" class="py-12">
    <hr />

    <div class="container py-20 sm:py-20">
      <div class="lg:w-[60%] mx-auto">
        <Card
          class="bg-background shadow-none text-center flex flex-col items-center justify-center border-0"
        >
          <CardHeader>
            <CardTitle class="text-4xl md:text-5xl font-bold">
              <Component
                :is="WhatsAppIcon"
                class="w-20 h-20 m-auto mb-4 text-green-500"
              />

              Join our free trading
              <span
                class="text-transparent bg-gradient-to-r from-[#D247BF] to-primary bg-clip-text"
              >
                community
              </span>
            </CardTitle>
          </CardHeader>

          <CardContent class="lg:w-[80%] text-xl text-muted-foreground">
            Get live market updates, trade ideas, and real-time insights directly
            on WhatsApp. Learn and grow with smart traders — completely free.
          </CardContent>

          <CardFooter>
            <Button as-child>
              <a
                href="https://chat.whatsapp.com/FzWkNru4RUmL3ar5FRo24a"
                target="_blank"
                rel="noopener noreferrer"
              >
                Join WhatsApp Community
              </a>
            </Button>
          </CardFooter>
        </Card>
      </div>
    </div>

    <hr />
  </section>
</template>
