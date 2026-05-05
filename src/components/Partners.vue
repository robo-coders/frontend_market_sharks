<script setup lang="ts">
import { Marquee } from "@selemondev/vue3-marquee";
import "@selemondev/vue3-marquee/dist/style.css";
import { TrendingUp } from "lucide-vue-next";

const partners: string[] = [
  "Alpin Markets",
  "Vantage Market",
  "Exness",
  "Ic Markets",
  "Bookmap",
  "Binance",
];
</script>

<template>
  <section id="sponsors" class="max-w-[75%] mx-auto pb-24 sm:pb-32">
    <h2 class="text-lg md:text-xl text-center mb-6">Trusted Partners</h2>

    <div class="mx-auto">
      <Marquee
        class="gap-[3rem]"
        :pauseOnHover="true"
        :fade="true"
        innerClassName="gap-[3rem]"
      >
        <div v-for="name in partners" :key="name">
          <div class="flex items-center text-xl md:text-2xl font-medium">
            <TrendingUp class="mr-2" stroke-width="3" />
            {{ name }}
          </div>
        </div>
      </Marquee>
    </div>
  </section>
</template>