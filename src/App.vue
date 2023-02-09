<script lang="ts">
import axios from "axios";
import { ref } from "vue";

export default {
  data() {
    return {
      themeStyles: {},
      logMsg: ref(""),
      loading: false,
      myTheme: ref("dark"),
    };
  },
  emits: ["logMessage"],
};
</script>

<script setup lang="ts">
import Form from "./components/Form.vue";
</script>

<template>
  <div
    :data-theme="myTheme"
    :class="myTheme === 'light' ? 'light-bg' : 'dark-bg'"
    style="height: 100%; width: 100%; position: absolute"
  >
    <header></header>
    <section></section>
    <main class="container">
      <details>
        <summary>📣 ข้อความสำคัญโปรดอ่านด้วยครับ</summary>
        <p style="text-align: center" class="container thai-p">
          สวัสดีครับอาจาร์ย ถ้าเกิดอาจาร์ยเห็น message ด้านล่างขวา
          เขียนประมาณว่า "You have exceeded the MONTHLY quota for Characters on
          your current plan, BASIC" ไม่ใช่ว่า app ผมมันทำงานไม่ได้นะครับ
          แต่หมายความ API ที่ใช่มันเกิน limit ที่ว่ามาแล้วในเดือนนี้ (500
          requests per month) ถ้าอาจาร์ยมาตรวจงานในเดือน ก.พ นี้
          รบกวนช่วยมาตรวจอีกทีในวันที่ 10 มี.ค ด้วยครับ
          ขออภัยในความไม่สะดวกด้วยนะครับ ขอบคุณครับ 🙏
        </p>
      </details>
      <div class="flex-inline">
        <div class="">
          <h1>Translate Me!</h1>
        </div>
        <div
          class="btn"
          @click="
            () =>
              myTheme === 'light' ? (myTheme = 'dark') : (myTheme = 'light')
          "
        >
          <a role="button" style="text-transform: uppercase">{{ myTheme }}</a>
        </div>
      </div>
      <hr />
      <Form
        @log-message="(msg: string, loaded?: boolean) => (logMsg = msg)"
      ></Form>
    </main>
    <footer class="buttom">
      <button
        v-if="logMsg !== ''"
        role="button"
        aria-busy="true"
        class="secondary outline"
      >
        {{ logMsg }}
      </button>
    </footer>
  </div>
</template>

<style scoped lang="css">
@import url("https://fonts.googleapis.com/css2?family=Kanit:wght@200&family=Monoton&display=swap");
.thai-p {
  font-family: "Kanit", sans-serif;
}

.light-bg {
  background-color: #fff;
}

.dark-bg {
  background-color: #111920;
}

.buttom {
  position: absolute;
  bottom: 0;
  right: 0;
  margin-right: 1rem;
  margin-bottom: 1rem;
}

.btn {
  cursor: pointer;
}

.flex-inline {
  display: flex;
  gap: 2rem;
}
</style>
