<script setup lang="ts">
import { Icon } from '@iconify/vue'
import L from 'leaflet'
import {
  computed,
  nextTick,
  onBeforeUnmount,
  onMounted,
  ref,
  watch,
} from 'vue'
import CityCard from '@/components/CityCard.vue'
import DestinationCard from '@/components/DestinationCard.vue'
import TheFooter from '@/components/TheFooter.vue'
import TheNavigation from '@/components/TheNavigation.vue'
import cityData from '@/data/cities.json'
import destinationData from '@/data/destinations.json'
import searchData from '@/data/search.json'

type SearchCity = {
  name: string
  coordinates: [number, number]
  interest: string
}

type SearchCountry = {
  country: string
  cities: SearchCity[]
}

const sections = [
  { id: 'discover', label: 'Discover' },
  { id: 'destinations', label: 'Destinations' },
  { id: 'cities', label: 'Cities' },
  { id: 'categories', label: 'Categories' },
  { id: 'map', label: 'Map' },
]

const interests = [
  'Arts & Culture',
  'Adventure',
  'Wellness',
  'Food',
  'Nature',
  'Luxury',
]

const categories = [
  { label: 'Adventure', icon: 'solar:mountains-linear' },
  { label: 'Wellness', icon: 'solar:heart-angle-linear' },
  { label: 'Culture', icon: 'solar:buildings-2-linear' },
  { label: 'Food', icon: 'solar:chef-hat-linear' },
  { label: 'Nature', icon: 'solar:leaf-linear' },
  { label: 'Luxury', icon: 'solar:crown-linear' },
]

const categoryThemes: Record<string, string> = {
  Adventure: 'category-panel--adventure',
  Wellness: 'category-panel--wellness',
  Culture: 'category-panel--culture',
  Food: 'category-panel--food',
  Nature: 'category-panel--nature',
  Luxury: 'category-panel--luxury',
}

const searchOptions = searchData as SearchCountry[]
const selectedCountry = ref(searchOptions[0]?.country ?? 'France')
const selectedCity = ref(searchOptions[0]?.cities[0]?.name ?? 'Paris')
const selectedInterest = ref('Arts & Culture')
const selectedCategory = ref('Adventure')

const mapContainer = ref<HTMLDivElement | null>(null)
let map: L.Map | null = null
let tileLayer: L.TileLayer | null = null
let markerLayer: L.LayerGroup | null = null

const availableCities = computed(
  () =>
    searchOptions.find((entry) => entry.country === selectedCountry.value)?.cities ?? [],
)

const filteredMapCities = computed(() => {
  const matchesCountry =
    searchOptions.find((entry) => entry.country === selectedCountry.value)?.cities ?? []

  return matchesCountry.filter(
    (city) => !selectedInterest.value || city.interest === selectedInterest.value,
  )
})

const activeMapCity = computed(
  () =>
    availableCities.value.find((city) => city.name === selectedCity.value) ??
    filteredMapCities.value[0] ??
    availableCities.value[0],
)

const categoryPanelClass = computed(
  () => categoryThemes[selectedCategory.value] ?? categoryThemes.Adventure,
)

const syncSelectedCity = () => {
  if (!availableCities.value.some((city) => city.name === selectedCity.value)) {
    selectedCity.value = availableCities.value[0]?.name ?? ''
  }
}

const createLeafletMap = () => {
  if (!mapContainer.value || map) return

  map = L.map(mapContainer.value, {
    zoomControl: false,
    attributionControl: false,
    scrollWheelZoom: true,
  }).setView([48.85, 8.5], 4)

  tileLayer = L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
    subdomains: 'abcd',
    maxZoom: 19,
  }).addTo(map)

  markerLayer = L.layerGroup().addTo(map)
  renderMarkers()
}

