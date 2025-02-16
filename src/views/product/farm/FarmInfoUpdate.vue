<template>
    <div class="farm-create-page">
        <SellerSidebar />
        <h3 class="title">농장 정보 수정</h3>

        <!-- 배너 이미지 업로드 -->
        <div class="image-upload banner-upload">
            <img :src="bannerImageUrl || defaultBanner" alt="배너 이미지" class="banner-image"
                @click="isEditing ? triggerBannerUpload() : null" />
            <input type="file" @change="onBannerImageUpload" class="image-input" ref="bannerInput" />

            <!-- 수정하기 버튼을 눌렀을 때만 보이도록 설정 -->
            <div class="upload-button banner-upload-button" @click="triggerBannerUpload" v-if="isEditing">+</div>

            <div class="profile-upload-wrapper">
                <img :src="profileImageUrl || defaultProfile" class="profile-image"
                    @click="isEditing ? triggerProfileUpload() : null" />
                <input type="file" @change="onProfileImageUpload" class="image-input" ref="profileInput" />
                <!-- 수정하기 버튼을 눌렀을 때만 보이도록 설정 -->
                <div class="upload-button profile-upload-button" @click="triggerProfileUpload" v-if="isEditing">+
                </div>
            </div>
        </div>
        <div class="farm-name">
            <h3 v-if="!isEditing">{{ farmName }}</h3>
        </div>

        <br />

        <!-- 수정하기 버튼을 눌렀을 때만 input 보이기 -->
        <input type="text" v-model="farmName" placeholder="농장 이름을 입력해주세요." class="farm-name-input" v-if="isEditing" />
        <br />

        <div class="selected-category-space">
            <h4 class="left-align" v-if="!isEditing">선택된 카테고리</h4><br />
            <div class="selected-categories">
                <span v-for="categoryId in selectedCategories" :key="categoryId" class="category-chip">
                    {{ getCategoryName(categoryId) }}
                    <!-- 수정하기 버튼을 눌렀을 때만 X 버튼 보이기 -->
                    <button v-if="isEditing" @click="removeCategory(categoryId)">
                        <span style="font-size: 12px;">X</span></button>
                </span>
            </div>
        </div>

        <div class="category-selection" v-if="isEditing">
            <h4 class="left-align">농장 카테고리 선택</h4>
            <br />
            <div class="category-list">
                <button v-for="category in categories" :key="category.id" @click="selectCategory(category.id)"
                    class="category-button" :class="{ selected: selectedCategories.includes(category.id) }">
                    {{ category.title }}
                </button>
            </div>
        </div>

        <div class="farm-intro">
            <h4 class="left-align" v-if="isEditing">농장 설명을 적어주세요.</h4><br />
            <!-- 수정하기 버튼을 눌렀을 때만 textarea 보이기 -->
            <textarea v-model="farmIntro" class="intro-textarea" v-if="isEditing" maxlength="50"></textarea>
            <div class="char-count" v-if="isEditing">{{ farmIntro.length }}/50</div>
            <h4 v-if="!isEditing" class="left-align">농장 한 줄 소개</h4>
            <div class="intro-area" v-if="!isEditing">
                <p v-if="!isEditing">{{ farmIntro }}</p>
            </div>
        </div>

        <div class="button-group" v-if="isEditing">
            <button @click="submitFarm" class="submit-button">저장</button>
            <button @click="cancelEdit" class="submit-button cancel-button">취소</button>
        </div>        
        <button @click="editFarm" class="edit-button" style="margin-top: 10%;" v-else>수정하기</button>
    </div>

    <!-- 모달 -->
    <v-dialog v-model="cancelModal" max-width="260px">
        <v-card class="modal" style="width:150%; padding: 10px; padding-right: 20px; text-align: center;">
            <v-card-text>변경 사항이 모두 삭제됩니다. <br />
                취소하시겠습니까?</v-card-text>
            <div class="modal-buttons">
                <v-btn @click="confirmCancel" class="submit-btn half-width">확인</v-btn>
                <v-btn @click="cancelModal = false" class="submit-btn half-width"
                    style="background-color: lightgrey;">취소</v-btn>
            </div>

        </v-card>
    </v-dialog>

    <v-dialog v-model="alertModal" max-width="260px">
        <v-card class="modal" style="padding: 15px; text-align: center;">
            <v-card-text style="text-align: center;">완료되었습니다.</v-card-text>
            <v-btn @click="closeModalAndRedirect" class="submit-btn" style="margin-top: -2px; margin-right: 10px;
            font-weight: 400;">닫기</v-btn>
        </v-card>
    </v-dialog>


    <v-dialog v-model="cropModal" max-width="800px">
        <v-card class="modal" style="padding: 10px; text-align: center;">
            <v-card-text>
                <div class="cropper-container flex">
                    <div class="cropper-image-container w-3/4">
                    <img ref="cropperImage" :src="cropImageSrc" alt="Cropper Image" style="max-width: 100%;" />
                    </div>
                    <div class="cropper-preview-container w-1/4">
                    <h4>프리뷰</h4>
                    <div class="preview overflow-hidden w-full h-32 bg-gray-200"></div>
                    </div>
                </div>

            <!-- 이미지 처리 중일 때 메시지 표시 -->
            <div v-if="loading" class="loading-message" style="color: gray; margin-top: 20px;">
                이미지를 처리하고 있습니다...
            </div>
            </v-card-text>
            <div class="cropper-controls">
                <button @click="saveCrop" class="cropper-control-btn save-btn" :disabled="loading">저장</button>
                <button @click="cancelCrop" class="cropper-control-btn cancel-btn" :disabled="loading">취소</button>
            </div>
        </v-card>
        </v-dialog>
