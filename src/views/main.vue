<template>
    <div class="ex1">
        <div class="container">
            <v-btn @click="$router.push('/write')" div class="box1">
                <div class="box1_text"><b>루틴 작성하기</b></div>
            </v-btn>
            <v-btn @click="$router.push('/mypage')" div class="box2">
                <div class="box2_text"><b>마이페이지</b></div>
            </v-btn>
            <v-btn @click="$router.push('/share')" div class="box3">
                <div class="box3_text"><b>루틴 공유하기</b></div>
            </v-btn>
        </div>
    </div>

    <div class="ex3">
        <div v-if="!ISROUTINEID" style="height:5em;"></div>
        <div class="btn" v-if="!ISROUTINEID">
            <div data-bs-toggle="modal" data-bs-target="#routinedatamodal" style="margin-top: 10px;" @click="onModalShow">
                <i class="bi bi-search"></i>
            </div>
        </div>
        <div v-if="!ISROUTINEID" style="height:5em;"></div>
        <div class="daily" v-if="ISROUTINEID">
            <div class="routine-header">
                <v-list-subheader class="routine-title">{{ selectedRoutineTitle }}</v-list-subheader>
                <v-btn class="routineChangeBtn" block rounded="lg" variant="elevated" color="#616D78" min-width="10em"
                    data-bs-toggle="modal" data-bs-target="#routinedatamodal" @click="onModalShow">다른 루틴으로 변경하기</v-btn>
            </div>

            <div v-for="dayIndex in getUniqueDays(selectedRoutineData)" :key="dayIndex">
                <div class="day-routine-box">
                    <div class="day-routine-box-title">Day {{ dayIndex }}</div>
                    <div class="day-routine-exercise-box">
                        <!-- Filter exercises for the current day -->
                        <div v-for="(exercise, exindex) in selectedRoutineData.filter(ex => ex.day === dayIndex)"
                            :key="exindex">
                            <div class="day-routine-exercise-box-name">
                                <div class="exercise-usebody-name"
                                    :style="{ 'background-color': getBackgroundColor(exercise.usebody_name) }">
                                    {{ exercise.usebody_name }}
                                </div>
                                {{ exercise.exercise_name }}
                                <v-checkbox label="완료" color="success" value="success" class="v-checkbox"
                                    density="compact"></v-checkbox>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>


    <!-- 모달내용 -->
    <div class="modal fade" id="routinedatamodal" tabindex="-1">
        <div class="modal-dialog modal-dialog-centered modal-lg">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">내가 담은 루틴</h5>
                </div>


                <div class="modal-body" id="routin-table">
                    <table class="table">
                        <thead>
                            <tr>
                                <th scope="col">#</th>
                                <th scope="col">Routine ID</th>
                                <th scope="col">User Routin ID</th>
                                <th scope="col">상세</th>
                                <th scope="col"></th>
                                <!-- <th scope="col">수정</th> -->
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-if="userData" v-for="(routine, index) in userData" :key="index">
                                <th scope="row">{{ index + 1 }}</th>
                                <td>{{ routine.routine }}</td>
                                <td>{{ routine.user }}</td>
                                <td>
                                    <button @click="showRoutineId(routine)">더보기</button>
                                </td>
                                <td>
                                    <v-btn elevation="1" data-bs-dismiss="modal" @click="saveRoutineId(routine)"
                                        color="black">
                                        선택
                                    </v-btn>
                                </td>
                            </tr>
                        </tbody>
                    </table>


                    <div v-if="selectedRoutine">
                        <div class="goback-btn">
                            <button @click="goBack" id="goback">접기</button>
                        </div>
                        <div class="card">
                            <ul class="list-group list-group-flush">
                                <li class="list-group-item">Routine ID: {{ selectedRoutine.routine_id }}</li>
                                <li class="list-group-item">Routine Name: {{ selectedRoutine.routine_name }}</li>
                                <li class="list-group-item">Routine Comment: {{ selectedRoutine.routine_comment }}</li>
                                <li class="list-group-item">Recommend Count: {{ selectedRoutine.recommend_count }}</li>
                                <li class="list-group-item">Routine Day: {{ selectedRoutine.routine_day }}</li>
                                <li class="list-group-item">Nickname: {{ selectedRoutine.nickname }}</li>
                                <li class="list-group-item">Created At: {{ selectedRoutine.created_at }}</li>
                            </ul>
                        </div>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" data-bs-dismiss="modal" class="close-btn">닫기</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, watchEffect } from 'vue';
