<script setup lang="ts">
import {
  Card,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";

import { Badge } from "@/components/ui/badge";

enum ProService {
  YES = 1,
  NO = 0,
}

interface ServiceProps {
  title: string;
  pro: ProService;
  description: string;
}

const serviceList: ServiceProps[] = [
  {
    title: "Live Trading Sessions",
    description:
      "Trade live with clear explanations behind entries, exits, and risk.",
    pro: 0,
  },
  {
    title: "VIP Chart Alerts",
    description:
      "High-probability setups with clean levels and trade context.",
    pro: 0,
  },
  {
    title: "Level 2 & Order Flow",
    description:
      "Read liquidity and intent using futures Level 2 data.",
    pro: 1,
  },
  {
    title: "Premium Mentorship",
    description:
      "Structured guidance to improve discipline and consistency.",
    pro: 1,
  },
];
</script>

<template>
  <section id="services" class="container py-24 sm:py-32">
    <h2 class="text-lg text-primary text-center mb-2 tracking-wider">
      Services
    </h2>

    <h2 class="text-3xl md:text-4xl text-center font-bold mb-4">
      What You Get Inside Market Sharks
    </h2>

    <h3 class="md:w-1/2 mx-auto text-xl text-center text-muted-foreground mb-8">
      Practical tools and live guidance designed to help you trade Gold with confidence.
    </h3>

    <div
      class="grid sm:grid-cols-2 lg:grid-cols-2 gap-4 w-full lg:w-[60%] mx-auto"
    >
      <div v-for="{ title, description, pro } in serviceList" :key="title">
        <Card class="bg-muted/60 dark:bg-card h-full relative">
          <CardHeader>
            <CardTitle>{{ title }}</CardTitle>
            <CardDescription>{{ description }}</CardDescription>
          </CardHeader>

          <Badge
            v-if="pro === ProService.YES"
            variant="secondary"
            class="absolute -top-2 -right-3"
          >
            PRO
          </Badge>
        </Card>
      </div>
    </div>
  </section>
</template>
