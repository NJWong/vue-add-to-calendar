<template>
  <div class="default">
    <script type="text/javascript">
      window.addeventasync = function() {
        addeventatc.settings({
          appleical: { show: true, text: 'iCalendar' },
          google: { show: true, text: 'Google calendar' },
          office365: { show: false },
          outlook: { show: false },
          outlookcom: { show: false },
          yahoo: { show: false },
        });
      };
    </script>
    <nuxt />
  </div>
</template>

<style>
html {
  font-family: 'Source Sans Pro', -apple-system, BlinkMacSystemFont, 'Segoe UI',
    Roboto, 'Helvetica Neue', Arial, sans-serif;
  font-size: 16px;
  word-spacing: 1px;
  -ms-text-size-adjust: 100%;
  -webkit-text-size-adjust: 100%;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  box-sizing: border-box;
}

*,
*:before,
*:after {
  box-sizing: border-box;
  margin: 0;
}
</style>
