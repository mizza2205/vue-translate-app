<template>
  <div id="app">
    <Translator />
  </div>
</template>
<script>
import Translator from './components/Translator.vue';
export default {
  name: 'App',
  components: {
    Translator
  },
  mounted() {
    // ✅ Voice load karne ke liye setup
    const loadVoices = () => {
      const voices = speechSynthesis.getVoices();
      if (voices.length > 0) {
        console.log("✅ Voices loaded:", voices);
      } else {
        // Retry if not loaded yet (especially for Chrome & Incognito)
        setTimeout(loadVoices, 200);
      }
    };
    loadVoices();
    if (speechSynthesis.onvoiceschanged !== undefined) {
      speechSynthesis.onvoiceschanged = () => {
        loadVoices();
      };
    }
  }
};
</script>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  margin-top: 30px;
}
</style>
