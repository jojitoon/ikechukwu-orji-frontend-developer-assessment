<template>
  <div class="home">
    <h1>USER DATA</h1>
    <Users />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Users from "@/components/Users.vue";

@Component({
  components: {
    Users
  }
})
export default class Home extends Vue {}
</script>