</template>

<script>
import SellerSidebar from '@/components/sidebar/SellerSidebar.vue';
import axios from 'axios';
import Cropper from 'cropperjs';
import 'cropperjs/dist/cropper.css';

export default {
    components: {
        SellerSidebar,
    },
    data() {
        return {
            isEditing: false,
            cancelModal: false,
            farmName: '',
            farmIntro: '',
            bannerImageUrl: '',
            profileImageUrl: '',
            projectImageFile: null,
            categories: [],
            selectedCategories: [],
            defaultBanner: '/baseBannerImage.png',
            defaultProfile: '',
            alertModal: false,
            cropModal: false,   // 크롭 모달을 위한 상태
            cropImageSrc: '',   // 크롭할 이미지 URL
            cropper: null,      // Cropper 인스턴스
            loading: false,     // 이미지 처리 중 로딩 상태
        };
    },
    created() {
        this.loadCategories();
        this.loadFarmInfo();
    },
    methods: {
        async loadFarmInfo() {
            try {
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/product-service/farm/info`);
                const farmInfo = response.data;
                this.farmName = farmInfo.farmName;
                this.farmIntro = farmInfo.farmIntro;
                this.bannerImageUrl = farmInfo.bannerImageUrl;
                this.profileImageUrl = farmInfo.profileImageUrl;
                this.selectedCategories = farmInfo.categories;
            } catch (error) {
                console.error('농장 정보 불러오기 실패:', error);
            }
        },

        async loadCategories() {
            try {
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/product-service/farm/categories`);
                this.categories = response.data;
            } catch (error) {
                console.error('카테고리 불러오기 실패:', error);
            }
        },

        editFarm() {
            this.isEditing = true; // 수정 모드 활성화
        },

        selectCategory(categoryId) {
            if (!this.selectedCategories.includes(categoryId)) {
                this.selectedCategories.push(categoryId);
            }
        },

        removeCategory(categoryId) {
            this.selectedCategories = this.selectedCategories.filter(id => id !== categoryId);
        },

        getCategoryName(categoryId) {
            const category = this.categories.find(cat => cat.id === categoryId);
            return category ? category.title : '';
        },

        triggerProfileUpload() {
            this.$refs.profileInput.click();
        },

        triggerBannerUpload() {
            this.$refs.bannerInput.click();
        },

        async uploadImage(blob) {
            const accessToken = localStorage.getItem('accessToken');
            const body = {
                prefix: "Farm",
                url: `${blob?.name}`,
            };
            const headers = {
                Authorization: `Bearer ${accessToken}`,
                "Content-Type": "application/json",
            };
            const getUrl = await fetch(`${process.env.VUE_APP_API_BASE_URL}/product-service/api/upload/presigned-url`, {
                method: "POST",
                headers: headers,
                body: JSON.stringify(body),
            });

            const urlContentType = getUrl.headers.get("content-type");
            let getUrlResult;
            if (urlContentType && urlContentType.includes("application/json")) {
                getUrlResult = await getUrl.json();
            } else {
                getUrlResult = await getUrl.text();
            }

            const awsUrl = {
                data: `${getUrlResult.split("?")[0]}`,
                auth: `?${getUrlResult.split("?")[1]}`,
            };

            const options = {
                method: "PUT",
                headers: {
                    "Content-Type": blob.type,
                },
                body: blob,
            };
            await fetch(awsUrl.data + awsUrl.auth, options);

            return awsUrl.data;
        },

        async onBannerImageUpload(event) {
            const file = event?.target?.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (e) => {
                this.cropImageSrc = e.target.result;  // 선택한 이미지를 크롭 모달에 표시
                this.cropModal = true;                // 크롭 모달 열기
                };
                reader.readAsDataURL(file);
                this.projectImageFile = file; // 원본 파일 저장
            }
        },

        saveCrop() {
            if (this.cropper) {
                this.loading = true; // 로딩 상태 시작
                const dpr = window.devicePixelRatio || 1;

                // 크롭된 캔버스를 가져와서 고해상도 처리
                const croppedCanvas = this.cropper.getCroppedCanvas({
                width: this.cropper.getCroppedCanvas().width * dpr,
                height: this.cropper.getCroppedCanvas().height * dpr,
                imageSmoothingEnabled: true,
                imageSmoothingQuality: 'high',
                });

                if (croppedCanvas) {
                    croppedCanvas.toBlob(async (blob) => {
                        const croppedFile = new File([blob], this.projectImageFile.name, { type: 'image/png' });
                        this.bannerImageUrl = await this.uploadImage(croppedFile); // 이미지 업로드
                        this.loading = false; // 로딩 상태 종료
                        this.cropModal = false; // 모달 닫기
                        this.cropper.destroy(); // Cropper 인스턴스 제거
                        this.cropper = null;
                    }, 'image/png', 1);
                }
            }
        },

        initializeCropper() {
            const image = this.$refs.cropperImage;
                if (this.cropper) {
                    this.cropper.destroy();
                }
                this.cropper = new Cropper(image, {
                    aspectRatio: 3 / 1, // 배너 이미지의 비율
                    viewMode: 2,
                    dragMode: 'crop',
                    preview: '.preview',
                    autoCrop: true,
                    autoCropArea: 1.0,
                    movable: false,
                    zoomable: false,
                    rotatable: false,
                    scalable: false,
                    cropBoxMovable: true,
                    cropBoxResizable: false,
                    modal: true,
                    guides: true,
                    center: true,
                    highlight: true,
                    background: true,
                    responsive: true,
                    checkCrossOrigin: true,
                    checkOrientation: true,
                    ready() {
                    const cropperData = this.cropper.getCropBoxData();
                    const containerData = this.cropper.getContainerData();
                    this.cropper.setCropBoxData({
                        left: (containerData.width - cropperData.width) / 2,
                        top: (containerData.height - cropperData.height) / 2,
                        width: cropperData.width,
                        height: cropperData.height
                    });
                },
            });
        },

        cancelCrop() {
            this.cropModal = false; // 모달 닫기
            if (this.cropper) {
                this.cropper.destroy(); // Cropper 인스턴스 제거
                this.cropper = null;
            }
        },

        async onProfileImageUpload(event) {
            this.projectImageFile = event?.target?.files[0];
            this.profileImageUrl = await this.uploadImage(this.projectImageFile);
        },

        closeModalAndRedirect() {
            this.alertModal = false;
            location.reload();
        },

        async submitFarm() {
            try {
                const farmData = {
                    farmName: this.farmName,
                    farmIntro: this.farmIntro,
                    bannerImageUrl: this.bannerImageUrl,
                    profileImageUrl: this.profileImageUrl,
                    categoryId: this.selectedCategories,
                };

                await axios.post(`${process.env.VUE_APP_API_BASE_URL}/product-service/farm/update`, farmData);
                this.alertModal = true;
                this.isEditing = false; // 수정 완료 후 수정 모드 종료
            } catch (error) {
                console.error('농장 수정 실패:', error);
                alert(error.response?.data?.message || "농장 수정 중 문제가 발생했습니다.");
            }
        },
        cancelEdit() {
            this.cancelModal = true;
        },
        confirmCancel() {
            this.isEditing = false;
            this.cancelModal = false;
            this.loadFarmInfo();
        },
    },
    watch: {
        cropModal(newVal) {
            if (newVal) {
                this.$nextTick(() => {
                this.initializeCropper(); // 모달 열릴 때 Cropper 초기화
                });
            } else {
                if (this.cropper) {
                    this.cropper.destroy(); // 모달 닫힐 때 Cropper 인스턴스 제거
                    this.cropper = null;
                }
            }
        }
    }
};
</script>


