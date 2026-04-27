<script setup lang="ts">
import { Icon } from '@iconify/vue'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import L from 'leaflet'
import { computed, nextTick, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import CityCard from '@/components/CityCard.vue'
import DestinationCard from '@/components/DestinationCard.vue'
import TheFooter from '@/components/TheFooter.vue'
import TheNavigation from '@/components/TheNavigation.vue'
import cityData from '@/data/cities.json'
import destinationData from '@/data/destinations.json'
import searchData from '@/data/search.json'

gsap.registerPlugin(ScrollTrigger)

type SearchCity = { name: string; coordinates: [number, number]; interest: string; image: string }
type SearchCountry = { country: string; cities: SearchCity[] }

const sections = [
  { id: 'discover', label: 'Discover' },
  { id: 'destinations', label: 'Destinations' },
  { id: 'cities', label: 'Cities' },
  { id: 'categories', label: 'Categories' },
  { id: 'map', label: 'Map' },
]

const interests = ['Arts & Culture', 'Adventure', 'Wellness', 'Food', 'Nature', 'Luxury']

const categories = [
  {
    label: 'Adventure',
    icon: 'mdi:mountain-outline',
    image: 'https://ik.imagekit.io/Ochoja01/euvoria/Spain.png',
  },
  {
    label: 'Wellness',
    icon: 'solar:heart-angle-linear',
    image: 'https://ik.imagekit.io/Ochoja01/euvoria/Amsterdam.png',
  },
  {
    label: 'Culture',
    icon: 'solar:buildings-2-linear',
    image: 'https://ik.imagekit.io/Ochoja01/euvoria/Italy.png',
  },
  {
    label: 'Food',
    icon: 'solar:chef-hat-linear',
    image: 'https://ik.imagekit.io/Ochoja01/euvoria/paris.png',
  },
  {
    label: 'Nature',
    icon: 'solar:leaf-linear',
    image: 'https://ik.imagekit.io/Ochoja01/euvoria/Categories.png',
  },
  {
    label: 'Luxury',
    icon: 'solar:crown-linear',
    image: 'https://ik.imagekit.io/Ochoja01/euvoria/London.png',
  },
  {
    label: 'History',
    icon: 'solar:diploma-linear',
    image: 'https://ik.imagekit.io/Ochoja01/euvoria/Italy.png',
  },
]

const searchOptions = searchData as SearchCountry[]
const selectedCountry = ref(searchOptions[0]?.country ?? 'France')
const selectedCity = ref(searchOptions[0]?.cities[0]?.name ?? 'Paris')
const selectedInterest = ref('Arts & Culture')
const selectedCategory = ref('Adventure')

const mapContainer = ref<HTMLDivElement | null>(null)
let map: L.Map | null = null
let tileLayer: L.TileLayer | null = null
let markerLayer: L.LayerGroup | null = null
const selectedMapCities = ref<string[]>([])

const availableCities = computed(
  () => searchOptions.find((e) => e.country === selectedCountry.value)?.cities ?? [],
)

const syncSelectedCity = () => {
  if (!availableCities.value.some((c) => c.name === selectedCity.value)) {
    selectedCity.value = availableCities.value[0]?.name ?? ''
  }
}

const buildPhotoIcon = (city: SearchCity) =>
  L.divIcon({
    className: '',
    html: `<div style="position:relative;background:#fff;border-radius:10px;overflow:visible;box-shadow:0 6px 20px rgba(0,0,0,0.26);width:88px;cursor:pointer;font-family:Manrope,sans-serif;">
        <img src="${city.image}" alt="${city.name}" style="width:88px;height:62px;object-fit:cover;display:block;border-radius:10px 10px 0 0;" />
        <div style="padding:4px 7px 6px;font-size:10px;font-weight:700;color:#1f242d;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;border-radius:0 0 10px 10px;">${city.name}</div>
        <div style="position:absolute;bottom:-7px;left:50%;transform:translateX(-50%);width:0;height:0;border-left:7px solid transparent;border-right:7px solid transparent;border-top:7px solid #fff;"></div>
      </div>`,
    iconSize: [88, 85],
    iconAnchor: [44, 92],
  })

const toggleCityOnMap = (cityName: string) => {
  const idx = selectedMapCities.value.indexOf(cityName)
  if (idx >= 0) selectedMapCities.value.splice(idx, 1)
  else if (selectedMapCities.value.length < 5) selectedMapCities.value.push(cityName)
  renderMarkers()
}

const renderMarkers = () => {
  if (!map || !markerLayer) return
  markerLayer.clearLayers()
  const cities = availableCities.value
  const bounds: L.LatLngExpression[] = []
  cities.forEach((city) => {
    bounds.push(city.coordinates)
    if (selectedMapCities.value.includes(city.name)) {
      const marker = L.marker(city.coordinates, { icon: buildPhotoIcon(city) })
      marker.on('click', () => toggleCityOnMap(city.name))
      markerLayer?.addLayer(marker)
    } else {
      const marker = L.circleMarker(city.coordinates, {
        radius: 7,
        weight: 2,
        color: '#ffffff',
        fillColor: '#1f242d',
        fillOpacity: 1,
      })
      marker.bindTooltip(city.name, {
        permanent: false,
        direction: 'top',
        offset: [0, -8],
        className: 'map-tooltip',
      })
      marker.on('click', () => toggleCityOnMap(city.name))
      markerLayer?.addLayer(marker)
    }
  })
  if (bounds.length)
    map.fitBounds(bounds as L.LatLngBoundsExpression, { padding: [40, 40], maxZoom: 7 })
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

const runSearch = () => {
  syncSelectedCity()
  selectedMapCities.value = []
  renderMarkers()
  const mapSection = document.getElementById('map')
  if (!mapSection) return
  window.scrollTo({
    top: mapSection.getBoundingClientRect().top + window.scrollY - 88,
    behavior: 'smooth',
  })
}

watch(selectedCountry, () => {
  syncSelectedCity()
  selectedMapCities.value = []
  renderMarkers()
})
watch(selectedInterest, renderMarkers)

let gsapCtx: gsap.Context | null = null

onMounted(() => {
  createLeafletMap()
  nextTick(() => {
    gsapCtx = gsap.context(() => {
      gsap.utils.toArray<HTMLElement>('.section-head').forEach((el) => {
        gsap.from(el, {
          scrollTrigger: { trigger: el, start: 'top 88%' },
          opacity: 0,
          y: 28,
          duration: 0.65,
          ease: 'power2.out',
        })
      })
      gsap.from('.destinations-grid > *', {
        scrollTrigger: { trigger: '.destinations-grid', start: 'top 82%' },
        opacity: 0,
        y: 36,
        duration: 0.6,
        stagger: 0.1,
        ease: 'power2.out',
      })
      gsap.from('.cities-grid > *', {
        scrollTrigger: { trigger: '.cities-grid', start: 'top 82%' },
        opacity: 0,
        y: 30,
        duration: 0.55,
        stagger: 0.07,
        ease: 'power2.out',
      })
      gsap.from('.category-chip', {
        scrollTrigger: { trigger: '.category-grid', start: 'top 85%' },
        immediateRender: false,
        opacity: 0,
        y: 18,
        duration: 0.5,
        stagger: 0.06,
        ease: 'power2.out',
      })
      gsap.from('.map-copy', {
        scrollTrigger: { trigger: '.map-section', start: 'top 82%' },
        opacity: 0,
        x: -36,
        duration: 0.75,
        ease: 'power2.out',
      })
      gsap.from('.map-card', {
        scrollTrigger: { trigger: '.map-section', start: 'top 82%' },
        opacity: 0,
        x: 36,
        duration: 0.75,
        ease: 'power2.out',
      })
    })
  })
})

onBeforeUnmount(() => {
  gsapCtx?.revert()
  ScrollTrigger.getAll().forEach((t) => t.kill())
  markerLayer?.clearLayers()
  tileLayer?.remove()
  map?.remove()
})
</script>

<template>
  <div class="overflow-x-clip bg-[#fafafa]">
    <!-- ── Hero ───────────────────────────────────────── -->
    <section
      id="discover"
      class="relative min-h-184 flex items-center"
      style="
        background:
          linear-gradient(180deg, rgba(0, 0, 0, 0.58) 0%, rgba(0, 0, 0, 0.58) 100%),
          url('https://ik.imagekit.io/Ochoja01/euvoria/Hero.png') center/cover no-repeat;
      "
    >
      <TheNavigation :sections="sections" />

      <div
        class="relative w-full flex flex-col items-center text-center px-[clamp(1.25rem,5vw,3.25rem)] pt-20 pb-12 max-w-[1280px] mx-auto"
      >
        <h1
          class="m-0 mb-2 text-white font-[Plus_Jakarta_Sans] leading-[1.02] sm:whitespace-nowrap"
          style="font-size: clamp(2.6rem, 5.5vw, 4.85rem)"
        >
          Discover Europe, <span class="text-[var(--accent)] font-medium">Your Way</span>
        </h1>

        <!-- Search bar -->
        <div
          class="hero__search mt-4 w-full max-w-204 overflow-hidden rounded-[999px] border border-white/10 backdrop-blur-[10px]"
          style="background: #fafafa33"
        >
          <!-- Country -->
          <label class="search-field relative grid items-center text-white/94 cursor-pointer">
            <div class="flex items-center gap-3">
              <Icon icon="solar:map-point-linear" class="text-[1.06rem] shrink-0" />
              <div class="min-w-0">
                <span class="block text-[0.6rem] uppercase tracking-[0.04em] text-white/60"
                  >Country</span
                >
                <strong class="block mt-0.5 text-[0.88rem] font-semibold truncate">{{
                  selectedCountry || 'Where to?'
                }}</strong>
              </div>
            </div>
            <Icon icon="solar:alt-arrow-down-linear" class="text-[1.06rem] ml-2 shrink-0" />
            <select
              v-model="selectedCountry"
              aria-label="Select country"
              class="absolute inset-0 opacity-0 cursor-pointer"
            >
              <option v-for="item in searchOptions" :key="item.country" :value="item.country">
                {{ item.country }}
              </option>
            </select>
          </label>

          <!-- City -->
          <label class="search-field relative grid items-center text-white/94 cursor-pointer">
            <div class="flex items-center gap-3">
              <Icon icon="solar:buildings-2-linear" class="text-[1.06rem] shrink-0" />
              <div class="min-w-0">
                <span class="block text-[0.6rem] uppercase tracking-[0.04em] text-white/60"
                  >City</span
                >
                <strong class="block mt-0.5 text-[0.88rem] font-semibold truncate">{{
                  selectedCity || 'Add Cities'
                }}</strong>
              </div>
            </div>
            <Icon icon="solar:alt-arrow-down-linear" class="text-[1.06rem] ml-2 shrink-0" />
            <select
              v-model="selectedCity"
              aria-label="Select city"
              class="absolute inset-0 opacity-0 cursor-pointer"
            >
              <option v-for="city in availableCities" :key="city.name" :value="city.name">
                {{ city.name }}
              </option>
            </select>
          </label>

          <!-- Interests -->
          <label class="search-field relative grid items-center text-white/94 cursor-pointer">
            <div class="flex items-center gap-3">
              <Icon icon="solar:camera-linear" class="text-[1.06rem] shrink-0" />
              <div class="min-w-0">
                <span class="block text-[0.6rem] uppercase tracking-[0.04em] text-white/60"
                  >Interests</span
                >
                <strong class="block mt-0.5 text-[0.88rem] font-semibold truncate">{{
                  selectedInterest
                }}</strong>
              </div>
            </div>
            <Icon icon="solar:alt-arrow-down-linear" class="text-[1.06rem] ml-2 shrink-0" />
            <select
              v-model="selectedInterest"
              aria-label="Select interest"
              class="absolute inset-0 opacity-0 cursor-pointer"
            >
              <option v-for="interest in interests" :key="interest" :value="interest">
                {{ interest }}
              </option>
            </select>
          </label>

          <!-- Search button -->
          <button
            type="button"
            class="inline-flex items-center justify-center gap-1.5 rounded-full bg-white text-[#171c24] font-bold cursor-pointer border-0 m-[0.3rem] px-5 py-[0.72rem] text-[0.9rem] shrink-0"
            @click="runSearch"
          >
            <Icon icon="solar:magnifer-linear" class="text-[1.06rem]" />
            Search
          </button>
        </div>
      </div>
    </section>

    <!-- ── Featured Destinations ──────────────────────── -->
    <section
      id="destinations"
      class="py-[clamp(4.5rem,7vw,7.5rem)] px-[clamp(1.25rem,5vw,3.25rem)] bg-[#fafafa]"
    >
      <div class="w-full max-w-[1280px] mx-auto">
        <header class="section-head mb-8">
          <h2 class="section-title">Featured Destinations</h2>
          <p class="section-copy">Hand-picked curations for the modern explorer</p>
        </header>
        <div class="destinations-grid grid gap-5">
          <DestinationCard v-for="d in destinationData" :key="d.name" :destination="d" />
        </div>
      </div>
    </section>

    <!-- ── Top Cities (bento) ─────────────────────────── -->
    <section
      id="cities"
      class="py-[clamp(4.5rem,7vw,7.5rem)] px-[clamp(1.25rem,5vw,3.25rem)] bg-[#fafafa]"
    >
      <div class="w-full max-w-[1280px] mx-auto">
        <header class="section-head section-head--centered mb-8 text-center">
          <h2 class="section-title">Top Cities</h2>
          <p class="section-copy">Urban escapes that blend culture and contemporary lives</p>
        </header>
        <div class="cities-grid gap-3">
          <CityCard v-for="city in cityData" :key="city.name" :city="city" />
        </div>
      </div>
    </section>

    <!-- ── Travel By Interests ────────────────────────── -->
    <section id="categories" class="cat-section">
      <div class="cat-inner">
        <span class="cat-badge">Categories</span>
        <h2 class="cat-title">Travel By Interests</h2>
        <div class="cat-grid category-grid">
          <button
            v-for="category in categories"
            :key="category.label"
            type="button"
            class="cat-card category-chip bg-white/5 backdrop-blur-[5px] border border-white/20 hover:bg-white/14 transition-all duration-300"
            :class="{ 'cat-card--active': selectedCategory === category.label }"
            @click="selectedCategory = category.label"
          >
            <Icon :icon="category.icon" class="cat-icon" />
            <span class="cat-label">{{ category.label }}</span>
          </button>
        </div>
      </div>
    </section>

    <!-- ── Map Your Journey ───────────────────────────── -->
    <section
      id="map"
      class="py-[clamp(4.5rem,7vw,7.5rem)] px-[clamp(1.25rem,5vw,3.25rem)] bg-[#fafafa]"
    >
      <div class="map-section w-full max-w-7xl mx-auto grid gap-10 items-center">
        <div class="map-copy max-w-120">
          <h2 class="section-title">
            Map Your Journey
            <span class="text-(--accent)">Across Europe</span>
          </h2>
          <p class="section-copy">
            Interact with our dynamic map to discover hidden gems tailored to your profile.
          </p>
          <div class="grid gap-5 mt-8">
            <div class="grid grid-cols-[auto_1fr] gap-3.5 items-start">
              <Icon
                icon="solar:map-point-rotate-linear"
                class="mt-0.5 text-(--text-strong) text-[1.15rem]"
              />
              <div>
                <strong class="block text-(--text-strong) font-bold">Interactive Pins</strong>
                <p class="m-0 mt-1 text-[var(--text-soft)] text-sm">
                  Click city markers to pin up to 5 cities and see their photo.
                </p>
              </div>
            </div>
            <div class="grid grid-cols-[auto_1fr] gap-3.5 items-start">
              <Icon
                icon="solar:routing-2-linear"
                class="mt-0.5 text-[var(--text-strong)] text-[1.15rem]"
              />
              <div>
                <strong class="block text-[var(--text-strong)] font-bold">Custom Routes</strong>
                <p class="m-0 mt-1 text-[var(--text-soft)] text-sm">
                  Country search updates the map in real time.
                </p>
              </div>
            </div>
          </div>
        </div>

        <div
          class="map-card justify-self-center w-[min(100%,31rem)] border border-[rgba(79,102,128,0.28)] rounded-[28px] bg-[#f8f1d8] shadow-[var(--shadow-md)] p-1.5"
        >
          <div
            ref="mapContainer"
            class="leaflet-map overflow-hidden aspect-square rounded-3xl"
          ></div>
        </div>
      </div>
    </section>

    <TheFooter />
  </div>
</template>

<style scoped>
/* ── Search bar layout ──────────────────────────── */
.hero__search {
  display: grid;
}

.search-field {
  grid-template-columns: 1fr auto;
  min-height: 3.1rem;
  padding: 0.45rem 1.1rem;
}

/* Divider between fields */
.search-field::after {
  content: '';
  position: absolute;
  top: 15%;
  right: 0;
  width: 1px;
  height: 70%;
  background: rgba(255, 255, 255, 0.16);
}

.search-field:last-of-type::after {
  display: none;
}

/* Mobile: stack vertically */
@media (max-width: 959px) {
  .hero__search {
    border-radius: 26px !important;
  }

  .search-field {
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  }

  .search-field::after {
    display: none;
  }

  .search-field:nth-child(3) {
    border-bottom: none;
  }
}

/* Desktop: 4-col horizontal */
@media (min-width: 960px) {
  .hero__search {
    grid-template-columns: 1fr 1fr 1fr auto;
    align-items: center;
  }
}

/* ── Destinations grid ──────────────────────────── */
@media (min-width: 700px) {
  .destinations-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (min-width: 960px) {
  .destinations-grid {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }
}

/* ── Cities bento grid ──────────────────────────── */
/* Mobile: single column stack */
.cities-grid {
  display: grid;
}

/* Desktop: asymmetric bento — narrow | wide | tall */
@media (min-width: 960px) {
  .cities-grid {
    grid-template-columns: repeat(10, minmax(0, 1fr));
    grid-template-rows: 16rem 16rem;
  }

  /* Row 1: narrow(3) | wide(4) | tall-start(3) */
  .cities-grid > :nth-child(1) {
    grid-column: 1 / span 3;
    grid-row: 1;
  }
  .cities-grid > :nth-child(2) {
    grid-column: 4 / span 4;
    grid-row: 1;
  }
  .cities-grid > :nth-child(3) {
    grid-column: 8 / span 3;
    grid-row: 1 / span 2;
  }

  /* Row 2: wide(4) | narrow(3) | tall-end(3 — already placed) */
  .cities-grid > :nth-child(4) {
    grid-column: 1 / span 4;
    grid-row: 2;
  }
  .cities-grid > :nth-child(5) {
    grid-column: 5 / span 3;
    grid-row: 2;
  }
}

/* ── Categories section ─────────────────────────── */
.cat-section {
  padding: clamp(4.5rem, 7vw, 7.5rem) clamp(1.25rem, 5vw, 3.25rem);
  background:
    linear-gradient(rgba(0, 0, 0, 0.52), rgba(0, 0, 0, 0.6)),
    url('https://ik.imagekit.io/Ochoja01/euvoria/Categories.png') center / cover no-repeat;
}

.cat-inner {
  width: 100%;
  max-width: 80rem;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  padding: 1rem 0;
}

.cat-badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(255, 255, 255, 0.12);
  backdrop-filter: blur(16px);
  color: rgba(255, 255, 255, 0.86);
  padding: 0.25rem 0.75rem;
  font-size: 0.85rem;
  margin-bottom: 0.75rem;
}

.cat-title {
  margin: 0;
  color: #fff;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-size: clamp(2rem, 4vw, 3rem);
  font-weight: 600;
  line-height: 1.1;
}

.cat-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1rem;
  margin-top: 2rem;
  width: 100%;
  max-width: 60rem;
}

@media (min-width: 960px) {
  .cat-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}

.cat-card {
  border-radius: 18px;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  padding: 2rem 1rem;
  min-height: 8rem;
  color: #fff;
  font-size: 0.95rem;
  font-weight: 500;
}

.cat-card--active {
  border-color: rgba(255, 255, 255, 0.55) !important;
  background: rgba(255, 255, 255, 0.16) !important;
  box-shadow: 0 10px 28px rgba(0, 0, 0, 0.35);
}

.cat-icon {
  font-size: 1.6rem;
}

/* ── Map section ────────────────────────────────── */
@media (min-width: 960px) {
  .map-section {
    grid-template-columns: minmax(0, 1fr) minmax(18rem, 31rem);
  }
}

/* ── Leaflet overrides ──────────────────────────── */
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

:deep(.map-tooltip) {
  background: rgba(20, 24, 32, 0.88);
  border: none;
  border-radius: 6px;
  color: #fff;
  font-family: 'Manrope', sans-serif;
  font-size: 0.75rem;
  font-weight: 600;
  padding: 3px 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.18);
}

:deep(.map-tooltip::before) {
  display: none;
}
</style>
