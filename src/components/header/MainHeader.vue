<template>
    <v-app-bar app dark dense height="75">
        <v-container class="bar" fluid>
            <v-row no-gutters class="flex-nowrap justify-space-between">
                <!-- Left-aligned buttons -->
                <v-col class="d-flex justify-start" style="margin-top: 12px; font-size: 15px; cursor:pointer;">
                    <span style="text-transform: none; margin-left: 5%;" @click="goTo('/farm')">농장</span>
                    <span @click="goTo(`/live/list`)" style="text-transform: none; margin-left: 5%;">라이브</span>
                    <span @click="goTo(`/product`)" style="text-transform: none; margin-left: 5%;">정기구독 패키지</span>
                </v-col>
                <v-col class="text-center">
                    <v-btn @click="goTo(`/`)" color="white" style="margin-top: 12px;">
                        <img src="/123.png" width=110 alt="Logo" class="main-logo-image" />
                    </v-btn>
                </v-col>
                <!-- Right-aligned buttons -->
                <v-col class="d-flex justify-end" style="margin-top: 12px; font-size: 15px; cursor:pointer;">
                    <span @click="goTo(`/member/my-page`)" style="text-transform: none; margin-right: 5%;"
                        v-if="!isSeller && isLogin">마이페이지</span>
                    <span style="text-transform: none; margin-right: 5%;" v-if="isSeller && isLogin" 
                    @click="checkFarmAndRedirect">내 농장
                        관리</span>
                    <span style="text-transform: none; margin-right: 1%;" v-if="!isLogin" class="reduce-spacing"
                    @click="goTo(`/member/sign-in`)">로그인</span>
                    <span style="text-transform: none; margin-right: 1%;" v-if="isLogin" class="reduce-spacing"
                        @click="alertModal = true">로그아웃</span> 

                    <v-menu>
                        <template v-slot:activator="{ props }">
                            <v-btn v-bind="props" color="white" class="reduce-spacing" style="margin-top: -8px;">
                                <img src="/notifications.png" width=25 alt="Logo" />
                                <span v-if="notifications.length > 0" class="notification-mark"></span>
                            </v-btn>
                        </template>
                        <v-list style="background-color: #EAEAEA; padding: 5px; width: 400px;">
                            <v-list-item v-if="notifications.length === 0">
                                <v-list-item-title>새로운 알림이 없네요!</v-list-item-title>
                            </v-list-item>
                            <v-card v-for="(notification, index) in notifications" :key="index"
                                style="margin: 5px; margin-bottom: 7px; padding-bottom: 5px;"
                                @click="readNotificationAndRoute(notification)">
                                <v-card-text style="font-weight: bold;">
                                    {{ notification.title }}
                                </v-card-text>
                                <v-card-subtitle style="font-size:small; margin-bottom: 10px;">{{
                                    notification.content }}</v-card-subtitle>

                            </v-card>
                            <v-list-item @click="markAsRead()" style="text-align: end; font-size: 15px;">✅ 전체 읽음 처리
                                표시
                            </v-list-item>
                        </v-list>
                    </v-menu>


                    <v-btn color="white" @click="openSearchModal" style="margin-top: -8px;">
                        <img src="/searchLogo.png" width=17 alt="Logo" />
                    </v-btn>
                </v-col>
            </v-row>
        </v-container>
    </v-app-bar>

    <v-dialog v-model="alertModal" max-width="300px">
        <v-card class="modal" style="align-items: center; text-align: center; height: 160px; padding-bottom: 20px; 
        overflow-y: hidden;">
            <v-card-text style="margin-top: 10%;">로그아웃 되었습니다.</v-card-text>
            <v-btn @click="handleLogout" class="submit-btn">닫기</v-btn>
        </v-card>
    </v-dialog>

    <v-dialog v-model="firstFarmModal" max-width="260px">
        <v-card class="farmModal" style="padding: 10px; padding-right: 20px; text-align: center;">
            <v-card-text>
                농장 정보가 존재하지 않네요.<br>농장 정보를 입력해주세요.
            </v-card-text>
            <v-btn @click="goToFarmCreate" class="submit-btn">확인</v-btn>
        </v-card>
    </v-dialog>

    <!-- 검색 모달 -->
    <v-dialog v-model="searchDialog" max-width="550px" class="search-modal" @close="closeModal">
        <v-card class="search-card" style="height: 430px;">
            <v-row style="height: 5%;">
                <v-text-field v-model="keyword" label="검색어를 입력하세요." prepend-inner-icon="mdi-magnify" outlined
                    style="margin-bottom: -15px;" @input="debouncedSearch"></v-text-field>
                <!-- <v-btn @click="performSearch" class="search-btn" style="margin-top: 17px;">검색</v-btn> -->
            </v-row>
            <v-row style="justify-content: center; padding-top: 20px;">
                <v-btn class="cat-btn" outlined style="border-color: #525252; border-width: 1px;"
                    :class="{ active: selectedCategory === 'all' }" @click="selectedCategory = 'all'">
                    전체
                </v-btn>
                <v-btn class="cat-btn" outlined style="border-color: #525252; border-width: 1px;"
                    :class="{ active: selectedCategory === 'farm' }" @click="selectedCategory = 'farm'">
                    농장
                </v-btn>
                <v-btn class="cat-btn" outlined style="border-color: #525252; border-width: 1px;"
                    :class="{ active: selectedCategory === 'package' }" @click="selectedCategory = 'package'">
                    패키지
                </v-btn>
            </v-row>

            <v-card class="result-card" style="height: 280px; overflow-y: auto;">
                <v-list v-if="total.length > 0 && selectedCategory === 'all'">
                    <v-card v-for="item in total" :key="item.id" class="list-card" @click="goToDetail(item)">
                        <v-card-title style="font-size: 15px;">
                            <span class="mdi mdi-magnify" style="margin-right: 10px;"></span>
                            <span v-html="highlightKeyword(item.farmName || item.packageName)"></span>
                        </v-card-title>
                        <v-card-text style="font-size: 14px;">
                            <!-- <span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span> -->
                            <span v-html="highlightKeyword(item.farmIntro || item.productDescription)"></span>
                        </v-card-text>
                    </v-card>
                </v-list>
                <v-list v-else-if="farms.length > 0 && selectedCategory === 'farm'">
                    <v-card v-for="farm in farms" :key="farm.id" class="list-card" @click="goToFarmDetail(farm.id)">
                        <v-card-title style="font-size: 15px;">
                            <span class="mdi mdi-magnify" style="margin-right: 10px;"></span>
                            <span v-html="highlightKeyword(farm.farmName)"></span>
                        </v-card-title>
                        <v-card-text style="font-size: 14px;">
                            <!-- <span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span> -->
                            <span v-html="highlightKeyword(farm.farmIntro)"></span>
                        </v-card-text>
                    </v-card>
                </v-list>
                <v-list v-else-if="products.length > 0 && selectedCategory === 'package'">
                    <v-card v-for="product in products" :key="product.id" class="list-card"
                        @click="goToPackageDetail(product.id)">
                        <v-card-title style="font-size: 15px;">
                            <span class="mdi mdi-magnify" style="margin-right: 10px;"></span>
                            <span v-html="highlightKeyword(product.packageName)"></span>
                        </v-card-title>
                        <v-card-text style="font-size: 14px;">
                            <!-- <span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span> -->
                            <span v-html="highlightKeyword(product.productDescription)"></span>
                        </v-card-text>
                    </v-card>
                </v-list>
                <v-list v-else>
                    <v-list-item>
                        <v-list-item-title style="text-align: center; margin-top: 20%;">검색 결과가 없습니다.</v-list-item-title>
                    </v-list-item>
                </v-list>
            </v-card>
        </v-card>
    </v-dialog>


    <!-- 비로그인 회원이 좋아요를 누를 때 뜨는 모달 -->
    <v-dialog v-model="this.loginModal" max-width="300px">
        <v-card class="modal"
            style="align-items: center; text-align: center; height: 160px; padding-bottom: 20px; overflow-y: hidden;">
            <v-card-text>
                로그인이 필요한 서비스입니다.<br>
                로그인 하시겠습니까?
            </v-card-text>
            <v-row>
                <v-btn @click="this.$router.push('/member/sign-in')" class="submit-btn"
                    style="background-color: #BCC07B;">로그인하기</v-btn>
                <v-btn @click="this.loginModal = false" class="submit-btn"
                    style="background-color: #e0e0e0;">close</v-btn>
            </v-row>

        </v-card>
    </v-dialog>


    <!-- 판매자 회원이 좋아요를 누를 때 뜨는 모달 -->
    <v-dialog v-model="this.sellerModal" max-width="300px">
        <v-card class="modal"
            style="align-items: center; text-align: center; height: 160px; padding-bottom: 20px; overflow-y: hidden;">
            <v-card-text>
                판매자 회원은 다른 농장을 즐겨찾기할 수 없습니다.
            </v-card-text>
            <v-row>
                <v-btn @click="this.loginModal = false" class="submit-btn"
                    style="background-color: #e0e0e0;">close</v-btn>
            </v-row>

        </v-card>
    </v-dialog>