const renderMarkers = () => {
  if (!map || !markerLayer) return

  markerLayer.clearLayers()

  const targetCities = filteredMapCities.value.length
    ? filteredMapCities.value
    : availableCities.value

  const bounds: L.LatLngExpression[] = []

  targetCities.forEach((city) => {
    bounds.push(city.coordinates)

    const isActive = city.name === activeMapCity.value?.name
    const marker = L.circleMarker(city.coordinates, {
      radius: isActive ? 9 : 7,
      weight: 2,
      color: isActive ? '#1f242d' : '#ffffff',
      fillColor: isActive ? '#6dd645' : '#1f242d',
      fillOpacity: 1,
    })

    marker.bindPopup(
      `<strong>${city.name}</strong><br />${selectedCountry.value}<br />${city.interest}`,
      {
        closeButton: false,
        offset: [0, -8],
      },
    )

    marker.on('mouseover', () => marker.openPopup())
    markerLayer?.addLayer(marker)
  })

  const focusCity = activeMapCity.value
  if (focusCity) {
    map.flyTo(focusCity.coordinates, 5, { duration: 0.7 })
    nextTick(() => {
      markerLayer?.eachLayer((layer) => {
        if (
          layer instanceof L.CircleMarker &&
          layer.getLatLng().lat === focusCity.coordinates[0] &&
          layer.getLatLng().lng === focusCity.coordinates[1]
        ) {
          layer.openPopup()
        }
      })
    })
  } else if (bounds.length) {
    map.fitBounds(bounds, { padding: [32, 32] })
  }
}

const runSearch = () => {
  syncSelectedCity()
  renderMarkers()

  const mapSection = document.getElementById('map')
  if (!mapSection) return

  const top = mapSection.getBoundingClientRect().top + window.scrollY - 88
  window.scrollTo({ top, behavior: 'smooth' })
}

watch(selectedCountry, () => {
  syncSelectedCity()
  renderMarkers()
})

watch([selectedCity, selectedInterest], () => {
  renderMarkers()
})

onMounted(() => {
  createLeafletMap()
})

onBeforeUnmount(() => {
  markerLayer?.clearLayers()
  tileLayer?.remove()
  map?.remove()
})
</script>

