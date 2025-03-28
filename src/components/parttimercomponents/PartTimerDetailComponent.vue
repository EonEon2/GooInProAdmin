<script setup>
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router'; // useRouter 및 useRoute 추가
import { detailPartTimer, deletePartTimer } from '../../api/parttimerapi/partTimerAPI.js';
import {findChatRoom} from "../../api/chatapi/chatAPI.js";
import {useAdminAuthStore} from "../../stores/adminAuthStore.js";


// 날짜 포맷팅 함수 (생년월일은 시간 제외, 등록일은 시간 포함)
const formatDate = (dateStr, includeTime = false) => {
  const date = new Date(dateStr);
  if (includeTime) {
    return new Intl.DateTimeFormat('ko-KR', {
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit',
      timeZone: 'Asia/Seoul',
    }).format(date);
  } else {
    return new Intl.DateTimeFormat('ko-KR', {
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
      timeZone: 'Asia/Seoul',
    }).format(date);
  }
};

// 상세 정보 데이터를 저장할 변수
const partTimerDetail = ref(null);
const loading = ref(true);
const error = ref(null);

// 라우터에서 전달된 pno 값을 사용
const route = useRoute();
const router = useRouter(); // Router 객체 생성
const pno = route.params.pno; // URL에서 pno 값을 받아옵니다.
const pemail = ref('');
const roomId = ref('');

const adminAuthStore = useAdminAuthStore();
const admid =  adminAuthStore.adminId;


// 페이지 번호를 쿼리에서 가져오기
const page = route.query.page;

console.log(route.query);

const fetchPartTimerDetail = async () => {
  try {
    loading.value = true;
    const response = await detailPartTimer(pno); // pno에 맞는 상세 정보 호출
    partTimerDetail.value = response;
    pemail.value = response.pemail
    console.log(response);
  } catch (err) {
    error.value = '근로자 상세 정보를 불러오는 데 실패했습니다.';
  } finally {
    loading.value = false;
  }
};

// 뒤로가기 버튼 클릭 시 실행될 함수
const goBackToList = () => {
  if (page) {
    router.push(`/partTimer/list?page=${page}`); // 쿼리 파라미터와 함께 리스트 페이지로 이동
  } else {
    router.push('/partTimer/list'); // 페이지 정보가 없으면 기본 리스트로 이동
  }
};

// 삭제 버튼 클릭 시 실행될 함수
const handleDelete = async () => {
  try {
    await deletePartTimer(pno); // pno에 맞는 근로자 삭제
    goBackToList(); // 삭제 후 리스트 페이지로 이동
  } catch (err) {
    error.value = '근로자 삭제에 실패했습니다.';
  }
};

const startChattingClick = () => {

  const roomName = `${admid},${pemail.value}/parttimer`;

  const dto = {roomName:roomName, senderEmail: admid, recipientEmail: pemail.value};

  console.log(dto);

  findChatRoom(dto).then((res) => {
    router.push(`/chat/chatting/${res.id}`);
  })
}


onMounted(fetchPartTimerDetail);
</script>

<template>
  <div class="p-4">
    <h1 class="text-2xl font-semibold mb-4">근로자 상세보기</h1>

    <!-- 로딩 상태 -->
    <div v-if="loading" class="text-lg">로딩 중...</div>

    <!-- 에러 처리 -->
    <div v-if="error" class="text-red-500 font-semibold">{{ error }}</div>

    <!-- 근로자 상세 정보 -->
    <div v-if="partTimerDetail && !loading && !error" class="space-y-4">
      <div>
        <strong>이름:</strong> {{ partTimerDetail.pname }}
      </div>
      <div>
        <strong>생년월일:</strong> {{ formatDate(partTimerDetail.pbirth) }}
      </div>
      <div>
        <strong>성별:</strong> {{ partTimerDetail.pgender === 'M' ? '남' : '여' }}
      </div>
      <div>
        <strong>이메일:</strong> {{ partTimerDetail.pemail }}
      </div>
      <div>
        <strong>등록일:</strong> {{ formatDate(partTimerDetail.pregdate, true) }}
      </div>
      <div>
        <strong>주소:</strong> {{ partTimerDetail.proadAddress }}
      </div>
      <div>
        <strong>상세 주소:</strong> {{ partTimerDetail.pdetailAddress }}
      </div>
      <div v-if="partTimerDetail.pifilename">
        <strong>프로필 이미지 :</strong>
        <img :src="`/path/to/images/${partTimerDetail.pifilename}`" alt="프로필 이미지" class="max-w-xs mt-2" />
      </div>
      <div v-if="partTimerDetail.pdifilename">
        <strong>이력서 :</strong>
        <img :src="`/path/to/images/${partTimerDetail.pdifilename}`" alt="이력서 이미지" class="max-w-xs mt-2" />
      </div>
    </div>

    <!-- 버튼들 -->
    <div v-if="!loading && !error" class="mt-8 flex justify-center gap-4">
      <!-- 삭제 버튼 -->
      <button @click="handleDelete" class="px-6 py-3 bg-red-600 text-white rounded-lg shadow-md hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500">
        삭제
      </button>

      <!-- 뒤로가기 버튼 -->
      <button @click="goBackToList" class="px-6 py-3 bg-blue-600 text-white rounded-lg shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500">
        뒤로가기
      </button>

      <button
          class="px-6 py-2 bg-orange-500 text-white font-semibold rounded-lg shadow-md hover:bg-red-400 transition"
          @click="startChattingClick"
      >
        채팅하기
      </button>


    </div>
  </div>
</template>

<style scoped>

</style>
