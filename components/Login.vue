<template>
  <div class="main">
    <!-- a  username and password input at the top of the page -->
    <div class="login-form">
      <div class="form-header">
        <h2>Login</h2>
      </div>
      <div class="form-body">
        <div class="form-group">
          <label for="username">Username</label>
          <input type="text" class="form-control" id="username" placeholder="Username">
        </div>
        <div class="form-group">
          <label for="password">Password</label>
          <input type="password" class="form-control" id="password" placeholder="Password">
        </div>
        <div class="form-group">
          <button class="btn btn-primary btn-block" @click = "login">Login</button>
        </div>
        <div class="form-group">
          <button class="btn btn-primary btn-block" @click = "signUp">Sign Up</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
    data(){
        return{
            username: '',
            password: '',
        }
    },
    methods:{
        login(){
            this.$emit('login', {
                username: this.username,
                password: this.password,
            });
        },
        signUp(){
            this.$emit('signup', {
                username: this.username,
                password: this.password,
            });
        }
    },
}
</script>

<style>

</style>