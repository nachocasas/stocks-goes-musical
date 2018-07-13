<template>
  <div v-bind:class="[{ searchanimated: isActive }, 'form-wrapper']">
    <div v-bind:class="[{ backanimated: isActive }, 'background']"></div>
    <div class='form-container'>
      <form v-on:submit.prevent="submit">
        <input required v-model.trim="searchTxt" v-on:keydown="checkActive" type="text" placeholder="Enter symbol" >
      </form>
    </div>
  </div>
</template>

<script>
/* eslint-disable */

export default {
  name: 'Search',
  props: ['firstUse'],
  data () {
    return {
      searchTxt: '',
      isActive: false
    }
  },
  methods: {
    checkActive: function(event){
      var inputElem = event.target;
      if(this.firstUse){
        this.isActive = inputElem.value != "";
      }
      if(event.keyCode == 13) {
        this.submit();
      }
    },
    submit: function(){
      if(this.searchTxt !== ""){
        this.$emit('search', this.searchTxt);
      }
      this.searchTxt = '';
      }
  }
}
</script>

<style lang="postcss" scoped>

.form-wrapper {
  display: flex;
  flex-direction: column;
  position:absolute;
  width: 80%;
  height:100%;
  top: 0%;
}

.background{
  position:absolute;
  align-self: center;
  width: 80%;
  height: 100%;
  background:url('../assets/img/home-back.svg') center no-repeat;
  background-size: contain;
}

.backanimated {
    animation: backopacity 1s;
    animation-fill-mode: forwards;
  }

.form-container {
  position:absolute;
  top:40%;
  width:100%;
  padding: 5%;
  border-radius: 20px;
  background:#5AAC87;
  z-index: 999;
}
.searchanimated {
    animation: searchanimation 2s;
    animation-fill-mode: forwards;
  }

.searchanimated .form-container{
    animation: formanimation 2s;
    animation-fill-mode: forwards;
}

.searchanimated .form-container input{
    animation: inputanimation 1s;
    animation-fill-mode: forwards;
}

@keyframes searchanimation{
    from { height: 100%; width:80%; } 
    to{  height: 10%; width: 100%; } 
}

@keyframes backopacity{
    from { opacity: 1; } 
    to{  opacity: 0;} 
}

@keyframes formanimation{
  from { border-radius: 20px; padding:5%; top: 40% } 
    to{  border-radius: 0px; padding:1%; top: 0} 
}

@keyframes inputanimation{
  from { border-width: 7px; padding: 2%; font-size: 1.5rem } 
    to{  border-width: 3px; padding: 1%; font-size: 1rem } 
}

input {
  width: 95%;
  border: 7px solid #F9B89A;  
  padding: 2%;
  font-size: 1.5rem;
}
</style>
