<template>
  <div class="quiz-result">
    <!-- 상단 이미지 및 텍스트 영역 -->
    <div class="content-wrapper">
      <div class="result-wrapper">
        <div class="result-image fade-up delay-1">
          <div>
            <MileageDisplay
              v-if="isCorrect"
              :size="120"
              color="#FAB809"
              class="delay-1"
            />

            <img
              v-else
              src="@/assets/images/sobble.png"
              class="sobble"
              alt="울머기"
            />
          </div>
        </div>

        <!-- 마일리지 텍스트 -->
        <MileageCounter
          v-if="isCorrect"
          :amount="mileage"
          :duration="1200"
          class="mileage fade-up delay-2"
        />

        <p
          class="result-text fade-up delay-3"
          :style="{ color: isCorrect ? '#000' : '#AE0000' }"
        >
          {{ isCorrect ? '정답이에요!' : '틀렸어요!' }}
        </p>

        <p class="sub-text fade-up delay-4">
          {{
            isCorrect
              ? `${mileage} 마일리지 획득!`
              : '해설을 보면 마일리지를 드릴게요'
          }}
        </p>

        <div v-if="showExplanation" class="explanation-box fade-up delay-5">
          {{ explanation }}
        </div>
      </div>
    </div>

    <!-- 하단 버튼 -->
    <div class="footer-button fade-up delay-6">
      <CustomButton
        :category="buttonCategory"
        size="medium"
        style="width: 80%; max-width: 400px"
        @click="handleButtonClick"
      >
        {{ buttonText }}
      </CustomButton>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useQuizResultStore } from '@/stores/quizResult';
import CustomButton from '@/components/common/CustomButton.vue';
import axios from 'axios';
import MileageDisplay from '@/components/quiz/MileageDisplay.vue';
import MileageCounter from '@/components/quiz/MileageCounter.vue';
import { useAuthStore } from '@/stores/authStore';
import { getUserInfo } from '@/apis/users';
import GetMileageSound from '@/assets/sounds/GetMileage.mp3';
import QuizWrongSound from '@/assets/sounds/QuizWrong.mp3';
import SelectSound from '@/assets/sounds/ButtonSound.mp3';

const quizResult = useQuizResultStore();
const route = useRoute();
const router = useRouter();
const isCorrect = quizResult.isCorrect;
const mileage = quizResult.mileage;
const explanation = quizResult.explanation;
const date = quizResult.date;
const authStore = useAuthStore();
const showExplanation = ref(false);

const buttonText = computed(() => {
  if (isCorrect) return '포켓몬 뽑으러 가기';
  if (!showExplanation.value) return '해설 보기';
  return '마일리지 받기'; // 오답 + 해설 본 경우
});

const buttonCategory = computed(() => {
  if (isCorrect || showExplanation.value) return 'primary';
  return 'secondary';
});

function playClickSound() {
  const audio = new Audio(SelectSound);
  audio.volume = 1.0; // 🎵 소리 크기 최대로
  audio.play().catch((err) => {
    console.warn('효과음 재생 실패:', err);
  });
}

function handleButtonClick() {
  playClickSound();
  if (isCorrect) {
    router.push('/pokedex');
  } else if (!showExplanation.value) {
    showExplanation.value = true;
  } else {
    router.push('/quiz/reward'); // 👉 새 화면으로 이동
  }
}

function getTodayDateString() {
  const today = new Date();
  return today.toISOString().split('T')[0];
}

async function updateAnsweredDate() {
  try {
    const userId = authStore.user.id;
    await axios.patch(`/api/users/${userId}`, {
      last_answered_date: getTodayDateString(),
    });
    console.log('✅ 날짜 업데이트 완료');
  } catch (err) {
    console.error('❌ 날짜 업데이트 실패:', err);
  }
}

