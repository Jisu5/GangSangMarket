<template>
    <v-row>
        <v-col cols="11" md="8" class="mx-auto">
            <!-- 프로필 사진 -->
            <v-list>
                <v-list-item>
                    <v-spacer></v-spacer>
                    <image-input v-model="avatar" @input="input" :rules="rules">
                        <div slot="activator">
                            <v-avatar size="10rem" v-ripple v-if="!profileImg" class="grey lighten-3 mb-3">
                                <v-icon size="5rem">mdi-account</v-icon>
                            </v-avatar>
                            <v-avatar size="10rem" v-ripple v-else class="mb-3">
                                <img :src="'http://i3a504.p.ssafy.io/static/image/account/'+profileImg" v-if="!change" alt="avatar">
                                <img :src="avatar.imageURL" v-else alt="avatar">
                            </v-avatar>
                        </div>
                    </image-input>

                    <v-spacer></v-spacer>
                </v-list-item>
                <v-list-item>
                    <v-text-field color="#643579" v-model="nickName"  label="닉네임" required></v-text-field>
                </v-list-item>
                <v-list-item>
                    <v-text-field color="#643579" v-model="phone" :rules="phoneRules" label="핸드폰번호" required></v-text-field>
                </v-list-item>
                <v-list-item>
                    <v-text-field color="#643579" :value="address" label="상세 주소" readonly @click="addressSearch"></v-text-field>
                </v-list-item>
                <v-list-item>
                    <v-textarea color="#643579" v-model="introduce" label="상점 소개"></v-textarea>
                </v-list-item>
            </v-list>
            <v-btn color="#e7d8eb" @click="submit" id="clickBtn">수정완료</v-btn>
        </v-col>

    </v-row>
</template>


<script>
import { mapState, mapActions } from 'vuex'
import http from "@/util/http-common"
import ImageInput from '@/components/form/ImageInput.vue'
export default {
    data() {
        return {
            rules: [
                value => !value || value.size < 1000000 || '프로필 사진 파일 크기는 1 MB 미만이여야 합니다.',
            ],
            phoneRules:[
                value => (value && !value.includes("-") && (/^[0-9]*$/).test(value) && value.length<=11) || "\'-\'를 제외한 숫자만 입력해주세요"
            ],
            nickName: "",
            address: "",
            introduce: "",
            profileImg: null,
            avatar:null,
            change:false,
            img:null,
            phone: null,
        }
    },
    components:{
        ImageInput,
    },
    computed: {
        ...mapState(['myProfile']),
    },
    methods: {
        ...mapActions(['getMyProfile', 'updateProfile']),
        addressSearch(){
            var address2;
            var changeAddress = this.changeAddress
            new daum.Postcode({
                oncomplete: function(data) {
                    address2 = data.roadAddress
                    changeAddress(data.roadAddress)
                }
            }).open();
            
        },
        input(file) {
            this.img = file.formData.entries().next().value[1]
            this.change=true
        },
        changeAddress(value){
            this.address=value
        },
        submit() {
            const frm = new FormData();
            frm.append("nickName", this.nickName)
            console.log(this.img)
            if (this.img !== null) {
                frm.append("img", this.img)
            }
            frm.append("phone",this.phone)
            frm.append("introduce", this.introduce)
            frm.append("address", this.address)
            
            // for (var pair of frm.entries()) { 
            //     console.log(pair)
            //     // console.log(pair[0]+ ', ' + pair[1]); 
            //     }

            http.put("/api/user", frm, {
                    headers: {
                        Authorization: this.$store.state.authorization,
                        'Content-Type': 'multipart/form-data'
                    }
                })
                .then(res => {
                    this.getMyProfile()
                    console.log(res)
                })
            
            this.$router.push({name:'UserProfile', params: {uid: this.myProfile.userId}})
        },
    },
    created() {
        this.nickName = this.myProfile.nickName
        this.introduce = this.myProfile.introduce
        this.address = this.myProfile.address
        this.profileImg = this.myProfile.profileImg
        this.phone = this.myProfile.phone
    },


}
</script>

<style scoped>
</style>