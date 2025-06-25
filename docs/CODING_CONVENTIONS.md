# Kodlama Kuralları

Bu belge, projedeki Vue.js bileşenleri için kodlama standartlarını ve en iyi uygulamaları içerir.

## Genel Kurallar

- **Dil:** Tüm kod ve yorumlar İngilizce olmalıdır.

## Vue.js Bileşenleri

### İsimlendirme

- **Bileşen Dosyaları:** Bileşen dosyaları `PascalCase` formatında isimlendirilmelidir. (örn. `VideoPlayer.vue`)
- **Kök Bileşenler:** Her bileşenin en üst seviye elementi için `class` ismi, bileşen ismiyle aynı ve `kebab-case` formatında olmalıdır. (örn. `video-player`)

### Yapı

- **Composition API:** Tüm bileşenler `<script setup>` ve Composition API kullanılarak yazılmalıdır.
- **Props Down, Events Up:** Bileşenler arası iletişimde "props down, events up" prensibine sadık kalınmalıdır. Bir alt bileşen, üst bileşenin durumunu doğrudan değiştirmemeli, bunun yerine bir olay (`emit`) yayınlamalıdır.
- **Reaktivite:** `ref` ve `reactive` doğru şekilde kullanılmalıdır. Genellikle, ilkel veri türleri (String, Number, Boolean) için `ref`, nesneler ve diziler için `reactive` tercih edilir.

### Örnek Bileşen Yapısı

```vue
<script setup>
import { ref, computed }.from 'vue'

// Props
const props = defineProps({
  someProp: {
    type: String,
    required: true,
  },
})

// Emits
const emit = defineEmits(['some-event'])

// State
const internalState = ref(0)

// Computed Properties
const formattedProp = computed(() => {
  return props.someProp.toUpperCase()
})

// Methods
function handleSomeAction() {
  emit('some-event', 'payload')
}
</script>

<template>
  <div class="my-component">
    <!-- ... -->
  </div>
</template>

<style scoped>
.my-component {
  /* ... */
}
</style>
```
