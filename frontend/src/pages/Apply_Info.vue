<template>
    <v-ons-page>
        <custom-toolbar v-bind="toolbarInfo"></custom-toolbar>
        <div class="travler" v-if="session_type()">
            <p align="right">
                <v-ons-button class="Delete_button" icon="trash" style="width:150px"
                              @click="cancel_info()"> 신청취소
                </v-ons-button>
            </p>
            <v-ons-list-header>세부 정보</v-ons-list-header>
            <v-ons-card>
                가이드 프로필
                <p align="right">
                    <v-ons-button class="show_info" v-show="show_info === false" icon='fa-angle-down'
                                  @click="show_guide_info(item.UserID)"> 보기
                    </v-ons-button>
                    <v-ons-button class="hide_info" v-show="show_info === true" icon='fa-angle-up'
                                  @click="show_guide_info(item.UserID)"> 숨기기
                    </v-ons-button>
                </p>
            </v-ons-card>
            <div class="guide_info" v-show="show_info === true">
                <v-ons-card v-for="item in guide_info">
                    <img v-bind:src="'http://13.125.164.72:8000/'+item.User_ImageURL" alt="MSG" width="275" height="230">
                    <v-ons-card>이메일 : {{item.Email}}</v-ons-card>
                    <v-ons-card>이름 : {{item.Name}}</v-ons-card>
                    <v-ons-card>핸드폰 번호 : {{item.PhoneNum}}</v-ons-card>
                    <v-ons-card>카카오 ID : {{item.kakaoID}}</v-ons-card>
                    <v-ons-card>가이드 등록번호 : {{item.Auth}}</v-ons-card>
                    <v-ons-card>평점 :
                        <ons-icon v-for="n in Math.floor(item.GuideGrade)" icon="fa-star"></ons-icon>
                        <ons-icon v-if="count(item.GuideGrade)" icon="fa-star-half-alt"></ons-icon>
                        {{item.GuideGrade}} / {{item.Total_Review}}명 평가</v-ons-card>
                    <v-ons-card>여행 진행 건수 : {{item.GuideGrade}}</v-ons-card>
                </v-ons-card>
            </div>
            <v-ons-card>제목 : {{item.TourTitle}}</v-ons-card>
            <v-ons-card>가격 : {{item.TourPrice}}</v-ons-card>
            <v-ons-card>지역 : {{item.TourLocation}}</v-ons-card>
            <v-ons-card>테마 : {{item.TourThema}}</v-ons-card>
            <v-ons-card>내용 : {{item.TourContent}}</v-ons-card>
            <v-ons-card>날짜 : <br>{{item.TourDayandTime_start.substring(0,21)}} 부터 <br>{{item.TourDayandTime_end.substring(0,21)}} 까지</v-ons-card>
            <v-ons-card>최소인원 : {{item.TourMinNum}}</v-ons-card>
            <v-ons-card>최대인원 : {{item.TourMaxNum}}</v-ons-card>
            <v-ons-list-header>신청인원 : {{item.TourNowPeopleNum}}명 / {{item.TourApplyList2.length}} 단체 </v-ons-list-header>
            <v-ons-list>
                <v-ons-card v-for="todo in item.TourApplyList2">
                    <div class="me" v-if="check_me(todo.user_apply_id)" style="font-family: 궁서">
                        단체대표 : {{todo.user_apply_id}} {{todo.user_num}}명 (ME)
                    </div>
                    <div class="other" v-if="!check_me(todo.user_apply_id)">
                        단체대표 : {{todo.user_apply_id}} {{todo.user_num}}명
                    </div>
                </v-ons-card>
            </v-ons-list>
        </div>
        <div class="travler" v-if="!session_type()">
            <p align="right">
                <v-ons-button class="Delete_button" icon="trash" style="width:150px"
                              @click="cancel_info()"> 신청취소
                </v-ons-button>
            </p>
            <v-ons-list-header>세부 정보</v-ons-list-header>
            <v-ons-list>
                <v-ons-card>작성자 : {{item.UserID }}</v-ons-card>
                <v-ons-card>제목 : {{item.RecruitTitle}}</v-ons-card>
                <v-ons-card>지역 : {{item.RecruitLocation}}</v-ons-card>
                <v-ons-card>내용 : {{item.RecruitContent}}</v-ons-card>
                <v-ons-card>날짜 : <br>{{item.From_time.substring(0,21)}} 부터 <br>{{item.To_time.substring(0,21)}} 까지</v-ons-card>
                <v-ons-card>인원 : {{item.RecruitPeopleNumber}}</v-ons-card>
                <v-ons-card>작성시간 : {{item.WriteTime}}</v-ons-card>
            </v-ons-list>
            <v-ons-list-header>지원자 : 총 {{item.ApplyGuideID.length}}명</v-ons-list-header>
            <v-ons-list>
                <v-ons-card v-for="todo in item.ApplyGuideID" @click="push1(page.component, page.label, todo)">
                    <div class="me" v-if="check_me(todo)" style="font-family: 궁서">
                        지원자: {{todo}} (ME)
                    </div>
                    <div class="other" v-if="!check_me(todo)">
                        지원자: {{todo}}
                    </div>
                </v-ons-card>
            </v-ons-list>
        </div>
    </v-ons-page>
