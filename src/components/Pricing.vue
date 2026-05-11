<script setup lang="ts">
import { Button } from "@/components/ui/button";
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Check } from "lucide-vue-next";

enum PopularPlan {
  NO = 0,
  YES = 1,
}

interface PlanProps {
  title: string;
  popular: PopularPlan;
  price: number;
  originalPrice?: number;
  discountLabel?: string;
  period: "month" | "quarter" | "hour";
  description: string;
  buttonText: string;
  benefitList: string[];
}

const plans: PlanProps[] = [
  {
    title: "Basic",
    popular: PopularPlan.NO,
    price: 25,
    period: "month",
    description:
      "Perfect for beginners seeking guided trades, live sessions, and essential market insights.",
    buttonText: "Get started",
    benefitList: [
      "Live 90-minute trading session",
      "Exclusive chart alerts",
      "Level 2 data insights",
      "Access to premium learning content",
    ],
  },
  {
    title: "Premium",
    popular: PopularPlan.YES,
    price: 60,
    originalPrice: 75,
    discountLabel: "20% OFF",
    period: "quarter",
    description:
      "Ideal for active traders who want full-day live trading, premium alerts, and deeper market analysis.",
    buttonText: "Get started",
    benefitList: [
      "Extended live trading sessions",
      "Premium chart alerts",
      "Advanced Level 2 data insights",
      "Full access to premium learning content",
    ],
  },
  {
    title: "Enterprise",
    popular: PopularPlan.NO,
    price: 100,
    period: "hour",
    description:
      "Built for serious traders who need complete access, advanced tools, and priority-level support.",
    buttonText: "Get started",
    benefitList: [
      "1:1 trading session (60 minutes)",
      "Strategy development",
      "Risk management planning",
      "Advanced Bookmap training",
      "Order flow concepts",
    ],
  },
];

const checkoutHref = (title: string) =>
  `/checkout?plan=${encodeURIComponent(title.toLowerCase())}`;
</script>

<template>
  <section id="pricing" class="container py-24 sm:py-32">
    <h2 class="text-lg text-primary text-center mb-2 tracking-wider">
      Pricing
    </h2>

    <h2 class="text-3xl md:text-4xl text-center font-bold mb-4">
      Get unlimited access
    </h2>

    <h3 class="md:w-1/2 mx-auto text-xl text-center text-muted-foreground pb-14">
      High-probability trade setups, live market coverage, and a community built
      to help you trade with confidence.
    </h3>

    <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-8 lg:gap-4">
      <Card
        v-for="{
          title,
          popular,
          price,
          originalPrice,
          discountLabel,
          period,
          description,
          buttonText,
          benefitList,
        } in plans"
        :key="title"
        class="relative"
        :class="{
          'drop-shadow-xl shadow-black/10 dark:shadow-white/10 border-[1.5px] border-primary lg:scale-[1.1] lg:z-10':
            popular === PopularPlan.YES,
        }"
      >
        <!-- Most Popular badge -->
        <span
          v-if="popular === PopularPlan.YES"
          class="absolute -top-4 left-1/2 -translate-x-1/2 bg-primary text-primary-foreground text-xs font-semibold px-4 py-1 rounded-full"
        >
          Most Popular
        </span>

        <!-- Discount badge -->
        <span
          v-if="discountLabel"
          class="absolute top-4 right-4 bg-green-500 text-white text-[11px] font-bold px-2.5 py-1 rounded-full tracking-wide"
        >
          {{ discountLabel }}
        </span>

        <CardHeader>
          <CardTitle class="pb-2">{{ title }}</CardTitle>
          <CardDescription class="pb-4">{{ description }}</CardDescription>

          <div class="flex items-end gap-2">
            <div>
              <span class="text-3xl font-bold">${{ price }}</span>
              <span class="text-muted-foreground">
                {{ period === "hour" ? "/hourly" : `/${period}` }}
              </span>
            </div>
            <!-- Struck-through original price -->
            <span
              v-if="originalPrice"
              class="text-muted-foreground line-through text-sm mb-1"
            >
              ${{ originalPrice }}
            </span>
          </div>
        </CardHeader>

        <CardContent class="flex">
          <div class="space-y-4">
            <span v-for="benefit in benefitList" :key="benefit" class="flex">
              <Check class="text-primary mr-2" />
              <h3>{{ benefit }}</h3>
            </span>
          </div>
        </CardContent>

        <CardFooter>
          <Button
            :variant="popular === PopularPlan.NO ? 'secondary' : 'default'"
            class="w-full"
            as-child
          >
            <a :href="checkoutHref(title)">
              {{ buttonText }}
            </a>
          </Button>
        </CardFooter>
      </Card>
    </div>
  </section>
</template>