</template>

<script>
import axios from 'axios';
import 'firebase/messaging';
import { initializeApp } from 'firebase/app';
import { getMessaging, getToken, onMessage } from 'firebase/messaging';
import { mapGetters } from 'vuex';
import debounce from 'lodash/debounce';

export default {
    data() {
        return {
            alertModal: false, // Initialize alertModal as a reactive data property
            isLogin: false, // 로그인 여부 확인 
            isSeller: false, // seller인지 member인지 여부 확인
            firstFarmModal: false,
            notiCount: 0,
            notifications: [],

            // 검색
            searchDialog: false,
            searchKeyword: '',
            farms: [],
            products: [],
            total: [],
            keyword: "",
            selectedCategory: 'all', // 선택된 카테고리를 저장하는 변수 추가
            loginModal: false,
            sellerModal: false,
        };
    },
    computed: {
        ...mapGetters(['getNotiCount']),
    },
    async created() {
        const token = localStorage.getItem('accessToken');
        this.isLogin = !!token;

        const role = localStorage.getItem('role');

        if (token) {
            this.isLogin = true;
        } else {
            this.isLogin = false;
        }

        if (role == "SELLER") {
            this.isSeller = true;
        } else {
            this.isSeller = false;
        }

        try {
            const role = localStorage.getItem("role");

            if (role === 'MEMBER') {
                const res = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/member-service/notification?unread=true`);
                console.log(res);
                this.notifications = res.data.content;
            } else if(role === 'SELLER') {
                const res = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/product-service/notification?unread=true`);
                console.log(res);
                this.notifications = res.data.content;

                console.log("line 244 >>>>>>>> ");
                console.log(res);
            }

        } catch (e) {
            console.log(e);
        }



        this.clearObjectStore(); // Object store 리셋
        this.initializeFCM();



        // 주기적으로 IndexedDB 상태 확인 (예: 5초마다)
        setInterval(() => {
            this.getNotificationsFromIndexedDB()
                .then(async (notifications) => {
                    if (notifications.length > 0) {
                        const res = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/member-service/notification?unread=true`);
                        console.log(res);
                        this.notifications = res.data.content;
                        console.log(this.notifications);
                        this.clearObjectStore();
                    }
                })
                .catch((error) => {
                    this.closeConnectionsAndDeleteDB("notificationDB");
                    console.log(error);
                });
        }, 5000); // 5초마다 확인

    },

    methods: {
        async checkFarmAndRedirect() {
            try {
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/product-service/farm/exists`, {
                    headers: {
                        Authorization: `Bearer ${localStorage.getItem('accessToken')}`,
                    }
                });
                if (response.data) {
                    this.$router.push('/seller/delivery-management');
                } else {
                    // 농장이 없을 경우 모달 띄우기
                    this.firstFarmModal = true;
                }
            } catch (error) {
                console.error('농장 확인 실패:', error);
            }
        },
        goToFarmCreate() {
            this.firstFarmModal = false;
            this.$router.push('/farm/farm-create'); // 농장 생성 페이지로 이동
        },

        async handleLogout() {
            try {
                const role = localStorage.getItem("role");
                if(role == 'MEMBER') {
                    await axios.post(`${process.env.VUE_APP_API_BASE_URL}/member-service/member/sign-out`);
                    const fcmToken = localStorage.getItem("fcmToken");
                    console.log(fcmToken);
                    await axios.delete(`${process.env.VUE_APP_API_BASE_URL}/member-service/fcm/${fcmToken}`);
                } else if(role == 'SELLER') {
                    const fcmToken = localStorage.getItem("fcmToken");
                    console.log(fcmToken);
                    await axios.delete(`${process.env.VUE_APP_API_BASE_URL}/product-service/fcm/${fcmToken}`);
                }

            } catch(e) {
                console.log(e);
            }
            localStorage.clear();
            this.isLogin = false;
            this.alertModal = false; // Close the modal
            window.location.href = '/'; // Redirect to home
        },
        async initializeFCM() {
            const firebaseConfig = {
                apiKey: `${process.env.VUE_APP_FIREBASE_API_KEY}`,
                authDomain: "dongsanginong-24f95.firebaseapp.com",
                projectId: "dongsanginong-24f95",
                storageBucket: "dongsanginong-24f95.appspot.com",
                messagingSenderId: "331466655968",
                appId: "1:331466655968:web:f9802c140ab1701b63a885",
                measurementId: "G-WK79L8VKTR"
            };

            // Initialize Firebase
            // const firebase = !getApps().length ? initializeApp(firebaseConfig) : getApp();
            const firebase = initializeApp(firebaseConfig);
            const messaging = getMessaging(firebase);

            if (!localStorage.getItem("fcmToken")) {
                try {
                    // Await the FCM token
                    const token = await getToken(messaging, {
                        vapidKey: `${process.env.VUE_APP_FIREBASE_VAPID_KEY}`
                    });

                    // Store the token in local storage
                    localStorage.setItem("fcmToken", token);
                } catch (err) {
                    console.log(err);
                }
            }

            const fcmToken = localStorage.getItem("fcmToken");
            const body = {
                "fcmToken": fcmToken
            };

            // DB에 fcm 토큰을 저장
            try {
                const role = localStorage.getItem("role");
                if (role == 'MEMBER' && fcmToken != undefined && fcmToken != "") {
                    await axios.post(`${process.env.VUE_APP_API_BASE_URL}/member-service/fcm/token`, body);
                } else if (role == 'SELLER' && fcmToken != undefined && fcmToken != "") {
                    await axios.post(`${process.env.VUE_APP_API_BASE_URL}/product-service/fcm/token`, body);
                }
            } catch (e) {
                console.log(e);
            }

            onMessage(messaging, (payload) => {

                // IndexedDB에 저장
                this.saveNotificationToIndexedDB(payload);
                // notifications 배열 업데이트
                this.notifications.push(payload);


                console.log("Received message ", payload);
                const notificationTitle = payload.notification.title;
                const notificationOptions = {
                    body: payload.notification.body,
                    icon: "favicon.ico"
                };



                if (Notification.permission === "granted") {
                    new Notification(notificationTitle, notificationOptions);
                }

            });
        },
        async markAsRead() {
            const role = localStorage.getItem("role");

            try {
                if(role == 'MEMBER') {
                    await axios.post(`${process.env.VUE_APP_API_BASE_URL}/member-service/notification/read`);

                } else if(role == 'SELLER') {
                    await axios.post(`${process.env.VUE_APP_API_BASE_URL}/product-service/notification/read`);
                }
            } catch(e) {
                console.log(e);
            }

            this.notifications = [];
            this.clearNotificationsFromIndexedDB();
        },
        saveNotificationToIndexedDB(notification) {
            const request = indexedDB.open("notificationDB", 1);

            request.onsuccess = (event) => {
                const db = event.target.result;
                try {
                    const transaction = db.transaction("notifications", "readwrite");
                    const store = transaction.objectStore("notifications");
                    store.add(notification);
                } catch (e) {
                    // IndexedDB에서 "notificationDB" 데이터베이스 삭제
                    const request = indexedDB.deleteDatabase("notificationDB");

                    request.onsuccess = function () {
                        console.log("notificationDB has been deleted successfully.");
                    };

                    request.onerror = function (event) {
                        console.error("Failed to delete notificationDB:", event);
                    };

                    request.onblocked = function () {
                        console.warn("notificationDB deletion blocked. Please close other connections to this database.");
                    };
                }

            };

            request.onerror = (event) => {
                console.error("Error saving notification to IndexedDB", event);
            };
        },
        getNotificationsFromIndexedDB() {
            return new Promise((resolve, reject) => {
                const request = indexedDB.open("notificationDB", 1);

                request.onupgradeneeded = (event) => {
                    const db = event.target.result;
                    if (!db.objectStoreNames.contains("notifications")) {
                        db.createObjectStore("notifications", { keyPath: "id", autoIncrement: true });
                    }
                };


                request.onsuccess = (event) => {
                    const db = event.target.result;
                    try {
                        const transaction = db.transaction("notifications", "readonly");
                        const store = transaction.objectStore("notifications");
                        const getAllRequest = store.getAll();

                        getAllRequest.onsuccess = () => {
                            resolve(getAllRequest.result);
                        };

                        getAllRequest.onerror = (error) => {
                            // IndexedDB에서 "notificationDB" 데이터베이스 삭제
                            this.closeConnectionsAndDeleteDB("notificationDB");
                            reject(error);
                        };
                    } catch (e) {
                        // IndexedDB에서 "notificationDB" 데이터베이스 삭제
                        const request = indexedDB.deleteDatabase("notificationDB");

                        request.onsuccess = function () {
                            console.log("notificationDB has been deleted successfully.");
                        };

                        request.onerror = function (event) {
                            console.error("Failed to delete notificationDB:", event);
                        };

                        request.onblocked = function () {
                            console.warn("notificationDB deletion blocked. Please close other connections to this database.");
                        };
                    }


                };

                request.onerror = (event) => {
                    reject(event);
                };
            });
        },
        clearNotificationsFromIndexedDB() {
            const request = indexedDB.open("notificationDB", 1);

            request.onsuccess = (event) => {
                const db = event.target.result;
                const transaction = db.transaction("notifications", "readwrite");
                const store = transaction.objectStore("notifications");

                // 모든 데이터를 삭제
                const clearRequest = store.clear();

                clearRequest.onsuccess = () => {
                    console.log("All notifications cleared from IndexedDB");
                };

                clearRequest.onerror = (event) => {
                    console.error("Error clearing notifications from IndexedDB", event);
                };
            };

            request.onerror = (event) => {
                console.error("Error opening IndexedDB", event);
            };
        },
        // 검색 모달창
        openSearchModal() {
            this.closeModal();
            this.searchDialog = true;
        },
        //  검색 버튼 
        async performSearch() {
            if (!this.selectedCategory) {
                this.selectedCategory = 'all';
            }
            // 검색어 지우면 빈값 
            if (!this.keyword) {
                this.total = [];
                this.farms = [];
                this.products = [];
                return;
            }

            this.total = [];
            if (this.selectedCategory === 'farm') {
                await this.searchFarms();
                this.total = [...this.farms];

            } else if (this.selectedCategory === 'package') {
                await this.searchPackageProduct();
                this.total = [...this.products];

            } else {
                // 전체 카테고리 선택 시, 두 검색을 모두 수행
                console.log("all로 들어옴")
                await this.searchFarms();
                await this.searchPackageProduct();
                this.total = [...this.farms, ...this.products];
            }
        },
        debouncedSearch: debounce(function () {
            this.performSearch();
        }, 300),

        highlightKeyword(text) {
            if (!this.keyword) {
                return text;
            }
            const regex = new RegExp(`(${this.keyword})`, 'gi');
            return text.replace(regex, '<strong>$1</strong>');
        },

        //  농장 카테고리로 검색
        async searchFarms() {
            try {
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/product-service/es/farms/search`, {
                    params: {
                        keyword: this.keyword
                    }
                });
                this.farms = response.data;
            } catch (error) {
                console.error('Error fetching search results:', error);
            }
        },
        // 상품 카테고리로 검색 
        async searchPackageProduct() {
            try {
                const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/product-service/es/product/search`, {
                    params: {
                        keyword: this.keyword
                    }
                });
                console.log(response.data);
                this.products = response.data;
            } catch (error) {
                console.error('Error fetching search results:', error);
            }
        },
        // 모달 닫기
        closeModal() {
            this.keyword = '';
            this.farms = [];
            this.products = [];
            this.total = [];
            this.selectedCategory = 'all';
        },
        // 농장 detail로 라우팅
        goToFarmDetail(farmId) {
            this.searchDialog = false;
            window.location.href = `${process.env.VUE_APP_MY_URL}/farm/${farmId}/packages`;
        },
        goToPackageDetail(productId) {
            this.searchPackageProduct = false;
            window.location.href = `${process.env.VUE_APP_MY_URL}/product/${productId}`;
        },
        goToDetail(item) {
            if (item.farmName) {
                this.goToFarmDetail(item.id);
            } else if (item.packageName) {
                this.goToPackageDetail(item.id);
            }
        },
        closeConnectionsAndDeleteDB(dbName) {
            const openRequest = indexedDB.open(dbName);

            openRequest.onupgradeneeded = function (event) {
                const db = event.target.result;

                // DB 버전 업그레이드 시 이전 연결이 있으면 강제로 닫음
                db.onversionchange = function () {
                    db.close();
                    console.log("All connections to IndexedDB have been closed due to version change.");
                };
            };

            openRequest.onsuccess = function (event) {
                const db = event.target.result;

                // 현재 열린 연결이 있으면 닫음
                if (db) {
                    db.close();
                    console.log("Closed all active connections to IndexedDB.");
                }

                // 데이터베이스 삭제 시도
                const deleteRequest = indexedDB.deleteDatabase(dbName);

                deleteRequest.onsuccess = function () {
                    console.log(`${dbName} has been deleted successfully.`);
                };

                deleteRequest.onerror = function (event) {
                    console.error(`Failed to delete ${dbName}:`, event);
                };

                deleteRequest.onblocked = function () {
                    console.warn(`Deletion of ${dbName} is blocked. Please close other connections manually.`);
                };
            };

            openRequest.onerror = function (event) {
                console.error(`Failed to open ${dbName} for connection closure:`, event);
            };
        },
        clearObjectStore() {
            let request = indexedDB.open("notificationDB", 1);

            request.onsuccess = function (event) {
                let db = event.target.result;

                try{
                    // 트랜잭션 생성 (읽기/쓰기 모드)
                    let transaction = db.transaction('notifications', 'readwrite');

                    // Object Store 가져오기
                    let objectStore = transaction.objectStore('notifications');

                    // Object Store 클리어 (모든 데이터 삭제)
                    let clearRequest = objectStore.clear();

                    clearRequest.onsuccess = function () {
                        console.log('Object Store cleared successfully.');
                    };

                    clearRequest.onerror = function (event) {
                        console.error('Error clearing Object Store:', event.target.error);
                        this.closeConnectionsAndDeleteDB("notificationDB");
                    };
                } catch(e) {
                    // this.closeConnectionsAndDeleteDB("notificationDB");
                    console.log(e);
                }


            };

            request.onerror = function (event) {
                console.error('Error opening database:', event.target.error);
            };
        },
        goTo(route) {
            this.$router.push(route).then(() => {
                location.reload();
            });
        },
        async readNotificationAndRoute(notification) {
            try {
                const role = localStorage.getItem("role");
                
                if(role == 'MEMBER') {
                    await axios.post(`${process.env.VUE_APP_API_BASE_URL}/member-service/notification/${notification.notificationId}/read`);
                } else if(role == 'SELLER') {
                    await axios.post(`${process.env.VUE_APP_API_BASE_URL}/product-service/notification/${notification.notificationId}/read`);
                }
                
                for(let i=0; i<this.notifications.length; i++) {
                    if(this.notifications[i].notificationId === notification.notificationId) {
                        this.notifications.splice(i, 1);
                        break;
                    }
                }
            } catch(e) {
                console.log(e);
            }
            this.$router.push(notification.url);
        }
    }
};
</script>

