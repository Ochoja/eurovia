<script setup lang="ts">
defineProps<{
  destination: {
    name: string
    cityCount: number
    image: string
    summary: string
  }
}>()
</script>

<template>
  <article class="destination-card">
    <img
      :src="destination.image"
      :alt="destination.name"
      class="destination-card__image"
    />
    <div class="destination-card__overlay"></div>
    <div class="destination-card__content">
      <div class="destination-card__base">
        <h3>{{ destination.name }}</h3>
        <p class="destination-card__meta">{{ destination.cityCount }} Cities</p>
      </div>

      <div class="destination-card__hover">
        <h3>{{ destination.name }}</h3>
        <p class="destination-card__summary">{{ destination.summary }}</p>
      </div>
    </div>
  </article>
</template>

<style scoped>
.destination-card {
  position: relative;
  isolation: isolate;
  overflow: hidden;
  min-height: 31rem;
  border-radius: 22px;
  box-shadow: var(--shadow-md);
}

.destination-card__image,
.destination-card__overlay {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
}

.destination-card__image {
  object-fit: cover;
  transition: transform 0.35s ease;
}

.destination-card__overlay {
  background:
    linear-gradient(180deg, rgba(9, 14, 22, 0.02) 0%, rgba(9, 14, 22, 0.82) 100%),
    linear-gradient(180deg, rgba(15, 23, 42, 0) 42%, rgba(15, 23, 42, 0.65) 100%);
}

.destination-card__content {
  position: relative;
  display: flex;
  min-height: 31rem;
  flex-direction: column;
  justify-content: flex-end;
  padding: 1.35rem;
  color: var(--text-inverse);
}

.destination-card__base,
.destination-card__hover {
  transition:
    opacity 0.28s ease,
    transform 0.28s ease;
}

h3 {
  margin: 0;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-size: 1.75rem;
  font-weight: 500;
}

.destination-card__meta,
.destination-card__summary {
  margin: 0;
}

.destination-card__meta {
  margin-top: 0.2rem;
  font-size: 0.95rem;
  opacity: 0.92;
}

.destination-card__hover {
  position: absolute;
  inset: auto 1.35rem 1.35rem;
  opacity: 0;
  transform: translateY(12px);
}

.destination-card__summary {
  margin-top: 0.35rem;
  max-width: 22ch;
  font-size: 0.92rem;
  color: rgba(248, 250, 252, 0.84);
  line-height: 1.7;
}

.destination-card:hover .destination-card__image {
  transform: scale(1.04);
}

.destination-card:hover .destination-card__base {
  opacity: 0;
  transform: translateY(8px);
}

.destination-card:hover .destination-card__hover {
  opacity: 1;
  transform: translateY(0);
}
</style>
