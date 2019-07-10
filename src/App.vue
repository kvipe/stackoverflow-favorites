<template>
  <div id="app">
    <div class="search" v-bind:class="{ 'search-on-top': userData }">
      <form class="searchbar" v-on:submit.prevent="getUser">
        <div class="greeting">Explore <i>stackoverflow</i> users favorites</div>
        <span class="home-btn" v-on:click="userData = null; userFavs = null" v-if="userData">&#60;&nbsp;Home</span>
        <input type="text" placeholder="type user ID" v-model="userId">
        <div class="error-info" v-if="error.id">
          <h2>{{error.message}}</h2>
        </div>
      </form>
    </div>
    <div class="user-info" v-if="userData">
      <h1 class="user-name"><a target="_blank" :href="userData.items[0].link">{{userData.items[0].display_name}}</a></h1>
      <div class="avatar-card">
        <div class="avatar">
          <img :src="userData.items[0].profile_image" alt="user avatar">
        </div>
        <div class="reputation">
          <h3 class="user-reputation">Reputation: {{userData.items[0].reputation}}</h3>
        </div>
        <div class="awards"></div>
      </div>  
    </div>
    <div class="user-favorites" v-if="userFavs">
      <div class="question-summary" v-for="favorite in userFavs.items">
        <div class="question-info">
          <span class="question-score">Score: <b>{{favorite.score}}</b></span>&nbsp;
          <span class="question-answers" v-bind:class="{accepted: favorite.accepted_answer_id }">Answers: <b>{{favorite.answer_count}}</b></span>&nbsp;
          <span class="question-views"> Views: <b>{{favorite.view_count}}</b></span>
        </div>
        <div class="question-title">
          <h3><a :href="favorite.link" target="_blank">{{favorite.title}}</a></h3>
          <div class="question-tags">
            <span class="tag" v-for="tag in favorite.tags">{{tag}}</span>
          </div>
        </div>
      </div>
      <paginate
        v-if="isNeedPagination"
        v-model="pagination.current"
        :page-count="pagesCount"
        :click-handler="gotoPage"
        :prev-text="'Prev'"
        :prev-class="'direction'"
        :next-text="'Next'"
        :next-class="'direction'"
        :container-class="'questions-pagination'">
      </paginate>
    </div>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import Paginate from 'vuejs-paginate'
import faves from '@/favs.json'
import user from '@/user.json'
import '@/styles.less'

export default {
  name: 'app',
  components: {
    HelloWorld,
    Paginate
  },
  data(){
    return {
      userId: null,
      userData: null,
      userFavs: null,
      userFavsTotal: null,
      error: {
        id: null,
        message: "Error occured :( Please, try again"
      },
      pagination:{
        current: 1,
        pagesize: 25
      }
    }
  },
  methods: {
    getUser(){
      this.error.id = null;
      this.pagination.current = 1;

      if(!this.userId || isNaN(this.userId)) return;

      this.getUserProfileData();
      this.getUserFavs();
      this.getUserFavsTotal();
    },
    getUserProfileData(){
      fetch(`https://api.stackexchange.com/2.2/users/${this.userId}?site=stackoverflow`)
      .then(response => response.json())
      .then(user => {
        if(!user.error_id)
          this.userData = user;
        else{
          this.error.id = user.error_id;
          return;
        }
      })
      .catch(error => {this.error.id = error; return;})
    },
    getUserFavs(){
      fetch(`https://api.stackexchange.com/2.2/users/${this.userId}/favorites?site=stackoverflow&pagesize=${this.pagination.pagesize}&page=${this.pagination.current}`)
      .then(response => response.json())
      .then(favs => {
        if(!favs.error_id)
          this.userFavs = favs;
        else{
          this.error.id = favs.error_id;
          return;
        }
      })
      .catch(error => {this.error.id = error; return;})
    },
    getUserFavsTotal(){
      fetch(`https://api.stackexchange.com/2.2/users/${this.userId}/favorites?site=stackoverflow&filter=total`)
      .then(response => response.json())
      .then(data => {
        if(!data.error_id)
          this.userFavsTotal = data.total;
        else{
          this.error.id = data.error_id;
          return;
        }
      })
      .catch(error => {this.error.id = error; return;})
    },
    gotoPage(pagenum){
      this.getUserFavs();
    }
  },
  computed:{
    isNeedPagination(){return this.userFavs && this.userFavsTotal > this.pagination.pagesize},
    pagesCount(){ return Math.ceil(this.userFavsTotal / this.pagination.pagesize)}
  }
}
</script>