</template>

<script>
    import Check_Apply_Info2 from "./Check_Apply_Info2.vue";

    export default {
        methods: {
            count(counter){
                var temp = counter;
                temp = temp - Math.floor(counter);
                if(temp>0)
                {this.checkGrade= true;}
                else
                    this.checkGrade =false;
                return this.checkGrade;
            },
            show_guide_info(key1){
                if(this.show_info === false){
                    this.show_info=true;
                    this.$http.post('http://13.125.164.72:8000/checkInfo/guide', {
                        params: {user: key1}
                    })
                        .then((response) => {  //로그인 성공;
                                this.guide_info = response.data.data;
                                console.log(this.guide_info);
                            },
                            (error) => { // error 를 보여줌
                                alert(error.response.data.error)
                            }
                        )
                        .catch(error => {
                            alert(error)
                        })
                }
                else{
                    this.show_info = false;
                }
            },
            session_type(){
                if(localStorage.getItem('newType') === '여행객'){
                    return true;
                }
                else{
                    return false;
                }
            },
            check_me(key){
                if(localStorage.getItem('newEmail') === key){
                    return true;
                }
                else{
                    return false;
                }
            },
            push1(page, key, todo) {
                this.$store.state.user = todo;
                this.$store.state.target = this.item.RecruitNum;
                this.$store.commit('navigator/push', {
                    extends: page,
                    data() {
                        return {
                            toolbarInfo: {
                                backLabel: 'Back',
                                title: key
                            }
                        }
                    }
                });
            },
            cancel_info(){
                if(localStorage.getItem('newType') === '여행객' ){
                    for(var i=0; i<this.item.TourApplyList2.length; i++){
                        if(this.item.TourApplyList2[i].user_apply_id === localStorage.getItem('newEmail')){
                            this.temp_people_num = this.item.TourApplyList2[i].user_num;
                        }
                    }
                        this.$http.post('http://13.125.164.72:8000/checkInfo/apply/delete', {
                            params: {
                                email: localStorage.getItem('newEmail'),
                                item: this.item,
                                People_num : this.temp_people_num,
                                type: localStorage.getItem('newType')
                            }
                        })
                            .then(response => {  //로그인 성공
                                    alert("삭제되었습니다.");
                                    location.reload();
                                },
                                (error) => { // error 를 보여줌
                                    alert(error.response.data.error)
                                }
                            )
                            .catch(error => {
                                alert(error)
                            })
                }
                else{
                    this.$http.post('http://13.125.164.72:8000/checkInfo/apply/delete', {
                        params: {
                            email: localStorage.getItem('newEmail'),
                            item: this.item,
                            type: localStorage.getItem('newType')
                        }
                    })
                        .then(response => {  //로그인 성공
                                alert("삭제되었습니다.");
                                location.reload();
                            },
                            (error) => { // error 를 보여줌
                                alert(error.response.data.error)
                            }
                        )
                        .catch(error => {
                            alert(error)
                        })
                }
            }
        },
        data() {
            return {
                page: {
                    component:Check_Apply_Info2,
                    label: '신청 가이드 정보'
                },
                temp_people_num: 0,
                item : this.$store.state.item,
                show_info: false,
                guide_info: []
            };
        }
    };
</script>

<style scoped>
</style>