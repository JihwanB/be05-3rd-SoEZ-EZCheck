<template>
    <div class="user-info">
        <h2>사용자 정보</h2>
        <div class="info-item">
            <label>아이디:</label>
            <p>{{ userInfo.userId }}</p>
        </div>
        <div class="info-item">
            <label>이름:</label>
            <p>{{ userInfo.name }}</p>
        </div>
        <div class="info-item">
            <label>전화번호:</label>
            <p>{{ userInfo.phoneNumber }}</p>
        </div>
        <div class="info-item">
            <label>이메일:</label>
            <p>{{ userInfo.email }}</p>
        </div>
        <button class="delete-account-btn" @click="showDeleteModal = true">회원 탈퇴</button>
        <button class="reset-password-btn" @click="showPwdResetModal = true">비밀번호 재설정</button>

        <!-- 회원 탈퇴 모달 -->
        <div v-if="showDeleteModal" class="modal">
            <div class="modal-content">
                <h2>회원 탈퇴</h2>
                <h6>확인을 위해 비밀번호를 다시 한 번 입력해주세요</h6>
                <br />
                <input v-model="password" type="password" placeholder="비밀번호">
                <div class="button-container">
                    <button class="cancel-btn" @click="cancelDeletion">취소</button>
                    <button class="delete-btn" @click="showDeleteConfirmModal = true">회원 탈퇴</button>
                </div>
            </div>
        </div>

        <div v-if="showDeleteConfirmModal" class="modal">
            <div class="modal-content">
                <h2>회원 탈퇴</h2>
                <h6>정말로 탈퇴하시겠습니까?</h6>
                <div class="button-container">
                    <button class="cancel-btn" @click="cancelDeletion">NO</button>
                    <button class="delete-btn" @click="deleteAccount">YES</button>
                </div>
            </div>
        </div>

        <!-- <div v-if="isDeleteClicked" class="modal">
            <div class="modal-content">
                <p>{{ deleteResult.value }}</p>
                <button class="cancel-btn" @click="routeToHomePage">확인</button>
            </div>
        </div> -->

        <!-- 비밀번호 재설정 모달 -->
        <div v-if="showPwdResetModal" class="modal">
            <div class="modal-content">

                <h2>비밀번호 재설정</h2>
                <h6>회원가입시 입력한 이메일을 입력해주세요</h6>
                <br />

                <div class="auth-wrapper">
                    <label for="email" class="">이메일</label>
                    <input class="input-class" v-model="email" type="email" placeholder="e-mail@example.com">
                </div>
                <div class="button-wrapper" @click="sendAuthEmail">인증메일 발송</div>


                <div v-if="isAuthEmailSent" class="auth-wrapper">
                    <label for="authCode" class="">인증번호</label>
                    <input class="input-class" v-model="authCode" type="text" placeholder="인증번호">
                </div>

                <div v-if="isAuthEmailSent" class="button-wrapper" @click="checkAuthCode">인증확인</div>

                <hr />

                <div v-if="isAuthCodeValid" class="auth-wrapper">
                    <label for="newPassword" class="">새 비밀번호</label>
                    <div class="">
                        <input v-model="newPassword" type="password" placeholder="변경할 비밀번호" class="input-class">
                    </div>
                </div>

                <div class=" button-container">
                    <button class="btn cancel-btn" @click="cancelReset">취소</button>
                    <button class="btn delete-btn" @click="resetPassword" :disabled="!isAuthCodeValid">재설정</button>
                </div>

            </div>
        </div>

        <div v-if="showResetResultModal" class="modal">
            <div class="modal-content">
                <h2>비밀번호 재설정 결과</h2>
                <p>{{ resetResult }}</p>
                <button class="cancel-btn" @click="closeAllModal">확인</button>
            </div>
        </div>

    </div>
</template>

