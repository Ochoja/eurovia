<script setup lang="ts">
import { Icon } from '@iconify/vue'
import { ref } from 'vue'

const email = ref('')
</script>

<template>
  <footer class="footer page-pad">
    <div class="footer__inner page-shell">
      <div class="footer__brand-col">
        <p class="footer__brand">EURO<span>VIA</span></p>
        <p class="footer__tagline">
          Lorem ipsum dolor sit amet, consectetur adipiscing. Et euismod volutpat congue eget cras
          malesuada massa mauris.
        </p>
      </div>

      <div class="footer__links-col">
        <p class="footer__col-heading">Destinations</p>
        <ul class="footer__list">
          <li><a href="#">Italy</a></li>
          <li><a href="#">Paris</a></li>
          <li><a href="#">Amsterdam</a></li>
          <li><a href="#">Amsterdam</a></li>
          <li><a href="#">Amsterdam</a></li>
        </ul>
      </div>

      <div class="footer__links-col">
        <p class="footer__col-heading">Support</p>
        <ul class="footer__list">
          <li>
            <a href="#" class="footer__social-link">
              <Icon icon="mdi:facebook" />
              <span>Facebook</span>
            </a>
          </li>
          <li>
            <a href="#" class="footer__social-link">
              <Icon icon="mdi:instagram" />
              <span>Instagram</span>
            </a>
          </li>
        </ul>
      </div>

      <div class="footer__newsletter-col">
        <p class="footer__col-heading">Subscribe to our Newsletter</p>
        <p class="footer__newsletter-sub">Sign up for our updates by subscribing to our newsletter</p>
        <div class="footer__newsletter-form">
          <input
            v-model="email"
            type="email"
            placeholder="Enter your email"
            class="footer__newsletter-input"
          />
          <button type="button" class="footer__newsletter-btn">Subscribe</button>
        </div>
      </div>
    </div>
  </footer>
</template>

<style scoped>
.footer {
  background: #0c0f14;
  padding-block: 4rem;
}

.footer__inner {
  display: grid;
  gap: 2.5rem;
  grid-template-columns: 1fr;
}

.footer__brand {
  margin: 0;
  color: #fff;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-size: 1.5rem;
  font-weight: 800;
}

.footer__brand span {
  color: var(--accent);
}

.footer__tagline {
  margin: 0.75rem 0 0;
  color: rgba(255, 255, 255, 0.42);
  font-size: 0.82rem;
  line-height: 1.6;
  max-width: 24ch;
}

.footer__col-heading {
  margin: 0 0 1rem;
  color: #fff;
  font-size: 0.9rem;
  font-weight: 600;
}

.footer__list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
}

.footer__list a {
  color: rgba(255, 255, 255, 0.5);
  font-size: 0.875rem;
  transition: color 0.2s ease;
}

.footer__list a:hover {
  color: rgba(255, 255, 255, 0.9);
}

.footer__social-link {
  display: inline-flex;
  align-items: center;
  gap: 0.45rem;
}

.footer__social-link :deep(svg) {
  font-size: 1.05rem;
}

.footer__newsletter-sub {
  margin: 0 0 1rem;
  color: rgba(255, 255, 255, 0.42);
  font-size: 0.8rem;
  line-height: 1.5;
}

.footer__newsletter-form {
  display: flex;
  gap: 0;
  border-radius: 999px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.12);
  background: rgba(255, 255, 255, 0.06);
  max-width: 20rem;
}

.footer__newsletter-input {
  flex: 1;
  border: 0;
  background: transparent;
  color: #fff;
  padding: 0.7rem 1rem;
  font-size: 0.82rem;
  outline: none;
  min-width: 0;
}

.footer__newsletter-input::placeholder {
  color: rgba(255, 255, 255, 0.35);
}

.footer__newsletter-btn {
  border: 0;
  border-radius: 999px;
  background: #fff;
  color: #0c0f14;
  cursor: pointer;
  padding: 0.65rem 1.1rem;
  font-size: 0.8rem;
  font-weight: 700;
  flex-shrink: 0;
  margin: 0.2rem;
}

@media (min-width: 768px) {
  .footer__inner {
    grid-template-columns: 1.4fr 1fr 1fr;
    gap: 3rem;
  }

  .footer__newsletter-col {
    grid-column: 1 / -1;
  }
}

@media (min-width: 1024px) {
  .footer__inner {
    grid-template-columns: 1.6fr 1fr 1fr 1.8fr;
    gap: 2.5rem;
    align-items: start;
  }

  .footer__newsletter-col {
    grid-column: auto;
  }
}
</style>
