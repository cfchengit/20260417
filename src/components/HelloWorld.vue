<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'

// 工具列的狀態
const isDarkMode = ref(false)
const showZhuyin = ref(false)
const isCountdown = ref(false)

// 時間與倒數計時相關變數
const currentTime = ref('')
const inputMinutes = ref(60) // 預設倒數分鐘數設定
const countdownSeconds = ref(3600) // 預設倒數時間 (60分鐘)
let timerInterval = null

// 表單資料綁定
const formData = ref({
  time: '',
  subject: '',
  proctor: ''
})

// 更新當前時間
const updateTime = () => {
  currentTime.value = new Date().toLocaleTimeString('zh-TW', { hour12: false })
}

// 設定倒數計時功能
const setCountdown = () => {
  countdownSeconds.value = (inputMinutes.value || 0) * 60
}

// 生命週期掛載與清除計時器
onMounted(() => {
  updateTime()
  timerInterval = setInterval(() => {
    updateTime()
    if (isCountdown.value && countdownSeconds.value > 0) {
      countdownSeconds.value--
    }
  }, 1000)
})

onUnmounted(() => {
  if (timerInterval) clearInterval(timerInterval)
})

// 格式化倒數顯示時間 (HH:MM:SS)
const formattedCountdown = computed(() => {
  const h = Math.floor(countdownSeconds.value / 3600).toString().padStart(2, '0')
  const m = Math.floor((countdownSeconds.value % 3600) / 60).toString().padStart(2, '0')
  const s = (countdownSeconds.value % 60).toString().padStart(2, '0')
  return `${h}:${m}:${s}`
})
</script>

<template>
  <!-- 透過綁定 Class 實現深淺色模式與注音模式切換 -->
  <div class="proctor-app" :class="{ 'dark-theme': isDarkMode, 'zhuyin-theme': showZhuyin }">
    <header class="header">
      <a href="https://www.et.tku.edu.tw/" target="_blank" class="tku-btn">
        <ruby>淡<rt>ㄉㄢˋ</rt>江<rt>ㄐㄧㄤ</rt>教<rt>ㄐㄧㄠˋ</rt>科<rt>ㄎㄜ</rt>系<rt>ㄒㄧˋ</rt></ruby>
      </a>
      <h1 class="title">
        <ruby>互<rt>ㄏㄨˋ</rt>動<rt>ㄉㄨㄥˋ</rt>程<rt>ㄔㄥˊ</rt>式<rt>ㄕˋ</rt>設<rt>ㄕㄜˋ</rt>計<rt>ㄐㄧˋ</rt></ruby>_12345
      </h1>
    </header>

    <!-- 小工具操作按鈕 -->
    <section class="toolbar">
      <button @click="isCountdown = !isCountdown" class="tool-btn">
        <ruby>時<rt>ㄕˊ</rt>間<rt>ㄐㄧㄢ</rt>切<rt>ㄑㄧㄝ</rt>換<rt>ㄏㄨㄢˋ</rt></ruby>
      </button>
      <button @click="showZhuyin = !showZhuyin" class="tool-btn">
        <ruby>注<rt>ㄓㄨˋ</rt>音<rt>ㄧㄣ</rt>開<rt>ㄎㄞ</rt>關<rt>ㄍㄨㄢ</rt></ruby>
      </button>
      <button @click="isDarkMode = !isDarkMode" class="tool-btn">
        <ruby>黑<rt>ㄏㄟ</rt>白<rt>ㄅㄞˊ</rt>主<rt>ㄓㄨˇ</rt>題<rt>ㄊㄧˊ</rt></ruby>
      </button>
    </section>

    <!-- 時鐘與倒數顯示區域 -->
    <section class="time-display">
      <div v-if="!isCountdown">
        <h2><ruby>現<rt>ㄒㄧㄢˋ</rt>在<rt>ㄗㄞˋ</rt>時<rt>ㄕˊ</rt>間<rt>ㄐㄧㄢ</rt></ruby></h2>
        <div class="clock">{{ currentTime }}</div>
      </div>
      <div v-else>
        <h2><ruby>倒<rt>ㄉㄠˋ</rt>數<rt>ㄕㄨˇ</rt>計<rt>ㄐㄧˋ</rt>時<rt>ㄕˊ</rt></ruby></h2>
        <div class="clock">{{ formattedCountdown }}</div>
        <div class="countdown-settings">
          <input type="number" v-model="inputMinutes" placeholder="分鐘" class="time-input" />
          <button @click="setCountdown" class="set-btn">設定(分鐘)</button>
        </div>
      </div>
    </section>

    <!-- 考試資訊輸入區域 -->
    <section class="form-section">
      <h2><ruby>資<rt>ㄗ</rt>料<rt>ㄌㄧㄠˋ</rt>輸<rt>ㄕㄨ</rt>入<rt>ㄖㄨˋ</rt></ruby></h2>
      <div class="form-inputs">
        <div class="input-group">
          <label>時間:</label>
          <select v-model="formData.time">
            <option value="" disabled>請選擇考試時間</option>
            <option value="08:10 - 10:00 (第1-2節)">08:10 - 10:00 (第1-2節)</option>
            <option value="10:10 - 12:00 (第3-4節)">10:10 - 12:00 (第3-4節)</option>
            <option value="13:10 - 15:00 (第5-6節)">13:10 - 15:00 (第5-6節)</option>
            <option value="15:10 - 17:00 (第7-8節)">15:10 - 17:00 (第7-8節)</option>
            <option value="18:10 - 20:00 (夜間部)">18:10 - 20:00 (夜間部)</option>
          </select>
        </div>
        <div class="input-group">
          <label>科目:</label>
          <input type="text" v-model="formData.subject" placeholder="請輸入考試科目" />
        </div>
        <div class="input-group">
          <label>監考老師:</label>
          <input type="text" v-model="formData.proctor" placeholder="請輸入監考老師姓名" />
        </div>
      </div>
    </section>

    <!-- 考試資訊展示區域 -->
    <section class="display-section">
      <h2><ruby>考<rt>ㄎㄠˇ</rt>試<rt>ㄕˋ</rt>資<rt>ㄗ</rt>料<rt>ㄌㄧㄠˋ</rt>顯<rt>ㄒㄧㄢˇ</rt>示<rt>ㄕˋ</rt></ruby></h2>
      <div class="display-card">
        <p><strong>時間:</strong> {{ formData.time || '尚未輸入' }}</p>
        <p><strong>科目:</strong> {{ formData.subject || '尚未輸入' }}</p>
        <p><strong>監考老師:</strong> {{ formData.proctor || '尚未輸入' }}</p>
      </div>
    </section>
  </div>
