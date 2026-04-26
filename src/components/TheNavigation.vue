<script setup lang="ts">
import { Icon } from '@iconify/vue'
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

type Section = {
  id: string
  label: string
}

const props = defineProps<{
  sections: Section[]
}>()

const activeSection = ref(props.sections[0]?.id ?? 'discover')
const mobileOpen = ref(false)
const isFloating = ref(false)
const isVisible = ref(true)

let observer: IntersectionObserver | null = null
let lastScrollY = 0

const navClass = computed(() => ({
  'nav-wrap--floating': isFloating.value,
  'nav-wrap--hidden': !isVisible.value && isFloating.value && !mobileOpen.value,
}))

const goToSection = (id: string) => {
  activeSection.value = id
  mobileOpen.value = false
  document.body.classList.remove('menu-open')

  const target = document.getElementById(id)
  if (!target) return

  const offset = 104
  const top = target.getBoundingClientRect().top + window.scrollY - offset
  window.scrollTo({ top, behavior: 'smooth' })
}

const toggleMobileMenu = () => {
  mobileOpen.value = !mobileOpen.value
  document.body.classList.toggle('menu-open', mobileOpen.value)
}

const handleScroll = () => {
  const currentY = window.scrollY
  const hero = document.getElementById('discover')
  const heroBottom = hero ? hero.offsetTop + hero.offsetHeight - 96 : 0

  isFloating.value = currentY > heroBottom

  if (currentY < heroBottom) {
    isVisible.value = true
  } else {
    isVisible.value = currentY < lastScrollY || currentY < heroBottom + 40
  }

  lastScrollY = currentY
}

onMounted(() => {
  lastScrollY = window.scrollY
  handleScroll()

  observer = new IntersectionObserver(
    (entries) => {
      const visible = entries
        .filter((entry) => entry.isIntersecting)
        .sort((a, b) => b.intersectionRatio - a.intersectionRatio)

      if (visible[0]?.target instanceof HTMLElement) {
        activeSection.value = visible[0].target.id
      }
    },
    {
      rootMargin: '-24% 0px -56% 0px',
      threshold: [0.2, 0.35, 0.55],
    },
  )

  props.sections.forEach((section) => {
    const element = document.getElementById(section.id)
    if (element) observer?.observe(element)
  })

  window.addEventListener('scroll', handleScroll, { passive: true })
})

onBeforeUnmount(() => {
  observer?.disconnect()
  document.body.classList.remove('menu-open')
  window.removeEventListener('scroll', handleScroll)
})
</script>

<template>
  <header class="nav-wrap page-pad" :class="navClass">
    <div class="nav-shell page-shell">
      <nav class="nav-bar">
        <a href="#discover" class="nav-group brand-group" @click.prevent="goToSection('discover')">
          <span class="brand">EURO<span>VIA</span></span>
        </a>

        <div class="nav-group nav-links">
          <button
            v-for="section in sections.slice(0, 4)"
            :key="section.id"
            type="button"
            class="nav-link"
            :class="{ 'nav-link--active': activeSection === section.id }"
            @click="goToSection(section.id)"
          >
            {{ section.label }}
          </button>
        </div>

        <div class="nav-group nav-actions">
          <button type="button" class="lang-pill">
            <Icon icon="mdi:translate" />
            <span>En</span>
            <Icon icon="solar:alt-arrow-down-linear" />
          </button>

          <button type="button" class="cta" @click="goToSection('map')">Explore Now</button>

          <button type="button" class="menu-btn" @click="toggleMobileMenu">
            <Icon :icon="mobileOpen ? 'solar:close-circle-linear' : 'solar:hamburger-menu-linear'" />
          </button>
        </div>
      </nav>

      <div class="mobile-panel" :class="{ 'mobile-panel--open': mobileOpen }">
        <button
          v-for="section in sections"
          :key="`${section.id}-mobile`"
          type="button"
          class="mobile-link"
          :class="{ 'mobile-link--active': activeSection === section.id }"
          @click="goToSection(section.id)"
        >
          {{ section.label }}
        </button>
      </div>
    </div>
  </header>
</template>

<style scoped>
.nav-wrap {
  position: absolute;
  inset: 1.1rem 0 auto;
  z-index: 40;
  transition:
    transform 0.3s ease,
    opacity 0.3s ease;
}

.nav-wrap--floating {
  position: fixed;
}

.nav-wrap--hidden {
  opacity: 0;
  transform: translateY(-120%);
  pointer-events: none;
}

.nav-shell {
  position: relative;
}

.nav-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.8rem;
}

.nav-group {
  border: 1px solid rgba(255, 255, 255, 0.14);
  background: rgba(36, 36, 36, 0.18);
  backdrop-filter: blur(18px);
  box-shadow: 0 8px 25px rgba(16, 24, 40, 0.14);
}

.brand-group {
  display: inline-flex;
  align-items: center;
  border-radius: 999px;
  padding: 0.82rem 1rem;
}

.brand {
  color: #f7fafc;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-size: 1.5rem;
  font-weight: 800;
}

.brand span {
  color: var(--accent);
}

.nav-links {
  display: none;
  align-items: center;
  gap: 0.15rem;
  border-radius: 999px;
  padding: 0.22rem;
}

.nav-link,
.mobile-link {
  border: 0;
  background: transparent;
  cursor: pointer;
  padding: 0.7rem 0.96rem;
  color: rgba(248, 250, 252, 0.8);
}

.nav-link {
  border-radius: 999px;
  font-size: 0.86rem;
  transition: background-color 0.25s ease;
}

.nav-link--active {
  background: rgba(23, 23, 23, 0.72);
  color: #fff;
}

.nav-actions {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  border-radius: 999px;
  padding: 0.32rem;
}

.lang-pill,
.cta,
.menu-btn {
  border: 0;
  cursor: pointer;
}

.lang-pill {
  display: none;
  align-items: center;
  gap: 0.35rem;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.08);
  color: #fff;
  padding: 0.68rem 0.88rem;
}

.cta {
  display: none;
  border-radius: 999px;
  background: #fff;
  color: #171c24;
  padding: 0.76rem 1rem;
  font-size: 0.88rem;
  font-weight: 700;
}

.menu-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.08);
  color: #fff;
  padding: 0.68rem;
  font-size: 1.2rem;
}

.mobile-panel {
  position: absolute;
  top: calc(100% + 0.85rem);
  right: 0;
  left: 0;
  display: grid;
  gap: 0.5rem;
  border: 1px solid rgba(255, 255, 255, 0.14);
  border-radius: 24px;
  background: rgba(19, 22, 28, 0.78);
  backdrop-filter: blur(18px);
  padding: 0.75rem;
  opacity: 0;
  pointer-events: none;
  transform: translateY(-10px);
  transition:
    opacity 0.25s ease,
    transform 0.25s ease;
}

.mobile-panel--open {
  opacity: 1;
  pointer-events: auto;
  transform: translateY(0);
}

.mobile-link {
  border-radius: 14px;
  color: rgba(248, 250, 252, 0.86);
  text-align: left;
}

.mobile-link--active {
  background: rgba(255, 255, 255, 0.14);
  color: #fff;
}

@media (min-width: 900px) {
  .nav-links,
  .lang-pill,
  .cta {
    display: inline-flex;
  }

  .menu-btn,
  .mobile-panel {
    display: none;
  }
}
</style>
