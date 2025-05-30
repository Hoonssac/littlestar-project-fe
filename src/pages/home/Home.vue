<template>
  <div class="home-container">
    <ProgressBar :degree="userMileageDegree" />
    <div class="mileage-container">
      <p class="mileage-text">{{ userMileage }} / 5000</p>
      <IconMileage color="#FAB809" size="20" />
    </div>
    <Help :show="showModal" @close="showModal = false" />

    <div v-if="pokedex !== null" class="info-container" size="small">
      <p>No.{{ pokedex.id }} {{ pokedex.name }}</p>
      <p>
        타입:
        {{ pokedex.types }}
      </p>
      <p>크기: {{ pokedex.height }}cm</p>
      <p>무게: {{ pokedex.weight }}kg</p>
    </div>

    <div class="main-pokemon-container">
      <img
        v-if="pokedex"
        class="main-pokemon-img"
        :src="pokedex.image_url"
        :alt="pokedex.name"
      />
    </div>

    <div class="add-button-container">
      <CustomButton class="add-button" size="large" @click="showModal = true"
        >>
        <p>사용 방법</p>
      </CustomButton>
    </div>
  </div>
</template>

<script setup>
import { getPokedex } from '@/apis/pokedex';
import CustomButton from '@/components/common/CustomButton.vue';
import { IconMileage } from '@/components/common/icons';
import ProgressBar from '@/components/common/ProgressBar.vue';
import { useAuthStore } from '@/stores/authStore';
import { computed, onMounted, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
import Help from '@/components/home/Help.vue';
import IconHowto from '@/components/common/icons/IconInfo.vue';
import { getSessionUser, getUserInfo } from '@/apis/users';

const showModal = ref(false);
const route = useRoute();
const authStore = useAuthStore();
const pokedex = ref(null);
const userMileage = ref(0);

const fetchHomeData = async () => {
  const userData = await getSessionUser();
  if (!userData) {
    console.warn('로그인된 사용자 정보 없음!');
    return;
  }

  authStore.login(userData);

  if (userData.main_pokemon_id) {
    try {
      const data = await getPokedex(userData.main_pokemon_id);
      pokedex.value = data;
    } catch (e) {
      console.error('대표 포켓몬 데이터 불러오기 실패:', e);
    }
  }

  userMileage.value = userData.mileage ?? 0;
};

onMounted(fetchHomeData);

watch(
  () => route.path,
  (newPath) => {
    if (newPath === '/home') {
      fetchHomeData();
    }
  }
);

onMounted(fetchHomeData);

watch(
  () => route.path,
  (newPath) => {
    if (newPath === '/home') {
      fetchHomeData();
    }
  }
);

const userMileageDegree = computed(() => {
  return Math.min((userMileage.value / 5000) * 100, 100);
});
</script>

<style scoped>
.home-container {
  height: 100%;
  position: relative;
}

.mileage-container {
  display: flex;
  align-items: center;
  gap: 3px;
  padding-left: 35px;
  margin-top: -10px;
}
.mileage-text {
  font-size: 20px;
}

.info-btn {
  cursor: pointer;
  position: absolute;
  top: 90px;
  left: 35px;
}
.info-container {
  background-color: white;
  color: #4a483f;
  padding: 10px 20px;
  font-size: 20px;
  display: flex;
  flex-direction: column;
  width: fit-content;
  height: fit-content;
  position: absolute;
  top: 80px;
  right: 25px;

  border-style: solid;
  border-color: #4a483f;
  border-radius: 13px;
  border-top-width: 1px;
  border-right-width: 4px;
  border-bottom-width: 4px;
  border-left-width: 1px;
}

.main-pokemon-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);

  width: 320px;
  overflow: hidden;

  display: flex;
  justify-content: center;
  align-items: center;
}
.main-pokemon-img {
  position: relative;
  z-index: 10;
  width: 400px;
}

.add-button-container {
  width: 100%;
  position: absolute;
  bottom: 0;
  display: flex;
  padding: 0 25px 45px 25px;
}
.add-button {
  display: flex;
  gap: 15px;
  align-items: center;
  flex: 1;
  justify-content: center;
}

.plus-text {
  font-size: 60px;
  line-height: 0.8;
}

.home-container::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 100%;
  height: 100%;
  max-width: 400px;
  max-height: 400px;
  background-image: url('@/assets/images/monster-ball-back.png');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  transform: translate(-50%, -50%);
  z-index: 0;
}

.help-button-container {
  position: absolute;
  top: 25px;
  left: 25px;
  z-index: 50;
}
</style>