</template>

<style scoped>
/* 基本與淺色佈局 */
.proctor-app {
  min-height: 100vh;
  width: 100%;
  padding: 20px;
  font-family: 'Helvetica Neue', Arial, sans-serif;
  transition: background-color 0.3s, color 0.3s;
  box-sizing: border-box;
  background-color: #ffffff;
  color: #333333;
}

/* 深色模式(黑底白字) */
.proctor-app.dark-theme {
  background-color: #1a1a1a;
  color: #f0f0f0;
}

/* 注音設定 - 預設隱藏 */
rt {
  display: none;
  font-size: 0.6em;
  color: #ff6b6b;
  text-align: center;
}
.zhuyin-theme rt {
  display: block;
}
ruby {
  ruby-align: center;
}

/* 頂部排版 */
.header {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 20px;
  padding-bottom: 20px;
  border-bottom: 2px solid #ccc;
}
.title {
  margin: 0;
  flex-grow: 1;
  text-align: center;
  font-size: 24px;
}
.tku-btn {
  text-decoration: none;
  background-color: #004d99;
  color: white;
  padding: 10px 20px;
  border-radius: 8px;
  font-weight: bold;
  transition: background-color 0.2s;
}
.tku-btn:hover {
  background-color: #003366;
}

/* 工具列按鈕 */
.toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  margin: 20px 0;
  justify-content: center;
}
.tool-btn {
  padding: 10px 20px;
  font-size: 16px;
  border: 2px solid #004d99;
  background-color: transparent;
  color: #004d99;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
  font-weight: bold;
}
.tool-btn:hover {
  background-color: #004d99;
  color: #fff;
}
.dark-theme .tool-btn {
  border-color: #4da6ff;
  color: #4da6ff;
}
.dark-theme .tool-btn:hover {
  background-color: #4da6ff;
  color: #1a1a1a;
}

/* 時間與倒數區塊 */
.time-display {
  text-align: center;
  margin: 40px 0;
}
.clock {
  font-size: 48px;
  font-weight: bold;
  font-family: monospace;
  margin: 20px 0;
}
.countdown-settings {
  display: flex;
  justify-content: center;
  gap: 10px;
}

/* 表單輸入區域 */
input, select {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  width: 100%;
  box-sizing: border-box;
  font-family: inherit;
  background-color: #fff;
}
.time-input {
  width: 120px;
}
.set-btn {
  padding: 10px 15px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}
.dark-theme input, .dark-theme select {
  background-color: #333;
  color: #fff;
  border-color: #555;
}

.form-section, .display-section {
  max-width: 600px;
  margin: 0 auto 30px auto;
  background-color: #f9f9f9;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}
.dark-theme .form-section, .dark-theme .display-section {
  background-color: #2a2a2a;
  box-shadow: 0 4px 6px rgba(0,0,0,0.5);
}
.form-inputs {
  display: flex;
  flex-direction: column;
  gap: 15px;
}
.input-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
  text-align: left;
}
.input-group label {
  font-weight: bold;
}
.display-card p {
  font-size: 18px;
  margin: 10px 0;
  text-align: left;
}

/* 響應式 (RWD) 對應桌機版型 */
@media (min-width: 768px) {
  .header {
    flex-wrap: nowrap;
  }
  .title {
    text-align: center;
  }
}
</style>
