<script setup>
const languages = ref([
  { name: 'Choose Locale', locale: '', selected: true },
  { name: 'English (en-GB)', locale: 'en-GB' },
  { name: 'French (fr-CA)', locale: 'fr-CA' },
  { name: 'German (de-DA)', locale: 'de-DE' },
  { name: 'Italian (it-IT)', locale: 'it-IT' },
  { name: 'Spanish (es-ES)', locale: 'es-ES' },
]);

const loading = ref(false);
const text = ref('');
const locale = ref('');
const translation = ref('');

async function onSubmit() {
  if (!text.value || !locale.value) {
    alert('Text and locale required');
    return;
  }
  // TODO Convert to use pending maybe? Or some sort of global loader state?
  loading.value = true;
  // TODO Convert to composable maybe `useTranslation(text, locale)`
  const { data: translatedText } = await useFetch('/api/translate', {
    method: 'post',
    query: {
      text,
      locale,
    },
  });
  loading.value = false;
  translation.value = translatedText.value || '';
}

function onCopy() {
  if (location.protocol !== 'https:') {
    alert('Copy will only work over https');
    return;
  }
  navigator.clipboard.writeText(translation.value);
}
</script>


<template>
  <section class="text-translate container" :class="{
    'container__loading': loading === true,
  }">
    <h2 class="text-translate__heading">Translate Text</h2>
    <form @submit.prevent="onSubmit">
        <label for="translate-text">Text (in English)</label>
        <textarea name="translate-text" v-model="text"></textarea>

        <label for="translate-locale">Translate To</label>
        <select name="translate-locale" v-model="locale">
          <option
            v-for="language in languages"
            :key="language.locale"
            :value="language.locale"
            :selected="language.selected">
              {{ language.name }}
          </option>
        </select>

        <div class="buttons">
          <button type="submit">Submit</button>
          <button type="reset">Reset</button>
        </div>
      </form>

      <div class="text-translate__translation" v-if="translation">
          <label for="translate-text-translated">Translation</label>
          <textarea
            name="translate-text-translated"
            class="text-translate__translation-text"
            readonly
            v-model="translation"></textarea>
          <div class="buttons">
            <button @click="onCopy">Copy Translation</button>
          </div>
      </div>
    </section>
</template>

<style lang="scss">
.text-translate {
    &__heading {
      color: var(--font-heading-color);
      text-align: center;
    }

    &__translation {
        margin-top: var(--spacer-primary);
    }

    &__translation-text {
        margin-bottom: var(--spacer-secondary);
    }
}
</style>
