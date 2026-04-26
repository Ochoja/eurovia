<script setup lang="ts">
import { Icon } from '@iconify/vue'
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

type Section = { id: string; label: string }

const props = defineProps<{ sections: Section[] }>()

const activeSection = ref(props.sections[0]?.id ?? 'discover')
const mobileOpen = ref(false)
const isFloating = ref(false)
const isVisible = ref(true)

let observer: IntersectionObserver | null = null
let lastScrollY = 0

const wrapClass = computed(() => [
  !isVisible.value && isFloating.value && !mobileOpen.value
    ? 'opacity-0 -translate-y-full pointer-events-none'
    : 'opacity-100 translate-y-0',
])

const goToSection = (id: string) => {
  activeSection.value = id
  mobileOpen.value = false
  document.body.classList.remove('menu-open')
  const target = document.getElementById(id)
  if (!target) return
  const top = target.getBoundingClientRect().top + window.scrollY - 104
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
        .filter((e) => e.isIntersecting)
        .sort((a, b) => b.intersectionRatio - a.intersectionRatio)
      if (visible[0]?.target instanceof HTMLElement) {
        activeSection.value = visible[0].target.id
      }
    },
    { rootMargin: '-24% 0px -56% 0px', threshold: [0.2, 0.35, 0.55] },
  )
  props.sections.forEach((s) => {
    const el = document.getElementById(s.id)
    if (el) observer?.observe(el)
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
  <!-- Outer wrapper: handles show/hide and positioning -->
  <header
    class="px-[var(--page-pad-inline)] z-40 transition-all duration-300"
    :class="[isFloating ? 'fixed inset-x-0 top-3' : 'absolute inset-x-0 top-4', ...wrapClass]"
  >
    <div class="relative w-full max-w-[1280px] mx-auto">
      <!-- Nav bar: on hero = separated pills; floating = one dark container -->
      <nav
        class="flex items-center justify-between gap-3 transition-all duration-300"
        :class="
          isFloating
            ? 'bg-[rgba(10,14,22,0.82)] backdrop-blur-2xl border border-white/10 rounded-full py-1.5 pl-5 pr-1.5'
            : ''
        "
      >
        <!-- Brand – no container on hero; plain text inside floating bar -->
        <a
          href="#discover"
          class="inline-flex items-center shrink-0"
          :class="isFloating ? 'py-0' : 'py-1'"
          @click.prevent="goToSection('discover')"
        >
          <span class="text-white font-extrabold text-2xl font-[Plus_Jakarta_Sans]">
            EURO<span class="text-[var(--accent)]">VIA</span>
          </span>
        </a>

        <!-- Nav links: own pill container only on hero -->
        <div
          class="hidden md:inline-flex items-center gap-0.5 rounded-full px-1 py-1 transition-all duration-300 text-white"
          :class="
            isFloating
              ? 'bg-transparent border-transparent shadow-none'
              : 'border border-white/15 bg-[#FAFAFA]/20 backdrop-blur-[18px] shadow-[0_8px_25px_rgba(16,24,40,0.14)]'
          "
        >
          <button
            v-for="section in sections.slice(0, 4)"
            :key="section.id"
            type="button"
            class="rounded-full px-4 py-2.5 text-sm text-white/80 bg-transparent border-0 cursor-pointer transition-colors duration-200"
            :class="
              activeSection === section.id
                ? 'bg-[rgba(23,23,23,0.72)] text-white'
                : 'hover:bg-black/20'
            "
            @click="goToSection(section.id)"
          >
            {{ section.label }}
          </button>
        </div>

        <!-- Actions: own pill-style on hero; inline inside floating bar -->
        <div class="flex items-center gap-2">
          <!-- Language -->
          <button
            type="button"
            class="hidden md:inline-flex items-center gap-1.5 rounded-full bg-[#FAFAFA]/50 text-white px-3.5 py-2.5 text-sm border border-[#9E9B9B] cursor-pointer transition-all duration-300"
          >
            <Icon icon="mdi:translate" class="text-white" />
            <span class="text-white">En</span>
            <Icon icon="solar:alt-arrow-down-linear" class="text-white" />
          </button>

          <!-- CTA -->
          <button
            type="button"
            class="hidden md:inline-flex rounded-full bg-white text-[#171c24] font-bold px-4 py-2.5 text-sm border-0 cursor-pointer"
            @click="goToSection('map')"
          >
            Explore Now
          </button>

          <!-- Mobile hamburger -->
          <button
            type="button"
            class="md:hidden inline-flex items-center justify-center rounded-full text-white p-2.5 text-xl border-0 cursor-pointer transition-all duration-300"
            :class="
              isFloating
                ? 'bg-white/10'
                : 'border border-white/20 bg-[rgba(36,36,36,0.22)] backdrop-blur-[18px]'
            "
            @click="toggleMobileMenu"
          >
            <Icon
              :icon="mobileOpen ? 'solar:close-circle-linear' : 'solar:hamburger-menu-linear'"
            />
          </button>
        </div>
      </nav>

      <!-- Mobile panel -->
      <div
        class="absolute left-0 right-0 grid gap-2 rounded-3xl border border-white/15 bg-[rgba(19,22,28,0.88)] backdrop-blur-[18px] p-3 transition-all duration-200 md:hidden"
        :class="
          mobileOpen
            ? 'top-[calc(100%+0.75rem)] opacity-100 pointer-events-auto translate-y-0'
            : 'top-[calc(100%+0.75rem)] opacity-0 pointer-events-none -translate-y-2'
        "
      >
        <button
          v-for="section in sections"
          :key="`${section.id}-m`"
          type="button"
          class="rounded-2xl px-4 py-3 text-left text-white/86 border-0 bg-transparent cursor-pointer"
          :class="activeSection === section.id ? 'bg-white/15 text-white' : ''"
          @click="goToSection(section.id)"
        >
          {{ section.label }}
        </button>
      </div>
    </div>
  </header>
</template>