<style scoped>
.farm-create-page {
    padding: 20px;
    max-width: 800px;
    margin: 10px 100px auto;
}

.title {
    font-weight: bold;
    margin-bottom: 20px;
    text-align: left;
}

.image-upload {
    margin-bottom: 20px;
    text-align: center;
    position: relative;
    /* 배너 위에 프로필을 겹치기 위해 상대 위치 지정 */
}

.banner-upload img {
    width: 100%;
    height: 250px;
    object-fit: cover;
    position: relative;
    border: 1px solid #ccc;
}

.profile-name-wrapper {
    display: flex;
}

.profile-upload-wrapper {
    position: absolute;
    bottom: -70px;
    left: 10%;
    transform: translateX(-50%);
    /* 가로 중앙 정렬 */
    width: 120px;
    height: 120px;
    border-radius: 50%;
    overflow: hidden;
    background-color: #5f5f5f;
    /* 배경색 */
    border: 1px solid #ccc;
}

.profile-upload-wrapper img {
    background-color: #5f5f5f;
    width: 110%;
    height: 110%;
    object-fit: cover;
}

.image-input {
    display: none;
}

/* + 모양 버튼 스타일 */
.upload-button {
    position: absolute;
    width: 30px;
    height: 30px;
    background-color: lightgray;
    color: white;
    font-size: 24px;
    text-align: center;
    border-radius: 50%;
    cursor: pointer;
    line-height: 30px;
}