<style scoped>
/* 나눔 스퀘어라운드 */
@font-face {
    font-family: 'NanumSquareRound';
    src: url('https://fastly.jsdelivr.net/gh/projectnoonnu/noonfonts_two@1.0/NanumSquareRound.woff') format('woff');
    font-weight: normal;
    font-style: normal;
}

.bar {
    /* font-weight: 400; */
    font-family: 'NanumSquareRound';
}

.main-logo-image {
    transform: translateY(-21px);
}

.reduce-spacing {
    padding-right: 0px;
}

.modal {
    background-color: rgb(255, 255, 255);
    border: none;
    box-shadow: none;
    border-radius: 10px;
}

.farmModal {
    background-color: rgb(255, 255, 255);
    border: none;
    box-shadow: none;
    border-radius: 10px;
    width: 300px;
}

.submit-btn {
    margin-left: 10px;
    margin-top: 8px;
    background-color: #BCC07B;
    color: black;
    border-radius: 50px;
}

.notification-mark {
    background-color: #FFAF6E;
    border-radius: 20px;
    width: 10px;
    height: 10px;
    margin-bottom: 10px;
}

.search-card {
    border-radius: 10 !important;
    padding: 20px;
    padding-top: 20px;
}

.search-btn {
    border: none;
    box-shadow: none;
    background-color: #BCC07B;
    border-radius: 50px;
    margin-right: 3px;
    margin-left: 3px;
}

.cat-btn {
    box-shadow: none;
    border-radius: 50px;
    margin-right: 3px;
    margin-left: 3px;
    background-color: #e0e0e0;
    border:none;
}

.result-card {
    border: none;
    box-shadow: none;
}

.list-card {
    border: none;
    box-shadow: none;
    border-radius: 0px;
}

.cat-btn.active {
    background-color: #FFE2A6 !important;
    color: #000 !important;
}
</style>