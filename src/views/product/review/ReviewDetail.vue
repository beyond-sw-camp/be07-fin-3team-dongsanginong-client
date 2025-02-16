<template>
  <FarmMenuComponent :currentMenu="3" />
  <v-container class="d-flex justify-center">
    <v-card class="review-card elevation-0" outlined>
      <!-- 별점과 업데이트 날짜를 표시하는 섹션 -->
      <v-row class="review-header">
        <v-col cols="12" class="review-info">
          <div class="header-content">
            <!-- 별점 표시 -->
            <div class="rating">
              <span v-for="star in 5" :key="star">
                <v-icon v-if="star <= review.rating" class="star-icon">mdi-star</v-icon>
                <v-icon v-else class="star-icon">mdi-star-outline</v-icon>
              </span>
            </div>
            <!-- 업데이트 날짜 표시 -->
            <div class="review-date">
              {{ formatDate(review.updateAt) }}
            </div>
          </div>

          <!-- 리뷰 제목 -->
          <div class="review-title">
            <strong>{{ review.title }}</strong>
          </div>

          <!-- 수정/삭제 버튼: 현재 로그인한 유저와 리뷰 작성자가 같을 경우에만 표시 -->
          <v-row class="button-row">
            <v-col v-if="isReviewOwner" cols="12" class="action-buttons" style="margin-top: 1px;">
              <v-btn @click="openEditDialog" class="edit-btn">수정</v-btn>
              <v-btn @click="openDeleteConfirmation" class="delete-btn">삭제</v-btn>
            </v-col>
          </v-row>
        </v-col>
      </v-row>

      <!-- 리뷰 내용과 이미지가 표시되는 섹션 -->
      <v-row class="review-content">
        <v-col cols="12">
          <!-- 리뷰 내용 -->
          <p class="review-text">{{ review.contents }}</p>
          
          <!-- 리뷰 이미지들 -->
          <div class="review-images" v-if="review.imageUrls && review.imageUrls.length">
            <div class="image-slider-container">
              <!-- 이미지가 3장 이상일 때 페이지네이션 버튼 표시 -->
              <button v-if="review.imageUrls.length > imagesPerPage" @click="scrollLeft" class="scroll-button left-button">‹</button>
              <div ref="imageSlider" class="image-slider">
                <img
                  v-for="(imageUrl, i) in visibleImages"
                  :key="i"
                  :src="imageUrl"
                  class="review-image"
                  alt="리뷰 이미지"
                />
              </div>
              <button v-if="review.imageUrls.length > imagesPerPage" @click="scrollRight" class="scroll-button right-button">›</button>
            </div>
          </div>
          <br>
        </v-col>
      </v-row>

      <!-- 삭제 확인 모달 -->
      <v-dialog v-model="deleteModal" max-width="300">
        <v-card class="modal">
          <v-card-title class="modal-title">정말 삭제하시겠습니까?<br><br></v-card-title>
          <v-card-actions class="modal-actions justify-center">
            <v-btn @click="confirmDelete" class="delete-confirm-btn">삭제</v-btn>
            <v-btn @click="closeDeleteModal" class="cancel-btn">닫기</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- 삭제 완료 모달 -->
      <v-dialog v-model="alertModal" max-width="260px">
        <v-card class="modal" style="padding: 10px; padding-right: 20px; text-align: center;">
          <v-card-text>리뷰가 삭제되었습니다.</v-card-text>
          <v-btn @click="closeAlertModal" class="submit-btn">닫기</v-btn>
        </v-card>
      </v-dialog>

      <!-- 리뷰 수정 모달 -->
      <review-update 
        :review-id="review.id" 
        ref="editModal" 
        @review-updated="fetchReviewDetail"
      />
    </v-card>
  </v-container>
</template>

<script>
import ReviewUpdate from '@/views/product/review/ReviewUpdate.vue';
import FarmMenuComponent from '@/components/menubar/FarmMenuComponent.vue';

