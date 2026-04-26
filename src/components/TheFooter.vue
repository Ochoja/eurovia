<script setup lang="ts">
import { Icon } from '@iconify/vue'
import { ref } from 'vue'

const email = ref('')
</script>

<template>
  <footer class="bg-[#0c0f14] px-[clamp(1.25rem,5vw,3.25rem)] py-16">
    <div class="w-full max-w-[1280px] mx-auto grid gap-10 footer-grid">

      <!-- Brand -->
      <div>
        <p class="m-0 text-white font-[Plus_Jakarta_Sans] text-2xl font-extrabold">
          EURO<span class="text-[var(--accent)]">VIA</span>
        </p>
        <p class="mt-3 text-white/40 text-[0.82rem] leading-[1.65] max-w-[22ch]">
          Curated European journeys for the modern explorer. Discover destinations, cities,
          and culture-driven escapes.
        </p>
      </div>

      <!-- Destinations -->
      <div>
        <p class="m-0 mb-4 text-white text-[0.9rem] font-semibold">Destinations</p>
        <ul class="list-none m-0 p-0 flex flex-col gap-2.5">
          <li v-for="dest in ['Italy', 'Paris', 'Amsterdam', 'London', 'Seville']" :key="dest">
            <a href="#" class="text-white/48 text-[0.875rem] hover:text-white/90 transition-colors duration-200">{{ dest }}</a>
          </li>
        </ul>
      </div>

      <!-- Support -->
      <div>
        <p class="m-0 mb-4 text-white text-[0.9rem] font-semibold">Support</p>
        <ul class="list-none m-0 p-0 flex flex-col gap-3">
          <li>
            <a href="#" class="inline-flex items-center gap-2 text-white/48 text-[0.875rem] hover:text-white/90 transition-colors duration-200">
              <Icon icon="mdi:facebook" class="text-[1.05rem]" />
              <span>Facebook</span>
            </a>
          </li>
          <li>
            <a href="#" class="inline-flex items-center gap-2 text-white/48 text-[0.875rem] hover:text-white/90 transition-colors duration-200">
              <Icon icon="mdi:instagram" class="text-[1.05rem]" />
              <span>Instagram</span>
            </a>
          </li>
        </ul>
      </div>

      <!-- Newsletter -->
      <div>
        <p class="m-0 mb-1 text-white text-[0.9rem] font-semibold">Subscribe to our Newsletter</p>
        <p class="mt-1 mb-4 text-white/40 text-[0.8rem] leading-[1.5]">
          Sign up for updates by subscribing to our newsletter
        </p>
        <div class="flex rounded-full overflow-hidden border border-white/10 bg-white/6 max-w-[20rem]">
          <input
            v-model="email"
            type="email"
            placeholder="Enter your email"
            class="flex-1 min-w-0 bg-transparent border-0 text-white text-[0.82rem] px-4 py-[0.68rem] outline-none placeholder-white/30"
          />
          <button
            type="button"
            class="shrink-0 rounded-full bg-white text-[#0c0f14] font-bold text-[0.8rem] px-4 py-[0.62rem] m-[0.2rem] border-0 cursor-pointer"
          >
            Subscribe
          </button>
        </div>
      </div>

    </div>
  </footer>
</template>

<style scoped>
.footer-grid {
  grid-template-columns: 1fr;
}

@media (min-width: 768px) {
  .footer-grid {
    grid-template-columns: 1.4fr 1fr 1fr;
  }

  .footer-grid > :last-child {
    grid-column: 1 / -1;
  }
}

@media (min-width: 1024px) {
  .footer-grid {
    grid-template-columns: 1.6fr 1fr 1fr 1.8fr;
    align-items: start;
  }

  .footer-grid > :last-child {
    grid-column: auto;
  }
}
</style>
