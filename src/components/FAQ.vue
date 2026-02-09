<script setup lang="ts">
import {
  Accordion,
  AccordionContent,
  AccordionItem,
  AccordionTrigger,
} from "@/components/ui/accordion";

interface FAQProps {
  question: string;
  answer: string;
  value: string;
}

/* Common Questions */
const commonFAQs: FAQProps[] = [
  {
    question: "What is MarketSharks?",
    answer:
      "MarketSharks is a live trading community where traders learn, analyze, and trade together in real time.",
    value: "common-1",
  },
  {
    question: "Can I get a refund for membership programs?",
    answer:
      "No, refunds are not provided for mentorship programs, learning materials, or private mentoring, as all information is delivered upfront.",
    value: "common-2",
  },
  {
    question: "How do I get Market Sharks indicators access?",
    answer:
      `Market Sharks indicators are available exclusively to members. Once you join via 
      <a href="https://marketsharks.live" target="_blank" class="underline text-primary">
        marketsharks.live
      </a>, you will receive access along with live trading sessions, VIP alerts, and training resources.`,
    value: "common-3",
  },
  {
    question: "How can I get the Bookmap tool?",
    answer:
      `You can access the free version of Bookmap using this link:
      <a href="https://bookmap.com/members/aff/go/wnjcdag" target="_blank" class="underline text-primary">
        Bookmap Free Version
      </a>.`,
    value: "common-4",
  },
  {
    question: "How can I open a zero-spread account for scalping?",
    answer:
      `Sign up and choose the <strong>Zero (MT5)</strong> account type using this link:
      <a href="https://one.exnesstrack.org/boarding/sign-up/a/o0d3ngiyp5" target="_blank" class="underline text-primary">
        Open Zero Spread Account
      </a>.`,
    value: "common-5",
  },
  {
    question: "How can I join the live trading session?",
    answer:
      `To join live trading sessions at Market Sharks, sign up on 
      <a href="https://marketsharks.live" target="_blank" class="underline text-primary">
        marketsharks.live
      </a>
      and select a membership plan starting from $25/month. This gives you access to Zoom-based sessions, private WhatsApp groups, and VIP alerts.`,
    value: "common-6",
  },
  {
    question: "What markets do you focus on?",
    answer:
      "We primarily trade Gold (XAUUSD) during the New York session, when market volatility is highest.",
    value: "common-7",
  },
  {
    question: "Do you use Level 2 data in your live trading sessions?",
    answer:
      "Yes, we use live Gold futures Level 2 data from COMEX to improve trade accuracy and execution.",
    value: "common-8",
  },
  {
    question: "What experience level is required to join the VIP membership?",
    answer:
      "Our VIP membership is suitable for all experience levels, from beginners to advanced traders.",
    value: "common-9",
  },
];

/* Level 2 Questions */
const level2FAQs: FAQProps[] = [
  {
    question: "How can I get a discount code for Bookmap?",
    answer:
      `Buy through this link:
      <a href="https://bookmap.com/members/aff/go/wnjcdag?i=194" target="_blank" class="underline text-primary">
        Bookmap Official Link
      </a><br/>
      Use discount code <strong>MS20</strong> to get 20% off your first purchase.`,
    value: "level2-1",
  },
  {
    question: "Which plan includes Footprint and DOM Pro?",
    answer:
      `Open the link below and choose the <strong>Global+</strong> plan (monthly or quarterly):
      <a href="https://bookmap.com/members/aff/go/wnjcdag?i=194" target="_blank" class="underline text-primary">
        View Bookmap Plans
      </a>.`,
    value: "level2-2",
  },
  {
    question: "Which data feed is required for Gold futures (GC live order flow)?",
    answer:
      `After subscribing to your tool plan, log in to your account at
      <a href="https://bookmap.com" target="_blank" class="underline text-primary">
        bookmap.com
      </a>.<br/>
      Go to your dashboard → Add Subscription → Select <strong>Bookmap Data (COMEX)</strong>.`,
    value: "level2-3",
  },
];
</script>

<template>
  <section id="faq" class="container md:w-[700px] py-24 sm:py-32">

    <!-- Common Questions -->
    <div class="text-center mb-10">
      <h2 class="text-lg text-primary mb-2 tracking-wider">FAQs</h2>
      <h2 class="text-3xl md:text-4xl font-bold">Common Questions</h2>
    </div>

    <Accordion type="single" collapsible>
      <AccordionItem
        v-for="{ question, answer, value } in commonFAQs"
        :key="value"
        :value="value"
      >
        <AccordionTrigger class="text-left">
          {{ question }}
        </AccordionTrigger>
        <AccordionContent v-html="answer" />
      </AccordionItem>
    </Accordion>

    <!-- Level 2 Questions -->
    <div class="text-center mt-20 mb-10">
      <h2 class="text-3xl md:text-4xl font-bold">
        Level 2 Questions
      </h2>
    </div>

    <Accordion type="single" collapsible>
      <AccordionItem
        v-for="{ question, answer, value } in level2FAQs"
        :key="value"
        :value="value"
      >
        <AccordionTrigger class="text-left">
          {{ question }}
        </AccordionTrigger>
        <AccordionContent v-html="answer" />
      </AccordionItem>
    </Accordion>

    <h3 class="font-medium mt-10 text-center">
      Still have questions?
      <a href="#" class="underline text-primary">Contact us</a>
    </h3>

  </section>
</template>