<script>
import { onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';
import axios from '@/axios';

export default {
    name: 'UserInfo',

    props: {
        userInfo: {
            type: Object,
            required: true
        }
    },

    setup(props, { emit }) {
        const router = useRouter();
        const showDeleteModal = ref(false);
        const showPwdResetModal = ref(false);
        const showDeleteConfirmModal = ref(false);
        const password = ref('');
        const email = ref('');
        const authCode = ref('');
        const newPassword = ref('');
        const isAuthEmailSent = ref(false);
        const isAuthCodeValid = ref(false);
        const showResetResultModal = ref(false);
        const resetResult = ref('');
        const deleteResult = ref('');
        const isDeleteClicked = ref(false);

        const deleteAccount = async () => {
            try {
                const response = await axios.delete('http://localhost:8080/user/delete-account', {
                    data: {
                        userId: localStorage.getItem('userId'),
                        password1: password.value,
                    }
                });
                deleteResult.value = response.data;
                if (response.status === 200) {
                    console.log('회원 탈퇴 성공:', response.data);
                    localStorage.removeItem('token');
                    localStorage.removeItem('userId');
                    alert('회원 탈퇴가 완료되었습니다.');
                    router.push('/');
                } else {
                    console.log('회원 탈퇴 실패', response.data);
                    alert('회원 탈퇴에 실패했습니다.');
                }
            } catch (error) {
                console.error('회원 탈퇴 오류:', error);
            }
            // isDeleteClicked.value = true;
            showDeleteModal.value = false;
        };

        const cancelDeletion = () => {
            showDeleteConfirmModal.value = false;
            showDeleteModal.value = false;
        };

        const cancelReset = () => {
            showPwdResetModal.value = false;
        };

        const fetchUserInfo = async () => {
            try {
                const response = await axios.post('http://localhost:8080/user/userinfo', {
                    userId: localStorage.getItem('userId')
                });
                emit('update:userInfo', response.data);
                console.log("사용자 정보:", response.data);
            } catch (error) {
                console.error('사용자 정보 불러오기 실패:', error);
            }
        };

        const sendAuthEmail = async () => {
            try {
                alert("인증 메일이 발송되었습니다.");
                await axios.post('http://localhost:8080/user/auth/send', {
                    email: email.value
                });
                console.log(email);
                isAuthEmailSent.value = true
            }
            catch (error) {
                console.error(error)
            }
        };

        const checkAuthCode = async () => {
            try {
                const response = await axios.post('http://localhost:8080/user/auth/check', {
                    email: email.value, authCode: authCode.value
                });
                isAuthCodeValid.value = response.data
                if (isAuthCodeValid.value === true) {
                    alert("인증되었습니다.");
                } else {
                    alert("인증코드가 일치하지 않습니다.");
                }
            }
            catch (error) {
                console.error(error)
            }
        };

        const resetPassword = async () => {
            try {
                const response = await axios.post('http://localhost:8080/user/change-password', {
                    userId: localStorage.getItem('userId'),
                    newPassword: newPassword.value,
                });
                resetResult.value = response.data;
                if (response.status === 200) {
                    console.log(response.data);
                    // alert("비밀번호가 재설정되었습니다.");
                }
                else {
                    console.log(response.data);
                    // alert("비밀번호 재설정에 실패했습니다.");
                }
            }
            catch (error) {
                console.error("비밀번호 재설정 오류", error);
                // alert("비밀번호 재설정에 실패했습니다.");
            }
            showResetResultModal.value = true;
            email.value = '';
            authCode.value = '';
            newPassword.value = '';
        }

        const closeAllModal = () => {
            showResetResultModal.value = false;
            showPwdResetModal.value = false;
        }

        const routeToHomePage = () => {
            isDeleteClicked.value = false;
            showDeleteConfirmModal.value = false;
            router.push('/');
        }


        onMounted(() => {
            fetchUserInfo();
        });

        return {
            showDeleteModal,
            showPwdResetModal,
            showResetResultModal,
            showDeleteConfirmModal,
            password,
            email,
            authCode,
            isAuthEmailSent,
            isAuthCodeValid,
            newPassword,
            resetResult,
            isDeleteClicked,
            deleteResult,
            sendAuthEmail,
            deleteAccount,
            cancelDeletion,
            cancelReset,
            checkAuthCode,
            resetPassword,
            closeAllModal,
            routeToHomePage,
        }
    }
}
</script>

<style scoped>
.user-info {
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 5px;
    background-color: #f9f9f9;
}

h2 {
    font-size: 1.5rem;
    margin-bottom: 20px;
}

.info-item {
    margin-bottom: 10px;
}

label {
    font-weight: bold;
}

button {
    padding: 10px 20px;
    margin-right: 10px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.reset-password-btn {
    background-color: #007bff;
    color: #fff;
}

.delete-account-btn {
    background-color: #dc3545;
    color: #fff;
}

.modal {
    position: fixed;
    z-index: 9998;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
}

.modal-content {
    background-color: #fff;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
    padding: 20px;
    max-width: 50%;
    /* Reduced width */
}

.modal-content h2 {
    margin-bottom: 20px;
}

.modal-content input[type="password"] {
    width: calc(100% - 20px);
    padding: 5px;
    border: 1px solid #ccc;
    border-radius: 5px;
    box-sizing: content-box;
}

.button-container {
    display: flex;
    justify-content: flex-end;
    margin-top: 20px;
    /* Increase spacing between buttons */
}

.cancel-btn,
.delete-btn {
    padding: 8px 16px;
    /* Reduced button size */
    margin-left: 10px;
    /* Increase spacing between buttons */
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.cancel-btn {
    background-color: #dc3545;
    color: #fff;
}

.delete-btn {
    background-color: #007bff;
    color: #fff;
}

.cancel-btn:hover,
.delete-btn:hover {
    opacity: 0.8;
}

.auth-wrapper {
    display: flex;
    flex-direction: row;
    justify-content: space-between;

}

.button-wrapper {
    background-color: #007bff;
    color: white;
    border-radius: 5px;
    padding: 10px 10px;
    text-align: center;
    cursor: pointer;
    margin-top: 10px;
    margin-bottom: 10px;
    margin-left: auto;
}

.input-class {
    width: 80%;
}
</style>