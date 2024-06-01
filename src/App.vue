<template>
  <v-app>
    <!-- <v-textarea/> тут только для того, чтобы автоматически подтянулись стили -->
    <v-textarea style="display: none;"/>

    <v-main>
      <v-card style="margin: 50px auto; width: 800px;">
        <v-card-text>
          <RichTextEditor
              v-model="text"
              label="Поле ввода текста"
              hide-details
              :error-messages="getTextErrors()"
          />
        </v-card-text>
        <v-card-text style="padding-bottom: 0" v-if="text">Результат:</v-card-text>
        <v-card-text style="font-size: small;">{{ text }}</v-card-text>
      </v-card>
    </v-main>
  </v-app>
</template>

<script>
import RichTextEditor from "@/components/RichTextEditor.vue";

export default {
  name: 'App',

  components: {
    RichTextEditor
  },

  data: () => ({
    text: ''
  }),

  methods: {
    getTextErrors() {
      if (this.text.length === 0) {
        return ['Не указан текст']
      }
      return [];
    }
  }
};
</script>
