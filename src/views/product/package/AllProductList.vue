<template>
    <v-carousel cycle :show-arrows="false" delimiter-icon="mdi-square" delimiter-color="light_green" height="267"
        interval="5000" hide-delimiter-background>
        <v-carousel-item v-for="(image, index) in images" :key="index" style="overflow: hidden;">
            <img :src="image.src" :alt="image.alt" class="banner-img" />
        </v-carousel-item>
    </v-carousel>
    <v-container class="custom-container">
        <!-- Top 10 패키지 시작 -->
        <v-card style="border-radius: 15px; padding: 20px; padding-bottom: 0px; max-width: 1200px; width: 100%;"
            rounded="0" flat>
            <v-card-title style="font-size: 20px;"> <span style="font-weight: bold;">🏆 BEST 10 </span>
                <span style="font-size: 15px; color: grey;"> 지금 가장 인기있는 패키지를 만나보세요 ! </span>
            </v-card-title>
            <br />
            <Best10PackageSlide />
        </v-card>
        <!-- Top 10 패키지 끝 -->

        <br>
        <div class="hr-style"></div>
        <br>

        <!-- 패키지 리스트 -->
        <v-container style="width: 100%; text-align: start;">

            <v-card-title style="font-size: 20px;"> <span style="font-weight: bold;"> 🥦 패키지 살펴보기 </span>
            </v-card-title>

            <v-row style="margin-top: 20px; padding-right:1.8%">
                <v-col cols="6"></v-col>
                <v-col cols="2">
                    <div class="select-wrapper">
                        <select v-model="sortOption" class="sort-select" @change="onSearch"> <!-- 변경 이벤트 추가 -->
                            <option v-for="option in sortOptions" :key="option" :value="option">{{ option }}</option>
                        </select>
                        <svg-icon type="mdi" :path="mdiMenuDown" class="dropdown-icon"></svg-icon>
                    </div>
                </v-col>
                <v-col cols="4">
                    <form class="searchbar">
                        <input style="width:100%; margin-left: 15px;" placeholder="구독 패키지를 검색해보세요!"
                            v-model="searchQuery" @input="onSearch" />
                        <button type="submit" @click.prevent="onSearch">
                            <img style="margin-right:15px; margin-top: 8px"
                                src="https://d3cpiew7rze14b.cloudfront.net/assets/svg/Search-icon-24x-24_qnmx4o57C.svg"
                                alt="검색">
                        </button>
                    </form>
                </v-col>


            </v-row>

            <v-container class="d-flex custom-card-container">
                <v-row>
                    <v-card v-for="(pkg, index) in packageList" :key="index" class="package-card" md="2" variant="text"
                        style="width:260px; height:500px; margin: 10px; margin-bottom: 15px;" :rounded="false">
                        <v-img class="package-image" :src="pkg.imageUrl" alt="Package 썸네일" cover
                            @click="this.$router.push(`/product/${pkg.id}`)" />
                            <v-chip
                            v-if="isNewProduct(pkg.createdAt)"
                            style="position: absolute; top: 10px; right: 10px; background-color: rgba(0, 128, 0, 0.8); color: white;">
                            NEW !
                        </v-chip>
                        <v-chip
                            style="position: absolute; top: 10px; left: 10px; padding: 5px 10px; border-radius: 8px; background-color: rgba(128, 128, 128, 0.9); color: white;">
                            {{ pkg.deliveryCycle }}일 주기 배송🚚
                        </v-chip>
                        <v-btn style="width: 100%; margin-top:10px; border: 0.5px solid gray; box-shadow: none;"
                            @click="addToWishList(pkg)" v-if="member">
                            <div v-if="wishAnimation.get(pkg.id)" class="heart-emoji">
                                <svg-icon type="mdi" :path="mdiHeart" class="icon-colored"></svg-icon>
                            </div>
                            <svg-icon type="mdi" :path="wishlistItems[pkg.id] ? mdiHeart : mdiHeartOutline"
                                :style="{ marginRight: '2px', color: wishlistItems[pkg.id] ? 'red' : 'black' }"
                                class="heart-icon"></svg-icon>
                            <span style="font-size: 14px;">{{ wishlistItems[pkg.id] ? '위시리스트 취소' : '위시리스트 담기' }}</span>
                        </v-btn>
                        <v-card-text style="padding-left: 0px;">
                            <span style="font-size:medium; font-weight: 400;" v-if="pkg.packageName.length > 20"> {{
                                pkg.packageName.substring(0, 10)
                            }}... </span>
                            <span style="font-size:medium; font-weight: 400;" v-else> {{ pkg.packageName }}</span>
                            <div class="item-info" v-if="pkg.discountId != null && pkg.discountActive == true">
                                <p style="text-decoration: line-through; color: #999; font-size: 14px;">{{
                                    formatPrice(pkg.price) }}</p>
                                <div style="margin-bottom: 2px;">
                                    <span style="color:darkgreen; font-size:medium;">{{ formatPrice(pkg.price -
                                        pkg.discount) }}&nbsp;&nbsp;</span>
                                    <span class="sale-style">SALE</span>
                                </div>
                                <p style="color:#999; font-size: small;"> 1회 제공 금액 {{
                                    formatPrice(getPerCyclePrice(pkg.price - pkg.discount, pkg.deliveryCycle)) }} </p>
                                <p style="color:#999; font-size: small;">
                                    🧾 누적 주문 {{ pkg.orderCount }}
                                </p>
                            </div>
                            <div class="item-info" v-else>
                                <p style="color:darkgreen; font-size:medium;">{{ formatPrice(pkg.price) }}</p>
                                <span style="color:#999; font-size: small;"> 1회 제공 금액 {{
                                    formatPrice(getPerCyclePrice(pkg.price, pkg.deliveryCycle)) }} </span>
                                <br />
                                <span style="color:#999; font-size: small;">
                                    🧾 누적 주문 {{ pkg.orderCount }}
                                </span>
                            </div>
                        </v-card-text>
                    </v-card>
                </v-row>
            </v-container>
        </v-container>
        <!-- 패키지 리스트 끝 -->
    </v-container>