<template>
  <div class="home-page">
    <section id="discover" class="hero page-pad">
      <div class="hero__backdrop"></div>
      <TheNavigation :sections="sections" />

      <div class="hero__content page-shell">
        <div class="hero__copy">
          <h1>Discover Europe, <span>Your Way</span></h1>
        </div>

        <div class="hero__search">
          <label class="search-field">
            <div class="search-field__lead">
              <Icon icon="solar:map-point-linear" />
              <div>
                <span>Country</span>
                <strong>{{ selectedCountry || 'Where to?' }}</strong>
              </div>
            </div>

            <select v-model="selectedCountry" aria-label="Select country">
              <option v-for="item in searchOptions" :key="item.country" :value="item.country">
                {{ item.country }}
              </option>
            </select>
            <Icon icon="solar:alt-arrow-down-linear" />
          </label>

          <label class="search-field">
            <div class="search-field__lead">
              <Icon icon="solar:buildings-2-linear" />
              <div>
                <span>City</span>
                <strong>{{ selectedCity || 'Add Cities' }}</strong>
              </div>
            </div>

            <select v-model="selectedCity" aria-label="Select city">
              <option v-for="city in availableCities" :key="city.name" :value="city.name">
                {{ city.name }}
              </option>
            </select>
            <Icon icon="solar:alt-arrow-down-linear" />
          </label>

          <label class="search-field">
            <div class="search-field__lead">
              <Icon icon="solar:camera-linear" />
              <div>
                <span>Interests</span>
                <strong>{{ selectedInterest }}</strong>
              </div>
            </div>

            <select v-model="selectedInterest" aria-label="Select interest">
              <option v-for="interest in interests" :key="interest" :value="interest">
                {{ interest }}
              </option>
            </select>
            <Icon icon="solar:alt-arrow-down-linear" />
          </label>

          <button type="button" class="hero__search-btn" @click="runSearch">
            <Icon icon="solar:magnifer-linear" />
            Search
          </button>
        </div>
      </div>
    </section>

    <section id="destinations" class="section-block page-pad section-surface">
      <div class="page-shell">
        <header class="section-head">
          <h2 class="section-title">Featured Destinations</h2>
          <p class="section-copy">Hand-picked curations for the modern explorer</p>
        </header>

        <div class="destinations-grid">
          <DestinationCard
            v-for="destination in destinationData"
            :key="destination.name"
            :destination="destination"
          />
        </div>
      </div>
    </section>

    <section id="cities" class="section-block page-pad section-surface">
      <div class="page-shell">
        <header class="section-head section-head--centered">
          <h2 class="section-title">Top Cities</h2>
          <p class="section-copy">
            Urban escapes that blend culture and contemporary lives
          </p>
        </header>

        <div class="cities-grid">
          <CityCard v-for="city in cityData" :key="city.name" :city="city" />
        </div>
      </div>
    </section>

    <section id="categories" class="section-block page-pad section-surface">
      <div class="page-shell">
        <div class="category-panel" :class="categoryPanelClass">
          <img
            src="https://ik.imagekit.io/Ochoja01/euvoria/Categories.png"
            alt="Travel categories"
            class="category-panel__image"
          />
          <div class="category-panel__overlay"></div>

          <div class="category-panel__content">
            <span class="category-pill">Categories</span>
            <h2 class="section-title section-title--light">Travel By Interests</h2>

            <div class="category-grid">
              <button
                v-for="category in categories"
                :key="category.label"
                type="button"
                class="category-chip"
                :class="{ 'category-chip--active': selectedCategory === category.label }"
                @click="selectedCategory = category.label"
              >
                <Icon :icon="category.icon" />
                <span>{{ category.label }}</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section id="map" class="section-block page-pad section-surface">
      <div class="page-shell map-section">
        <div class="map-copy">
          <h2 class="section-title">
            Map Your Journey
            <span>Across Europe</span>
          </h2>
          <p class="section-copy">
            Interact with our dynamic map to discover hidden gems tailored to your
            profile.
          </p>

          <div class="map-feature-list">
            <div class="map-feature">
              <Icon icon="solar:map-point-rotate-linear" />
              <div>
                <strong>Interactive Pins</strong>
                <p>Hover over cities for quick insights and available experiences.</p>
              </div>
            </div>

            <div class="map-feature">
              <Icon icon="solar:routing-2-linear" />
              <div>
                <strong>Custom Routes</strong>
                <p>Country and city search updates the map in real time.</p>
              </div>
            </div>
          </div>
        </div>

        <div class="map-card">
          <div ref="mapContainer" class="leaflet-map"></div>
        </div>
      </div>
    </section>

    <TheFooter />
  </div>
</template>

<style scoped>
.home-page {
  overflow-x: clip;
  background: #fafafa;
}

.hero {
  position: relative;
  min-height: 46rem;
  background:
    linear-gradient(180deg, rgba(7, 10, 15, 0.3) 0%, rgba(7, 10, 15, 0.48) 100%),
    url('https://ik.imagekit.io/Ochoja01/euvoria/Hero.png') center/cover no-repeat;
  padding-bottom: 3.5rem;
}

.hero__backdrop {
  position: absolute;
  inset: 0;
  background:
    radial-gradient(circle at top left, rgba(255, 255, 255, 0.14), transparent 30%),
    linear-gradient(180deg, rgba(7, 10, 15, 0.08), rgba(7, 10, 15, 0.44));
  pointer-events: none;
}

.hero__content {
  position: relative;
  display: grid;
  justify-items: center;
  min-height: calc(46rem - 5rem);
  padding-top: clamp(8.5rem, 17vh, 12rem);
}

.hero__copy {
  max-width: 56rem;
  text-align: center;
}

.hero__copy h1 {
  margin: 0;
  color: #fff;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-size: clamp(2.9rem, 7vw, 4.85rem);
  font-weight: 500;
  line-height: 1.02;
}

.hero__copy span,
.map-copy span {
  color: var(--accent);
}

.hero__search {
  display: grid;
  width: min(100%, 51.25rem);
  gap: 0;
  margin-top: 1.4rem;
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.14);
  backdrop-filter: blur(22px);
  box-shadow: 0 20px 45px rgba(17, 24, 39, 0.2);
  overflow: hidden;
}

