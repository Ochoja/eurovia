<template>
  <footer class="footer page-pad">
    <div class="footer__inner page-shell">
      <div>
        <p class="footer__brand">EURO<span>VIA</span></p>
        <p class="footer__copy">Curated European journeys for modern explorers.</p>
      </div>

      <p class="footer__meta">Discover destinations, cities, and culture-driven escapes.</p>
    </div>
  </footer>
</template>

<style scoped>
.footer {
  background: #0a0a0a;
  padding-block: 3.5rem;
}

.footer__inner {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  color: rgba(255, 255, 255, 0.72);
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

.footer__copy,
.footer__meta {
  margin: 0.5rem 0 0;
}

@media (min-width: 768px) {
  .footer__inner {
    flex-direction: row;
    align-items: flex-end;
    justify-content: space-between;
  }

  .footer__meta {
    margin: 0;
    max-width: 28ch;
    text-align: right;
  }
}
</style>