</template>

<script>
import axios from 'axios';
import SvgIcon from '@jamescoyle/vue-icon';
import { mdiHeartPlusOutline } from '@mdi/js';
import { mdiHeartOutline, mdiHeart } from '@mdi/js';
import { mdiMenuDown } from '@mdi/js';
import Best10PackageSlide from '@/components/slide/Best10PackageSlide.vue';
import debounce from 'lodash/debounce';

export default {
    name: "my-component",
    components: {
        SvgIcon,
        Best10PackageSlide
    },
    data() {
        return {
            path: mdiHeartPlusOutline,
            topPackageList: [],
            onboarding: 1,
            packageList: [],
            currentPage: 0,
            currentSlide: 1,
            pageSize: 10,
            searchQuery: "",
            sortOptions: [
                "최신순", "판매량 순"
            ],
            sortOption: "최신순",
            sortOptionMap: new Map(),
            isLoading: false,
            isLastPage: false,

            wishlistItems: [],
            mdiHeartOutline: mdiHeartOutline,
            mdiHeart: mdiHeart,
            mdiMenuDown: mdiMenuDown,

            member: localStorage.getItem("memberId"),

            wishAnimation: new Map(),
            images: [],
        }
    },
    computed: {
        windowCount() {
            return Math.ceil(this.topPackageList.length / 4); // 페이지당 4개의 패키지가 표시된다고 가정
        }
    },
    async created() {
        this.images = [
            { "src": "https://dongsanginong-bucket.s3.ap-northeast-2.amazonaws.com/banner/banner_package1.png", "alt": "배너사진1", "link": "/event2" },
            { "src": "https://dongsanginong-bucket.s3.ap-northeast-2.amazonaws.com/banner/banner_package2.png", "alt": "배너사진2", "link": "/event2" },
            { "src": "https://dongsanginong-bucket.s3.ap-northeast-2.amazonaws.com/banner/banner_package3.png", "alt": "배너사진3", "link": "/event2" },
        ];
        // Top 10 패키지 가져오기
        try {
            const topPackagesResponse = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/product-service/product/no-auth/top10`);
            this.topPackageList = topPackagesResponse.data;
            console.log(this.topPackageList)
            // 전체 패키지 리스트 가져오기
            let params = {
                page: this.currentPage,
                size: this.pageSize,
                sort: this.sortOptionMap.get(this.sortOption),
                packageName: this.searchQuery
            }
            const packageListResponse = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/product-service/product/no-auth`, { params });
            this.packageList = packageListResponse.data.content;
            console.log(this.packageList)
            // sortOptionMap 설정
            this.sortOptionMap.set("최신순", "id,desc");
            this.sortOptionMap.set("판매량 순", "orderCount,desc");
        } catch (e) {
            console.log(e);
        }


        // 페이지네이션을 위한 이벤트 리스너 추가
        window.addEventListener('scroll', this.scrollPagination); // 스크롤 이벤트
        window.addEventListener('keydown', (event) => {
            if (event.key === 'Enter') {
                this.onSearch();
            }
        }); // 엔터 키 이벤트
    },
    async mounted() {
        await this.fetchWishlistItems();
        this.startCarousel();
    },
    methods: {
        startCarousel() {
            setInterval(() => {
                this.nextSlide();
            }, 3000);
        },
        nextSlide() {
            this.currentSlide = this.currentSlide % this.windowCount + 1; // 다음 슬라이드로 이동
        },
        async fetchWishlistItems() {
            try {
                const memberId = localStorage.getItem('memberId');
                if (memberId) {
                    const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/member-service/wishlist`);
                    console.log(">>>>>>>>response : ", response.data);

                    const wishlistProductIds = response.data.map(product => product.id);
                    wishlistProductIds.forEach(id => {
                        this.wishlistItems[id] = true;
                    });
                }
            } catch (error) {
                console.error('위시리스트 정보를 가져오는데 실패했습니다:', error);
            }
        },
        formatPrice(value) {
            if (value == null) {
                return "0원";
            }
            return parseInt(value).toLocaleString('ko-KR') + ' 원'; // 한국어 화폐 양식으로 변환
        },
        getPerCyclePrice(price, deliveryCycle) {
            if (price == null || deliveryCycle == null || deliveryCycle == 0) {
                return 0; // 값이 없거나 deliveryCycle이 0일 경우 0 반환
            }
            // 10단위 반올림 처리
            const perCyclePrice = Math.round(price / (28 / deliveryCycle) / 10) * 10;
            return perCyclePrice;
        },
        next() {
            this.onboarding =
                this.onboarding + 1 > this.windowCount ? 1 : this.onboarding + 1;
        },
        prev() {
            this.onboarding =
                this.onboarding - 1 <= 0 ? this.windowCount : this.onboarding - 1;
        },
        isNewProduct(createdAt) {
            const createdDate = new Date(createdAt);
            const currentDate = new Date();
            const diffTime = Math.abs(currentDate - createdDate);
            const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
            return diffDays <= 7; // 최근 7일 이내 생성된 경우 true 반환
        },
        paginatedPackages(page) {
            // 페이지에 따라 패키지를 반환
            const packagesPerPage = 4;
            const start = (page - 1) * packagesPerPage;
            const end = start + packagesPerPage;
            return this.topPackageList.slice(start, end);
        },
        onSearch: debounce(async function () {
            this.currentPage = 0;
            const params = {
                page: this.currentPage,
                size: this.pageSize,
                sort: this.sortOptionMap.get(this.sortOption),
                packageName: this.searchQuery
            };
            console.log(params);
            this.isLoading = true;

            try {
                const packageListResponse = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/product-service/product/no-auth/search`, { params });
                this.packageList = packageListResponse.data.content;
                this.isLastPage = packageListResponse.data.last;
            } catch (error) {
                console.error(error);
            } finally {
                this.isLoading = false;
            }
        }, 300),
        async loadPackage() {
            try {
                if (this.isLoading || this.isLastPage) return;
                this.isLoading = true;
                this.currentPage++;
                let params = {
                    page: this.currentPage,
                    size: this.pageSize,
                    sort: this.sortOptionMap.get(this.sortOption),
                    packageName: this.searchQuery
                }
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/product-service/product/no-auth`, { params });
                const additionalData = response.data.content;
                this.packageList = [...this.packageList, ...additionalData];
                this.isLastPage = response.data.last;
                this.isLoading = false;
            } catch (e) {
                console.log(e);
                this.isLoading = false;
            }
            console.log(this.currentPage);
        },
        scrollPagination() {
            const isBottom = window.innerHeight + window.scrollY >= document.body.offsetHeight - 100;
            if (isBottom && !this.isLastPage && !this.isLoading) {
                this.loadPackage();
            }
        },
        async addToWishList(packageProduct) {
            try {
                const memberId = localStorage.getItem('memberId');
                await axios.post(`${process.env.VUE_APP_API_BASE_URL}/member-service/wishlist/product/${packageProduct.id}`, {
                    headers: {
                        myId: memberId,
                    }
                });
                this.wishlistItems[packageProduct.id] = !this.wishlistItems[packageProduct.id];
                if (this.wishlistItems[packageProduct.id]) {
                    // 애니메이션 시작
                    this.wishAnimation.set(packageProduct.id, true);
                    setTimeout(() => {
                        this.wishAnimation.set(packageProduct.id, false);
                    }, 1000); // 애니메이션 지속 시간 조절 가능
                }
            } catch (e) {
                console.log(e.message);
            }
        }
    }
}
</script>

<style scoped>
.custom-container {
    max-width: 1200px !important;
    /* 원하는 최대 폭 */
    margin: 0 auto !important;
    /* 중앙 정렬 */
    width: 100% !important;
    /* 컨테이너의 폭을 100%로 설정 */
}

.select-wrapper {
    width: 100%;
    position: relative;
    display: inline-block
}

.dropdown-icon {
    position: absolute;
    top: 37%;
    right: -10px;
    transform: translateY(-50%);
    pointer-events: none;
    /* 클릭 방지 */
}

.searchbar {
    display: flex;
    background-color: rgb(245 245 247);
    border-radius: 4px;
    justify-content: space-between;
    width: 100%;
}

.sort-select {
    background-color: rgb(245 245 247);
    width: 100%;
    margin: 0 2px 10px 15px;
    padding: 8px;
    padding-left: 12px;
    border-radius: 4px;
}

.search-icon {
    transition: color 0.3s ease;
}

/* Apply hover effect */
.search-icon:hover {
    cursor: pointer;
    transition: color 0.3s ease;
}

.package-image {
    object-fit: cover;
    transition: transform 0.3s ease;
    /* border-radius: 10px; */
    width: 260px;
    height: 320px;
}

.package-image:hover {
    transform: scale(1.05);
    cursor: pointer;
}

.custom-card-container {
    display: flex;
    justify-content: left;
}

.heart-icon {
    width: 17px;
    height: 17px;
}

.hr-style {
    border-bottom: 3px solid #efefef;
    border-radius: 3px;
}

.icon-colored {
    color: red;
}

.heart-emoji {
    position: absolute;
    top: -20px;
    /* 필요에 따라 위치 조정 */
    left: 30%;
    transform: translateX(-50%);
    font-size: 24px;
    opacity: 0;
    animation: popUp 1s ease-in-out forwards;
}

@keyframes popUp {
    0% {
        opacity: 0;
        transform: translate(-50%, 0) scale(0);
    }

    50% {
        opacity: 1;
        transform: translate(-50%, -50px) scale(1.5);
    }

    100% {
        opacity: 0;
        transform: translate(-50%, -100px) scale(0);
    }
}

.banner-img {
    object-fit: cover;
    width: 100%;
    /* 부모의 너비에 맞추기 */
    height: 100%;
    /* 부모의 높이에 맞추기 */
}

.sale-style {
    background-color: rgb(245, 77, 77);
    color: white;
    padding-right: 7px;
    padding-left: 7px;
    padding-bottom: 3px;
    padding-top: 5px;
    font-size: 10px;
    margin-bottom: 10px;
}
</style>
