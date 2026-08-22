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
              <span class="text-main">စျေးကွက်အရ ပတ်သီး</span>
              <div class="date-box-value">` {{ randomKey }} `</div>
            </div>
          </div>

            <div class="date-row">
            <div class="date-box">
              <span class="text-main">စျေးကွက်အရ ထိပ်</span>
              <div class="date-box-value">`  {{ randomSeqKey }} `</div>
            </div>
          </div>
        </div>

        <div class="subscribe">
          <button :class="{green:isGreen,red:!isGreen}">
            🔔 LIKE & SUBSCRIBE
          </button>
        </div>
      </div>

      <!-- ✅ RIGHT Panel -->
      <div class="right" v-if="twoDeeLive">
        <div class="phone">
          <h1 class="big-number">` {{ twoDeeLive.live.twod }} `</h1>
          <p class="updated">Updated: {{ updatedTime }}</p>

          <!-- ✅ Two red cards inside the phone -->
          <div class="card">
            <h2>12:01 PM</h2>
            <div class="info">
              <div class="info-item">
                <h3>စက်</h3>
                <p>{{ twoDeeLive.result[1].set }}</p>
              </div>
              <div class="info-item">
                <h3>တန်ဖိုး</h3>
                <p>{{ twoDeeLive.result[1].value }}</p>
              </div>
              <div class="info-item">
                <h3>တူးတူး</h3>
                <p>{{ twoDeeLive.result[1].twod }}</p>
              </div>
            </div>
          </div>

          <div class="card">
            <h2>4:30 PM</h2>
            <div class="info">
              <div class="info-item">
                <h3>စက်</h3>
                <p>{{ twoDeeLive.result[3].set }}</p>
              </div>
              <div class="info-item">
                <h3>တန်ဖိုး</h3>
                <p>{{ twoDeeLive.result[3].value }}</p>
              </div>
              <div class="info-item">
                <h3>တူးတူး</h3>
                <p>{{ twoDeeLive.result[3].twod }}</p>
              </div>
            </div>
          </div>

          <!-- 🎥 Random Video Section -->
          <div class="video-container">
            <video 
              :src="currentVideo" 
              autoplay 
              muted 
              playsinline 
              @ended="changeVideo"
              class="random-video"
            ></video>
          </div>

          <!-- ✅ NEW: "စက်" & "တန်ဖိုး" alignment left/right -->
          <div class="live-info">
            <div class="info-item">
              <p class="info-label">စက်</p>
              <p class="info-value">{{ twoDeeLive.live.set }}</p>
            </div>

            <div class="info-item">
              <p class="info-label">တန်ဖိုး</p>
              <p class="info-value">{{ twoDeeLive.live.value }}</p>
            </div>
          </div>

        </div>
      </div>


      <!-- ✅ CENTER Panel Now Right -->
      <div class="center" >
        <div class="center-content">
          <!-- 🙏 ပုံထည့်ထားတဲ့အပိုင်း -->
         <img src="@/assets/photo.png" alt="Respect Image" class="respect-img" />
          <div class="twod">{{ myanmarDate }} <br><span class="day-session"> {{ dayInMyanmar }}နေ့ <br>{{ session }}</span></div>
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
import { ref,computed, onMounted } from "vue";

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

    //  for API data from backend api laravel
    const currencies = ref([]);
    const loading = ref(true);
    const apiUrl = process.env.VUE_APP_API_URRS;
    const accessToken = process.env.VUE_APP_API_KEY;

    // ✅ reactive variable for key
    const randomKey = ref("");
    // ✅ reactive variable
    const randomSeqKey = ref("");


    const videoList = [
      "/videos/video1.mp4",
      "/videos/video2.mp4",
      "/videos/video3.mp4",
      "/videos/video4.mp4",
      "/videos/video5.mp4",
      "/videos/video6.mp4",
      "/videos/video7.mp4",
      "/videos/video8.mp4",
      "/videos/video9.mp4",
      "/videos/video10.mp4",
      "/videos/video11.mp4",
      "/videos/video12.mp4",
      "/videos/video13.mp4",
      "/videos/video14.mp4",
      "/videos/video15.mp4",
      "/videos/video16.mp4",
    ];

    const currentVideo = ref(videoList[0]);

    const changeVideo = () => {
      const randomIndex = Math.floor(Math.random() * videoList.length);
      currentVideo.value = videoList[randomIndex];
    };

    // ✅ Random key generator (တူတဲ့ digit မပါအောင် ၂ လုံး random ထုတ်)
    function generateUniqueKey() {
      let key;
      do {
        const temp = [];
        for (let i = 0; i < 2; i++) {
          temp.push(Math.floor(Math.random() * 10)); // 0-9 ထဲက random digit ၂ လုံး
        }
        key = temp;
      } while (new Set(key).size < 2); // ထပ်နေတဲ့ digit မဖြစ်အောင် စစ်
      return key.join("-"); // ဥပမာ output: "3-7"
    }

    // ✅ Random Key တစ်ခု ထုတ်ပြီး randomKey ထဲသိမ်း
    function generateRandomKey() {
      randomKey.value = generateUniqueKey();
    }


    // ✅ 3-digit sequential key generator
    function generateSequentialKey() {
      // 0 - 9 ထဲက random start digit တစ်ခုရွေး
      const start = Math.floor(Math.random() * 10);

      // start + 1, +2 နဲ့ အစဉ်လိုက် သုံးလုံး ပြုလုပ်
      const digits = [
        start,
        (start + 1) % 10,   // 9 နောက်ထပ် 0 ပြန်သွားအောင် (%10)
        (start + 2) % 10
      ];

      return digits.join("-"); // ဥပမာ: "7-8-9"
    }

    // ✅ Random Sequential Key တစ်ခု ထုတ်ပြီး randomSeqKey ထဲသိမ်း
    function generateRandomSequentialKey() {
      randomSeqKey.value = generateSequentialKey();
    }

    function isWeekend() {
      const day = new Date().getDay();
      return day === 0 || day === 6; // Sunday = 0, Saturday = 6
    }

    function generateRandomKey() {
      randomKey.value = isWeekend() ? "CLOSED" : generateUniqueKey();
    }

    function generateRandomSequentialKey() {
      randomSeqKey.value = isWeekend() ? "CLOSED" : generateSequentialKey();
    }

    // ✅ Computed for currentTwoD
    const currentTwoD = computed(() => {
      if (!twoDeeLive.value) return "";
      const now = new Date();
      const cutoff = new Date();
      cutoff.setHours(14, 0, 0, 0);
      return now < cutoff
        ? twoDeeLive.value.result[1].twod
        : twoDeeLive.value.result[3].twod;
    });


    
    // 🔹 Methods
    const convertToMyanmarNumber = (num) => {
      const myanmarNumbers = ["၀","၁","၂","၃","၄","၅","၆","၇","၈","၉"];
      return num?.toString().split("").map(d => myanmarNumbers[Number(d)]).join("") || "";
    };

    const updateDay = () => {
      const daysMm = ["တနင်္ဂနွေ","တနင်္လာ","အင်္ဂါ","ဗုဒ္ဓဟူး","ကြာသပတေး","သောကြာ","စနေ"];
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
      const hours = String(now.getHours()).padStart(2,"0");
      const minutes = String(now.getMinutes()).padStart(2,"0");
      const seconds = String(now.getSeconds()).padStart(2,"0");
      countdown.value = `${hours}:${minutes}:${seconds}`;
    };

    const loadTwoDeeLive = async () => {
      try {
        const response = await fetch("https://api.thaistock2d.com/live");
        const datas = await response.json();
        if (JSON.stringify(datas.live) !== JSON.stringify(previousLiveData.value)) {
          isAnimated.value = true;
          setTimeout(() => { isAnimated.value = false; }, 10000);
        }
        previousLiveData.value = datas.live;
        twoDeeLive.value = datas;
      } catch (err) {
        error.value = err.message;
      }
    };



    // Fetch data from backend API
    const fetchData = async () => {
      try {
        const response = await fetch(apiUrl, {
          method: 'GET',
          headers: {
            "Content-Type": 'application/json',
            'Authorization': `Bearer ${accessToken}`,
          }
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

      mainTime.value = (hour >= 7 && hour < 14) ? time12 : time430;
    };


    // ✅ Schedule key auto update (တစ်နေ့ ၂ ကြိမ် update)
    function scheduleKeyUpdate(fn) {
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
      setTimeout(() => {
        fn();
        scheduleKeyUpdate(fn);
      }, diff);
    }


    // ✅ Schedule auto update (တစ်နေ့ ၂ ကြိမ် update)
    function scheduleSequentialKeyUpdate(fn) {
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
      setTimeout(() => {
        fn();
        scheduleSequentialKeyUpdate(fn);
      }, diff);
    }

    // 🌟 Mounted Equivalent
    onMounted(() => {
      if (screen.orientation && screen.orientation.lock) {
        screen.orientation.lock("landscape").catch(err => console.warn("Orientation lock failed:", err));
      }

      // ✅ Initial random key generation and scheduling
      generateRandomKey(); // Start with first random key
      scheduleKeyUpdate(generateRandomKey); // Auto update at 10 AM / 2 PM

      // ✅ Initial random sequential key generation and scheduling
      generateRandomSequentialKey();              // စတင်အချိန်မှာ တစ်ခုပဲ ထုတ်
      scheduleSequentialKeyUpdate(generateRandomSequentialKey);  // နောက်အချိန် auto update

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

      setInterval(changeVideo, 15000); // 15 seconds


      // Intervals
      setInterval(updateDateTime, 1000);
      setInterval(updateCountdown, 1000);
      setInterval(loadTwoDeeLive, 10000);
      setInterval(updateMainTime, 60000);
      setInterval(() => { isGreen.value = !isGreen.value; }, 2000);
      setInterval(() => {
        updateDateTimeMyanmar();
        updateSession();
      }, 60000);

      // Fetch data every 5 minutes
      setInterval(() => {fetchData();}, 1 * 60 * 1000); 
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
      // API data from backend
      currencies,
      loading,

      // ⬇⬇⬇ ထည့်ဖို့လိုတာ
      currentVideo,
      changeVideo,
      randomKey,
      randomSeqKey,
      currentTwoD,
    };
  }
};
</script>



<style>
html, body, .app {
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
.left, .center, .right {
  border-radius: 0;
  padding: 10px;
  box-sizing: border-box;
  background: #ffffff;
  font-weight: bold;
  border: none !important;
}

/* ✅ Panel width ratios */
.left { flex: 0 0 33%; }
.center { flex: 0 0 30%; }
.right { flex: 0 0 37%; }


/* 🚫 Panel ကြားမျဉ်းများဖြုတ် */
.left, .center, .right {
  border: none !important;
}

/* ✅ LEFT PANEL */
.left {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.left ul {
  list-style: none;
  padding: 0;
  margin-top: 50px;
    /* Floating animation */
  animation: floatBounce 2s ease-in-out infinite;
  transform-origin: center;
  
}
.left li {
  font-size: 1.8rem;
  font-weight: 900;
  text-align: center;
  color: #000000;
  margin-bottom: 20px;
  font-weight: bold;
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
  background: #ff000000;
  color: #111;
}
.text-main {
  display: block;
  font-size: 3.5rem;
  font-weight: bold;
  color: #000000;
}

.text-main-value {
  display: block;
  font-size: 2.5rem;
  font-weight: bold;
  color: #aa0000;
}


.date-box .date-box-value  {
  border: 3px solid #000;
  border-radius: 12px;
  padding: 10px 30px;
  font-size: 5rem;
  font-weight: bold;
  background: #fefefe;
  color: #d10000;
  box-shadow: 2px 2px 5px rgba(0,0,0,0.2);
  min-width: 110px;
  display: inline-block;
  margin-top: 10px;
}

/* Subscribe Button */
.subscribe {
  text-align: center;
  margin-top: 30px;
}
/* Subscribe Button - Floating/Bounce Effect */
.subscribe button {
  padding: 15px 40px;
  border-radius: 12px;
  font-size: 2.3rem;
  font-weight: bold;
  border: none;
  color: white;
  cursor: pointer;

  /* Floating animation */
  animation: floatBounce 2s ease-in-out infinite;
  transform-origin: center;
}



@keyframes blink { 50% { opacity: 0.7; } }

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
  box-shadow: 0 10px 25px rgba(0,0,0,0.3);
  padding: 20px 15px 20px 15px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  overflow-y: auto;
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
  text-shadow: 3px 3px 8px rgba(0,0,0,0.4);
  margin: 50px 0 5px 0;
}

.phone .updated {
  font-size: 1rem;
  font-weight: 700;
  color: #000;
}

/* ✅ Cards inside the phone */
.phone .card {
  background: #ff3030;
  color: #fff;
  border-radius: 18px;
  padding: 10px;
  width: 100%;
  margin-top: 15px;
  text-align: center;
  font-weight: bold;
  box-shadow: 0 4px 6px rgba(0,0,0,0.3);
}

.phone .card h2 {
  font-size: 1.3rem;
  margin-bottom: 8px;
}

.phone .card .info {
  display: flex;
  justify-content: space-around;
  align-items: center;
  gap: 8px;
}

.phone .card .info-item h3 {
  font-size: 1.2rem;
  color: #fff;
  margin: 0 0 4px 0;
}

.phone .card .info-item p {
  font-size: 1.4rem;
  font-weight: bold;
  color: yellow;
  margin: 0;
}
/* ✅ Live info box (left/right layout) */
/* ✅ Live info box */
.live-info {
  display: flex;
  justify-content: space-around;
  align-items: center;
  width: 90%;
  margin-top: 20px;
  border: 3px solid #000;
  border-radius: 15px;
  padding: 15px 20px;
  background: #f1efef;
  box-shadow: 0 4px 10px rgba(0,0,0,0.2);
}

/* စက် / တန်ဖိုး တစ်ခုချင်းစီ */
.live-info .info-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

/* ခေါင်းစဉ် */
.live-info .info-label {
  font-size: 2.2rem;
  font-weight: bold;
  color: #000;
  margin: 0;
}

/* တန်ဖိုး */
.live-info .info-value {
  font-size: 3.2rem;
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

/* Main content alignment */
.center-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 12px;
  font-weight: bold;

}

/* 🔹 မြန်မာနေ့/အချိန် စာသား */
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
/* 🔹 “ထွက်မည့် အချိန်” စာသား */
.center-content .label {
  font-size: 3rem;
  color: #000000;
  margin-top: 5px;
}

/* 🔹 အချိန် */
.center-content .main-time {
  font-size: 4.3rem;
  color: #000;
  margin: 10px 0 5px 0;
}

/* 🔹 Countdown timer */
.center-content .countdown {
  font-size: 3.7rem;
  color: #d10000;
  margin-bottom: 10px 0 5px 0;
}

/* 🔹 ထွက်မည့် ဂဏန်း (ကြီးကြီးပြ) */
.center-content .text-output {
  font-size: 6rem;
  font-weight: 900;
  color: #ff0000;
  margin-top: 5px;
  letter-spacing: 2px;
}


/* 🙏 ပုံအတွက် style */
.center-content .respect-img {
  width: 180px;
  height: auto;
  margin-bottom: 80px;
  animation: growPulse 4s ease-in-out infinite;
  transform-origin: center;
}
.currency-label {
  margin-top: 15px;
  color: #c00000;
  font-size: 6rem;
  animation: floatBounce 4s ease-in-out infinite;
  transform-origin: center;
}
/* ✨ အစမှာ သေးပြီး နည်းနည်းကြီးလာပြီး pulse လှုပ် */
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


/* ✅ Responsive */
@media (max-width: 1600px) {
  .main-time { font-size: 3.5rem; }
  .text-output { font-size: 4.2rem; }
  .big-number { font-size: 6.5rem; }
  .label { font-size: 2rem; }
  .date-box { font-size: 2rem; }
}

/* Floating / Bouncing Keyframes */
@keyframes floatBounce {
  0%   { transform: translateY(0) scale(1); }
  25%  { transform: translateY(-10px) scale(1.05); }
  50%  { transform: translateY(-5px) scale(1.1); }
  75%  { transform: translateY(-10px) scale(1.05); }
  100% { transform: translateY(0) scale(1); }
}

/* Color variants remain the same */
.subscribe button.red { background: red; }
.subscribe button.green { background: green; }

.video-container {
  width: 300px;
  height: 200px;
  border: 4px solid #000;
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0,0,0,0.4);
  margin-top: 20px;
  animation: floatBounce 10s ease-in-out infinite;
}

.random-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 15px;
}

</style>
