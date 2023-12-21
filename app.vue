<template>
  <div>
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </div>
</template>

<script>
export default {
  setup() {
    useHead({
        title: "Hello World",
        meta: [
          { name: 'viewport', content: 'width=device-width, user-scalable=no' },
          { name: 'description', content: 'My amazing site.' }
        ],
        link: [
          { rel: 'stylesheet', href: 'https://api.mapbox.com/mapbox-gl-js/v2.14.1/mapbox-gl.css' },
        ],
        script: [
          { src: 'https://api.mapbox.com/mapbox-gl-js/v2.14.1/mapbox-gl.js', body: true },
        ],
    })
  }
}
</script>