<template>
  <div class="relative z-10 mx-auto w-full max-w-[1240px] px-5 pb-10 pt-2 md:px-8">
    <footer class="flex flex-col gap-4 border-t border-[var(--hairline)] pt-6 text-[13px] text-[var(--ink-2)] md:flex-row md:items-center md:justify-between">
      <p>&copy; {{ currentYear }} iOS Club of XAUAT</p>

      <nav class="flex flex-wrap items-center gap-x-5 gap-y-2">
        <a class="footer-link" href="https://cn.xauat.edu.cn/" target="_blank" rel="noopener">
          西安建筑科技大学
        </a>
        <a class="footer-link" href="https://gitee.com/XAUATiOSClub" target="_blank" rel="noopener">
          Gitee 组织
        </a>
        <a class="footer-link" href="https://beian.miit.gov.cn/" target="_blank" rel="noopener">
          陕ICP备2024031872号
        </a>
      </nav>
    </footer>
  </div>
</template>

<script setup lang="ts">
const currentYear = new Date().getFullYear()
</script>

<style scoped>
.footer-link {
  color: inherit;
  text-decoration: none;
  transition: color 0.2s ease;
}

.footer-link:hover {
  color: var(--accent);
}

.footer-link:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 4px;
  border-radius: 4px;
}
</style>
