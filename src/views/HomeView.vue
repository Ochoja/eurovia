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
    icon: 'solar:mountains-linear',
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
]

const searchOptions = searchData as SearchCountry[]
const selectedCountry = ref(searchOptions[0]?.country ?? 'France')
const selectedCity = ref(searchOptions[0]?.cities[0]?.name ?? 'Paris')
const selectedInterest = ref('Arts & Culture')
const selectedCategory = ref('Adventure')
const hoveredCategory = ref('')

const activeCategoryImage = computed(
  () =>
    categories.find((c) => c.label === (hoveredCategory.value || selectedCategory.value))?.image ??
    categories[0]!.image,
)

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
      gsap.from('.category-panel', {
        scrollTrigger: { trigger: '.category-panel', start: 'top 85%' },
        opacity: 0,
        scale: 0.975,
        duration: 0.75,
        ease: 'power2.out',
      })
      gsap.from('.category-chip', {
        scrollTrigger: { trigger: '.category-grid', start: 'top 85%' },
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
      class="relative min-h-[46rem] flex items-center"
      style="background: linear-gradient(180deg, rgba(0,0,0,0.58) 0%, rgba(0,0,0,0.68) 100%), url('https://ik.imagekit.io/Ochoja01/euvoria/Hero.png') center/cover no-repeat;"
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
          class="hero__search mt-4 w-full max-w-204 overflow-hidden rounded-[999px] border border-white/10 backdrop-blur-[3xl]"
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
    <section
      id="categories"
      class="py-[clamp(4.5rem,7vw,7.5rem)] px-[clamp(1.25rem,5vw,3.25rem)] bg-[#fafafa]"
    >
      <div class="w-full max-w-[1280px] mx-auto">
        <div
          class="category-panel relative overflow-hidden min-h-[30rem] rounded-[28px] bg-[#151a20] shadow-[var(--shadow-lg)]"
        >
          <!-- Background image — fades between categories -->
          <Transition name="cat-fade">
            <img
              :key="activeCategoryImage"
              :src="activeCategoryImage"
              alt=""
              class="absolute inset-0 w-full h-full object-cover"
            />
          </Transition>

          <!-- Dark overlay -->
          <div class="absolute inset-0 bg-[rgba(10,14,21,0.55)]"></div>

          <!-- Content -->
          <div
            class="relative flex flex-col items-center justify-center min-h-[30rem] px-8 py-10 text-center"
          >
            <span
              class="inline-flex items-center justify-center rounded-full border border-white/16 bg-white/12 backdrop-blur-[16px] text-white/86 px-3 py-1 text-[0.78rem] mb-3"
            >
              Categories
            </span>
            <h2 class="section-title text-white">Travel By Interests</h2>

            <div class="category-grid grid w-full max-w-[60rem] gap-4 mt-8">
              <button
                v-for="category in categories"
                :key="category.label"
                type="button"
                class="category-chip inline-flex min-h-[5.35rem] items-center justify-center gap-2.5 rounded-[18px] border backdrop-blur-[16px] text-white cursor-pointer px-5 py-4 text-[1.05rem] transition-all duration-300"
                :class="
                  selectedCategory === category.label || hoveredCategory === category.label
                    ? 'border-white/70 bg-white/22 shadow-[0_10px_28px_rgba(0,0,0,0.22)]'
                    : 'border-white/38 bg-[rgba(21,24,31,0.2)] hover:border-white/55 hover:bg-white/15'
                "
                @mouseenter="hoveredCategory = category.label"
                @mouseleave="hoveredCategory = ''"
                @click="selectedCategory = category.label"
              >
                <Icon :icon="category.icon" class="text-[1.1rem]" />
                <span>{{ category.label }}</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ── Map Your Journey ───────────────────────────── -->
    <section
      id="map"
      class="py-[clamp(4.5rem,7vw,7.5rem)] px-[clamp(1.25rem,5vw,3.25rem)] bg-[#fafafa]"
    >
      <div class="map-section w-full max-w-[1280px] mx-auto grid gap-10 items-center">
        <div class="map-copy max-w-[30rem]">
          <h2 class="section-title">
            Map Your Journey
            <span class="text-[var(--accent)]">Across Europe</span>
          </h2>
          <p class="section-copy">
            Interact with our dynamic map to discover hidden gems tailored to your profile.
          </p>
          <div class="grid gap-5 mt-8">
            <div class="grid grid-cols-[auto_1fr] gap-3.5 items-start">
              <Icon
                icon="solar:map-point-rotate-linear"
                class="mt-0.5 text-[var(--text-strong)] text-[1.15rem]"
              />
              <div>
                <strong class="block text-[var(--text-strong)] font-bold">Interactive Pins</strong>
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

/* ── Category grid ──────────────────────────────── */
@media (min-width: 700px) {
  .category-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (min-width: 960px) {
  .category-grid {
    grid-template-columns: repeat(4, minmax(0, 1fr));
  }

  /* Centre the bottom two chips */
  .category-grid .category-chip:nth-child(5) {
    grid-column: 2;
  }
  .category-grid .category-chip:nth-child(6) {
    grid-column: 3;
  }
}

/* ── Category bg crossfade ──────────────────────── */
.cat-fade-enter-active,
.cat-fade-leave-active {
  transition: opacity 0.45s ease;
  position: absolute;
  inset: 0;
}

.cat-fade-enter-from,
.cat-fade-leave-to {
  opacity: 0;
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
