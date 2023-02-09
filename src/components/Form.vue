<script lang="ts">
import axios from "axios";
import { AxiosError } from "axios";
import { defineAsyncComponent } from "vue";

export default {
  setup() {},
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
            "76c2d6824emsh545f738d51c9119p1b26e7jsnb146b260feec",
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
    this.$emit(
      "logMessage",
      "Languages mode are fetching, please wait...",
      true
    );
    new Promise((resolve) => setTimeout(resolve, 2000)).then(() => {
      this.fetchtargetLang().then(() => this.$emit("logMessage", "", false));
    });
  },
  methods: {
    async fetchtargetLang() {
      const apiTarget =
        "https://google-translate1.p.rapidapi.com/language/translate/v2/languages";
      this.options.url = apiTarget;
      this.options.method = "GET";
      axios
        .request(this.options)
        .then((res) => {
          try {
            this.targetOpts = res.data.data.languages;
            console.log(this.targetOpts);
          } catch (err) {
            console.error(err);
          }
        })
        .catch((err: AxiosError) => {
          this.ErrorHandlerMiddleware(err as AxiosError);
        });
      this.options.url = "";
      this.options.method = "";
    },
    async onDetectedLang(text: string) {
      this.$emit("logMessage", "Detecting language... ");

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
              this.$emit(
                "logMessage",
                `Language detected on: "${this.translatedObj.source}"`
              );

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
          .catch((err: AxiosError) => {
            this.ErrorHandlerMiddleware(err as AxiosError);
          });
      } else {
        console.log("text not found");
      }
    },

    async onTranslated() {
      this.options.url =
        "https://google-translate1.p.rapidapi.com/language/translate/v2";
      this.options.method = "POST";
      let validation = await this.ErrorHandlerMiddleware();
      if (validation === false) {
        await new Promise((resolve) => setTimeout(resolve, 4000)).then(() => {
          this.$emit("logMessage", ``);
        });
      }

      if (this.translatedObj && validation === true) {
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
                this.$emit("logMessage", `Translated successfully. `);
                new Promise((resolve) => setTimeout(resolve, 1500)).then(() =>
                  this.$emit("logMessage", ``)
                );
              } catch (err) {
                console.error(err);
              }
            })
            .catch((err: AxiosError) => {
              this.ErrorHandlerMiddleware(err as AxiosError);
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
        "X-RapidAPI-Key": "76c2d6824emsh545f738d51c9119p1b26e7jsnb146b260feec",
        "X-RapidAPI-Host": "google-translate1.p.rapidapi.com",
      };
    },

    async ErrorHandlerMiddleware(err?: AxiosError) {
      if (err?.code) {
        this.$emit("logMessage", err.response?.data);
        await new Promise((resolve) => setTimeout(resolve, 10000)).then(() => {
          this.$emit("logMessage", ``);
        });
        return false;
      } else if (
        this.translatedObj.text === "" ||
        this.translatedObj.target === "" ||
        this.translatedObj.source === ""
      ) {
        this.$emit(
          "logMessage",
          "Some of required inputs is empty please try again"
        );
        return false;
      } else if (this.translatedObj.target === this.translatedObj.source) {
        this.$emit(
          "logMessage",
          "Source and target language cannot be same..., but why you want to translate the same language you wrote by the way."
        );
        return false;
      } else {
        return true;
      }
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
<template>
  <div>
    <form @submit.prevent="onSubmitTranslated">
      <label for="target">Pick a language to translate</label>
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
</template>
<style lang="css"></style>
