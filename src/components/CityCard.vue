<script setup lang="ts">
import { Icon } from '@iconify/vue'

defineProps<{
  city: {
    name: string
    country: string
    image: string
    rating: number
    description: string
    layout: string
    previewTitle: string
    previewDescription: string
  }
}>()
</script>

<template>
  <article
    class="city-card relative isolate overflow-hidden rounded-[18px] shadow-[var(--shadow-md)] min-h-64"
  >
    <img
      :src="city.image"
      :alt="city.name"
      class="city-card__image absolute inset-0 w-full h-full object-cover transition-transform duration-[350ms] ease-out"
    />
    <div
      class="absolute inset-0 w-full h-full"
      style="
        background:
          linear-gradient(180deg, rgba(10, 14, 21, 0.08) 0%, rgba(10, 14, 21, 0.45) 100%),
          linear-gradient(180deg, rgba(10, 14, 21, 0.02) 40%, rgba(10, 14, 21, 0.45) 100%);
      "
    ></div>

    <!-- Rating badge -->
    <div
      class="absolute top-3 right-3 inline-flex items-center gap-1 rounded-full border border-white/30 bg-[rgba(31,36,45,0.48)] backdrop-blur-[10px] text-white px-2 py-0.5 text-[0.7rem]"
    >
      <Icon icon="solar:star-bold" class="text-[#d6f958] text-[0.82rem]" />
      <span>{{ city.rating.toFixed(1) }}</span>
    </div>

    <!-- Default info label -->
    <div class="city-card__content absolute inset-0 flex items-end p-3.5 text-white">
      <div
        class="city-card__default border border-white/24 rounded-md bg-[rgba(18,18,18,0.1)] backdrop-blur-sm px-2.5 py-2 w-fit max-w-[15rem] transition-all duration-300"
      >
        <h3 class="m-0 font-[Plus_Jakarta_Sans] font-medium text-[1.3rem] leading-tight">
          {{ city.name }}
        </h3>
        <p class="m-0 mt-0.5 text-white/76 text-xs">{{ city.description }}</p>
      </div>
    </div>

    <!-- Hover preview -->
    <div
      class="city-card__preview absolute inset-0 flex flex-col items-center justify-center gap-3 text-center px-2 py-6 opacity-0 transition-all duration-300 translate-y-3"
      style="
        background: linear-gradient(180deg, rgba(10, 14, 21, 0.26) 0%, rgba(10, 14, 21, 0.78) 100%);
      "
    >
      <h3
        class="m-0 font-[Plus_Jakarta_Sans] font-semibold text-white"
        style="font-size: clamp(1.1rem, 1.8vw, 1.55rem); line-height: 1.15"
      >
        {{ city.previewTitle }}
      </h3>
      <p class="m-0 max-w-[95%] text-white/92 text-sm leading-[1.6]">
        {{ city.previewDescription }}
      </p>
      <button
        type="button"
        class="rounded-full bg-white text-[#1a1f27] font-semibold px-5 py-2.5 text-[0.88rem] border-0 cursor-pointer w-[min(100%,14rem)]"
      >
        Explore
      </button>
    </div>
  </article>
</template>

<style scoped>
/* Grid context: cards must fill their cell exactly, no min-height fighting the grid */
@media (min-width: 960px) {
  .city-card {
    min-height: 0 !important;
    height: 100%;
  }
}

.city-card:hover .city-card__image {
  transform: scale(1.05);
}

.city-card:hover .city-card__default {
  opacity: 0;
  transform: translateY(8px);
}

.city-card:hover .city-card__preview {
  opacity: 1;
  transform: translateY(0);
}
</style>
