<template>
  <form @submit.prevent="submitForm">
    <div>
      <label for="username">id: </label>
      <input id="username" type="text" v-model="username" />
    </div>
    <div>
      <label for="password">pw: </label>
      <input id="password" type="text" v-model="password" />
    </div>

    <button v-bind:disabled="!isUsernameValid || !password" type="submit">
      로그인
    </button>
    <p>{{ logMessage }}</p>
  </form>
</template>

<script>
import { loginUser } from '@/api/auth';
import { validateEmail } from '@/util/validation';
import { saveAuthToCookie, saveUserToCookie } from '@/util/cookies';

export default {
  data() {
    return {
      username: '',
      password: '',
      //log
      logMessage: '',
    };
  },
  computed: {
    isUsernameValid() {
      return validateEmail(this.username);
    },
  },
  methods: {
    async submitForm() {
      try {
        //비지니스 로직
        const userData = {
          username: this.username,
          password: this.password,
        };
        const { data } = await loginUser(userData);
        console.log(data.token);
        // this.initForm();
        this.logMessage = `${data.user.username}님 환영합니다.`;
        this.$store.commit('setToken', data.token);
        this.$store.commit('setUsername', data.user.username);
        saveAuthToCookie(data.token);
        saveUserToCookie(data.user.username);
        this.$router.push('/main');
      } catch (error) {
        //에러핸들링
        console.log(error.response.data);
        this.logMessage = error.response.data;
        // this.initForm();
      } finally {
        this.initForm();
      }
    },
    initForm() {
      this.username = '';
      this.password = '';
      this.nickname = '';
    },
  },
};
</script>

<style>
.btn {
  color: white;
}
</style>