.banner-upload-button {
    top: 120px;
    right: 370px;
}

.profile-upload-button {
    bottom: 43px;
    right: 44px;
}

.farm-name {
    display: flex;
    text-align: left;
    margin: 1% 0 0 20%;
}

.farm-name-input {
    padding: 10px;
    font-size: 16px;
    background-color: #f0f0f0;
    border: 1px solid #ccc;
    border-radius: 8px;
    margin-left: 152px;
    position: relative;
    top: -30px;

}

.selected-category-space {
    margin-top: 5%;
    margin-bottom: 20px;
    text-align: center;
}

.selected-categories {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    padding: 10px;
    background-color: #f0f0f0;
    border-radius: 8px;
    border: 1px solid #ccc;
    min-height: 70px;
    /* 초기 높이를 설정합니다 */
    overflow: visible;
    /* 스크롤이 생기지 않게 설정합니다 */
}

.category-chip {
    background-color: #BCC07B;
    padding: 5px 10px;
    border-radius: 20px;
    margin-right: 10px;
    margin-bottom: 10px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    /* 양쪽에 텍스트와 버튼을 배치 */
    color: white;
}

.category-chip button {
    margin-left: 8px;
    background-color: #FFE2A6;
    border: none;
    border-radius: 50%;
    color: black;
    width: 20px;
    height: 20px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    /* X 버튼을 중앙에 위치 */
}

.category-selection {
    margin-bottom: 20px;
    text-align: center;
}

.category-list {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
}

.category-button {
    margin-right: 10px;
    margin-bottom: 10px;
    background-color: #e0e0e0;
    padding: 5px 10px;
    border: none;
    border-radius: 20px;
    cursor: pointer;
}

.category-button.selected {
    background-color: #BCC07B;
    color: black;
}

.farm-intro {
    text-align: center;
}

.intro-area {
    height: 100px;
    text-align: start;
    padding: 15px 0 0 15px;
    margin-top: 3%;
    background-color: #f0f0f0;
    border-radius: 8px;
    border: 1px solid #ccc;
}

.intro-textarea {
    width: 100%;
    height: 200px;
    padding: 10px;
    font-size: 16px;
    background-color: #f0f0f0;
    border: 1px solid #ccc;
    border-radius: 8px;
}

.submit-button {
    background-color: #bcc07b;
    padding: 10px 10px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    width: 48%;
}

.edit-button {
    background-color: #bcc07b;
    padding: 10px 20px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    width: 100%;
}

.profile-image {
    top: -2%;
    right: 2%;
}

.left-align {
    text-align: left;
}

.modal {
    background-color: rgb(255, 255, 255);
    border: none;
    box-shadow: none;
    border-radius: 10px;
}

.submit-btn {
    margin-left: 10px;
    margin-top: 8px;
    background-color: #BCC07B;
    color: black;
    border-radius: 50px;
}

.modal-buttons {
    display: flex;
    justify-content: space-around;
    margin-top: 20px;
}

.half-width {
    width: 45%;
}

.button-group {
    display: flex;
    justify-content: space-between;
    margin-top: 10px; /* 필요한 경우 여백 추가 */
}

.cancel-button {
    background-color: lightgrey; /* 취소 버튼 스타일 */
}

.cropper-container {
  display: flex;
  justify-content: space-between;
}

.cropper-image-container {
  width: 70%; /* 이미지를 표시할 공간 */
}

.cropper-preview-container {
  width: 25%; /* 프리뷰 공간 */
  text-align: center;
}

.preview {
  width: 100%;
  height: 150px;
  overflow: hidden;
  background-color: #f0f0f0;
  border: 1px solid #ddd;
  margin-top: 10px;
}

.loading-message {
  font-size: 14px;
  color: #999;
}

.cropper-controls {
  display: flex;
  justify-content: center;
  gap: 10px; /* 버튼 사이의 간격을 추가 */
  margin-top: 10px;
}

.cropper-control-btn {
  color: black;
  border: none;
  border-radius: 20px;
  padding: 10px 20px;
  margin-bottom: 20px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.save-btn {
  background-color: #bcc07b;
}

.save-btn:hover {
    background-color: #a8b05b;
}

.cancel-btn {
  background-color: #e0e0e0;
  color: black;
}

.cancel-btn:hover {
  background-color: #c0c0c0;
}

.char-count {
    font-size: 12px;
    color: #666;
    text-align: right;
    margin-top: 5px;
}
</style>