.search-field {
  position: relative;
  display: grid;
  grid-template-columns: 1fr auto;
  align-items: center;
  color: rgba(248, 250, 252, 0.94);
  min-height: 4.45rem;
  padding: 0.88rem 1rem;
}

.search-field::after {
  content: '';
  position: absolute;
  top: 16%;
  right: 0;
  width: 1px;
  height: 68%;
  background: rgba(255, 255, 255, 0.18);
}

.search-field:nth-child(3)::after {
  display: none;
}

.search-field select {
  position: absolute;
  inset: 0;
  opacity: 0;
  cursor: pointer;
}

.search-field__lead {
  display: grid;
  grid-template-columns: auto 1fr;
  align-items: center;
  gap: 0.75rem;
}

.search-field__lead :deep(svg),
.hero__search-btn :deep(svg),
.map-feature :deep(svg),
.category-chip :deep(svg) {
  font-size: 1.06rem;
}

.search-field span {
  display: block;
  font-size: 0.65rem;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: rgba(248, 250, 252, 0.62);
}

.search-field strong {
  display: block;
  margin-top: 0.14rem;
  font-size: 0.92rem;
  font-weight: 600;
}

.hero__search-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.4rem;
  border: 0;
  border-radius: 999px;
  background: #fff;
  color: #171c24;
  cursor: pointer;
  margin: 0.45rem;
  padding: 1rem 1.3rem;
  font-weight: 700;
}

.section-surface {
  background: #fafafa;
}

.section-head {
  margin-bottom: 2rem;
}

.section-head--centered {
  text-align: center;
}

.destinations-grid {
  display: grid;
  gap: 1.25rem;
}

.cities-grid {
  display: grid;
  gap: 1rem;
}

.category-panel {
  position: relative;
  overflow: hidden;
  min-height: 30rem;
  border-radius: 28px;
  background: #151a20;
  box-shadow: var(--shadow-lg);
}

.category-panel__image,
.category-panel__overlay {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
}

.category-panel__image {
  object-fit: cover;
}

.category-panel__overlay {
  background:
    linear-gradient(180deg, rgba(10, 14, 21, 0.12) 0%, rgba(10, 14, 21, 0.56) 100%),
    rgba(0, 0, 0, 0.12);
}

.category-panel--adventure .category-panel__overlay {
  background:
    linear-gradient(180deg, rgba(33, 65, 48, 0.22) 0%, rgba(10, 14, 21, 0.58) 100%),
    rgba(0, 0, 0, 0.1);
}

.category-panel--wellness .category-panel__overlay {
  background:
    linear-gradient(180deg, rgba(65, 91, 88, 0.24) 0%, rgba(10, 14, 21, 0.58) 100%),
    rgba(0, 0, 0, 0.1);
}

.category-panel--culture .category-panel__overlay {
  background:
    linear-gradient(180deg, rgba(84, 69, 58, 0.26) 0%, rgba(10, 14, 21, 0.6) 100%),
    rgba(0, 0, 0, 0.1);
}

.category-panel--food .category-panel__overlay {
  background:
    linear-gradient(180deg, rgba(98, 58, 39, 0.24) 0%, rgba(10, 14, 21, 0.6) 100%),
    rgba(0, 0, 0, 0.1);
}

.category-panel--nature .category-panel__overlay {
  background:
    linear-gradient(180deg, rgba(48, 83, 53, 0.24) 0%, rgba(10, 14, 21, 0.56) 100%),
    rgba(0, 0, 0, 0.1);
}

.category-panel--luxury .category-panel__overlay {
  background:
    linear-gradient(180deg, rgba(83, 72, 47, 0.26) 0%, rgba(10, 14, 21, 0.6) 100%),
    rgba(0, 0, 0, 0.1);
}

.category-panel__content {
  position: relative;
  display: flex;
  min-height: 30rem;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  text-align: center;
}

.category-pill {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: 1px solid rgba(255, 255, 255, 0.16);
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.12);
  backdrop-filter: blur(16px);
  color: rgba(255, 255, 255, 0.86);
  padding: 0.35rem 0.8rem;
  font-size: 0.78rem;
  margin-bottom: 0.8rem;
}

.section-title--light {
  color: #fff;
}

