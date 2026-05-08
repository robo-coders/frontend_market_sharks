<script setup lang="ts">
import Separator from "./ui/separator/Separator.vue";
</script>

<template>
  <footer id="footer" class="container py-24 pb-16 sm:py-32 sm:pb-24">
    <div class="p-10 bg-muted/50 dark:bg-card border rounded-2xl">
      <div class="grid grid-cols-2 md:grid-cols-4 xl:grid-cols-6 gap-x-12 gap-y-8">

        <div class="col-span-full xl:col-span-2">
          <a href="/" class="flex font-bold items-center">
            <img src="/final.png" alt="Market Sharks" class="w-9 h-9 mr-2" />
            <h3 class="text-2xl">Market Sharks</h3>
          </a>
        </div>

        <div class="flex flex-col gap-2">
          <h3 class="font-bold text-lg">Quick Links</h3>
          <div><a href="#community" class="opacity-60 hover:opacity-100">Community</a></div>
          <div><a href="#pricing" class="opacity-60 hover:opacity-100">Pricing</a></div>
          <div><a href="#contact" class="opacity-60 hover:opacity-100">Contact</a></div>
          <div><a href="#faq" class="opacity-60 hover:opacity-100">FAQ</a></div>
        </div>

        <div class="flex flex-col gap-2">
          <h3 class="font-bold text-lg">Community</h3>
          <div><a target="_blank" href="https://chat.whatsapp.com/GIuCPbJ7V9W3wtYsQQz1x4" class="opacity-60 hover:opacity-100">WhatsApp Group</a></div>
          <div><a target="_blank" href="https://t.me/marketsharkslive" class="opacity-60 hover:opacity-100">Telegram Channel</a></div>
        </div>

        <div class="flex flex-col gap-2">
          <h3 class="font-bold text-lg">Help</h3>
          <div><a href="#contact" class="opacity-60 hover:opacity-100">Contact Us</a></div>
          <div><a href="#faq" class="opacity-60 hover:opacity-100">FAQ</a></div>
          <div><a href="/login" class="opacity-60 hover:opacity-100">Member Login</a></div>
        </div>

        <div class="flex flex-col gap-2">
          <h3 class="font-bold text-lg">Socials</h3>
          <div><a target="_blank" href="https://www.facebook.com/share/1JCie4aR12/?mibextid=wwXIfr" class="opacity-60 hover:opacity-100">Instagram</a></div>
          <div><a target="_blank" href="https://www.instagram.com/marketsharks.live?igsh=ZGF2YWl0OTdzd2Nj&utm_source=qr" class="opacity-60 hover:opacity-100">Facebook</a></div>
          <div><a target="_blank" href="https://youtube.com/@marketsharkss?si=cxx0w5sQC9F_AXxl" class="opacity-60 hover:opacity-100">YouTube</a></div>
          <div><a target="_blank" href="https://www.tiktok.com/@marketsharks.live?_r=1&_t=ZS-967AwUFEDu4" class="opacity-60 hover:opacity-100">Tiktok</a></div>
        </div>

      </div>

      <Separator class="my-6" />

      <!-- Disclaimer -->
      <div class="mb-6 text-xs text-muted-foreground leading-relaxed space-y-3">
        <p>
          The information presented on the Market Sharks website is solely for informational and
          educational purposes. It is not, nor is it intended to be, trading or investment advice
          or a recommendation that any securities, strategies, or trading methodologies described
          by Market Sharks are suitable for any particular investor. Furthermore, the information
          and opinions presented by Market Sharks are not an attempt to induce any particular
          trading behavior, investment, or strategy.
        </p>
        <p>
          Market Sharks is not a broker dealer, and its representatives are not securities brokers,
          licensed financial advisors, or registered investment advisors. Trading securities
          including equities, options, and futures involves risk of loss which can be substantial.
        </p>
        <p>
          Results of transactions described on the Market Sharks website and social media, and any
          presentation therein, may vary due to differences in market conditions, execution prices,
          the time of trade executions, fees for brokerage services, and other factors. Past
          performance is not necessarily indicative of future results. The information and any
          opinions contained on this website have been obtained from sources that Market Sharks
          considers reliable, but Market Sharks does not represent that such information and
          opinions are accurate or complete, and thus should not be relied upon as such.
        </p>
      </div>

      <Separator class="my-6" />

      <section>
        <h3 class="">
          &copy; 2026 Designed and developed by
          <a
            target="_blank"
            href="https://www.robocoders.dev/"
            class="text-primary transition-all border-primary hover:border-b-2"
          >
            Robo Coders
          </a>
        </h3>
      </section>
    </div>
  </footer>
</template>