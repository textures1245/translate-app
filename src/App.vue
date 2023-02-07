<script lang="ts">
import axios from "axios";

export default {
  data() {
    return {
      onLoaded: false,
      encodedParams: new URLSearchParams(),
      options: {
        method: "",
        url: "",
        headers: {
          "Accept-Encoding": "application/gzip",
          "X-RapidAPI-Key":
            "84214a2573msh8eef85f0805fb7bp11f306jsn01780e8a3986",
          "X-RapidAPI-Host": "google-translate1.p.rapidapi.com",
        },
      },
      translatedObj: {
        translatedText: "",
        text: "",
        source: "",
        target: "",
      },
      targetOpts: [
        {
          language: "",
        },
      ],
    };
  },
  watch: {
    "translatedObj.target"(newtarget) {
      if (newtarget !== "") {
        this.translatedObj.translatedText = "";
        this.translatedObj.target = newtarget;
        console.log(this.translatedObj.target);
      }
    },

    "translatedObj.text"() {
      this.translatedObj.translatedText = "";
    },
  },
  mounted() {
    this.fetchtargetLang();
  },
  methods: {
    async fetchtargetLang() {
      const apiTarget =
        "https://google-translate1.p.rapidapi.com/language/translate/v2/languages";
      this.options.url = apiTarget;
      this.options.method = "GET";
      axios.request(this.options).then((res) => {
        try {
          this.targetOpts = res.data.data.languages;
          console.log(this.targetOpts);
        } catch (err) {
          console.error(err);
        }
      });
      this.options.url = "";
      this.options.method = "";
    },
    async onDetectedLang(text: string) {
      const apiTarget =
        "https://google-translate1.p.rapidapi.com/language/translate/v2/detect";
      this.options.url = apiTarget;
      this.options.method = "POST";

      if (this.translatedObj.text) {
        this.encodedParams.append("q", text);

        //- assigned return content type
        Object.assign(this.options.headers, {
          "content-type": "application/x-www-form-urlencoded",
        });
        Object.assign(this.options, {
          data: this.encodedParams,
        });

        await axios
          .request(this.options)
          .then((response) => {
            try {
              this.translatedObj.source =
                response.data.data.detections[0][0].language;
              console.log(this.translatedObj.source);

              //* reset
              this.encodedParams = new URLSearchParams(); // reset params
              this.options.url = "";
              this.options.method = "";

              this.onTranslated().then(() => {
                console.log("Translated Success");
              });
            } catch (err) {
              console.error(err);
            }
          })
          .catch(function (error) {
            console.error(error);
          });
      } else {
        console.log("text not found");
      }
    },

    async onTranslated() {
      this.options.url =
        "https://google-translate1.p.rapidapi.com/language/translate/v2";
      this.options.method = "POST";

      if (this.translatedObj) {
        try {
          this.encodedParams.append("q", this.translatedObj.text);
          this.encodedParams.append("target", this.translatedObj.target);
          this.encodedParams.append("source", this.translatedObj.source);
          Object.assign(this.options, {
            data: this.encodedParams,
          });

          axios
            .request(this.options)
            .then((response) => {
              try {
                console.log(response.data);
                console.log(response.data.data.translations[0]);
                this.translatedObj.translatedText =
                  response.data.data.translations[0].translatedText;
                console.log(this.translatedObj.translatedText);
              } catch (err) {
                console.error(err);
              }
            })
            .catch(function (error) {
              console.error(error);
            });
        } catch (err) {
          console.error(err);
        }
      } else {
        console.error("Object not found!");
      }

      //- resets
      this.encodedParams = new URLSearchParams(); // reset params
      this.options.url = "";
      this.options.method = "";

      //- reset both input and headers
      this.options.headers = {
        "Accept-Encoding": "application/gzip",
        "X-RapidAPI-Key": "84214a2573msh8eef85f0805fb7bp11f306jsn01780e8a3986",
        "X-RapidAPI-Host": "google-translate1.p.rapidapi.com",
      };
    },

    async onSubmitTranslated() {
      this.onLoaded = true;
      await this.onDetectedLang(this.translatedObj.text).then(() => {
        console.log(
          `detected! on lang on ${this.translatedObj.target} then translated to ${this.translatedObj.source} `
        );
        this.onLoaded = false;
      });
    },
  },
};
</script>

<script setup lang="ts">
import Form from "./components/Form.vue";
</script>

<template>
  <header></header>
  <section></section>
  <main class="container">
    <h1>Translate Me!</h1>
    <hr />
    <div>
      <form @submit.prevent="onSubmitTranslated">
        <select
          required
          v-if="targetOpts.length > 0"
          name="target"
          v-model="translatedObj.target"
          id="target"
        >
          <option :value="s.language" v-for="s in targetOpts">
            {{ s.language }}
          </option>
        </select>
        <input
          required
          type="text"
          v-model="translatedObj.text"
          placeholder="Type something you want to translate!"
        />
        <button type="submit" role="button">Translate it!</button>
      </form>
      <section v-if="onLoaded">
        <progress></progress>
      </section>
      <section v-if="translatedObj.translatedText">
        <details>
          <summary>Translate: {{ translatedObj.translatedText }}</summary>
          <ul>
            <li>Language Input Detect: {{ translatedObj.source }}</li>
            <li>Language to Translated {{ translatedObj.target }}</li>
            <li>Original text: {{ translatedObj.text }}</li>
          </ul>
        </details>
      </section>
    </div>
  </main>
</template>

<style scoped></style>
