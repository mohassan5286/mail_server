<template>
    <div v-if = 'sender'>
      <SideBar></SideBar>
      <HeaderPage></HeaderPage>
      <ReceivedMails></ReceivedMails>
      <ComposeMails></ComposeMails>
      <DraftMails></DraftMails>
      <SentMails></SentMails>
      <TrashMails></TrashMails>
      <FilterComponent></FilterComponent>
    </div>
 
    <div v-if = '!sender'>
      <SignUp></SignUp>
      <SignIn></SignIn>
    </div>  
</template>

<script>
  import SideBar from "./components/SideBar.vue";
  import HeaderPage from "./components/HeaderPage.vue" ;
  import ReceivedMails from "./components/ReceivedMails.vue";
  import ComposeMails from "./components/ComposeMails.vue";
  import DraftMails from "./components/DraftMails.vue";
  import SentMails from "./components/SentMails.vue";
  import TrashMails from "./components/TrashMails.vue";
  import FilterComponent from "./components/FilterComponent.vue";
  import SignUp from "./components/SignUp.vue";
  import SignIn from './components/SignIn.vue';

  export default {
    name: 'App',
    components: {
      SideBar,
      ReceivedMails,
      HeaderPage,
      ComposeMails,
      DraftMails,
      SentMails,
      TrashMails,
      FilterComponent,
      SignUp,
      SignIn,
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
