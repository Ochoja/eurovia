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
  <article class="city-card" :class="`city-card--${city.layout}`">
    <img :src="city.image" :alt="city.name" class="city-card__image" />
    <div class="city-card__overlay"></div>

    <div class="city-card__rating">
      <Icon icon="solar:star-bold" />
      <span>{{ city.rating.toFixed(1) }}</span>
    </div>

    <div class="city-card__content">
      <div class="city-card__default">
        <h3>{{ city.name }}</h3>
        <p>{{ city.description }}</p>
      </div>

      <div class="city-card__preview">
        <h3>{{ city.previewTitle }}</h3>
        <p>{{ city.previewDescription }}</p>
        <button type="button">Explore</button>
      </div>
    </div>
  </article>
</template>

<style scoped>
.city-card {
  position: relative;
  isolation: isolate;
  overflow: hidden;
  min-height: 16rem;
  border-radius: 18px;
  box-shadow: var(--shadow-md);
}

.city-card--tall {
  min-height: 33.25rem;
}

.city-card__image,
.city-card__overlay {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
}

.city-card__image {
  object-fit: cover;
  transition: transform 0.35s ease;
}

.city-card__overlay {
  background:
    linear-gradient(180deg, rgba(10, 14, 21, 0.08) 0%, rgba(10, 14, 21, 0.66) 100%),
    linear-gradient(180deg, rgba(10, 14, 21, 0.02) 40%, rgba(10, 14, 21, 0.78) 100%);
}

.city-card__rating {
  position: absolute;
  top: 0.8rem;
  right: 0.8rem;
  display: inline-flex;
  align-items: center;
  gap: 0.2rem;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 999px;
  background: rgba(31, 36, 45, 0.48);
  backdrop-filter: blur(10px);
  color: var(--text-inverse);
  padding: 0.22rem 0.45rem;
  font-size: 0.7rem;
}

.city-card__rating :deep(svg) {
  font-size: 0.82rem;
  color: #d6f958;
}

.city-card__content {
  position: relative;
  display: flex;
  min-height: 100%;
  align-items: flex-end;
  padding: 0.85rem;
  color: var(--text-inverse);
}

.city-card__default,
.city-card__preview {
  width: 100%;
}

.city-card__default {
  border: 1px solid rgba(255, 255, 255, 0.24);
  border-radius: 12px;
  background: rgba(18, 18, 18, 0.24);
  backdrop-filter: blur(14px);
  padding: 0.5rem 0.65rem;
  width: fit-content;
  max-width: min(100%, 15rem);
  transition:
    opacity 0.3s ease,
    transform 0.3s ease;
}

.city-card__default h3,
.city-card__preview h3 {
  margin: 0;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-weight: 500;
}

.city-card__default h3 {
  font-size: 1.6rem;
}

.city-card__default p {
  margin: 0.08rem 0 0;
  color: rgba(248, 250, 252, 0.76);
  font-size: 0.62rem;
}

.city-card__preview {
  position: absolute;
  inset: 0;
  display: flex;
  min-height: 100%;
  flex-direction: column;
  justify-content: center;
  gap: 1rem;
  background:
    linear-gradient(180deg, rgba(10, 14, 21, 0.26) 0%, rgba(10, 14, 21, 0.72) 100%),
    rgba(10, 14, 21, 0.16);
  padding: 2rem 1rem;
  text-align: center;
  opacity: 0;
  transition:
    opacity 0.3s ease,
    transform 0.3s ease;
  transform: translateY(14px);
}

.city-card__preview h3 {
  font-size: clamp(2rem, 5vw, 3.3rem);
  font-weight: 500;
}

.city-card__preview p {
  margin: 0 auto;
  max-width: 16ch;
  color: rgba(248, 250, 252, 0.92);
  font-size: 0.95rem;
  line-height: 1.7;
}

.city-card__preview button {
  border: 0;
  border-radius: 999px;
  background: #fff;
  color: #1a1f27;
  cursor: pointer;
  padding: 0.95rem 1.2rem;
  font-size: 0.95rem;
  font-weight: 600;
  width: min(100%, 17.5rem);
  margin-inline: auto;
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

@media (max-width: 767px) {
  .city-card {
    min-height: 18rem;
  }

  .city-card--tall {
    min-height: 18rem;
  }

  .city-card__default {
    opacity: 0;
    transform: translateY(8px);
  }

  .city-card__preview {
    opacity: 1;
    transform: none;
  }

  .city-card__preview h3 {
    font-size: 2.4rem;
  }

  .city-card__preview p {
    font-size: 0.9rem;
    line-height: 1.6;
  }
}
</style>
