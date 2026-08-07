<script setup lang="ts">
import { Marquee } from "@selemondev/vue3-marquee";
import "@selemondev/vue3-marquee/dist/style.css";
import { TrendingUp } from "lucide-vue-next";

const partners = [
  { name: "Alpin Markets",  href: "https://my.alpinmarkets.com/links/go/1" },
  { name: "Vantage Market", href: "https://www.vantagemarkets.com/open-live-account/?affid=NzM2NDMwOA==&invitecode=rcva4Nwa" },
  { name: "Exness",         href: "https://one.exnessonelink.com/boarding/sign-up/a/o0d3ngiyp5" },
  { name: "IC Markets",     href: "https://icmarkets.com/?camp=89447" },
  { name: "Bookmap",        href: "https://bookmap.com/members/aff/go/wnjcdag?i=194" },
  { name: "Binance",        href: "https://www.binance.com/register?ref=41848978" },
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
        <a
          v-for="partner in partners"
          :key="partner.name"
          :href="partner.href"
          target="_blank"
          rel="noopener noreferrer"
          class="flex items-center text-xl md:text-2xl font-medium transition-opacity duration-200 hover:opacity-60 no-underline"
        >
          <TrendingUp class="mr-2" stroke-width="3" />
          {{ partner.name }}
        </a>
      </Marquee>
    </div>
  </section>
</template>