onMounted(async () => {
  updateAnsweredDate(); // ✅ 정답/오답 관계없이 무조건 실행

  // 🎵 효과음 재생
  const audio = new Audio(isCorrect ? GetMileageSound : QuizWrongSound);
  audio.volume = 0.6;

  audio.play().catch((err) => {
    console.warn('🔇 효과음 자동재생 실패:', err);
  });

  if (!isCorrect) return; // ❌ 오답이면 마일리지는 지급하지 않음

  try {
    const userId = authStore.user.id;
    const res = await axios.get(`/api/users/${userId}`);
    const currentMileage = res.data.mileage;

    await axios.patch(`/api/users/${userId}`, {
      mileage: currentMileage + 1000,
    });

    const fetchUser = async () => {
      if (authStore.user) {
        await getUserInfo(authStore.user.id);
      }
    };

    await fetchUser();
    console.log('✅ 정답 보상 1000 마일리지 지급 완료!');
  } catch (err) {
    console.error('❌ 마일리지 지급 실패:', err);
  }
});
</script>

<style scoped>
.quiz-result {
  display: flex;
  flex-direction: column;
  justify-content: space-between; /* 상단 + 하단 분리 */
  height: 100%;
  text-align: center;
}

.content-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 10rem;
  flex: 1;
}

.result-wrapper {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.result-image {
  display: flex;
  justify-content: center;
  margin-bottom: 0.5rem;
}

.mileage {
  font-size: 40px;
  font-weight: bold;
  font-family: 'Pretendard Variable', Pretendard, -apple-system,
    BlinkMacSystemFont, system-ui, Roboto, 'Helvetica Neue', 'Segoe UI',
    'Apple SD Gothic Neo', 'Noto Sans KR', 'Malgun Gothic', 'Apple Color Emoji',
    'Segoe UI Emoji', 'Segoe UI Symbol', sans-serif;
  font-weight: bold;
  color: #fab809;
}

.result-text {
  font-size: 35px;
  font-weight: bold;
  margin-top: 2rem;
  margin-bottom: 0.5rem;
}

.sub-text {
  font-size: 20px;
  color: #555;
  margin-bottom: 4rem;
}

.explanation-box {
  background-color: #f4f4f4;
  padding: 1rem;
  border-radius: 10px;
  font-size: 20px;
  line-height: 1.5;
  white-space: pre-line;
  margin: 0 auto;
  width: 80%;
  max-width: 500px;
}

.footer-button {
  display: flex;
  justify-content: center;
  margin-bottom: 3rem;
}

.explanation-button {
  padding: 1rem 2rem;
  font-size: 1.1rem;
  background-color: #fed337;
  border: none;
  border-radius: 13px;
  font-family: 'Press Start 2P', cursive;
  box-shadow: 4px 4px 0px #000000;
  cursor: pointer;
  transition: transform 0.2s;
}

.explanation-button:hover {
  transform: scale(1.03);
}

.sobble {
  width: 180px;
  height: auto;
}

.fade-up {
  opacity: 0;
  transform: translateY(40px);
  animation: fadeUp 0.5s ease-out forwards;
}

.delay-1 {
  animation-delay: 0.2s;
}
.delay-2 {
  animation-delay: 0.4s;
}
.delay-3 {
  animation-delay: 0.6s;
}
.delay-4 {
  animation-delay: 0.8s;
}
.delay-5 {
  animation-delay: 0.3s;
}
.delay-6 {
  animation-delay: 1.2s;
}

@keyframes fadeUp {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.spin-in {
  animation: spinInY 0.8s ease-out forwards;
  transform-style: preserve-3d; /* ✅ 3D 회전 자연스럽게 */
}

@keyframes spinInY {
  0% {
    opacity: 0;
    transform: rotateY(90deg) scale(0.8);
  }
  60% {
    opacity: 1;
    transform: rotateY(180deg) scale(1.05);
  }
  100% {
    opacity: 1;
    transform: rotateY(360deg) scale(1);
  }
}
</style>