import axios from 'axios';
import 'bootstrap-icons/font/bootstrap-icons.css';
//import { store } from '@/store';
import { useRouter } from 'vue-router';
import { useStore } from 'vuex';



const userData = ref(null);
const selectedRoutine = ref(null);

const router = useRouter();
const store = useStore(); // 밖으로 빼서 선언해야함.

const ISROUTINEID = ref(false);
const selectedRoutineTitle = ref(null);

const selectedRoutineData = ref(null);

const getUniqueDays = (data) => {
    // Check if data is not null before using map
    return data ? [...new Set(data.map(exercise => exercise.day))] : [];
};

const getExercisesByDay = (dayIndex) => {
    // Check if selectedRoutineData is an array before using filter
    const exercises = Array.isArray(selectedRoutineData) ? selectedRoutineData : [];

    // Log the selectedRoutineData and dayIndex for debugging
    console.log('Selected Routine Data:', selectedRoutineData);
    console.log('Day Index:', dayIndex);

    // Filter exercises based on the selected day
    const filteredExercises = exercises.filter(exercise => exercise.day === dayIndex);

    // Log the filtered exercises to the console for debugging
    console.log('Filtered Exercises for Day ' + dayIndex + ':', filteredExercises);

    // Return only the exercise names
    return filteredExercises.map(exercise => exercise.exercise_name);
};

const getBackgroundColor = (usebodyName) => {
    // usebody_name에 따른 색상 추가
    switch (usebodyName) {
        case 'neck':
            return '#6C8495';
        case 'lower arms':
            return '#7F7B9B';
        case 'cardio':
            return '#8A7EAE';
        case 'shoulders':
            return '#9B718C';
        case 'upper arms':
            return '#A0878E';
        case 'waist':
            return '#6C7D83';
        case 'legs':
            return '#7D8B92';
        case 'back':
            return '#9D5478';
        case 'chest':
            return '#8E8495';
    }
}

const fetchRoutineById = async () => {
    try {
        const ROUTINE_ID = localStorage.getItem('routineId');
        const response = await axios.get(`/api/routine/check/${ROUTINE_ID}/`);
        console.log('루틴 아이디로 루틴 정보 불러오기 : ', response.data);
        selectedRoutineTitle.value = response.data.routine_name + " 루틴";      // 루틴 제목 보여주기

        const response_2 = await axios.get(`/api/routine/detail/check/${ROUTINE_ID}/`);
        console.log('루틴 아이디로 운동 불러오기 : ', response_2.data);
        selectedRoutineData.value = response_2.data;

        console.log('루틴 day : ', response_2.data.length);
    } catch (error) {
        console.log("루틴 아이디로 루틴 정보 불러오기 오류");
    }
}

watchEffect(() => {
    const storedRoutineId = localStorage.getItem('routineId');

    // Check if routineId is set in localStorage
    if (storedRoutineId) {
        // Update ROUTINE_ID to true
        ISROUTINEID.value = true;
        // routine detail 가져오기
        fetchRoutineById();

    } else {
        // Update ROUTINE_ID to false if routineId is not set
        ISROUTINEID.value = false;
    }
});

const onModalShow = () => {
    fetchRoutineData();
};

onMounted(async () => {
    try {
        await fetchRoutineData();
    } catch (error) {
        console.error(error);
    }
});

const fetchRoutineData = async () => {
    try {
        const response = await axios.get('/api/routine/box/check/');
        userData.value = response.data;
        console.log('Received userData:', userData.value);
    } catch (error) {
        console.error(error);
        userData.value = [];
    }
};
const showRoutineId = async (routine) => {
    try {
        const routineId = routine.routine;
        const apiUrl = `/api/routine/check/${routineId}/`;

        const response = await axios.get(apiUrl);
        console.log('Response from fetchRoutineId:', response);

        if (!response || !response.data) {
            console.error('Invalid response data');
            return;
        }

        // Display routine details in the modal
        console.log('Routine ID:', response.data.routine_id);
        console.log('Routine Name:', response.data.routine_name);

        selectedRoutine.value = response.data;
    } catch (error) {
        console.error('Error occurred while loading routine ID:', error);
        throw error;
    }
};

