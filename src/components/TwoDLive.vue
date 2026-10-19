<template>
  <div class="app">
    <div class="container">
      <!-- Left Panel -->
      <div class="left">
        <ul>
          <li>🎓 EDUCATIONAL PURPOSE ONLY</li>
          <li>📊 Market Data & Probability Learning</li>
          <li>🚫 NOT Gambling Advice / Promotion</li>
        </ul>

        <div class="time-box">
          <div class="date-row">
            <div class="date-box">
              <span class="text-main">စျေးကွက်အရ ထိပ်</span>
              <div class="date-box-value">{{ randomSeqKey }}</div>
            </div>
          </div>
        </div>

        <!-- ✅ "Set" & "Value" alignment left/right (Safe Optional Chaining) -->
        <div class="live-info">
          <div class="info-item">
            <p class="info-label">Set</p>
            <p class="info-value">{{ twoDeeLive?.live?.set || "--" }}</p>
          </div>

          <div class="info-item">
            <p class="info-label">Value</p>
            <p class="info-value">{{ twoDeeLive?.live?.value || "--" }}</p>
          </div>
        </div>

        <div class="subscribe">
          <button :class="{ green: isGreen, red: !isGreen }">
            🔔 LIKE & SUBSCRIBE
          </button>
        </div>
      </div>

      <!-- ✅ RIGHT Panel -->
      <div class="right" v-if="twoDeeLive">
        <div class="phone">
          <h1 class="big-number">` {{ twoDeeLive?.live?.twod || "--" }} `</h1>
          <p class="updated">Updated: {{ updatedTime }}</p>

          <!-- ✅ အပေါ်ကတ်ကြီး ၂ ခု (100% Full Size) -->
          <div class="card">
            <h2>12:01 PM</h2>
            <div class="info">
              <div class="info-item">
                <h3>Set</h3>
                <p>{{ twoDeeLive?.result?.[1]?.set || "--" }}</p>
              </div>
              <div class="info-item">
                <h3>Value</h3>
                <p>{{ twoDeeLive?.result?.[1]?.value || "--" }}</p>
              </div>
              <div class="info-item">
                <h3>Result</h3>
                <p>{{ twoDeeLive?.result?.[1]?.twod || "--" }}</p>
              </div>
            </div>
          </div>

          <div class="card">
            <h2>4:30 PM</h2>
            <div class="info">
              <div class="info-item">
                <h3>Set</h3>
                <p>{{ twoDeeLive?.result?.[3]?.set || "--" }}</p>
              </div>
              <div class="info-item">
                <h3>Value</h3>
                <p>{{ twoDeeLive?.result?.[3]?.value || "--" }}</p>
              </div>
              <div class="info-item">
                <h3>Result</h3>
                <p>{{ twoDeeLive?.result?.[3]?.twod || "--" }}</p>
              </div>
            </div>
          </div>

          <!-- ✅ အောက်ကတ် ၂ ခု (အပေါ်ကတ်များ၏ ၃ ပုံ ၂ ပုံ - 2/3 Size) -->
          <div class="card sub-card">
            <h2>03:00 PM</h2>
            <div class="info">
              <div class="info-item">
                <h3>Set</h3>
                <p>{{ twoDeeLive?.result?.[2]?.set || "--" }}</p>
              </div>
              <div class="info-item">
                <h3>Value</h3>
                <p>{{ twoDeeLive?.result?.[2]?.value || "--" }}</p>
              </div>
              <div class="info-item">
                <h3>Result</h3>
                <p>{{ twoDeeLive?.result?.[2]?.twod || "--" }}</p>
              </div>
            </div>
          </div>

          <div class="card sub-card">
            <h2>11:00 AM</h2>
            <div class="info">
              <div class="info-item">
                <h3>Set</h3>
                <p>{{ twoDeeLive?.result?.[0]?.set || "--" }}</p>
              </div>
              <div class="info-item">
                <h3>Value</h3>
                <p>{{ twoDeeLive?.result?.[0]?.value || "--" }}</p>
              </div>
              <div class="info-item">
                <h3>Result</h3>
                <p>{{ twoDeeLive?.result?.[0]?.twod || "--" }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ✅ CENTER Panel Now Right -->
      <div class="center">
        <div class="center-content">
          <!-- 🙏 ပုံထည့်ထားတဲ့အပိုင်း -->
          <img
            src="@/assets/photo.png"
            alt="Respect Image"
            class="respect-img"
          />
          <div class="twod">
            {{ myanmarDate }} <br />
            <span class="day-session">
              {{ dayInMyanmar }}နေ့ <br />{{ session }}
            </span>
          </div>
          <p class="main-time">{{ mainTime }}</p>
          <p class="countdown">{{ countdown }}</p>
          <div>
            <div class="currency-box" v-if="twoDeeLive">
              <span class="currency-label">` {{ currentTwoD }} `</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from "vue";

export default {
  setup() {
    // 🌟 Reactive Data
    const today = ref("");
    const updatedTime = ref("");
    const countdown = ref("");
    const isGreen = ref(false);
    const twoDeeLive = ref(null);
    const error = ref("");
    const isAnimated = ref(false);
    const previousLiveData = ref(null);
    const mainTime = ref("");
    const dayInMyanmar = ref("");
    const session = ref(""); // မနက်ပိုင်း / ညနေပိုင်း
    const myanmarDate = ref("");

    // For API data from backend api laravel
    const currencies = ref([]);
    const loading = ref(true);
    const apiUrl = process.env.VUE_APP_API_URRS;
    const accessToken = process.env.VUE_APP_API_KEY;

    // ✅ reactive variables for keys
    const randomKey = ref("");
    const randomSeqKey = ref("");

    // Intervals & Timeouts Cleanup မှတ်ထားရန်
    const intervalIds = [];
    const timeoutIds = [];

    // ✅ Weekend စစ်ဆေးခြင်း
    function isWeekend() {
      const day = new Date().getDay();
      return day === 0 || day === 6; // Sunday = 0, Saturday = 6
    }

    // ✅ Random key generator (တူတဲ့ digit မပါအောင် ၂ လုံး random ထုတ်)
    function generateUniqueKey() {
      let key;
      do {
        const temp = [];
        for (let i = 0; i < 2; i++) {
          temp.push(Math.floor(Math.random() * 10));
        }
        key = temp;
      } while (new Set(key).size < 2);
      return key.join("-");
    }

    // ✅ 3-digit sequential key generator
    function generateSequentialKey() {
      const start = Math.floor(Math.random() * 10);
      const digits = [start, (start + 1) % 10, (start + 2) % 10];
      return digits.join("-");
    }

    // ✅ Random Key တစ်ခု ထုတ်ပြီး randomKey ထဲသိမ်း (Duplicate ဖြစ်နေသည်ကို ပေါင်းထားသည်)
    function generateRandomKey() {
      randomKey.value = isWeekend() ? "CLOSED" : generateUniqueKey();
    }

    // ✅ Random Sequential Key တစ်ခု ထုတ်ပြီး randomSeqKey ထဲသိမ်း
    function generateRandomSequentialKey() {
      randomSeqKey.value = isWeekend() ? "CLOSED" : generateSequentialKey();
    }

    // ✅ Computed for currentTwoD
    const currentTwoD = computed(() => {
      if (!twoDeeLive.value || !twoDeeLive.value.result) return "--";
      const now = new Date();
      const cutoff = new Date();
      cutoff.setHours(14, 0, 0, 0);
      return now < cutoff
        ? twoDeeLive.value.result[1]?.twod || "--"
        : twoDeeLive.value.result[3]?.twod || "--";
    });

    // 🔹 Methods
    const convertToMyanmarNumber = (num) => {
      const myanmarNumbers = ["၀", "၁", "၂", "၃", "၄", "၅", "၆", "၇", "၈", "၉"];
      return (
        num
          ?.toString()
          .split("")
          .map((d) => myanmarNumbers[Number(d)] ?? d)
          .join("") || ""
      );
    };

    const updateDay = () => {
      const daysMm = [
        "တနင်္ဂနွေ",
        "တနင်္လာ",
        "အင်္ဂါ",
        "ဗုဒ္ဓဟူး",
        "ကြာသပတေး",
        "သောကြာ",
        "စနေ",
      ];
      const todayIdx = new Date().getDay();
      dayInMyanmar.value = daysMm[todayIdx];
    };

    const updateDateTimeMyanmar = () => {
      const now = new Date();
      const options = { year: "numeric", month: "numeric", day: "numeric" };
      myanmarDate.value = new Intl.DateTimeFormat("my-MM", options).format(now);
    };

    const updateSession = () => {
      const hour = new Date().getHours();
      if (hour >= 10 && hour < 14) session.value = "မနက်ပိုင်း";
      else if (hour >= 14) session.value = "ညနေပိုင်း";
      else session.value = "မနက်ပိုင်း";
    };

    const updateDateTime = () => {
      const now = new Date();
      today.value = now.toLocaleDateString("en-GB");
      updatedTime.value = now.toLocaleString();
    };

    const updateCountdown = () => {
      const now = new Date();
      const hours = String(now.getHours()).padStart(2, "0");
      const minutes = String(now.getMinutes()).padStart(2, "0");
      const seconds = String(now.getSeconds()).padStart(2, "0");
      countdown.value = `${hours}:${minutes}:${seconds}`;
    };

    const loadTwoDeeLive = async () => {
      try {
        const response = await fetch("https://api.thaistock2d.com/live");
        const datas = await response.json();
        if (
          JSON.stringify(datas.live) !== JSON.stringify(previousLiveData.value)
        ) {
          isAnimated.value = true;
          const animTimeout = setTimeout(() => {
            isAnimated.value = false;
          }, 10000);
          timeoutIds.push(animTimeout);
        }
        previousLiveData.value = datas.live;
        twoDeeLive.value = datas;
      } catch (err) {
        error.value = err.message;
      }
    };

    // Fetch data from backend API
    const fetchData = async () => {
      if (!apiUrl) {
        loading.value = false;
        return;
      }
      try {
        const response = await fetch(apiUrl, {
          method: "GET",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${accessToken}`,
          },
        });

        if (!response.ok) {
          throw new Error("Network response was not ok");
        }

        const jsonData = await response.json();
        currencies.value = jsonData;
      } catch (err) {
        error.value = "Failed to load data: " + err.message;
      } finally {
        loading.value = false;
      }
    };

    const updateMainTime = () => {
      const hour = new Date().getHours();
      const time12 = "12:01 PM";
      const time430 = "04:30 PM";

      mainTime.value = hour >= 7 && hour < 14 ? time12 : time430;
    };

    // ✅ Schedule key auto update (တစ်နေ့ ၂ ကြိမ် 10:00 AM နှင့် 2:00 PM တွင် update လုပ်ရန်)
    function scheduleDailyUpdate(fn) {
      const now = new Date();
      const morning = new Date();
      morning.setHours(10, 0, 0, 0);
      const afternoon = new Date();
      afternoon.setHours(14, 0, 0, 0);

      let nextUpdate;
      if (now < morning) {
        nextUpdate = morning;
      } else if (now < afternoon) {
        nextUpdate = afternoon;
      } else {
        nextUpdate = new Date();
        nextUpdate.setDate(now.getDate() + 1);
        nextUpdate.setHours(10, 0, 0, 0);
      }

      const diff = nextUpdate - now;
      const tId = setTimeout(() => {
        fn();
        scheduleDailyUpdate(fn);
      }, diff);
      timeoutIds.push(tId);
    }

    // 🌟 Mounted
    onMounted(() => {
      if (screen.orientation && screen.orientation.lock) {
        screen.orientation
          .lock("landscape")
          .catch((err) => console.warn("Orientation lock failed:", err));
      }

      // ✅ Initial random keys generation and scheduling
      generateRandomKey();
      scheduleDailyUpdate(generateRandomKey);

      generateRandomSequentialKey();
      scheduleDailyUpdate(generateRandomSequentialKey);

      // Initial updates
      updateMainTime();
      updateDateTime();
      updateCountdown();
      loadTwoDeeLive();
      updateDay();
      updateDateTimeMyanmar();
      updateSession();

      // Fetch data from backend API
      fetchData();

      // Intervals
      intervalIds.push(setInterval(updateDateTime, 1000));
      intervalIds.push(setInterval(updateCountdown, 1000));
      intervalIds.push(setInterval(loadTwoDeeLive, 10000));
      intervalIds.push(setInterval(updateMainTime, 60000));
      intervalIds.push(
        setInterval(() => {
          isGreen.value = !isGreen.value;
        }, 2000),
      );
      intervalIds.push(
        setInterval(() => {
          updateDay();
          updateDateTimeMyanmar();
          updateSession();
        }, 60000),
      );
      intervalIds.push(
        setInterval(() => {
          fetchData();
        }, 1 * 60 * 1000),
      );
    });

    // 🌟 Cleanup on Unmount
    onUnmounted(() => {
      intervalIds.forEach((id) => clearInterval(id));
      timeoutIds.forEach((id) => clearTimeout(id));
    });

    return {
      today,
      updatedTime,
      countdown,
      isGreen,
      twoDeeLive,
      error,
      isAnimated,
      previousLiveData,
      mainTime,
      dayInMyanmar,
      session,
      myanmarDate,
      convertToMyanmarNumber,
      updateMainTime,
      updateDateTime,
      updateCountdown,
      loadTwoDeeLive,
      updateDay,
      updateDateTimeMyanmar,
      updateSession,
      currencies,
      loading,
      randomKey,
      randomSeqKey,
      currentTwoD,
    };
  },
};
</script>

<style>
html,
body,
.app {
  width: 100%;
  height: 100%;
  margin: 0;
  background: #f9f9f9;
  overflow: hidden;
  font-family: "Noto Sans Myanmar", sans-serif;
}

/* 🌕 Main Container */
.container {
  display: flex;
  justify-content: space-between;
  align-items: stretch;
  width: 100vw;
  height: 100vh;
  box-sizing: border-box;
}

/* 🌕 Panels */
.left,
.center,
.right {
  border-radius: 0;
  padding: 10px;
  box-sizing: border-box;
  background: #ffffff;
  font-weight: bold;
  border: none !important;
}

/* ✅ Panel width ratios */
.left {
  flex: 0 0 33%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
.center {
  flex: 0 0 30%;
}
.right {
  flex: 0 0 37%;
}

/* ✅ LEFT PANEL */
.left ul {
  list-style: none;
  padding: 0;
  margin-top: 50px;
  animation: floatBounce 2s ease-in-out infinite;
  transform-origin: center;
}
.left li {
  font-size: 2rem;
  font-weight: 900;
  text-align: center;
  color: #000000;
  margin-bottom: 20px;
}

/* Time Box */
.time-box {
  border: 3px solid #000;
  border-radius: 15px;
  text-align: center;
  padding: 15px 10px;
  background: #f1efef;
}
.label {
  font-size: 2.7rem;
  font-weight: 900;
  color: #ff0000;
  margin-bottom: 12px;
}

.date-row {
  margin-top: 18px;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

/* ✅ Box section */
.date-box {
  text-align: center;
  background: transparent;
  color: #111;
}
.text-main {
  display: block;
  font-size: 3rem;
  font-weight: bold;
  color: #000000;
}

.text-main-value {
  display: block;
  font-size: 2.5rem;
  font-weight: bold;
  color: #aa0000;
}

.date-box .date-box-value {
  padding: 10px 30px;
  font-size: 10rem;
  font-weight: bold;
  color: #d10000;
  min-width: 110px;
  display: inline-block;
  margin-top: 10px;
}

/* Subscribe Button */
.subscribe {
  text-align: center;
  margin-top: 30px;
}
.subscribe button {
  padding: 15px 40px;
  border-radius: 12px;
  font-size: 2.3rem;
  font-weight: bold;
  border: none;
  color: white;
  cursor: pointer;
  animation: floatBounce 2s ease-in-out infinite;
  transform-origin: center;
}

@keyframes blink {
  50% {
    opacity: 0.7;
  }
}

/* ✅ RIGHT Panel = Full Mobile UI Design */
.right {
  display: flex;
  justify-content: center;
  align-items: center;
  background: #f3f3f3;
  width: 100%;
  height: 100%;
  padding: 0;
  box-sizing: border-box;
}

/* 📱 Phone Container - full height panel */
.phone {
  position: relative;
  width: 90%;
  height: 95%;
  background: #ffffff;
  border-radius: 40px;
  border: 6px solid #000;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
  padding: 20px 15px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  overflow-y: auto;
  box-sizing: border-box;
}

/* 📱 Top notch (camera/speaker bar) */
.phone::before {
  content: "";
  position: absolute;
  top: 15px;
  left: 50%;
  transform: translateX(-50%);
  width: 120px;
  height: 25px;
  background: #000;
  border-radius: 12px;
}

/* Inside phone content */
.phone h1.big-number {
  font-size: 10rem;
  color: green;
  text-shadow: 3px 3px 8px rgba(0, 0, 0, 0.4);
  margin: 50px 0 5px 0;
}

.phone .updated {
  font-size: 2rem;
  font-weight: 700;
  color: #000;
}

/* ✅ Main Cards inside the phone (12:01 PM & 4:30 PM - 100% Size) */
.phone .card {
  background: #ff3030;
  color: #fff;
  border-radius: 18px;
  padding: 10px;
  width: 100%;
  margin-top: 15px;
  text-align: center;
  font-weight: bold;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
  box-sizing: border-box;
}

.phone .card h2 {
  font-size: 1.3rem;
  margin: 0 0 8px 0;
}

.phone .card .info {
  display: flex;
  justify-content: space-around;
  align-items: center;
  gap: 8px;
}

.phone .card .info-item h3 {
  font-size: 1.4rem;
  color: #fff;
  margin: 0 0 4px 0;
}

.phone .card .info-item p {
  font-size: 3rem;
  font-weight: bold;
  color: yellow;
  margin: 0;
}

/* ✅ Sub Cards (03:00 PM & 11:00 AM - အပေါ်ကတ်များ၏ ၃ ပုံ ၂ ပုံ (2/3 = 66.67%) အရွယ်အစား) */
.phone .card.sub-card {
  width: 100%;
  padding: 6.5px;
  border-radius: 12px;
  margin-top: 30px;
}

.phone .card.sub-card h2 {
  font-size: 1rem; /* 1.3rem ၏ 2/3 */
  margin: 0 0 5px 0;
}

.phone .card.sub-card .info {
  gap: 5px;
}

.phone .card.sub-card .info-item h3 {
  font-size: 1rem; /* 1.4rem ၏ 2/3 */
  margin: 0 0 3px 0;
}

.phone .card.sub-card .info-item p {
  font-size: 2rem; /* 3rem ၏ 2/3 */
}

/* ✅ Live info box */
.live-info {
  display: flex;
  justify-content: space-around;
  align-items: center;
  width: 100%;
  margin: 20px auto 0;
  border: 3px solid #000;
  border-radius: 15px;
  padding: 15px 20px;
  background: #f1efef;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  box-sizing: border-box;
}

.live-info .info-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.live-info .info-label {
  font-size: 3rem;
  font-weight: bold;
  color: #000;
  margin: 0;
}

.live-info .info-value {
  font-size: 4.7rem;
  font-weight: bold;
  color: #c00000;
  margin: 0;
  animation: floatBounce 2s ease-in-out infinite;
  transform-origin: center;
}

/* ✅ Hide scrollbar for better mobile look */
.phone::-webkit-scrollbar {
  width: 0;
  height: 0;
}

/* ✅ CENTER Panel */
.center {
  display: flex;
  justify-content: center;
  align-items: center;
  background: #f1efef;
  text-align: center;
  height: 100%;
  padding: 20px;
  box-sizing: border-box;
}

.center-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 12px;
  font-weight: bold;
}

.center-content .twod {
  font-size: 3.5rem;
  font-weight: 700;
  color: #c00000;
  margin-bottom: 6px;
}

.center-content .day-session {
  font-size: 3.7rem;
  font-weight: 700;
  color: #c00000;
  margin-bottom: 6px;
}

.center-content .label {
  font-size: 3rem;
  color: #000000;
  margin-top: 5px;
}

.center-content .main-time {
  font-size: 4.3rem;
  color: #000;
  margin: 10px 0 5px 0;
}

.center-content .countdown {
  font-size: 3.7rem;
  color: #d10000;
  margin: 10px 0 5px 0; /* ယခင် margin-bottom syntax အမှားကို ပြင်ထားသည် */
}

.center-content .text-output {
  font-size: 6rem;
  font-weight: 900;
  color: #ff0000;
  margin-top: 5px;
  letter-spacing: 2px;
}

.center-content .respect-img {
  width: 600px;
  height: auto;
  margin-bottom: 80px;
  transform-origin: center;
}

.currency-label {
  display: inline-block;
  margin-top: 15px;
  color: #c00000;
  font-size: 6rem;
  animation: floatBounce 4s ease-in-out infinite;
  transform-origin: center;
}

@keyframes growPulse {
  0% {
    transform: scale(0.5);
  }
  25% {
    transform: scale(1.5);
  }
  50% {
    transform: scale(1.2);
  }
  75% {
    transform: scale(2.1);
  }
  100% {
    transform: scale(2.5);
  }
}

/* Floating / Bouncing Keyframes (Media query အပြင်သို့ ပြန်ထုတ်ပေးထားသည်) */
@keyframes floatBounce {
  0% {
    transform: translateY(0) scale(1);
  }
  25% {
    transform: translateY(-10px) scale(1.05);
  }
  50% {
    transform: translateY(-5px) scale(1.1);
  }
  75% {
    transform: translateY(-10px) scale(1.05);
  }
  100% {
    transform: translateY(0) scale(1);
  }
}

/* ✅ Responsive */
@media (max-width: 1600px) {
  .main-time {
    font-size: 3.5rem;
  }
  .text-output {
    font-size: 4.2rem;
  }
  .phone h1.big-number {
    font-size: 6.5rem;
  }
  .label {
    font-size: 2rem;
  }
  .date-box {
    font-size: 2rem;
  }
}

/* Color variants */
.subscribe button.red {
  background: red;
}
.subscribe button.green {
  background: green;
}
</style>
