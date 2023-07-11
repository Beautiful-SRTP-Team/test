<template>
  <v-app>
    <v-main>
      <Simple/>
<!--      <HelloWorld/>-->
      <Rank/>
      <SignIn/>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
  import HelloWorld from '@/components/HelloWorld.vue'
  import Simple from "@/components/Simple.vue";
  import Rank from "@/components/Rank.vue";
  import SignIn from "@/components/SignIn.vue";
</script>
