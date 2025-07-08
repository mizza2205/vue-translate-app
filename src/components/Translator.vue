<template>
  <div class="translator">
    <h2>Multi-Language Translator</h2>
    <input v-model="text" placeholder="Enter English text" />
    <select v-model="targetLang">
      <option v-for="(name, code) in languages" :key="code" :value="code">
        {{ name }}
      </option>
    </select>
    <button @click="translateText">Translate</button>
    <p v-if="translatedText">
      <strong>Translated:</strong> {{ translatedText }}
      <button @click="speakTranslation" title="Speak">🔊</button>
    </p>
    <p v-if="error" style="color: red;">{{ error }}</p>
    <div v-if="history.length > 0">
      <h3>Translation History (Last 10)</h3>
      <ul>
        <li v-for="(item, index) in history" :key="index">
          {{ item.english }} → {{ item.hindi }} ({{ languages[item.lang] || item.lang }})
        </li>
      </ul>
      <button @click="clearHistory">Clear History</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      text: '',
      translatedText: '',
      targetLang: 'hi',
      error: '',
      history: [],
      voices: [],
      languages: {
        hi: 'Hindi',
        fr: 'French',
        es: 'Spanish',
        de: 'German',
        it: 'Italian',
        ru: 'Russian',
        ja: 'Japanese',
        zh: 'Chinese',
        ar: 'Arabic',
      }
    };
  },
  mounted() {
    const loadVoices = () => {
      const voices = speechSynthesis.getVoices();
      if (voices.length > 0) {
        this.voices = voices;
      } else {
        setTimeout(loadVoices, 200);
      }
    };
    loadVoices();
    if (speechSynthesis.onvoiceschanged !== undefined) {
      speechSynthesis.onvoiceschanged = () => {
        this.voices = speechSynthesis.getVoices();
      };
    }
    const storedHistory = localStorage.getItem('translationHistory');
    if (storedHistory) {
      this.history = JSON.parse(storedHistory);
    }
  },
  methods: {
    async translateText() {
      if (!this.text.trim()) {
        this.error = 'Please enter some text.';
        return;
      }
      try {
        this.error = '';
        const response = await fetch(
          `https://api.mymemory.translated.net/get?q=${encodeURIComponent(this.text)}&langpair=en|${this.targetLang}`
        );
        const data = await response.json();
        const translated = data.responseData.translatedText;
        this.translatedText = translated;
        this.history.unshift({
          english: this.text,
          hindi: translated,
          lang: this.targetLang
        });
        if (this.history.length > 10) this.history.pop();
        localStorage.setItem('translationHistory', JSON.stringify(this.history));
        this.text = '';
      } catch (err) {
        this.error = 'Translation failed. Please try again later.';
        console.error(err);
      }
    },
    clearHistory() {
      this.history = [];
      localStorage.removeItem('translationHistory');
    },
    speakTranslation() {
      if (!this.translatedText) return;
      const langMap = {
        hi: 'hi-IN',
        fr: 'fr-FR',
        es: 'es-ES',
        de: 'de-DE',
        it: 'it-IT',
        ru: 'ru-RU',
        ja: 'ja-JP',
        zh: 'zh-CN',
        ar: 'ar-SA'
      };
      const voiceLang = langMap[this.targetLang] || 'en-US';
      const voices = this.voices.length ? this.voices : speechSynthesis.getVoices();
      const utterance = new SpeechSynthesisUtterance(this.translatedText);
      let matchedVoice = voices.find(v => v.lang === voiceLang);
      if (!matchedVoice) {
        matchedVoice = voices.find(v => v.lang.startsWith(this.targetLang)) || voices[0];
      }
      if (matchedVoice) {
        console.log("🎤 Using voice:", matchedVoice.name, matchedVoice.lang);
        utterance.voice = matchedVoice;
      } else {
        console.warn("❌ No suitable voice found, using system default.");
      }
      utterance.lang = voiceLang;
      speechSynthesis.speak(utterance);
    }
  }
};
</script>
<style scoped>
.translator {
  max-width: 500px;
  margin: auto;
  padding: 1rem;
  border: 2px solid #ccc;
  border-radius: 10px;
  text-align: center;
}
input,
select {
  width: 80%;
  padding: 0.5rem;
  margin-bottom: 0.5rem;
}
button {
  padding: 0.5rem 1rem;
  margin: 0.5rem;
  cursor: pointer;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
}
ul {
  text-align: left;
  padding-left: 1rem;
}
</style>