.category-grid {
  display: grid;
  width: 100%;
  max-width: 60rem;
  gap: 1rem;
  margin-top: 2.15rem;
}

.category-chip {
  display: inline-flex;
  min-height: 5.35rem;
  align-items: center;
  justify-content: center;
  gap: 0.6rem;
  border: 1px solid rgba(255, 255, 255, 0.38);
  border-radius: 18px;
  background: rgba(21, 24, 31, 0.2);
  backdrop-filter: blur(16px);
  color: #fff;
  cursor: pointer;
  padding: 1rem 1.25rem;
  font-size: 1.15rem;
  transition:
    background-color 0.25s ease,
    border-color 0.25s ease,
    box-shadow 0.25s ease;
}

.category-chip--active {
  border-color: rgba(255, 255, 255, 0.65);
  background: rgba(255, 255, 255, 0.18);
  box-shadow: 0 10px 28px rgba(0, 0, 0, 0.18);
}

.map-section {
  display: grid;
  gap: 2.5rem;
  align-items: center;
}

.map-copy {
  max-width: 30rem;
}

.map-feature-list {
  display: grid;
  gap: 1.3rem;
  margin-top: 2rem;
}

.map-feature {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 0.9rem;
  align-items: start;
}

.map-feature strong {
  display: block;
  color: var(--text-strong);
  font-weight: 700;
}

.map-feature p {
  margin: 0.2rem 0 0;
  color: var(--text-soft);
}

.map-feature :deep(svg) {
  margin-top: 0.2rem;
  color: var(--text-strong);
  font-size: 1.15rem;
}

.map-card {
  justify-self: center;
  width: min(100%, 31rem);
  border: 1px solid rgba(79, 102, 128, 0.28);
  border-radius: 28px;
  background: #f8f1d8;
  box-shadow: var(--shadow-md);
  padding: 0.35rem;
}

.leaflet-map {
  overflow: hidden;
  aspect-ratio: 1 / 1;
  border-radius: 24px;
}

:deep(.leaflet-popup-content-wrapper) {
  border-radius: 12px;
  box-shadow: 0 12px 30px rgba(17, 24, 39, 0.16);
  font-family: 'Manrope', sans-serif;
}

:deep(.leaflet-popup-content) {
  margin: 0.7rem 0.8rem;
  color: #1f242d;
  line-height: 1.45;
}

:deep(.leaflet-popup-tip) {
  background: #fff;
}

@media (min-width: 700px) {
  .destinations-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .category-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (max-width: 959px) {
  .hero__search {
    border-radius: 26px;
  }

  .search-field {
    border-bottom: 1px solid rgba(255, 255, 255, 0.12);
  }

  .search-field::after {
    display: none;
  }

  .search-field:nth-child(3) {
    border-bottom: 0;
  }
}

@media (min-width: 960px) {
  .hero__search {
    grid-template-columns: 1.12fr 1.12fr 1.12fr auto;
    align-items: center;
  }

  .destinations-grid {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }

  .cities-grid {
    grid-template-columns: repeat(12, minmax(0, 1fr));
    grid-template-rows: repeat(2, 12.5rem);
  }

  .cities-grid > :nth-child(1) {
    grid-column: 1 / span 4;
    grid-row: 1;
  }

  .cities-grid > :nth-child(2) {
    grid-column: 5 / span 4;
    grid-row: 1;
  }

  .cities-grid > :nth-child(3) {
    grid-column: 9 / span 4;
    grid-row: 1 / span 2;
  }

  .cities-grid > :nth-child(4) {
    grid-column: 1 / span 4;
    grid-row: 2;
  }

  .cities-grid > :nth-child(5) {
    grid-column: 5 / span 4;
    grid-row: 2;
  }

  .category-grid {
    grid-template-columns: repeat(4, minmax(0, 1fr));
  }

  .category-grid .category-chip:nth-child(5) {
    grid-column: 2;
  }

  .category-grid .category-chip:nth-child(6) {
    grid-column: 3;
  }

  .map-section {
    grid-template-columns: minmax(0, 1fr) minmax(18rem, 31rem);
    justify-content: space-between;
  }
}
</style>