const saveRoutineId = async (routine) => {
    try {
        const routineId = routine.routine;
        // Access the store directly from the context
        store.commit('setChosenRoutineId', routineId);  // 루틴 아이디 상태관리 변수에 저장
        // localStorage에 routineId 저장(새로고침, 페이지 이동, 창 끄기 방지)
        localStorage.setItem('routineId', routineId);
        location.reload();  // 페이지 새로고침
    } catch (error) {
        console.error('Error occurred while saving routine ID:', error);
        throw error;
    }
}

const goBack = () => {
    selectedRoutine.value = null;
};

// const router = useRouter();

// const editRoutine = async (routine) => {
//   try {
//     const routineId = routine.routine;
//     // Assuming your edit route is named 'edit'
//     router.push({ name: 'edit', params: { id: routineId } });
//   } catch (error) {
//     console.error('Error navigating to edit page:', error);
//   }
// };


</script>

<style scoped>
.ex1 {
    position: relative;
    background-color: #181B21;
    border-radius: 20px;
    height: 490px;
    margin-bottom: 10px;
}

/*-------------------------------- 위에 박스 세개---------------------------- */
.container {
    display: flex;
}

.box1 {
    position: relative;
    background-color: #6790CE;
    border-radius: 20px;
    width: 420px;
    height: 400px;
    margin-top: 50px;
    margin-left: 10px;
    float: left;
}

.box1_text {
    position: relative;
    color: white;
    font-size: 40px;
    padding-bottom: 50px;
}

.box2 {
    position: relative;
    background-color: #C75B6F;
    border-radius: 20px;
    width: 420px;
    height: 400px;
    margin-top: 50px;
    margin-left: 10px;
    float: left;
}

.box2_text {
    position: relative;
    color: white;
    font-size: 40px;
    padding-bottom: 50px;
}

.box3 {
    position: relative;
    background-color: #8EC693;
    border-radius: 20px;
    width: 420px;
    height: 400px;
    margin-top: 50px;
    margin-left: 10px;
    float: left;
}

.box3_text {
    position: relative;
    color: white;
    font-size: 40px;
    padding-bottom: 50px;
}

/* ------------------------------위에 박스 세개 ---------------------------*/
/* ------------------------------루틴 정보 불러오는 박스 ---------------------------*/
.ex3 {
    background-color: #181B21;
    border-radius: 20px;
    /* height: 650px; */
    margin-bottom: 10px;
}

.daily {
    padding: 20px;
}

.routine-title {
    display: flex;
    justify-content: center;
    color: #FFFFFF;
    font-size: 30px;
    line-height: 30px;
    padding: 35px 0;
}

.routineChangeBtn {
    color: #181B21;
    position: relative;
    float: right;
    margin-right: 5%;
    margin-top: -5%;
}

.day-routine-box {
    margin: 35px;
    color: white;
}

.day-routine-exercise-box {
    padding-left: 1em;
    font-size: 130%;
}

.day-routine-exercise-box-name {
    display: flex;
    height: 40px;
    margin-top: 1em;
    /* line-height: 56px; */
}

.v-checkbox {
    margin-left: 1em;
}

.exercise-usebody-name {
    /* background: #C36C82; */
    padding: 0 0.6em;
    border-radius: 10px;
    margin-right: 0.5em;
}

.day-routine-box-title {
    font-size: x-large;
    background-color: #4B8AAF;
    padding: 0.5em;
    border-radius: 20px;
}

/* ------------------------------루틴 정보 불러오는 박스 ---------------------------*/
/* ----------------------모달 ------------------------------------*/
.btn {
    /* position: relative; */
    display: block;
    background-color: #55B570;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    color: white;
    font-size: 40px;
    /* top: 250px; */
    /* margin-left: 600px; */
    margin: auto;
}

.modal-content {
    background-color: #181B21;
    width: 1100px;
    height: 600px;
    border-radius: 20px;
}

.modal-title {
    font-size: 30px;
    color: white;
    padding-top: 25px;
}

.close-btn {
    color: white;
    font-size: 20px;
    margin: 0 auto;
}

.object {
    background-color: #3D5143;
    border-radius: 20px;
    padding: 80px;
    margin-top: 30px;
    color: white;
}

.login {
    color: white;
}

#routin-table {
    background-color: white;
}

.login-button {
    position: absolute;
    top: 10px;
    right: 10px;
    background-color: #007bff;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.modal {
    transition: opacity 0.5s ease;
}

.modal.fade:not(.show) {
    opacity: 0;
}
</style>