export default {
  components: {
    ReviewUpdate, // ReviewUpdate 컴포넌트 등록
    FarmMenuComponent
  },
  data() {
    return {
      review: {},
      currentImageIndex: 0,
      imagesPerPage: 2,
      deleteModal: false, // 삭제 확인 모달 상태
      alertModal: false,  // 리뷰 삭제 완료 모달 상태
    };
  },
  computed: {
    visibleImages() {
      return this.review.imageUrls.slice(
        this.currentImageIndex,
        this.currentImageIndex + this.imagesPerPage
      );
    },
    isReviewOwner() {
      // 현재 로그인한 사용자의 memberId와 리뷰 작성자의 memberId 비교
      return localStorage.getItem('memberId') == this.review.memberId;
    },
  },
  mounted() {
    this.fetchReviewDetail();
  },
  methods: {
    async fetchReviewDetail() {
      const reviewId = this.$route.params.reviewId;
      try {
        const response = await fetch(
          `${process.env.VUE_APP_API_BASE_URL}/product-service/reviews/no-auth/detail/${reviewId}`
        );
        if (response.ok) {
          this.review = await response.json();
        } else {
          console.error("리뷰 디테일을 불러오는 중 오류 발생:", response.status);
        }
      } catch (error) {
        console.error("리뷰 디테일을 불러오는 중 오류 발생:", error);
      }
    },
    formatDate(date) {
      if (!date) return '';
      return date.split('T')[0]; // 날짜를 'YYYY-MM-DD' 형식으로 반환
    },
    scrollLeft() {
      this.currentImageIndex = Math.max(this.currentImageIndex - this.imagesPerPage, 0);
    },
    scrollRight() {
      const maxIndex = this.review.imageUrls.length - this.imagesPerPage;
      this.currentImageIndex = Math.min(this.currentImageIndex + this.imagesPerPage, maxIndex);
    },
    // 삭제 확인 모달 열기
    openDeleteConfirmation() {
      this.deleteModal = true;
    },
    // 삭제 모달 닫기
    closeDeleteModal() {
      this.deleteModal = false;
    },
    // 삭제 완료 모달 닫기
    closeAlertModal() {
      this.alertModal = false;
      this.$router.go(-1); // 이전 페이지로 이동
    },
    // 리뷰 삭제
    async confirmDelete() {
      const accessToken = localStorage.getItem('accessToken');
      try {
        await fetch(
          `${process.env.VUE_APP_API_BASE_URL}/product-service/reviews/${this.review.id}/delete`, 
          {
            method: 'DELETE',
            headers: {
              Authorization: `Bearer ${accessToken}`,
            },
          }
        );
        this.alertModal = true;  // 리뷰 삭제 완료 모달 표시
      } catch (error) {
        console.error('리뷰 삭제 실패:', error);
        alert('리뷰 삭제 중 오류가 발생했습니다.');
      }
      this.closeDeleteModal(); // 삭제 확인 모달 닫기
    },
    // 수정 모달 열기
    openEditDialog() {
      this.$refs.editModal.openEditDialog();
    },
  },
};
</script>

<style scoped>
/* 리뷰 카드 디자인 */
.review-card {
  margin-top: 40px;
  width: 800px;
  border: 1px solid #d4d4d4;
  border-radius: 10px;
  padding: 20px 30px;
}

.review-detail-container {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.review-header {
  display: flex;
  justify-content: space-between;
  width: 100%;
  margin-bottom: 10px;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.rating {
  display: flex;
  align-items: center;
}

.star-icon {
  color: #FFCC80;
  font-size: 30px;
  padding: 0 5px;
}

.review-date {
  font-size: 14px;
  color: #888;
  margin-left: auto;
}

.review-title {
  margin-top: 10px;
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 10px;
}

.review-text {
  font-size: 16px;
  line-height: 1.5;
  margin-top: 20px;
}

.image-slider-container {
  display: flex;
  align-items: center;
  position: relative;
  width: 100%;
  margin-top: 20px;
}

.image-slider {
  display: flex;
  gap: 10px;
  overflow: hidden;
}

.review-image {
  width: 300px;
  height: 300px;
  object-fit: cover;
  border-radius: 10px;
}

.scroll-button {
  background-color: white;
  color: rgb(94, 94, 94);
  border: none;
  width: 30px;
  height: 30px;
  cursor: pointer;
  z-index: 1;
  font-size: 12px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.left-button {
  margin-right: 20px; /* 이미지 왼쪽에서 버튼 떨어뜨림 */
}

.right-button {
  margin-left: 20px; /* 이미지 오른쪽에서 버튼 떨어뜨림 */
}

.action-buttons {
  display: flex;
  gap: 10px;
  margin-top: -35px;
  justify-content: flex-end;
}

.edit-btn {
  margin-top: -30px;
  background-color: #BCC07B;
  color: black;
  border-radius: 30px;
  padding: 10px 20px;
  font-size: 13px;
  line-height: 1.5;
  max-width: 200px;
}

.delete-btn{
  margin-top: -30px;
  background-color: #e0e0e0;
  color: black;
  border-radius: 30px;
  padding: 10px 20px;
  font-size: 13px;
  line-height: 1.5;
  max-width: 200px;
}

/* 모달 스타일 */
.modal {
  background-color: rgb(255, 255, 255);
  border: none;
  box-shadow: none;
  border-radius: 10px;
}

.modal-title {
  text-align: center;
  font-weight: bold;
  font-size: 16px;
  margin-top: 10px;
}

.modal-actions {
  margin-top: 10px;
  gap: 10px; /* 버튼 사이 간격 추가 */
}

.delete-confirm-btn{
  background-color: #BCC07B;
  color: black;
  border-radius: 50px;
  padding: 8px 30px;
}

.cancel-btn {
  background-color: #e2e2e2;
  color: black;
  border-radius: 50px;
  padding: 8px 30px;
}


.submit-btn {
  margin-left: 10px;
  margin-top: 8px;
  background-color: #BCC07B;
  color: black;
  border-radius: 50px;
}
</style>
